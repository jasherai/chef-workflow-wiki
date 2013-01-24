Writing a Provisioner
---------------------

Provisioners are how most of the orchestration required in Chef-Workflow
happens. Up to this point in the documentation we've seen how provisioners can
change how machines are created, but almost all automated work with the
machines in your chef-workflow environment are done with provisioners,
including creating test chef servers and bootstrapping new machines with
chef-client.

Provisioners are objects controlled by the scheduler, implementing a [Strategy
Pattern](http://en.wikipedia.org/wiki/Strategy_pattern) of sorts. Each object
conforms to an interface and each part of the interface has an expectation of
how it'll be used. An array of provisioners is passed to the scheduler along
with a server group name, and provisioning is considered "successful" if all
the provisioners in the pipeline report success. See [[Understanding the
Scheduler]] for more information on how the scheduler deals with provisioners.

A special note here are "Machine Provisioners", which are configured by the end
user to define how the machines they use are created (via vagrant, ec2, lxc,
openstack, solaris zones etc). The only real distinction is that the existing
tooling puts these at the head of any pipeline so that they are executed first,
and they return IP addresses to the next provisioner.  Otherwise, there is no
difference.

Constraints
-----------

The big constraint is that a provisioner must be fully marshallable. This is
how the scheduler tracks these objects between runs.

The ruby [Marshal](http://www.ruby-doc.org/core-1.9.3/Marshal.html)
documentation covers what is and is not marshallable, and also provides some
information on how you can train your classes that have unmarshallable data to
be marshal-friendly.

In the same vein as above, your provisioner is expected to survive the program
being shutdown and started again. The scheduler does most of the dirty work for
you, but if you open sockets or other ethereal data, be sure you record state
as early as possible so you can deal with it if the unexpected happens. [[Using
the State Database]] is a guide on how to leverage the built-in persistence
layer to make that easy -- the vagrant provisioner does this with the
marshallable vagrant environments it creates, if you want an example.

Interface
---------

A provisioner, again, is an object that responds to certain methods. It doesn't
matter how this object is created, what class it belongs to, etc.

The methods that must exist are:

* `name` and `name=`: this is the name of the server group. The scheduler sets
  this when it first gets its hands on the provisioners.
* `startup(*args)`: This is what's called when this provisioner needs to create
  something, like a VM or a chef-client. The arguments that are passed are
  provided from the return value of the previous provisioner. The first
  provisioner gets an explicit nil here. This call must return a truthy value
  for the provisioning process to continue.
* `shutdown`: What's called when deprovisioning. Must return a truthy value for
  the deprovisioning process to continue unless `force_deprovision` is set on
  the scheduler.  
* `report`: Returns an array of strings; used in status output from the
  scheduler.
