Using Chef-Workflow In Your Gem
-------------------------------

If you want to extend Chef-Workflow in a consistent way and share that with
others, a separate gem would be the ideal place to do that. Maybe it's just
making a tool you already provide work with the system, or something like a new
machine provisioner.

If you're writing a rake task, it's best to get familiar with the [[Filesystem
Layout]]. This has some bonuses on the integration front, namely with the
require wrappers that we use elsewhere in this guide.

If you want to replace things in chef-workflow-tasklib itself with extended
functionality, you'll want to get familiar with the
[Rake::Task](http://rake.rubyforge.org/Rake/Task.html) documentation. It has
methods to manipulate tasks that already exist, invoke others that may or may
not exist, and is very useful when solving this problem.

Additionally, be sure to instruct your users to add your gem to their Gemfile.
