Configuration Sections
----------------------

There are four configuration sections; most of this is well-partitioned, but in
particular, things in `configure_knife` and `configure_general` are a little
more confusing than most -- this is largely an artifact of `configure_knife`
having everything before `configure_general` existed. Expect this to be
resolved in future versions.

Each section is used as such, for example for `configure_general`:

```ruby
configure_general do
  some_property some_value
end
```

Note that explicit assignment works, but has a subtlety:

```ruby
configure_general do
  self.some_property = some_value
end
```

Note the `self`. This is due to how ruby treats the assignment operator.

Wise Practices
--------------

Do what you want, but it's in your best interest to stick to a few rules:

* All configuration should go in `lib/chef-workflow-config.rb`. Do not modify
  configuration in rake tasks or test files.
* If you add something in that expects additional configuration such as the
  foodcritic plugin, require this at the top of your
  `lib/chef-workflow-config.rb` or in your `Rakefile`, depending on the
  context. This ensures the configuration parameters exist before configuration
  runs.

configure_general
-----------------

Top-Level basics.

**Note**: Right now, both vagrant and ec2 configuration sections will exist
whether or not you configure them, and no matter what your
`machine_provisioner` argument is set to. One is unused, but both will be
"loaded".  This will be resolved in future releases, but just be aware of this.

* `workflow_dir`: the path to where chef-workflow keeps its stuff.
* `vm_file`: the path to the state database. by default, based off the `workflow_dir`
* `machine_provisioner`: the provisioner used to create virtual machines.
  Accepts `:vagrant`, `:ec2`, or a class name (which must exist when
  `lib/chef-workflow-config.rb` is loaded). Default is `:vagrant`. Read [[Stock
  Provisioners]] for more information.

configure_knife
---------------

Mostly things related to chef, but some more surprising things as this
originally was all of the configuration.

* `search_index_wait`: machines bootstrapped with the knife provisioner wait
  until they appear in the chef servers Solr index so that they are available
  for search. This is done to ensure the machine is fully ready to be tested
  and depended on. This value is an integer in seconds on how long to wait for
  this before timing out. The default is 60 seconds.
* `cookbooks_path`: The path to your cookbooks. Used by `chef:cookbooks:upload`
  and optional tasks like `chef:cookbooks:resolve`.
* `chef_config_path`: path to configuration bits for `knife`, namely
  authentication keys. default is `.chef-workflow/chef`.
* `knife_config_path`: path to `knife.rb` as used by all knife commands.
  default is `knife.rb` under `chef_config_path`.
* `roles_path`: path to your roles directory
* `environments_path`: path to your environments directory
* `data_bags_path`: path to your data bags. Must have this tree structure:
```
data_bags/
  bag_name/
    item1.json
    item2.json
  another_bag_name/
    item1.json
```
* `ssh_user`: The user account to use for ssh. The default is `vagrant`.
* `ssh_password`: The password to use for ssh. The default is `vagrant`.
* `ssh_identity_file`: The private key to use for ssh. The default is nil.
* `use_sudo`: If true, will perform most ssh commands with `sudo` prepended.
  Default true.
* `test_environment`: Environment name to use for bootstrapping. **You must
  create this yourself**. Default is `vagrant`.
* `test_recipes`: See [[Running Cookbook Tests]].
* `knife_config_template`: The template used to write out a new `knife.rb` when
  creating a chef server with the `chef_server:create` rake task. It would be
  in your interest to review the default before trying to change this.

