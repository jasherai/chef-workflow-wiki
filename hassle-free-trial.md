Try Chef-Workflow Hassle Free
-----------------------------

Given the complexity of getting started with Chef-Workflow, it's not at all
surprising how one might say "forget that", or something less suitable for
documentation.

We recognize this, so we've configured an example repository that's
bootstrapped and ready to go with some basic cookbooks (that have tests) and
some basic integration tests. Chef-Workflow is completely self-hosted, from the
chef-server up, so the environment is not only isolated but you should just be
able to follow the instructions below and start playing around with both
following the documentation and making your own additions.

[Here's the repository](https://github.com/chef-workflow/chef-workflow-example).
These simple instructions will get you started. 

* You need a copy of Ruby MRI 1.9.3 (minimum), with [Bundler](http://gembundler.com) installed to it.
* You also need an installed and working copy of [VirtualBox](http://virtualbox.org).
* Running the tests and chef server will probably take about 1.5G of RAM which will be "locked", or unswappable.
* If you have locally addressable machines in the `10.10.10.0/24` subnet, set TEST_CHEF_SUBNET in your environment to a dotted-quad that ends in a 0, that isn't locally accessible.
* Clone it to your machine, or fork it and clone your fork.
* cd into the clone and type 'bundle install'
* If you can't wait and just want to see it run, try `bundle exec rake test:full`, which will do an end-to-end full test, including setting up a brand new chef server, and will clean up after it finishes.
* You should be ready to follow the rest of the documentation.
