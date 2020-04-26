---
layout: post
---

# Metaprogramming Ruby 2 Paolo Perrotta

Spell is notation of a usefull patterns that you can use.

Metaprogramming is used to create a wrap around external system, for example
call *any* method which will be dynamically called on the system (no need to
define methods). You can mimics specific language which is more suitable for
solving that particular problem.
Also you can add logging functionality around method that you want to monitor,
run custom code whenever a client inherits from your class...

In C++ (not interpreted language) at runtime (after compilation finishes) you
can not ask for class methods, but in Ruby, you can use *introspection* for
example `my_object.class`.

Metaprogramming is writting code that manipulates language constructs at
runtime.

Instead of calling functional style `to_alphanumeric('string')` you can reopen
String class and use like `'string'.to_alphanumeric`. This is called
Monkeypatching.

Spell Refinement

You can refine classes inside module so you do not monkeypatch global class
```
module StringExtensions
  refine String do
    def to_alphanumeric
      gsub(/[^\w\s]/, '')
    end
  end
end

"a*".to_alphanumeric # NoMethodError
using StringExtensions
"a*".to_alphanumeric # 'a'

module StringStuff
  using StringExtensions # this is active only till the end of module
  "a*".to_alphanumeric # 'a'
end
```

Even `object.instance_variables` live in the object itself, its `object.methods`
is actually using class instance methods `object.class.instance_methods`.
Each class is also an object and as an object, it contains instance methods
```
Class.instance_methods(false) # => [:new, :allocate, :superclass]
```
You can use `.superclass` method to find a chain
```
Array.superclass # => Object
Object.superclass # => BasicObject
BasicObject.superclass # => nil
```
For `Class.superclass == Module` so `class` and `module` is almost the same,
except classes can call `.new` so we use them to instantiate, and we use module
to include it somewhere (since `Class` is a subclass of `Module` it can be used
as module, but we keep using only where we instantiate them).
Using `class MyClass` you are actually defining (or reopening) a instance of
`Class` and assigning to constant `MyClass`. Since `MyClass` is an object and
class in the same time, we can call `.class` (right ->) and `.superclass` (up ^)
Note that `MyClass.class == Class` and `MyClass.superclass == Object`.
Also `Class.superclass == Module` and `Module.superclass == Object` so all
classes are objects and they inherit from `Object` class. `Object.class ==
Class`, `Module.class == Class` and `Class.class == Class`.

Constant differs from variables in that way you can access them in any scope
```
M::C::MyConst
# `Module` class provides instance method and class method `constants`.
M.constants # => [:C]
Module.constants # => all global constants
```

Method lookup: to find a method, ruby goes in the receiver's class (class of the
object on which you call the method) and than climbs the ancestors chain.
Anchestors chain also include modules that are prepended or included in class.

Spell KernelMethod

You can add method to `Kernel` so you can use it in every object, like awesome
print `ap 'string'`

Every line of ruby code is executed inside an object, current object or `self`.

In static type checking, whether method exists is checked at compile time. It is
an advantage, but the price for that is that you have to write a lot of
boilerplate methods like getters, setters, and delegate methods.

Spell Dynamic Dispatch

Using `send` like `my_object.send :my_method, 123` you can decide which method
to call at runntime. With `.send` you can call private methods (use
`.public_send` to prevent calling private methods).

Spell Dynamic Method

You can define methods on the fly using `Module#define_method`
```
class MyClass
  define_method :my_method do |my_arg|
    my_arg * 3
  end

  # or use it inside class method
  def self.define_component(name)
    define_method(name) do
    end
  end
end
```

If you need to define method name for each associated object method, you can
iterate over `associated_object.methods.grep(/^get_(.*)/) {
MyClass.define_component $1 }`.

Spell Ghost Method

You can override `Object#method_missing` so you can respond to any method
```
class MyClass
  def method_missing(method_name, *args, &blk)
  end
end
```

Spell Dynamic Proxy

Object that catches Ghost Methods and forwards them to another object is called
dynamic proxy.
```
class MyClass
  class ResourceProxy
    def method_missing(message, *args, &block)
      subject.send(message, *args, &block)
    end

    def subject
      @subject ||= connection.get(path_prefix).body
    end
  end
end

class MyClass
  module API
    module Gists
      class Proxy < ::MyClass::ResourceProxy
        def star
          connection.put("#{path_prefix}/star").status == 204
        end
      end
    end
  end
end

# when you call gist#star it will call put request, but for other
# like, gist.description it proxy it to subject which will read description
# attribute from response (using Hashie)
```

If you call `.respond_to? :dynamic_method` than you will get false, so when
overriding `method_missing` we should also override `respond_to_missing?`
Note that you should call `super` for method names that you do not proccess,
otherwise you can get stack level too deep because of infinite loop when some
undefined method is called. Because of this it is prefered to use Dynamic
Methods instead Ghost Methods.
```
class MyClass
  def method_missing(name)
    super if !@data_source.respond_to?("get_#{name}_info")

    info = @data_source.send("get_#{name}_info", @id)
    price = @data_source.send("get_#{name}_price", @id)
    result = "#{name.capitalize}: #{info} ($#{price})"
    return "* #{result}" if price >= 100
    result
  end

  def respond_to_missing?(method, include_private = false)
    @data_source.respond_to?("get_#{method}_info") || super
  end
end
```

Skill Blank Slate

When the name of method could be inherited from Object (like method `.display`)
than we can not use it (since it is not method missing), there are two
solutions: inherit from BasicObject (not implicit Object) or removing methods
with `#undef_method` or `#remove_method` for each `instance_methods`.

```
class MyClass < BasicObject
  # no need respond_to_missing since BasicObject.instance_methods are few
  # => [:__send__, :!, :==, :!=, :equal?, :__id__, :instance_eval, :instance_exec]
end

# or undefining all except __methods
class MyClass
  def self.hide(name)
    if name !~ /^(__|instance_eval$)/
      undef_method name
    end
  end

  instance_methods.each { |m| hide(m) }
end
```

You can also use `.const_missing` as in Rake
```
class Module
  def const_missing(const_name)
    case const_name
    when :Task
      Rake.application.const_warning(const_name)
      Rake::Task
    when :FileTask
      Rake.application.const_warning(const_name)
      Rake::FileTask
    when :FileCreationTask
      # ...
    end
  end
end
```

Blocks are defined only when you call a method. Inside method you can call
`yield` to evaluate that block. You can check if block is given with
`block_given?`. Using `yield` without params will call the block using the same
params as method. Here is example to always call `resource.dispose` (when
exception occurs or without exception in block).
```
module Kernel
  def using(resource)
    begin
      yield
    ensure
      resource.dispose
    end
  end
end

use(r) { |r| r.read }
```

When you create a block, it captures local bindings at that moment into a
closure. When you yield, it will use that scope (not the scope of method where
yield is called).
Use `Kernel#local_variables` to list all current object local variables.

Spell Scope Gate

Scope changes when program enters a `class` or `module` definition or enters the
method `def` (method definition is executed when it is called).
```
my_var = "Success"
class MyClass
  # We want to print my_var here...
  def my_method
    # ..and here
  end
end
```

Spell Flat Scope

To pass local variable through `class` gate, you can use `Class.new block` and
`define_method name, block` to create nested lexical scopes (flattening the
scope). You can inherit from existing class using `Class.new(MyParrent) do`.
```
my_var = "Success"
MyClass = Class.new do
  "#{my_var} in the class definition"

  define_method :my_method do
    "#{my_var} in the method"
  end
end
```

Spell Shared Scope

Control the sharing of variables by using Dynamic Dispatch inside the same flat
scope.
```
def define_methods
  shared = 0

  Module.send :define_method, :counter do
    shared
  end
  Module.send :define_method, :inc do |x|
    shared += x
  end
end
```

Spell Context Probe

Use `object.instance_eval block` to evaluate block with the object as `self`.
It's like a snippet of code that you dip inside an object.
```
v = 2
obj.instance_eval { @v = v}
obj.instance_eval { @v} # => 2
```

There is also `Module#class_eval` (we are not changing only `self` but also
current_class). It is more flexible than using `class MyClass` because we do not
open new scope, we keep current bindings like Spell Flat Scope.
```
a_class.class_eval do
  # define instance_method
  def m; 'Hi'; end
end
```

Spell Clean Room

Create an object just to evaluate blocks inside it (in clear env, otherwise
blocks could clash with current env)
```
class CleanRoom
  def current_temperature
    # ...
  end
end
clean_room = CleanRoom.new
clean_room.instance_eval do
  if current_temperature < 20
    # TODO: wear jacket
  end
end
```

Spell Deferred Evaluation

Use `obj = Proc.new block` to create object that holds a block and later use it
with `obj.call 123`.
```
inc = Proc.new {|x| x + 1}
inc.call 2
```

Spell Class Instance Variable

Define instance variable in context of class object. Using double `@@my_var` is
class variable which can be accessed from subclasses and instance methods.
```
class MyClass
  @my_class_var = 1
  def m
    # we can not access @my_class_var
  end
end
```

Spell Singleton Method

Define method for one specific object
```
str = 'Hi'

def str.title?
  self.upcase == self
end
# or
str.define_singleton_method(:title?){ self.upcase == self }
```

Spell Class Macro

`Module#attr_accessor` class method is called Class macro. They looks like
keywords, but there are just regular class methods.
```
class MyClass
  attr_accessor :my_attribute

  # it will actually do something like
  def self.attr_accessor(name)
    define_method "#{name}=" do |value|
      send "@#{name}=", value
    end
  end
end
```

It can be used for deprecation warnings
```
class Book
  def title
  end

  def self.deprecate(old_method, new_method)
    define_method(old_method) do |*args, &block|
      warn "Warning: #{old_method}() is deprecated. Use #{new_method}()."
      send(new_method, *args, &block)
    end
  end

  deprecate :GetTitle, :title
end
```

Singleton class of the object (metaclass, eigenclass) ie each object can have
it's own special hidden class. Special syntax needs to be used `class << object`
and you can get it with `object.singleton_class`. It is a place where singleton
methods live. It's superclass is object class.

```
obj = My.new
def obj.my_singleton_method; end
obj.singleton_class.instance_methods.grep /my_/
obj.singleton_class.superclass == My
```
So method lookup also includes singleton class, let's call it `#obj` class.

We can also add class method using same notation
```
class C
  class << self
    def a_class_method
      'C.a_class_method'
    end
  end

  # same as
  # def self.a_class_method
  #   'C.a_class_method'
  # end
end
# or third way of defining class methods is outside of class
# def C.a_class_method

class D < C
end

D.a_class_method # => 'C.a_class_method()'
```

So method lookup for `D.a_class_method` also looks at singleton class of
superclass `#C`.
Seven rules or ruby object model
* there is only one kind of object: regular or a module
* there is only one kind of module: regular module, class or singleton class
* there is only one kind of method and it lives in a module (most often in a
  class)
* every object (classes included), has its own 'real class', be it a regular
  class or a singleton class
* every class, except BasicObject, has exactly one ancestor, either a superclass
  or a module.
* superclass of singleton class of an object is the object's class. The
  superclass of singleton class of a class is the singleton class of the class's
  superclass
* when you call a method, ruby goes right to the receiver's real class and than
  'up' the ancestors chain.

Spell Class Extension

When you want to include module methods as class methods
```
module MyModule
  def my_method; 'hello'; end
end

class MyClass
  class << self
    include MyModule
  end
end

# short way is using
class MyClass
  extend MyModule
end
```

Spell Object Extension

You can extend objects with methods
```
module MyModule
  def my_method; 'hello'; end
end

obj = Object.new

class << obj
  include MyModule
end

obj.my_method # => 'hello'

# another way is to use
obj.extend MyModule
```

Spell Around Alias

Three steps: alias the method, redefine original method, call old method from
new method
```
module Kernel
  alias_method :new_method, :original_method

  def original_method(args)
    # add code here
    new_method(args)
  end
end
```
for example
```
class String
  alias_method :real_length, :length

  def length
    real_length > 5 ? 'long' : 'short'
  end
end
```

Spell Refinement Wrapper

Using `super` inside refinement will call original method

```
module StringRefinement
  refine String do
    def length
      super > 5 ? 'long' : 'short'
    end
  end
end
```

Spell Prepended Wrapper

Using prepend (which is searched before class method, or included methods) so
also you can use `super` to call original method. It is much cleaner than spell
around alias (which looks like Monkey patching).

```
module ExplicitString
  def length
    super > 5 ? 'long' : 'short'
  end
end

String.class_eval do
  prepend ExplicitString
end
```

So you can use this to wrap some library methods so you can catch exceptions
from it
```
module AmazonWrapper
  def reviews_of(book)
    result = super
    result
  rescue
    []
  end
end

Amazon.class_eval do
  prepend AmazonWrapper
end
```

Spell String of Code

`Kernel#eval` can run string of code.

```
array = [1]
el = 2
eval "array << el"

# or define methods which also depend on verb
POSSIBLE_VERBS = ['get', 'put', 'post', 'delete']
POSSIBLE_VERBS.each do |m|
  eval <<-end_eval
    def #{m}(path, *args, &b)
      r[path].#{m}(*args, &b)
    end
  end_eval
end
```

It is used with `binding` so you can evaluate code in some context (similar to
closure but it does not contains code just scope). In ruby there is
`TOPLEVEL_BINDING`.

Spell Code Processor

`eval statements, @bindings, file, line`

Instead `eval string` you can use `obj.instance_eval` which can take string or
block (both can access local variables).
```
array = ['a', 'b', 'c']
x = 'd'
array.instance_eval "self[1] = x"
```

Spell Hook Method

You can override `Class#included` to provide a hook into particular event
(`inherited`, `included`, `prepended`, `extended`, also `method_added`,
`method_removed` and `method_undefined`)
```
class String
  def self.included(subclass)
    puts "#{self} was included by #{subclass}"
  end
end

class MyString < String; end
```

This can be used to `include MyModule` which will extend methods so you got
class macros which you can use. This is also called include and extend trick.
```
module CheckedAttributes
  def self.included(base)
    base.extend ClassMethods
  end

  module ClassMethods
    def my_class_macro(name, &block)
    ...

class MyClass
  include CheckedAttributes
```

Instead of overriding `self.included` you can override `include` and use super
but it is not so clear since you have to call `super`
```
class C
  def self.include(*modules)
    puts "Called: C.include(#{modules})"
    super
  end

  include M
end
```

Problem with extending is with chained extensions, since at the momen first
module `include SecondLevel` module, it's `def self.included(base)` base will
point to first level module, not the target class.
