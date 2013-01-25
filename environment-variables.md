Environment Variables
---------------------

Environment variables are a great way to partition shared configuration from
personal or one-time configuration. Most of the variables in here are tailored
to suit that need.

CHEF_WORKFLOW_DEBUG
-------------------

Used to tune the amount of noise chef-workflow generates. It defaults to `2`.
Setting it to `0` is a good idea if you're running under CI. The current levels
recognized are 0-3.

You can leverage this environment variable by using the tools provided in the
DebugSupport library -- see [[Support Libraries]].

CHEF_CONFIG
-----------

Points at the working knife configuration. Useful for driving rake tasks from a
configuration that's not the one defined with `configure_knife`.

Note that this is also what's consulted when building a chef server with the
`knife-server` gem, which will use the information inside of your configuration
to generate new credentials such as a client and validation key.

**In other words, don't set this to your production credentials and create a
chef server.**

TEST_CHEF_SUBNET
----------------

This is only used by the vagrant provisioner as of this writing, but as other
local machine provisioners are used this may be re-used there as well.

Defines a `/24` subnet -- machines created with chef-workflow will get an
unused IP address within this subnet. The default is `10.10.10.0`.

It's most useful to change this when you're on a network that has addressable
machines in 10.10.10/24 as VirtualBox will overtake these addresses. ARP is a
hell of a drug.

The code that does this is less than ideal. If you change this, be sure the
subnet definition always ends in `.0`. Also, it only does /24 right now. Sorry.

AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
-------------------------------------------

For the EC2 provisioner, these are used if an access and secret key are *not*
provided in the chef workflow configuration. While the provisioner and
orchestration should be fairly safe, this has the potential to make someone's
day very bad. Consider yourself warned.

REFACTOR
--------

This is more of a development tool -- turns deprecated calls from warnings into
exceptions. Pretty much only useful if a pull request is in your near future.
