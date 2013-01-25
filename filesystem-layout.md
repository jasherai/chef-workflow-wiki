Filesystem Layout
-----------------

The "filesystem" as it is, is merely a directory structure of common
components. The reason documenting this is necessary is twofold:

* What directory a component lives in is important to certain end-user tooling,
  such as `chef_workflow_task`.
* The chef-workflow gems are designed to be a pick-and-choose deal, but present
  a universal hierarchy. You don't need the testing framework, but if it
  defines a support library and you *are* using it, it'll exist alongside all
  the chef-workflow gem's support libraries. This allows developers to extend
  the system in a way that easy for anyone familiar with the system to
  understand.

You don't obviously have to conform to this, it's just probably a good idea.

Directories and Purpose
-----------------------

Here's a table. Note that the "Primary Gem" field merely indicates what gem
provides most of the baked-in content in this directory, it's not necessarily
where all of it lives.

All chef-workflow tooling across all gems lives has a root of `chef-workflow/`.
Anything outside of that is probably a build product.

<table>
<tr><td><b>Directory</b></td><td><b>Purpose</b></td><td><b>Primary gem</b></td></tr>

<tr><td>`support/`          </td><td>Basic support and configuration libraries</td><td>chef-workflow        </td></tr>
<tr><td>`support/db`        </td><td>Database Layer                           </td><td>chef-workflow        </td></tr>
<tr><td>`support/vm`        </td><td>Provisioners                             </td><td>chef-workflow        </td></tr>
<tr><td>`support/vm/helpers`</td><td>Helpers specific to Provisioners         </td><td>chef-workflow        </td></tr>
<tr><td>`tasks/`            </td><td>Basic Rake Tasks (see below)             </td><td>chef-workflow-tasklib</td></tr>
<tr><td>`task-helpers/`     </td><td>Helpers for Rake Tasks                   </td><td>chef-workflow-tasklib</td></tr>
<tr><td>`helpers/`          </td><td>Test Helpers                             </td><td>chef-workflow-testlib</td></tr>
<tr><td>`runner/`           </td><td>Test Runners                             </td><td>chef-workflow-testlib</td></tr>
<tr><td>`test-case/`        </td><td>MiniTest TestCase subclasses             </td><td>chef-workflow-testlib</td></tr>
</table>

Rake Tasks
----------

There's a pretty gigantic tree under `tasks/`. It's partitioned by section,
then for the most part by task name, although there are a few exceptions. See
[[Standard Tasks]] for more information on the tasks that chef-workflow
provides.

Exceptions
----------

Of course there are exceptions. :)

* `bin/chef-workflow-bootstrap` is in the `chef-workflow` gem and is a script
  to bootstrap ... chef-workflow. See [[Bootstrapping Chef Workflow]] for more
  information.
* `chef-workflow-tasklib`, the require, is used to bootstrap functionality to
  require tasks with `chef_workflow_task`. Unsurprisingly, it lives in the
  `chef-workflow-tasklib` gem.
