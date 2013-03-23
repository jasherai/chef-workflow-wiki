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
  `lib/chef-workflow-config.rb` is loaded). Default is `:vagrant`. Read [[Stock Provisioners]] for more information.

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

configure_vagrant
-----------------

Things related to vagrant. Note that these are only consulted if the
`machine_provisioner` is set to `:vagrant`

* `box_url`: The url to your vagrant box -- currently chef-workflow does not
  support local-only boxes.

configure_ec2
-------------

Things related to EC2. Note that these are only consulted if the
`machine_provisioner` is set to `:ec2`.

**BIG UGLY NOTE ABOUT ACCESS KEYS**: You can specify them in the configuration,
and it is a very good idea you do, on a separate testing account. If you don't,
your environment will be consulted -- the somewhat standard `AWS_ACCESS_KEY_ID`
and `AWS_SECRET_ACCESS_KEY`. The EC2 provisioner creates its own security
groups by default, and only destroys the machines it knows about, but
**consider yourself warned**. See [[Stock Provisioners]] for more information
on using EC2.

* `access_key_id`: The AWS access key used to provision machines and manipulate
  security groups.
* `secret_access_key`: The AWS secret access key.
* `ami`: The Amazon Machine Image identifier used for provisioning machines.
* `instance_type`: Called a "flavor" by other systems, this is the machine
  class. `m1.large`, etc.
* `region`: The AWS region to provision in. Unlike most tools, there is **no
  default**. You must set this.
* `ssh_key`: The name of the SSH key registered with AWS. Be sure to couple
  this with `ssh_identity_file` from `configure_knife`.
* `security_groups`: a list of group names or `:auto`, the default. `:auto`
  will create a new group and provision all your machines underneath it. It
  will also be cleaned up when tasks like `chef:clean` are run.
* `security_group_open_ports`: only used when `security_groups` is set to
  `:auto`. Authorizes all ports in the list ingress on both tcp and udp.
  Default is `[22, 4000]`.
* `provision_wait`: The time in seconds to wait after an instance provision is
  requested before giving up. The default is 300 seconds (5 minutes).
