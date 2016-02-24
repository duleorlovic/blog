---
layout: post
title: Ruby Books
---

Here is what I did not think about long time ago so I am writting here to better
memorize.

But first, more imporant is to list Resources:

* http://ruby-doc.com/docs/ProgrammingRuby/
* https://en.wikibooks.org/wiki/Ruby_Programming
* http://www.toptal.com/ruby-on-rails
* http://www.toptal.com/ruby
* [Ruby User's Guide](http://www.rubyist.net/~slagell/ruby/)
* [The Book Of
  Ruby](http://www.sapphiresteel.com/ruby-programming/The-Book-Of-Ruby.html) or
  shorter version *The Little Book Of Ruby*


Listing free books:

* http://www.allrubybooks.com/
* https://www.ruby-lang.org/en/documentation/
* http://www.e-booksdirectory.com/programming.php
* http://www.dmoz.org/Computers/Programming/Languages/Ruby/
* http://iwanttolearnruby.com/
* https://rubymonk.com/

Maybe not so nice to share, but also interesing index of books:

* https://it-ebooks.info/tag/rails/

Math and algorith tasks

* https://www.hackerrank.com
* https://projecteuler.net/
* http://www.beatmycode.com/
* http://codility.com/programmers/


# Ruby

## Object and classes

As everything in Ruby is object, so `class A;end` is also object with type
Class.  Instead of keyword `class` we can create a class object with `a =
Class.new` and instance with `o = a.new`. `class Name;end` defines constant
`Name` which holds class object.  Class `Class` extends a class called `Module`
ie `A.class.superclass #=> Module`. That is why classes has more features than
modules (can be instantied, can be extended by other classes). On other side,
module can be included (ie all module's instance methods become avaiable as
instance methods in the class) very similar to inheritance. Module can also be
extended (ie all module's instance methods become class methods). You can see
all superclasses and mixin modules with
[A.ancestors](http://ruby-doc.com/docs/ProgrammingRuby/html/ospace.html).
Module class object have `new` method, but it's instance (Module instance) does
not have (on other hand `Class` object `a = Class.new` have `a.new` method).

Ruby gems use `self.included` hook to modify class that is including a module
(Rails version of this is `ActiveSupport::Concern`). So when you include some
module in your class, beside instance methods, it will extend and add some class
methods as well. For example
[Linkable](https://gist.github.com/duleorlovic/724b8ab1eb44d7f847ee)

Singleton mixin is used when you want to disallow multiple instance of some
class, ie we set `private_class_method :new`, and create it in some other class
method `def Logger.create;@@loger = new unless @logger;end`

Access:

* public methods can be called by anyone, all methods are public by default
  (except `initialize` which is private)
* protected methods can be invoked by anyone, but only within context of methods
  of defining class or its subclasses `a=A.new;a.protected_method` does not work
  if the call is outside of instance methods, but we can extend the class and
  than call it `class B<A;def
  call_protected_method(a);a.protected_method;protected_method;end;end`
  `B.new.call_protected_method(a)`
* private methods can be called only for that object of defining class or
  subclasses. `a.private_method` does not work, only exception is for
  private writter methods `self.value=1` that can we called within context. It
  works inside any method `class B<A;def p;priv_math;end;end`

`load "filename.rb"` includes that resource every time method is executed (like
copy paste code), but `require` only once and only when needed. `ruby -e 'puts
$:'` will list all load paths. `load` is usefull to overwrite with new changes
of a particular file.

`require` is more suitable for features (no need `.rb`).
It does not know for current folder, so it needs explicit `require "./f.rb"` or
`$: << "."` or `require_relative "f"`.

Variables are not objects. They just hold a reference to objects. You can use
`var.dup` to create another object or you can `var.freeze` to prevent
modifications. `var.clone` is the same as `dup` but if you clone freezed object,
result is also freezed. `freeze` is only one level, nested objects are not
freezed `a=["a","b"];a.freeze;a[0][0]='b'` or `a={a:{b:1}};a.freeze;a[:a][:b]=2`

Ruby look up for methods for current object metaclass (eigenclass) than in
ancestors classes. Object metaclass contains those singleton methods defined
only for that particular object after it was created `o={};def o.t;puts
1;end;o.t()` - common in GUI - different action for different buttons.

Module can be mixed in in two ways: `include M` and `prepend M`. Methods are
searched in prepend modules, than in class methods and than in included methods.
Last defined (first time, multiple includes do not have effect) module is
searched first, the same as with same_name methods in class definition. You can
see the order of modules with `C.ancestors` (prepended, class, included,
prepended of superclass, superclass, included of superclass). `super` calls the
same method at upper level. Since modules don't have instances they generally
represent properties of something of some class, and they have adjective names
(albeit class tend to be nouns). Modules also can be used for namespacing some
classes. When we see `M::C` we don't know if `C` is constant, module or class,
but `M` is a class or module since it has nested items.

Proc are objects that can be called (executed) `p = proc { puts 1 };p.call`

Ruby define scope of variable using its name, precisely, first char:

  * `$` global `$FIRST_NAME`
  * `@` instance `@first_name`
  * `[a-z]|_` local `first_name`
  * `[A-Z]` constant `FIRST_NAME` (only exception is `nil` which is constant and
    `self` which is global maintaned by interpreter). Constant can be changed
    (with warning) so you can include one file several times. Object that
    constant reffers can be changed freely.

`defined?` is operator that can say if variable is defined. Global are
accessible everywhere, local only inside proc, loop, def ... end, class ... end,
module ... end. But procedure local objects share the same scope with parent.
That way we can create something similar to closures to simulate classes.

~~~
ruby> def box
    |   contents = nil
    |   get = proc{contents}
    |   set = proc{|n| contents = n}
    |   return get, set
    | end
   nil
ruby> reader, writer = box
   [#<Proc:0x40170fc0>, #<Proc:0x40170fac>] 
ruby> reader.call
   nil
ruby> writer.call(2)
   2
ruby> reader.call
   2
~~~

* `attr_accessor :price` creates getter and setter method. Its better to write
  all instance variable like that at beggining so we know what defines state
* required and optional arguments for methods `def f(a, b=1, *c, d)` prefix
  asterix means that all other arguments will be sponged up an array `c`. `a`
  and `d` are required, ie you can not call `f(1)`. `b` has default value
* [Matrix](http://docs.ruby-lang.org/en/2.2.0/Matrix.html#method-c-5B-5D) `[]=`
is private but we can
[open](http://stackoverflow.com/questions/12683772/how-to-modify-a-matrix-ruby-std-lib-matrix-class/21559458#21559458)

~~~
class Matrix
  public :"[]=", :set_element, :set_component
end
~~~

* `echo "puts RbConfig::CONFIG['sitedir']" | ruby - | xargs ls -R` will show all
  C extensions (.so files). Use `CONFIG["site_ruby"]` to list libraries.

# Keywords

* `C.superclass # => Object` shows superclass
* `ruby -e "class C;end;puts C.ancestors" # [C, Object, Kernel, BasicObject]`
  shows mixed in modules `Kernel` and superclases
* `C.public_method_defined? 'my_method'` to chech if method is defined
* `puts caller` to print callstack
* `p method(:my_method).source_location` to find method implementation
* if `method` is overwritten in some class, we can unbind from kernel and
  rebind to request object
  [link](https://tenderlovemaking.com/2016/02/05/i-am-a-puts-debuggerer.html)

  ~~~
  method = Kernel.instance_method(:method)
  p method.bind(request).call(:headers).source_location
  ~~~

* to list all methods that are called from `render`

  ~~~
  def index
    @users = User.all
    tp = TracePoint.new(:call) do |x|
      p x
    end
    tp.enable
    render 'index'
  ensure
    tp.disable
  end
  ~~~
