Rake Task Helpers
-----------------

Task Helpers are just there to make some things easier when writing your own
tasks. They live under the `chef-workflow/task-helpers` directory in the
[[Filesystem Layout]], and perform a set of monkey patches to `Rake::DSL` and
the toplevel namespace so they're accessible.

SSH Helper
----------

`chef-workflow/task-helpers/ssh` brings in the functionality from
`ChefWorkflow::SSHHelper`, which will be familiar API as it's the same used in
integration testing -- see [[Testing Multi-Machine Behavior]] for a list of
calls.

with_scheduler
--------------

`chef-workflow/task-helpers/with_scheduler` instantiates the scheduler
properly, and then yields it to a block where further commands can be run on
it. It's useful for pretty much anything involving provisioning machines. You
may wish to read [[Understanding the Scheduler]] before you dig into this,
though.
