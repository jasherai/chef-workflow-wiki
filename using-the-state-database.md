Using the State Database
------------------------

The state database API is used to persist items about the running state of
machines managed by chef-workflow. Things like public IP addresses and
provisioning information are stored in it. This allows independent invocations
of chef-workflow tools access to everything that chef-workflow knows that
cannot be determined with hand configuration.

A Note About chef-workflow 0.1.1 and earlier
--------------------------------------------

chef-workflow 0.1.1 and below uses a Marshalling system to implement much of
this which in no way is similiar to this API (0.2.0 and greater), which is
backed by SQLite. It is strongly recommended you upgrade for this reason alone.
The marshal system was something that grew beyond its usefulness and has a lot
of problems, namely surrounding consistency, that the new layer does not have.

If you did not deal with the VM or IP databases directly, but did so with calls
like `get_role_ips`, you should be unaffected.

Basic API
---------

First off, the SQLite database is initialized at chef-workflow boot, as a
singleton. You can access it by calling
`ChefWorkflow::DatabaseSupport.instance`, which probably is only necessary if
you're writing a DB model.

There are a set of primitive data structures in
`chef-workflow/support/db/basic` as `Object`, `List`, `Set`, and `Map`. These
are mapped to tables with a consistent schema. Those familiar with perl's `tie`
will see the resemblance to the pattern.

Constructors take a table name, and with the exception of `Object`, take an
additional `object_name` argument which lets you partition many objects within
a single table, which allows you to organize related lists by type. Tables are
created if necessary during construction.

With the exception of `Set` which does not account for arbitrary values (just
strings), values are sent to SQLite by way of Marshal.

After that, you operate on the object like you would with similar objects. As
hinted at above, ruby has no real analogue to perl's `tie`, so these are not
actually `Array` or `Set` objects, for example. Not all methods will work (but
many will).  Some effort has been made to provide methods to allow copying to a
native type.

Example:

```ruby
a_set = ChefWorkflow::DatabaseSupport::Set.new("some_sets", "set_1")
another_set = ChefWorkflow::DatabaseSupport::Set.new("some_sets", "set_2")

# both objects have all their operations applied to the database at call time.
# They use the same table for their operation, "some_sets", but different object
# names.

a_set.add("woot")
another_set.add("f00f")
a_set.add("f00f")
a_set.include?("woot") #=> true
a_set.to_set & another_set.to_set # => Set.new("f00f")

# hashes (aka Maps)

a_map = ChefWorkflow::DatabaseSupport::Map.new("some_maps", "map_1")
a_map["hey"] = "there"
# That's right, you can nest them. No, they don't have to be in the same table.
a_map["hello"] = ChefWorkflow::DatabaseSupport::Map.new("some_maps", "map_1_sub")
a_map["hello"]["world"] = 1
```

Note that calls like `to_a` and `to_set` copy to native types -- modifications
of the return values they yield will *not* be persisted.

The basic API is not designed to be fast or fancy. It's designed to be simple and
durable. Things like `each` over large data sets will be very slow. It was
written largely because frameworks that do this already are huge and
unnecessary for what chef-workflow needs. This said, the tables created are
well-indexed and constrained where possible.

VMGroup API
-----------

The VMGroup API lives in `chef-workflow/support/db/group` and is a conflation
of `Map` and `List` for the purposes of managing server groups in the
scheduler. There is more detail in [[Understanding the Scheduler]]. 
