Using Chef-Workflow In Your Gem
-------------------------------

If you want to extend Chef-Workflow in a consistent way and share that with
others, a separate gem would be the ideal place to do that. Maybe it's just
making a tool you already provide work with the system, or something like a new
machine provisioner.

Rake Tasks
----------

If you're writing a rake task, it's best to get familiar with the [[Filesystem
Layout]]. This has some bonuses on the integration front, namely with the
require wrappers that we use elsewhere in this guide.

If you want to replace things in chef-workflow-tasklib itself with extended
functionality, you'll want to get familiar with the
[Rake::Task](http://rake.rubyforge.org/Rake/Task.html) documentation. It has
methods to manipulate tasks that already exist, invoke others that may or may
not exist, and is very useful when solving this problem.

Provisioners and Other Tooling
------------------------------

There could be a lot better support here, and I hope to add it eventually.
Conforming to the [[Filesystem Layout]] is probably still a good idea for that
reason.

Users need to require your stuff either in the configuration library, or before
that library is loaded (e.g., top of Rakefile). For a `machine_provisioner`,
they can use the class name of your provisioner presuming it conforms to a
specific interface for the constructor. They set this in the
`configure_general` section.

Test helpers and so forth are largely the same deal, just they need to be
required by parts of the user's test suite.

Everyone
--------

Additionally, be sure to instruct your users to add your gem to their Gemfile.
