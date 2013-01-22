Understanding the Scheduler
---------------------------

The scheduler is the choke point where all VM provisioning happens. It is
responsible for validating and orchestrating a number of things:

* Orchestrating the execution of provisioner objects
* Ensuring machines are provisioned successfully, and failing gracefully if they don't
* Communicating and tracking the state of provisions
  * Even over multiple scheduler runs, from the same or different program execution
* Orchestrating provisions in an order that honors known dependencies on other provisions. 

Basic Terminology
-----------------

* VM Database: This is the system used to persist changes in the scheduler.
  Most of the data you get back from chef-workflow in rake tasks and tests
  comes from this system.
* Provisioner: An object ( *not* a class ) that has a known interface which
  describes how machine relationships (such as a VM provision, or bootstrapping
  a node) can be applied and reverted. See [[Writing a Provisioner]] for
  further exposition.
* Serial and Parallel: while not divergent from their canonical definitions,
  these are actually mutually exclusive modes that the scheduler runs in that
  determine how it tackles provisioning.
* Server Group or Role: A name associated with a set of provisioners. This is
  additionally applied to the machines that make up the result of a
  provisioning operation. This term is conflated with Chef's roles due to an
  early design decision that may be changed later. It's best to use "Server
  Group".

Know Your Threading
-------------------

Reading the code, and some of the terms used here will require some background
with parallel programming, particularly related to threads and concurrency
problems like [Dining
Philosophers](http://en.wikipedia.org/wiki/Dining_Philosophers). It is also
useful to understand how MRI's GVL works in relation to POSIX `select(2)`, and
why concurrency works with it but parallelism doesn't. Yehuda Katz [details the
issues](http://yehudakatz.com/2010/08/14/threads-in-ruby-enough-already/) much
better than I could or would attempt to do for the purposes of this document.

The Basics
----------

The scheduler solves the Dining Philosophers problem in a somewhat classic way:

* All programmed operations (called waiters) are added to a set with their
  dependencies.
* A polling mechanism exhausts a solved queue (by way of adding it to a set for
  persistence and de-duplication) to determine set items that are fully
  resolved.
* A thread is spawned to service these resolved waiters, by executing their
  provisioners. Adds itself to the working list. This is a bastardized
  [Actor](http://en.wikipedia.org/wiki/Actor_(programming) of sorts, the actor
  itself being less of an object but more of a pre-defined execution plan on a
  set of objects which conform to a very specific interface (Provisioners).
* Actors are polled for status periodically to ensure errors are bubbled up to
  the main thread if they occur. There is no recovery from these errors, and is
  an intentional feature of the system.
* After the actor has completed, the information the waiter used to spawn it is
  added to the solved queue. 

Note that there is no thread pool or other mechanism constraining the number of
actors that may be spawned. If there are 1000 satisified waiters at the time
new actors are to be spawned, 1000 actors *will* be spawned. This reduces
complexity while making the assumption that this happening is extremely
unlikely.

Serial and Parallel Modes: How They Differ
------------------------------------------

As previously mentioned, the scheduler has two running modes: serial and
parallel. While mostly the same internally, there are some differences that
matter to the consumer:

* Parallel mode does not block, ever. Serial mode blocks the main thread during
  `run` execution until the waiters set is exhausted.
* Cascading, the `stop` call is never necessary for Serial mode, but may be
  needed in some parallel usage.
* `run` in parallel mode never does anything after the first invocation,
  presuming the solver is running. In serial mode it will always attempt to
  exhaust the waiters set.
* `wait_for` in serial mode always immediately returns, presuming there's no
  way it could ever block on anything.

To summarize, calling `run` has very different behavior on how the main thread
runs depending on what mode is in use. Consequently, three things are very, very
important:

* always use `wait_for` to block on the scheduler's state. This ensures
  blocking will always occur when it's desired.
* **never** change a running scheduler's mode.
* **never** let more than one scheduler run at one time.

The scheduler is referenced as a global variable (`$SCHEDULER`) with some
helpers to abstract that use: `chef-workflow-tasklib` has the `with_scheduler`
task helper and `chef-workflow-testlib` has the `ProvisionHelper` mixin which
is applied to the custom TestCase class. Do not be surprised if this becomes a
`Singleton` in a later version.

Provisioners
------------

Provisioners are objects which have a specific interface used for raising and
tearing down resources related to a server group. The server group is
provisioned with a list of provisioners, execution of the `startup` method
occurs for each one, the return value of the most recently executed passed to
the next, `apply` style. `nil` is passed to the first `startup` call.

Shutdown occurs by reversing the list and calling the `shutdown` method.

Any non-true value returned from the `startup` or `shutdown` calls indicates
that the (de)provisioning process failed. If `force_deprovision` is set,
`shutdown` will continue to execute, otherwise it will log the failed
deprovision and abort the attempt (but the scheduler will still continue).
Failed `startup` will raise which will bubble up to the main thread.

All state transitions for provisioners are tracked in the VM database, by way
of storing them as marshalled objects associated with the server group name.

Read [[Writing A Provisioner]] for what's expected from a provisioner at an
interface level.

The VM Database
---------------

The VM Database is core usage of the state database and exists to separate the
concerns between persisting the state of the scheduler and actually performing
the scheduling.

If you plan to get your hands dirty, [[Using the State Database]] will probably
be worth reading.

The VM database has four partitions:

* Scheduled Server Groups (`groups` attribute, VMGroup type)
* Server Group Dependencies (`dependencies` attribute, VMGroup type)
* Groups in the "provisioned" state (`provisioned` attribute, Set type)
* Groups in the "working" state (`working` attribute, Set type)

VMGroup is a specialized Hash table where the value is an array of Marshalled
objects. They are used to keep track of arbitrary data.

The state sets allow the scheduler to persist any change to its internals in a
durable fashion that can be interrogated to restore state after a termination
for any reason. The scheduler always talks through the database, there is no
intermediate data structure requiring synchronization, allowing it to be very
consistent with regards to what it knows has happened.
