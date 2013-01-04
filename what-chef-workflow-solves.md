What Does Chef-Workflow Solve?
------------------------------

The project is divided into 3 major parts:

* The task library formalizes how development with chef gets done in a customizable and malleable way.
* The test library provides integration testing features, which expose classes of bugs you would not find with other testing methods.
* The core is a framework for doing all the things you do by hand to support chef now. This largely consists of library support for VM/cloud systems and state tracking.

The *goal* is to provide a system that allows you to keep a clear record in a
repository of your expectations of your stack, both the machines that power it
and the team that administers it.

The tasklib allows you to clearly define, with code, the procedure your team
should use to perform rote tasks. Everything from "run a unit test for a
cookbook" to "upload cookbooks to the chef server" is a covered topic, with
various smaller steps composed to create them. While the defaults are what we
think most people will want to do, there's nothing keeping you from changing
that -- the whole system is designed to support it. Need to resolve cookbooks
before you upload? How about encrypting certain databags during upload? Maybe
you need to gpg encrypt something you check in. These are handled in a
consistent way for anyone who has a copy of the repository.

The testlib provides integration testing and can literally stand up and
exercise your whole stack if you tell it to. Instead of repeating [[Why
Integration Testing Matters]] here, you should probably read that if you're
interested.

But most importantly when these two projects are used together, you can
completely isolate any work you need to do with chef and get real-world
results. Throwing chef-workflow in jenkins is totally doable and recommended,
but if you're developing there are a lot of ways to abuse the isolation
features without clogging up a chef server with 15 environments and maybe you
have the right version of the cookbook up there. Additionally, you can actively
(and easily) bring up multiple machines from different roles and observe the
interactions between them.

In a sentence, Chef-Workflow lets you simulate production on your laptop.
