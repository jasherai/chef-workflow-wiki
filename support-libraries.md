Support Libraries
-----------------

These libraries are a part of the `chef-workflow` gem and are meant to
facilitate things that are common to all aspects of the system. They are
intended for people looking to extend `chef-workflow-tasklib`,
`chef-workflow-testlib`, or other things like [[Writing a Provisioner]]. All of
these libraries live under `chef-workflow/support` as per the [[Filesystem
Layout]].

ChefWorkflow::AttrSupport
-------------------------

`attr.rb`, This is intended to be extended in classes and provides a single method, called
`fancy_attr`. It provides attributes similar to `attr_accessor` that work
without assignment operators, similar to how chef itself deals with attributes.

```ruby

class Foo
  extend ChefWorkflow::AttrSupport

  fancy_attr :bar
end

f = Foo.new
f.bar = :hello  # sets @bar to :hello
f.bar :hello    # the same
f.bar           # returns :hello
```

ChefWorkflow::DatabaseSupport
-----------------------------

`db.rb`, This is a small heirarchy of classes to manage the state database. Given the
breadth of this subject, [[Using the State Database]] is dedicated to
discussing this.

ChefWorkflow::DebugSupport
--------------------------

`debug.rb`, including this module into your class allows you to use the `if_debug`
statement, which runs a block conditionally based on the value of
`CHEF_WORKFLOW_DEBUG`. (See [[Environment Variables]] for more information).
Values are numeric and `if_debug` treats any lesser value than what is in
`CHEF_WORKFLOW_DEBUG` as a successful condition.

If an additional argument is given as a proc, it will be executed if the
condition is *not* met.

```ruby
class Foo
  include ChefWorkflow::DebugSupport

  def something_normal
    $stderr.puts "this is what happens when CHEF_WORKFLOW_DEBUG is less than 1"
  end

  def do_something
    if_debug(1, method(:something_normal)) do
      $stderr.puts "CHEF_WORKFLOW_DEBUG is set to 1 or greater"
    end

    if_debug(3) do
      $stderr.puts "CHEF_WORKFLOW_DEBUG is set to 3 or greater"
    end
  end
end

# CHEF_WORKFLOW_DEBUG cannot be altered after program start. The examples here
# are used to demonstrate the difference, but this code will not work differently 
# if ENV["CHEF_WORKFLOW_DEBUG"] is altered at runtime.
ENV["CHEF_WORKFLOW_DEBUG"] = 0
f = Foo.new
f.something # something_normal is run
ENV["CHEF_WORKFLOW_DEBUG"] = 1
f.something # first message is printed only
ENV["CHEF_WORKFLOW_DEBUG"] = 2
f.something # same
ENV["CHEF_WORKFLOW_DEBUG"] = 3
f.something # both debug messages are printed
```

ChefWorkflow::EC2Support
------------------------

`ec2.rb`, A facilitator for the EC2 provisioner. Contains some basic logic to deal with
security groups. See [[Configuration Sections]] for how to use this. 

ChefWorkflow::GeneralSupport
----------------------------

`general.rb`, A facilitator for "general" configuration. See [[Configuration
Sections]] for more information.

ChefWorkflow::GenericSupport
----------------------------

`generic.rb`, When included into a class, makes it a
[Singleton](http://www.ruby-doc.org/stdlib-1.9.3/libdoc/singleton/rdoc/Singleton.html)
and gives it two class methods: `configure` which instance evals a block
against the singleton, and `method_missing`, which allows methods executed
against the class to be delegated to the singleton instance. Used for managing
configuration classes in a uniform way.

```ruby
class Thing
  extend ChefWorkflow::AttrSupport
  include ChefWorkflow::GenericSupport

  fancy_attr :stuff
  fancy_attr :more_stuff
end

Thing.configure do
  stuff :quux
end

Thing.stuff # => :quux
Thing.more_stuff :foo # sets @more_stuff to :foo
```

ChefWorkflow::IPSupport
-----------------------

`ip.rb`, The IP database. This is largely responsible for maintaining what IPs
belong to what Server Groups. Uses `GenericSupport`, so most methods are
executed against the class itself.

It's probably best to read the RDoc for the class itself.

ChefWorkflow::KnifePluginSupport
--------------------------------

`knife-plugin.rb`, when included provides a method named `init_knife_plugin`.
This can be used to take the classes that would normally be used by `knife` as
a plugin and run them without shelling out. Does not execute anything, does not
require what you need. Just returns the constructed plugin after doing the
dance that Chef wants to do. Useful for provisioners and anywhere you want to
run something like knife would. See
[knife-dsl](http://github.com/chef-workflow/knife-dsl) for a way to do this
that's a little less wordy for things like rake tasks.

The first argument is the class name, the second argument is the ARGV presented
to it.

```ruby
require 'chef/knife/cookbook_sync'
class Foo
  include ChefWorkflow::KnifePluginSupport

  def run_knife_cookbook_sync
    command = init_knife_plugin(Chef::Knife::CookbookSync, %w[-a])
    command.run
  end
end

f = Foo.new
f.run_knife_cookbook_sync # => runs 'knife cookbook sync -a'
```

ChefWorkflow::KnifeSupport
--------------------------

`knife.rb`, this is configuration relating to the use of chef and knife. See
[[Configuration Sections]] for more information.

ChefWorkflow::Scheduler
-----------------------

`scheduler.rb`, The scheduler manages all things that have to do with machines.
A great deal of the overall documentation is used discussing this topic, but
[[Understanding the Scheduler]] is probably the most direct resource for those
understanding how to work with this class. See also [[Rake Task Helpers]] for
those looking to use this class with `chef-workflow-tasklib`.

ChefWorkflow::VagrantSupport
----------------------------

`vagrant.rb`, this is the configuration relating to the use of vagrant. See
[[Configuration Sections]] for more information.

ChefWorkflow::VM
----------------

`vm.rb` and the `vm` subdirectory, this is the VM database and the provisioners
themselves. Read [[Using the State Database]], [[Understanding the Scheduler]],
and [[Writing a Provisioner]] for more information on this system.

ChefWorkflow::SSHHelper
-----------------------

`ssh.rb`, when included or extended adds several ssh orchestration methods.
Used both in testlib and tasklib.
