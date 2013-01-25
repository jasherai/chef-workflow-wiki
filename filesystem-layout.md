Filesystem Layout
-----------------

The "filesystem" as it is, is merely a directory structure of common
components. The reason documenting this is necessary is twofold:

* What directory a component lives in is important to certain end-user tooling,
  such as `chef_workflow_task`.
* The chef-workflow gems are designed to be a pick-and-choose deal, but present
  a universal hierarchy. You don't need the testing framework, but if it
  defines a support library and you *are* using it, it'll exist alongside all
  the chef-workflow gem's support libraries. This allows developers to extend
  the system in a way that easy for anyone familiar with the system to
  understand.

You don't obviously have to conform to this, it's just probably a good idea.

Directories and Purpose
-----------------------

Here's a table. Note that the "Primary Gem" field merely indicates what gem
provides most of the baked-in content in this directory, it's not necessarily
where all of it lives.

All chef-workflow tooling across all gems lives has a root of `chef-workflow/`.
Anything outside of that is probably a build product.

<table>
<th><td><b>Directory</b></td><td><b>Purpose</b></td><td><b>Primary gem</b></td></th>
</table>
