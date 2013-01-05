Writing Rake Tasks
------------------

Chef-Workflow is built to let you write your own tools to enhance your usage of
the system. One of the ways to do that is with rake tasks.

We have no intention of teaching you rake, and to be really effective with your
rake tasks, it's strongly advised you delve into the Advanced Topics section of
the [[Home]] wiki.

Instead of trying to bombard you with everything at once, let's create a small
rake task to use
[knife_cookbook_sync](https://github.com/erikh/knife_cookbook_sync) to perform
our cookbook uploads. This makes use of the
[knife-dsl](https://github.com/chef-workflow/knife-dsl) project which is
already included with chef-workflow.

It assumes `knife_cookbook_sync` is in your `Gemfile` and you have run `bundle install`.

First, to be useful with `chef_workflow_task`, we conform to the chef-workflow
[[Filesystem Layout]]. This isn't strictly necessary, but `chef_workflow_task`
won't work unless you do -- you can just use `require` if you wish to avoid
this.

* `mkdir -p lib/chef-workflow/tasks`
* Open `lib/chef-workflow/tasks/sync.rb` in an editor.

First, we will have likely already loaded our `chef:cookbooks:upload` task
already. Let's use some of the rake API to clear that task before we overwrite
it. 

```ruby
Rake::Task["chef:cookbooks:upload"].clear
```

Let's create our task:

```ruby
chef_workflow_task 'bootstrap/knife'

namespace :chef do
  namespace :cookbooks do
    task :upload => ["bootstrap:knife"] do
      Rake::Task["chef:cookbooks:resolve"].invoke rescue nil # optional, see below
      knife %w[cookbook sync -a]
    end
  end
end
```

The first line in the task invokes `chef:cookbooks:resolve` but only if it
exists. This is largely to allow the optional use of cookbook resolvers. See
[[Standard Tasks]] for more information.

The second line executes `knife` just as you would on the command line, but
it's already configured for you -- that's what the "bootstrap:knife" task does
for you. It worries about what chef server you want to point to. In this case
we execute `knife cookbook sync -a`, which will sync all cookbooks to the
chef server, depending on if the on-disk cookbooks differ from the server.

Because we overwrote "chef:cookbooks:upload", we see this benefit throughout
the whole workflow -- for example, `chef:upload` will use this task as will
`test:full`.

To use this task, make your `Rakefile` look like this:

```ruby
require 'chef-workflow-tasklib'
chef_workflow_task 'default'
chef_workflow_task 'sync'
```

Order *is* important here as we want to overwrite the previously-defined
`chef:cookbooks:upload` task.

You literally have everything available to you in rake tasks, with a bevy of
[[Support Libraries]] and [[Rake Task Helpers]] to assist you with just about
anything.
