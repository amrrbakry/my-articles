# Runtime Introspection in Ruby


Ruby offers a variety of _methods_ that allows you to ask an object about its capabilities (which messages/methods does it respond to?), its variables and constants, and its backstory (the object's class and ancestors).



## **Methods Introspection**

Let's start by asking an object about its non-private `methods`:

```ruby
str = 'a string'
str.methods

# => [:include?, :%, :*, :+, :unicode_normalize, :to_c, :unicode_normalize!, :unicode_normalized?, :count, ...]

```

`methods` returns an array of symbols representing the names of the public and protected methods for the receiver string object *`str`*.

We could also ask the object about:

* `private_methods`
* `public_methods`
* `protected_methods`
* `singleton_methods`

by default, Ruby will look up and list all the methods in the object's class and its ancestor classes and modules. If you want to list methods in the object's class only, you could pass the argument *false* or *nil* to any of these methods.

```ruby
str = 'a string'
str.public_methods(false)

# public methods defined in String only
# => [:include?, :%, :*, :+, :unicode_normalize, :to_c, :unicode_normalize!, :unicode_normalized?, :count, ...]

```

Sometimes, you'll only need to know if an object knows about a specific method, and that's when we use the aptly-named `respond_to?`:

```ruby
str = 'a string'
str.respond_to?(:include?)
# => true
str.respond_to?(:first)
# => false
```

## **Variables and Constants Introspection**

In the same sense, we can query a class about `instance_variables`, `class_variables`, and `constants`.

```ruby
class Cat
  @@cats_count = 2

  CATS = ['Luna', 'Milo']

  attr_accessor :name, :age

  def initialize(name)
    @name = name
  end
end

cat = Cat.new('Max')
cat.age = 2

cat.instance_variables
# => [:@name, :@age]

cat.class.class_variables
# => [:@@cats_count]

cat.class.constants
# => [:CATS]
```

Also, there are top-level methods to list `local_variables` and `global_variables`.

```ruby
# irb session
>> local_variables
# => [:_]

```
This _underscore_ local variable is actually a very interesting one; it always holds the last evaluated expression:

```ruby
Cat.new('Milo')
# => #<Cat:0x00000002415440 @name="Milo">
cat = _
# => #<Cat:0x00000002415440 @name="Milo">
```

## **The Object's Backstory**

Finally, let's ask the object about its `class`, `superclass`, and `ancestors`

```ruby
cats = ['luna', 'milo']
cats.class
# => Array

cats.class.superclass
# => Object

cats.class.ancestors
# => [Array, Enumerable, Object, Kernel, BasicObject]
```

`ancestors` returns a list which includes its receiver (Array) and modules included in Array (Enumerable, Kernel), Array's superclass (Object), and lastly the superclass of Object which is (BasicObject).


But that's not it. There are also more convenient ways to ask about the object's class...

- `is_a?`
- `kind_of?`
- `instance_of?`

this trio of predicate methods could also help us with that query:

```ruby
cats = ['luna', 'milo']
cats.is_a?(Array)
# => true
```

One important difference to note here is that `instance_of?` will only return true if the receiver object is a direct instance of the class and not of a subclass:

```ruby
class A; end
class B < A; end

b = B.new
# => #<B:0x00000001fac638>
b.kind_of?(A)
# true
b.instance_of?(A)
# false

```

This was just a glimpse of ruby's introspection capabilities, but there are always very interesting things you can learn about your objects!

_This article can also be found on [Dev.to](https://dev.to/amrrbakry/runtime-introspection-in-ruby-2po1)._

_This article can also be found on [Medium](https://medium.com/@amrrbakry/runtime-introspection-in-ruby-b2d718ec704f)._
