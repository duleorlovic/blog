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

Bundle

* http://www.railsbookbundle.com/
* http://rubybookbundle.com/

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
You can see all constants for particular class `Object.constants; A.constants`

Ruby gems use `self.included` hook to modify class that is including a module
(Rails version of this is `ActiveSupport::Concern`). So when you include some
module in your class, beside instance methods, it will extend and add some class
methods as well. For example
[Linkable](https://gist.github.com/duleorlovic/724b8ab1eb44d7f847ee)

For rails concerns you need to [video](https://youtu.be/bHpVdOzrvkE?t=1640)

~~~
module Emailable
  included do
    # before_ has_ macros
  end

  # instance methods

  def ClassMethods
    # class methods without self.
  end
~~~

Singleton mixin is used when you want to disallow multiple instance of some
class, ie we set `private_class_method :new`, and create it in some other class
method `def Logger.create;@@loger = new unless @logger;end`

`true.class # TrueClass` there is only one copy of these objects (true, false,
nil). This is example of singleton pattern.

Access:

* public methods can be called by anyone, all methods are public by default
  (except `initialize` which is private)
* protected methods can be invoked by anyone (explicit receiver), but only
  within context of methods of defining class or its subclasses (at calling time
  self is from same class hierarchy) `a=A.new;a.protected_method` does not work
  if the call is outside of instance-method for A, but we can extend the class
  and than call it `class B<A;def call_protected_method_for_a_and_self(a);
  a.protected_method;protected_method;end;end`
  `B.new.call_protected_method_for_a_and_self(a)`. Protected is like private but
  caller (self) and receiver object are from same class inheritance.
* private methods can be called only for that object in defining class or
  subclasses. Can not be called with explicit receiver: `a.private_method` does
  not work, only exception is that you can call private writter method with
  explicit receiver as long as the receiver is exactly the self `self.value=1`.
  So private methods can be called inside any method of descendand class `class
  B<A;def p;private_method_from_A;end;end`. Note that top-level methods are
  private instance method of Object class. Thats why thay can be easily called
  in bareword style (no explicit receiver).


Methods:

* top-level methods (outside of any definition block) are private to all objects
* instance-method in class `class C;def m;self;end;end;` self is instance of C
* instance-method in module `module M;def m;end;end;` self is object extended by
  M or instance of class that mixes in M
* singleton method on specific object `def obj.m;self;end` self is that obj
* class definition `class C;puts self;def self.class_method;self;end;end` self
  in both class definition (singleton on class object) and class method is class
  object
* module definition is the same as class definition

When you call `some_method` it is called on current self `self.some_method`.
Only place where you need self is assignment `self.some_identifier = 1`.

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

Variables and scope

Ruby define scope of variable using its name, precisely, first char:

  * `$` global `$FIRST_NAME` available on every scope
  * `@` instance `@first_name`
  * `[a-z]|_` local `first_name` in every definition block: proc, loop, def end,
    class end, module end, and is reset on each call. Local variable is very
    scoped (you can not access in another nested method)

    ~~~
    a = 1
    def p
      puts a
    end
    p # NameError: undefined local variable 'a'
    ~~~

    but you can access from closure definitions such: `define_method`, `proc` or
    `lambda` or plain old `begin end` block

  * `[A-Z]` constant `FIRST_NAME` (only exception is `nil` which is constant and
    `self` which is global maintaned by interpreter). Constant can be changed
    (with warning) so you can include one file several times. Object that
    constant reffers can be changed freely. Constant has global reachability
    when you know the chain of nested definition `C::M::X`, absolute path is
    `::String`
  * `@@` class variable `@@class_variable` are shared only between class and all
    instances of that class. Instance variable for class object are not
    available in instances of class so can not be shared between instances :(
    also globals and constants can be changed everywhere so that's bad too.
    It can be used in class definition, class methods and instance methods. It
    is shared with all ancestor classes so better is to store data at class
    object instance and create attr_accessor for that.

Every method call create its own local scope. Roby does some preprocessing
compile time where it recognizes all local variables (class @@x instance @x and
global $x are recognized by their appearance)

~~~
if false
  x = 1
end
puts x # x is created but not assigned
puts y # Fatal error: y is unknown
~~~

In recursion, self is the same but
scope is generated each time. But procedure local objects share the same scope
with parent. That way we can create something similar to closures to simulate
classes.

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

* `attr_accessor :price` creates getter and setter method. its better to write
  all instance variable like that at beggining so we know what defines state
* required and optional arguments for methods `def f(a, b=1, *c, d)` prefix
  asterix means that all other arguments will be sponged up an array `c`. `a`
  and `d` are required, ie you can not call `f(1)`. `b` has default value
* [matrix](http://docs.ruby-lang.org/en/2.2.0/Matrix.html#method-c-5B-5D) `[]=`
is private but we can
[open](http://stackoverflow.com/questions/12683772/how-to-modify-a-matrix-ruby-std-lib-matrix-class/21559458#21559458)

  ~~~
  class Matrix
    public :"[]=", :set_element, :set_component
  end
  ~~~

* params can be exploded

  ~~~
  def initialize(name:, address:)
    @name = name
    @address = address
  end
  ~~~

* `(1..10).tap {|x| puts x.inspect}.to_a` is used to perfom operation on
  intermediate results of operations. It returns object `x`. Usually used with
  initialization of objects: `Class.new(name: name).tap do |c| ... end`

#loops

# Keywords

* `C.superclass # => Object` shows superclass
* `ruby -e "class C;end;puts C.ancestors" # [C, Object, Kernel, BasicObject]`
  shows mixed in modules `Kernel` and superclases
* `C.public_method_defined? 'my_method'` to chech if method is defined
* `echo "puts RbConfig::CONFIG['sitedir']" | ruby - | xargs ls -R` will show all
  C extensions (.so files). Use `CONFIG["site_ruby"]` to list libraries.
* global variables `$0` filename, `$$` processID, `$:` or `$LOAD_PATH` paths
(you can add folder to load path with `$:.unshift 'lib'` or `$: << '.'`)
* `defined? a` is operator that can say if variable is defined.
* `ruby -e 'p Kernel.private_instance_methods.sort'` prints all usefull script
  commands (require, load, raise)
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

* `{}` and `do end` have [different
  precedence](https://github.com/bbatsov/rubocop/issues/1520) so when rubocop
  alerts for `Style/Lambda: Use the `lambda` method for multi-line lambdas` you
  need parenthesis for example `scope :active, (lambda do ... end)`
* `! a == b` is not the same as `a != b`
* you should memoize with `return @current_isp if defined? @current_isp` instead
  of `@current_isp ||= blabla; return @current_isp` since it handles `false`
  value as well (it will not rerun the check if @current_isp = false)
* when you iterate over some hashes you can assign variables with `name:`

  ~~~
  [{name: 'Duke', value: 1},].each do |name:, value:|
    puts name, value
  end
  ~~~

* big multiline strings can be concatenated from lines similar to HERE_DOC. It
will show `\n` and will insert spaces because text is indented. There is
`~HERE_DOC` version to strip spaces for all lines based on first line indent.

  ~~~
  MY_TEMPLATE = <<HTML
    <html>
    </html>
  HTML

  MY_TEMPLATE #=> "  <html>\n  </html>\n"

  INDENT = <<~HERE_DOC
    1
      2
    3
  HERE_DOC

  INDENT #=> "1\n  2\n3\n"
  ~~~

  Same HERE_DOC functionality can be achieved with `%()` or `%Q()` with
  interpolation (downcase `%q()` is without interpolation, same as `'HTML'`).
  Indent is important since lines are joined with `\n`.  First character is not
  important `%Q()` is the same as `%Q{}`

  ~~~
  my_str = %(
  This is multiline
  1 + 1 is #{1+1}
  )
  ~~~

  You can also write as single line strings, without new line `\n`

  ~~~
  string = "this is a \
            long string with a lot of spaces"
  string = "this is also one "\
           "long string but without spaces"
  ~~~

* `%W` and `%w` returns arrays (interpolated or not). Same is with
`HERE_DOC.split`

  ~~~
  >> %W(#{foo} Bar Bar\ with\ space)
  => ["Foo", "Bar", "Bar with space"]
 
  >> %w(#{foo} Bar Bar\ with\ space)
  => ["\#{foo}", "Bar", "Bar with space"]
  ~~~

* `%x(pwd)` is used to call system bash commands

* to send some data as json you can do it `user.templates.map {|t| t.slice :id,
  :name}.to_json`
* return multiple values from method, you can use arrays and `a, b = method` but
  it is better to create new class for result.
* if your service needs a lot of validation, raise exceptions to return message.
  Look at [rails tips service example]({{ site.baseurl }}
  {% post_url 2016-04-12-rails-tips %}).
* its convetion to use keyword `fail` instead of `raise` (it is used only if you re-raise exception)
* ruby caret operator `^` is bitwise XOR, if you want square use power
  (exponent) `**`
* if you do not know where something is called, for example some validation
  errors is added, you can dynamically extend objects
  [link](http://blog.iempire.ru/2016/09/23/breaking-points). So create module in
  runtime and extend the object or class, so it will stop on next `errors#add`
  method

  ~~~
  mod = Module.new do
    def add(*args)
      binding.pry
      super
    end
  end
  credit_card.errors.extend(mod)
  ~~~

* you can open singleton_class of the object and redefine

  ~~~
  num = Object.new
  num.instance_exec {
    def == other
      other == 3
    end
  }
  num == 4
  # => false
  num == 3
  # => true
  ~~~

* you can open a class with module_exec

  ~~~
  class Thing
  end

  Thing.module_exec(arg) do |arg|
    def hello
      "Hi"
    end
  end

  puts Thing.new.hello
  ~~~

* random number `[*1..100].sample`
* iterate over elements until first match `a.take_while {|i| i < 3}`
* get a class from value `my_string.constantize`
* you can call lambda in different ways

  ~~~
  my_lambda = -> { puts "hi" }

  my_lambda.call # preferred
  my_lambda[]
  my_lambda.()
  ~~~

* lamda are strict about arguments but procs are not

  ~~~
  my_lambda = ->(a, b)  { a + b }
  my_proc   = Proc.new  { |a, b| a + b }

  my_lambda.call(2)
  # ArgumentError: wrong number of arguments (1 for 2)

  my_proc.call(2)
  # TypeError: nil can't be coerced into Fixnum
  ~~~

* you can find method using grep `o.methods.grep /iden/`
* you can list all instance methods in current class but not in parrent class
with `o.methods - o.class.superclass.instance_methods`
* in rails it is enough to exclude all default object methods `o.methods -
Object.methods`

# Irb

* to suppress out of commands use `;` in irb

  ~~~
  require 'rest-client'
  RestClient.get('blackbytes.info');
  ~~~

* underscore variable `_` holds value of the last command. When used in code
than it means we are not going to use it.

# Expections

* [exception ruby video](https://www.youtube.com/watch?v=BlTjn_SZQT0)
* before ruby program exists it uses some callbacks

  ~~~
  trap("EXIT") { puts 'trap("EXIT")' }
  at_exit { puts "at_exit" }
  END { puts "END" }
  raise "Not handled"

  output:

  trap("EXIT")
  END
  at_exit
  a.rb:4:in `<main>': Not handled (RuntimeError)
  ~~~

# Tips

* Hash#invert `{a: 1, b: 2}.invert # {1: a, 2: b}`
* insert in strings with percentage `"Nums are %f %f" % [1, 2]`
* [101 ruby code factoids](http://6ftdan.com/allyourdev/2016/01/13/101-ruby-code-factoids/)
* you can return only from methods, but not from `do end` blocks
* method default parameters can be set using ruby 2.0 keyword arguments `def
my_method(name:, position: 1);end` They looks better than position arguments. It
will be an error if we call without required arguments, for example `my_method
position: 2`. It will be an error if we call with non existing arg `my_method
name: 'me', date: 3`. You can mix with position arguments. It does not work if
you instead of hash use object `ActionController::Parameters.new name: 'me'` so
you need to call with `my_method name: ActionController::Parameters.new(name:
'me').name`
* ruby regex match will return
[matchData](https://ruby-doc.org/core-2.2.0/MatchData.html) for which you can
call `captures` to get matched groups. You can use block instead of `if`

  ~~~
  exception.message.match(/for column (.*) at row/) do |match_data|
    detail += " for the field #{match_data.captures.first}"
  end
  ~~~

* decorators poro presenters
[thoughtbot](https://robots.thoughtbot.com/evaluating-alternative-decorator-implementations-in)
example code [slides](http://nithinbekal.com/slides/decorator-pattern/#/)
[video railsconf](https://www.youtube.com/watch?v=bHpVdOzrvkE)
* safe navigation operator `&.` can be used instead of `.try` for example: `user
&& user.name` can be written as `user&.name`. In rinu 2.3 there is also
`Array#dig` and `Hash#dig` so instead of `params[:a].try(:[], :b)` you can
`params.dig(:a, :b)`
* you can call methods with dot but also with double colon `"a".size` or
`"a"::size`. You can put spaces or new lines anywhere `a   .   size`.
* destructuring block arguments (params), for example `a = [[1, 2], [3, 4]]` can
be iterated with `a.each_with_index do |(first, last), i|`

* if you see error `method_missing': undefined method this` than you need to
reinstall ruby `rvm reinstall 2.3.1`
* in rails there is
[args.extract_options!](https://simonecarletti.com/blog/2009/09/inside-ruby-on-rails-extract_options-from-arrays/)
modify args and return last hash (or empty hash if there is no hash).

  ~~~
  def my_method(*args)
    options = args.extract_options!
    puts "Arguments:  #{args.inspect}"
    puts "Options:    #{options.inspect}"
  end

  my_method(1, 2, :a => :b)
  # Arguments:  [1, 2]
  # Options:    {:a=>:b}
  ~~~

* in rails you can set up default options with [reverse
merge](https://apidock.com/rails/Hash/reverse_merge)

  ~~~
  options = options.reverse_merge(size: 25, velocity: 10)
  # is equivalent to
  options = { size: 25, velocity: 10 }.merge(options)
  ~~~

* to run script and require byebug you can use

  ~~~
  ruby -rbyebug my_script_with_byebug.rb
  ~~~

  if you want to debug some ruby cli (file that begins with `#!/usr/bin/env
  ruby...`), you can call it (if you use Gemfil, add byebug to it and wrap with
  bundle exec)

  ~~~
  ruby -rbyebug $(which vagrant) up
  bundle exec ruby -rbyebug $(which vagrant) up
  ~~~

* you can use Resolv ruby class, but sometimes you get error `uninitialized
constant Resolv`. You should reload all services or you can add to Gemfile

  ~~~
  # http://stackoverflow.com/questions/42967546/ruby-puma-error-unable-to-load-application-nameerror-uninitialized-constant
  gem 'rubysl-resolv'
  ~~~

# Fast ruby using optimizations for speed

List of methods that are faster than other methods
[fast-ruby](https://github.com/PerfectMemory/fast-ruby)
[video](https://www.youtube.com/watch?v=fGFM_UrSp70)
[benches](https://github.com/ombulabs/benches)

* every method has implicit argument "block" which you can yield

~~~
def slow(&block)
  block.call
end

def fast
  yield
end
~~~

* `enum.map { }.flatten(1)` should be replaced with `enum.flat_map {}` since we
do not need to traverse twice
* use mutation methods (with !) so you do not need to create new objects. For
example `return hash.merge a: 1` should be replaced with `return hash.merge! a:
1` but for hash we can also use `hash[:a] = 1` whih is faster.
* hash fetch method can be used to define default value if the key does not
exists. You can use block instead of second argument to define default value,
and it is faster since block is not called if key is not found so
`hash.fetch(:foo, :my_default_value_computation)` should be replaced with
`hash.fetch(:foo) { my_default_value_computation }`
* `string.gsub("asd", "qwe")` should be replaced with `string.sub("asd", "qwe")`
if we need to replace only first occurence, so no need to scan string to the
end. Also you can use `string.tr(" ", "_")`
* do not use exception for controll flow, so better is to check before

~~~
begin
  foo
rescue NoMethodError
  bar
end
~~~

should be replaced

~~~
if respond_to?(:foo)
  foo
else
  bar
end
~~~

* `Time.parse('2001-01-01 06:06:06 UTC')` should be raplaced with
`Time.at(999232123).utc)` since we do not need to parse every time
* instead of `map(&:id)` use `pluck(:id)`


todo

http://norswap.com/ruby-dark-corners/
https://www.amazon.com/Eloquent-Ruby-Addison-Wesley-Professional/dp/0321584104
http://www.confidentruby.com/
http://www.poodr.com/
http://poignant.guide/book/chapter-5.html

