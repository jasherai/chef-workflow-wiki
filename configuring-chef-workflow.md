Configuring Chef Workflow
-------------------------

The file `lib/chef-workflow-config.rb` will be created in your repository if
you followed the instructions from [[Bootstrapping Chef Workflow]]. It contains
many options you can configure to suit your environment.

There are multiple sections, each corresponding to a different facet of the
system. Note that this configuration file is used by all functions of
chef-workflow, so it must exist even if you only wish to use, say, the test
library.

Sections are configured with the dsl included. For example:

```ruby
configure_general do
  machine_provisioner :ec2
end
```

Most options will come with comments describing what they do.

If you wish to inspect the running configuration, the rake task
`chef:info:config` will yield it.

If you want to understand more about the configuration system, check out the
[[Configuration Sections]] documentation.
