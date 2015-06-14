# Backbone.ModelFactory

Provides a function for generating model constructors which will never produce multiple instances with the same unique identifier.

When you create a new instance of a model created with `Backbone.ModelFactory` and give it an `id` it will never create duplicate instances of the same model with a given `id`.

    var user1 = new User({id: 1});
    var user2 = new User({id: 1});
    var user3 = new User({id: 2});

    console.log(user1 === user2); // true
    console.log(user3 === user1); // false

## Benefits

When architecting loosely-coupled web applications, it is generally considered a good practice for modules which control separate pieces of functionality to not rely on each other. `ModelFactory` helps with that!

Using `ModelFactory` models, you can eliminate the need to pass model instances between unrelated views.

Additionally, collections using `ModelFactory` models that `fetch` data will always be populated by existing model instances if they already exist in the cache. This prevents creating multiple model instances which represent the same resource.

`ModelFactory` makes sharing models between collections, views, routers, etc. almost completely hands-off.

## Dependencies

`Backbone.ModelFactory` 1.2.0+ depends on the following libraries:

- Underscore: 1.2.0+
- Backbone: 0.9.9+

Earlier versions of `ModelFactory` will work with Backbone 0.9.0-1.0.0 and do not depend on Underscore directly.

## Inclusion

`Backbone.ModelFactory` supports three methods of inclusion.

1. Node:

        var Backbone = require('backbone-model-factory');

2. AMD/[RequireJS](http://requirejs.org):

        require(['backbone-model-factory'], function (Backbone) {
            // Do stuff...
        });

3. Browser Globals:

        <script src="path/to/backbone.js"></script>
        <script src="path/to/backbone-model-factory.js"></script>

## Usage

Rather than extending `Backbone.Model`, constructors are created by a call to `Backbone.ModelFactory`. Instead of this:

    var User = Backbone.Model.extend({
      defaults: {
        firstName: 'John',
        lastName: 'Doe'
      }
    });

...do this:

    var User = Backbone.ModelFactory({
        defaults: {
            firstName: 'John',
            lastName: 'Doe',
            isAdmin: false
        }
    });

`ModelFactory` also supports inheritance, so model constructors can extend each other by providing a model constructor (whether generated by `ModelFactory` or not) as the first argument:

    var Admin = Backbone.ModelFactory(User, {
        defaults: _.extend({}, User.prototype.defaults, {
            isAdmin: true
        })
    });

### Consequences of Extending Models

Models created with `ModelFactory` will __not__ share their unique-enforcement capabilities with models which they extend or which extend them. For example, using the `User` and `Admin` models above giving each the same `id` would __not__ result in the same object:

    var user = new User({id: 1});
    var admin = new Admin({id: 1});

    console.log(user === admin); // false

## Potential Gotchas

### Relationships and Recursion

If you are using any sort of nested relationships (models or collections within models) and operating on those relationships in a recursive manner, it can cause an infinite loop if an identical model instance exists in its own relationship graph.

For example, let's say a User has a collection of Posts and each Post has a User. Having an infinite path of object references (User -> Posts -> Post -> User -> Posts -> ad infinitum) is not harmful, but if you do any _automated serialization_ of these relationships (into JSON, etc), you might encounter an infinite loop if the serialization is unchecked.

This entirely depends on how you're using Backbone and what you've built upon it.

### Caching and Wiping

Since model instances are cached, the potential exists for unneeded objects to hang around in memory.

As of `ModelFactory` 1.2.0 you can `wipe` model instances once you are done with them. Both the model instances themselves and the constructor function have a `wipe` method which can be used:

    var User = Backbone.ModelFactory();

    var jake = new User({id: 1});
    var joe1 = new User({id: 2});
    var joe2 = new User({id: 2});
    var jane = new User({id: 3});
    var jen = new User({id: 4});
    var josh = new User({id: 5});
    var james = new User({id: 6});

    // Wipe the instance of User with id: 2 from cache. "jake" will still exist in memory.
    jake.wipe();

    // Wipe a single cached instance from the model. "joe1" and "joe2" will still exist in memory.
    User.wipe(joe);

    // Wipe multiple cached instances in an array or a collection. As before, "jane", "jen", "josh", and "james" exist in memory still.
    User.wipe([jane, jen]);
    User.wipe(new Backbone.Collection([josh, james]));

    console.log(_.keys(User._cache).length); // 0

__Note:__ This will only remove them from the _internal cache_ - any references in your views, collections, or elsewhere will __not__ be deleted!

In the example here, all the local variables ("jake", "joe1", etc) will be garbage collected, but it's up to the developer to clean up references from his/her view, collection, etc. objects.

## Tests

### Inclusion

There are 3 files in `test/inclusion` and they account for the 3 supported methods of including this module. To execute these tests, simply open the HTML files in a browser or install the npm dependencies and run `node test/inclusion/node-module.js`.

### Unit

[Mocha](http://mochajs.org/) tests exist in `test/test.js`.

To run the tests, make sure development dependencies are installed and use `npm test`.

To test against different versions of Backbone (or Underscore), install the version desired (e.g., `npm install backbone@~1.0.0`).

## Inspriation

This is inspired by SoundCloud's approach detailed in [Building the Next SoundCloud](http://backstage.soundcloud.com/2012/06/building-the-next-soundcloud/) under "Sharing Models between Views."
