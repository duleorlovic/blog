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
* github list of books
  <https://github.com/EbookFoundation/free-programming-books/blob/master/free-programming-books.md#professional-development>
Bundle

* http://www.railsbookbundle.com/
* http://rubybookbundle.com/
* awesome coding style guides <http://awesome-ruby.com/#-coding-style-guides>

Questions

* <https://gist.github.com/ryansobol/5252653> 15 questions

Gems
* https://www.ruby-toolbox.com/
* https://ruby.libhunt.com/
* https://rubygems.org/

Docs

* https://docs.ruby-lang.org/en/master/index.html
* mobile responsive version https://rubyapi.org/
* older https://ruby-doc.org/stdlib-2.5.3/libdoc/observer/rdoc/Observable.html

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
extended (ie all module's instance methods become class methods). Also usefull
is that module extends self `module M; extend self; def m;end;end` so you can
use `M.m` instead of `M::m` for module methods. Also you can use
https://apidock.com/ruby/Module/module_function so subsequent method definitions
becomes module functions `M.m` and they will be also available as instance
methods if you include this module.

When you extend `class A; extend B;end` than `A.is_a?(B) == true`. When you
include `class A; include B; end` than `A.new.is_a?(B) == true`.

`self` in a definition of a class is class object (which is assigned to
constant `MyClass`) so calling any methods will run in context of that class
object (for example `instance_methods(false)` will print methods that are
defined with `def`). When you define a method it becomes instance method of
current object class (even when you define method inside other method).
When you reopen the class or module, current class is itself (not the
`self.class == Class`).

You can see all superclasses and mixin modules with
[A.ancestors](http://ruby-doc.com/docs/ProgrammingRuby/html/ospace.html).
Module class object have `new` method, but it's instance (Module instance) does
not have (on other hand `Class` object `a = Class.new` have `a.new` method).
You can see all constants for particular class `Object.constants; A.constants`

Ruby gems use `self.included` hook to modify class that is including a module
(Rails version of this is `ActiveSupport::Concern`). So when you include some
module in your class, beside instance methods, it will extend and add some class
methods as well. For example
[Linkable](https://gist.github.com/duleorlovic/724b8ab1eb44d7f847ee)


For rails concerns you can see [concerns]({{ site.baseurl }}
{% post_url 2016-04-12-rails-tips %}#concerns)

~~~
module Emailable
  included do
    # before_ has_ macros
  end

  # instance methods

  def ClassMethods
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
  M or instance of class that mixes in M. You can add module to object
  ```
  object = Object.new
  object.extend M
  o.m
  ```
* singleton method on specific object `def obj.m;self;end` or using `class <<`
  ```
  class << obj
    def a_singleton_method
    end
  end
  ```
  To access singleton class (which holds all singleton methods) you can use
  `s = obj.singleton_class` or this exotic `class <<` syntax
  ```
  s = class << obj
        self
      end
  ```
  Singleton superclaass is object class `s.superclass == obj.class` so that is
  why method lookup search singleton, than class, and ancestor classes.

  You can add singleton class method
  ```
  class MyClass
    class << self
      def a_class_method
      end
    end
  end
  ```
* class definition `class C;puts self;def self.class_method;self;end;end` self
  in both class definition (singleton on class object) and class method is class
  object
* module definition is the same as class definition

When you call `some_method` it is called on current self `self.some_method`.
Only place where you need self is assignment `self.some_identifier = 1`.

`load "filename.rb"` includes that resource every time method is executed (like
copy paste code), but `require` only once and only when needed. `ruby -e 'puts
$:'` will list all load paths. `load` is usefull to overwrite with new changes
of a particular file. Imported constants will stay, but if you want to remove
them than you can use `load('filename.rb', true)` to load into anonymous module
and than destroy the module. Load is not used to import the code, usually only
to execute the code.

`require` is more suitable for importing the code (no need `.rb`).  It does not
know for current folder, so you need to use `./` dot that explicitly define path
`require "./f.rb"` or `$: << "."` or `require_relative "f"`(extension is not
needed `.rb`)
Before we used to extend load path
```
$LOAD_PATH.unshift(Dir.pwd)
```

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
Last defined (multiple includes do not have effect, ie it will be ignored if
already in ancestor lists) module is searched first, the same as with same_name
methods in class definition. You can see the order of modules with `C.ancestors`
(prepended, class, included, prepended of superclass, superclass, included of
superclass). `super` calls the same method at upper level. Since modules don't
have instances they generally represent properties of something of some class,
and they have adjective names (albeit class tend to be nouns). Modules also can
be used for namespacing some classes. When we see `M::C` we don't know if `C` is
constant, module or class, but `M` is a class or module since it has nested
items.

Proc are objects that can be called (executed) `p = proc { puts 1 };p.call`.
Rubocop suggest using `proc` instead of `Proc.new`.

Variables and scope

Ruby define scope of variable using its name, precisely, first char:

* `$` global `$FIRST_NAME` available on every scope
* `@` instance `@first_name` you can list with `self.instance_variables` and
get value with `self.instance_variable_get :@first_name` (note that we need
both `:` and `@`) and set value `self.instance_variable_set :@first_name,
'me'`. You can not use `send("@first_name")`
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
  [class_variable_set](https://apidock.com/ruby/Module/class_variable_set)

  ~~~
  class Fred
    @@foo = 99
    def foo
      @@foo
    end
  end
  Fred.class_variable_set(:@@foo, 101)     #=> 101
  Fred.new.foo                             #=> 101
  ~~~

  In rails 5.2 we can have `class_attribute :settings, default: {}` so you can
  use `MyClass.settings = 'asd'` which will set `@@settings = 'asd'`.

Every method call create its own local scope. Ruby does some preprocessing
compile time where it recognizes all local variables (class @@x instance @x and
global $x are recognized by their appearance)

~~~
if false
  x = 1
end
puts x # x is created but not assigned
puts y # Fatal error: y is unknown
~~~

In recursion, self is the same but scope is generated each time. But procedure
local objects share the same scope with parent. That way we can create something
similar to closures to simulate classes.

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
(in rails when including `ActiveModel::Model` you can call `Model.new
model_params` and those accessor will be assigned, ie no need to write
initialized and manually assign)
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
* `defined? a` is operator that can say if variable is defined. Also works for
  Module for example `defined? My::Module`
* `ruby -e 'p Kernel.private_instance_methods.sort'` prints all usefull script
  commands (require, load, raise)
* `puts caller` to print callstack
* `p method(:my_method).source_location` to find method implementation. To print
  method source you can use `user.method(:name).source.display`. For class you
  need to grab some method for example
  `m=User.method(:initialize);m.source_location`
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

* `! a == b` is not the same as `a != b`
* you should memoize with `return @current_isp if defined? @current_isp` instead
  of `@current_isp ||= blabla; return @current_isp` since it handles `false`
  value as well (it will not rerun the check if @current_isp = false)
  NOTE Do not memoize has_belongs to items since it could change while you are
  using it...
* when you iterate over some hashes you can assign variables with `name:`

  ~~~
  [{name: 'Duke', value: 1},].each do |name:, value:|
    puts name, value
  end
  ~~~

* big multiline strings can be concatenated from lines similar to HERE_DOC
  heredoc. It will show `\n` and will insert spaces because text is indented.
  There is `~HERE_DOC` version to strip spaces for all lines based on first line
  indent. Minus in `<<-HERE_DOC` means that it will not strip spaces based on
  first line, so you need to start from 0 column on each line.
  Instead of `<<~` you can use `<<HEREDOC.strip_heredoc`, to create single line
  in Rails you can use `<<~HEREDOC.squish`
  If you do not want intepolation `#{i}` than use with quotes `<<~'TEXT'`

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

  # also without assignment
  def html_boilerplate
    <<~HTML
    HTML
  end

  # you can create single line from it, in rails use HEREDOC.squish
  sql = <<-SQL.gsub("\n", ' ').squish
    SELECT MIN(u.popularity)
    FROM users u
    WHERE
      u.created_at <= #{Time.now} AND u.created_at >= #{Time.now - 7.days}
  SQL

  # you can pass as parameter to method, with arguments also
    Person.where(<<-SQL, my_time).
      users.created_at > ?
    SQL
    where(managers: { id: Person.find_by!(name: 'Eve') })
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
  You can interpolate later in runtime using hash and percentage `%{}` instead
  of hash `#{}`
  ```
  'this %{value} is interpolated' % { value: 2 }
  ```

  You can also write as single line strings (long line), without new line `\n`

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

* `%x(pwd)` is used to call system bash commands. Use Open3 to call system shell
  commands
  ~~~
  require 'open3'
  stdout, stderr, status = Open3.capture3("sleep 10")
  ~~~

  capture3 will wait for all output, even you use ampersand `sleep 10 &` at the
  end of command. You can use `system 'sleep 10 &'` to get immediatelly back to
  ruby. To show stream output you can use
  ```
  STDOUT.sync = true
  # or $stdout.sync = true
  system('some-long-script-with-outputs')
  ```

  Also spawn

  ~~~
  pid = spawn('sleep 3') #=> 45376
  # do something while process is running
  Process.wait pid # wait for process to finishs
  ~~~

  Also Open3.capture3 can return back from background process if you
  dont use `stdout.read` and use ampersand (sometimes do not block event without
  ampersand).
  https://www.rubydoc.info/stdlib/open3/Open3.popen3

  ~~~
  # tmp/r.rb
  # run with: ruby tmp/r.rb
  require 'open3'
  puts 'start'
  # system 'sleep 10 &'
  stdin, stdout, stderr, wait_thr = Open3.popen3('sleep 10 && ls &')
  pid = wait_thr[:pid]  # pid of the started process
  stdin.close # stdin, stdout and stderr should be closed explicitly in this form.
  # if you uncommend this line, than it will wait for output
  # puts stdout.read
  stdout.close
  stderr.close
  exit_status = wait_thr.value  # Process::Status object returned.
  puts 'end'
  ~~~

  Another solution to run in background is using `IO`

  ~~~
  require 'open3'
  require 'byebug'

  puts "start in #{Process.pid}"
  cmd = 'touch tmp/txt && sleep 10 && ls'
  io = IO.popen cmd
  puts "cmd started in #{io.pid}"
  # does not exist the loop
  # begin
  #   while true do
  #     Process.getpgid io.pid
  #     printf '.'
  #     sleep 1
  #   end
  # rescue Errno::ESRCH
  #   puts 'cmd exits'
  # end
  Process.wait io.pid
  puts 'end'
  ~~~

  Also
  ```
  pid = Process.fork { sleep 1 }
  # or simply
  pid = fork { sleep 1 }
  ```
  https://ruby-doc.org/core-2.1.3/Process.html#method-c-exit

  ~~~
  Process.kill("HUP", pid)
  ~~~

  To start multithread app you can use Thread.new and join
  ```
  5.times.map do
    Thread.new do
      5.times do
        Net::HTTP.get('example.com', '/index.html')
      end
    end
  end.each(&:join)
  ```

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

* get class based on string `klass = Object.const_get "User"`
* get constant of class `User::MAX`, `User.const_get 'MAX'` or
`user.class.const_get :MAX`. To check if exists use `const_defined?`
* to check if class is defined you can use
  ```
  if eval("defined?(#{nameofclass}) == 'constant'")
  end
  ```
* random number `[*1000..9999].sample`
* iterate over elements until first match `a.take_while {|i| i < 3}`
* get a class from value `my_string.constantize`
* you can find method using grep `o.methods.grep /iden/`
* you can list all instance methods in current class but not in parrent class
with `o.methods - o.class.superclass.instance_methods` or using
`o.class.instance_methods(false)` (show only methods defined in that class, not
inherited from Object).
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

# Procs

<https://en.wikibooks.org/wiki/Ruby_Programming/Syntax/Method_Calls#Understanding_blocks.2C_Procs_and_methods>
Using Proc ruby can be used in functional programming (it enable first-class
functions - function can be created at runtime, assigned to variable, passed as
argument and be return value from other functions).
Ruby differs from other functional languages for which we can use name of
function as variable with function type (in ruby we need to use Proc).
First-class functions can be used for higher-order functions (functions which
takes other functions as arguments).

> Proc objects are blocks of code that have been bound to a set of local
> variables. Once bound, the code may be called in different contexts and still
> access those variables.

~~~
def gen_times(factor)
  shared = 1
  _proc = proc {|n| n*factor*shared }
  shared = 2
  _proc
end
times3 = gen_times(3)
times5 = gen_times(5)

puts times3.call(12) # 12 * 3 * 2
puts times5.call(5) # 5 * 5 * 2
~~~

That is very similar to [closures]({{ site.baseurl }} {% post_url 2016-02-10-javascript-theory %})

You can exit from proc with `next` (`return` exits from method, `break` exits
from loop). Or you can raise exception.

Proc object can be used as argument for functor (functor is higher-order
function, ie takes function as argument).

~~~
def functor(a, b)
  a.call b
end

p = proc { |i| puts i }
functor(p, 1) # 1
~~~

Shorthand notation for creating Procs is `p = lambda {|i| puts i}` but there are
differences:

* proc object does not check the number of params (you can call `p.call 1,2,3`
or `p.call` args will be nil). Lambda throws an error if number of params does
not match. lamda are strict about arguments but procs are not
  ~~~
  my_lambda = ->(a, b) { a + b }
  # stabbly lambda operator is the same as
  my_lambda = lambda {|a, b| a + b }
  call_with_multiline_lambda(lambda do
  end)
  my_proc   = proc { |a, b| a + b }

  my_lambda.call(2)
  # ArgumentError: wrong number of arguments (1 for 2)

  my_proc.call(2)
  # TypeError: nil can't be coerced into Fixnum
  ~~~
* also if you use `return` inside proc object, it will stop current scope, but
for lambda it will not stop, it will just return value (so `return` is not
usefull). Avoid explicit return since you can call proc in top level scope and
raise error that it can not be returned.
```
def double(callable_object)
  callable_object.call * 2
end
l = lambda { 10 }
double(l) # => 20
```
```
def another_double
  p = proc { return 10 }
  result = p.call
  return result * 2 # unreachable code!
end
another_double # => 10
```

You can check if proc object is lambda with `p.lambda?`. Lambda are more similar
to methods (checks arity and simply exit when call return)

You can call lambda in different ways (note that you can not call like regular
function)

~~~
p = -> { puts "hi" }

p.call # preferred
p[]
p.()
~~~

Block can't live alone, it is used as last param of methods. Every method has
implicit argument for that block and since we do not have name, we use `yield`
to call it. Someone prefer to use explicit argument so we know that method
expects block.

~~~
def f(a, &block)
  # block.class # => Proc
  # we can use explicit
  block.call a
  # but also we can use
  yield a
end

f(1) { |i| puts i }
~~~

Note that proc is not an actual argument and you can not use parenthesis around
`f(1, lambda {|i| puts i})`.
You can use ampersand `&` to convert back to block in method call or
`Method#to_proc`.

~~~
def f(a)
  yield a
end
p = lambda {|i| puts i}
f(1, &p)
~~~

This is used for example in  `map` functor. If we use ampersand for symbol, than
it will be converted `Symbol#to_proc` and passed in.

~~~
class Symbol
  def to_proc
    lambda {|x, *args| x.send(self, *args)}
  end
end

words = %w(Jane, aara, multiko)
upcase_words = words.map {|w| w.upcase}
upcase_words = words.map {|w| w.send :upcase}
# short notation
upcase_words = words.map(&:upcase)
# can not easilly add argument to map method
https://stackoverflow.com/questions/23695653/can-you-supply-arguments-to-the-mapmethod-syntax-in-ruby
~~~

You can create proc callable object using trailblazer uber. We can skip
extending (since it is empty module) but it is used for check if that class is
callable (so we can use `klass.call(options)`)
https://github.com/apotonick/uber/blob/master/lib/uber/options.rb#L60
```
class MyModel
  extend Uber::Callable
  def self.call(options)
    puts options
  end
end

MyModel.(a: 1)
# { :a => 1 }
MyModel.is_a? Uber::Callable
```
Another way is to `include` and use instance method `call`
```
class MyModel
  include Uber::Callable
  def call(ctx, *args)
    puts
  end
end

MyModel.new.call a: 1
# { :a => 1 }
MyModel.new.is_a? Uber::Callable
```

* there are special methods in ruby
  * `method_missing(method, *args)` can be used to catch all missing methods
  * `initialize` to instantiate class

* call method can be called (invoked) using three (maybe four) ways:
  * dot notation `o.my_method`
  * send `o.send :my_method` or `o.send :my_property=, 'some_value'`
  * grab method and call it `o.method(:my_method).call`
  * interpolated method call is using send like `current_user.send
  "is_#{@model}_admin?"`

  <https://gist.github.com/kidlab/72fff6e239b0af1dd3e5> for something that need
  test or we just want to showcase

* in rails there is
[args.extract_options!](https://simonecarletti.com/blog/2009/09/inside-ruby-on-rails-extract_options-from-arrays/)
modify args and return last hash (or empty hash if there is no hash). Userfull
if you need to pass attrs by position and as keyword arguments

  ~~~
  def my_method(*args)
    options = args.extract_options!
    puts "Arguments (array):  #{args.inspect}"
    puts "Options (hash):    #{options.inspect}"
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

  if you need to deep merge (deep_merge method is deprecated) you can not use
  `merge` since it will override existing value. In this case it is better to
  set in initialization

  ```
  # this will override params[:project][:subscriber_attributes]
  def _project_params
    params.require(:project).permit(
      :name, :description, :priority, subscriber_attributes: Subscriber::PARAMS
    ).merge(
      company: current_user.company,
      subscriber_attributes: {
        company_id: current_user.company.id
      }
    )
  end

  # so only way to put inside initialization
    @project = current_user.company.projects.new(
      subscriber: Subscriber.new(company: current_user.company),
    )
  ```

* to merge hash in conditional way you can use:
  ~~~
  hash = {
    a: 1
  }.merge(condition ? {b: 2} : {})
  ~~~

  ~~~
  hash.tap do |h|
    h[:b] = 2 if condition
  end
  ~~~

  ~~~
  hash = {
    a: 1,
    b: (2 if condition),
    b: (condition? ? 2 : 3),
    c: if another_condition
         3
       else
         4
       end,
  }.reject { |k, v| v.nil? }
  # or in Ruby 2.4
  }.compact
  ~~~

  ~~~
  hash = {
    a: 1,
    **(condition ? {b: 2} : {})
  }
  ~~~

* to run script and require byebug you can use

  ~~~
  ruby -rbyebug my_script_with_byebug.rb
  ~~~

  if you want to debug some ruby cli you can write scripts that check for rvm
  ruby version
  ```
  #!/usr/bin/env ruby
  ```
  If you use some gems like byebug, add `gem 'byebug'` to Gemfile and from that
  folder run

  ~~~
  ruby -rbyebug $(which vagrant) up
  bundle exec ruby -rbyebug $(which vagrant) up
  ~~~

  To exit from script you can use puts || since puts returns nil
  ```
  puts "max=#{max} should be greater than 10" || exit unless max > 10

  ```

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

* `enum.map { }.flatten(1)` should be replaced with `enum.flat_map {}` since we
do not need to traverse twice
* `[1,2,3].map{ |n| next if n.even? ; 2*n }` will return `[2, nil, 6]` so if you
need to filter you can apply `.compact`
* for big arrays you can use rails `User.find_each do |user| end` so it loads in
batch of 1000 (It returns `nil`). You can use map, `User.find_each.map ` but
than you again load all users at once.
* use mutation methods (with !) so you do not need to create new objects. For
example `return hash.merge a: 1` should be replaced with `return hash.merge! a:
1` but for hash we can also use `hash[:a] = 1` whih is faster.
* hash fetch method can be used to define default value if the key does not
exists. You can use block instead of second argument to define default value,
and it is faster since block is not called if key is not found so
`hash.fetch(:foo, :my_default_value_computation)` should be replaced with
`hash.fetch(:foo) { my_default_value_computation }` so
my_default_value_computation is only evaluated when `hash[:foo]` is nil
* `string.gsub("asd", "qwe")` should be replaced with `string.sub("asd", "qwe")`
if we need to replace only first occurence, so no need to scan string to the
end. Also you can use `string.tr(" ", "_")`
* replace all double new lines `"\n\n"` with single `"\n"` with
  `s.squeeze("\n")` so you can string with

  ~~~
  def strip(text)
    text.tr("\n", " ").squeeze(" ")
  end
  ~~~
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
* to return hash after map you can use to_h `[1,2].map { |x| [x, f(x)] }.to_h`
* to assign multiple attributes to active record object you can use `Hash.slice`
(`pluck` is for database query) and `assign_attributes` to self

  ~~~
  new_user.assign_attributes user.slice :email, :phone
  ~~~

  On hash there is `values_at` to extract and assign
  ```
  a, b = hash.values_at :a, :b
  ```
  To fetch all attributes use `user.attributes` or specific ones
  `user.attributes.values_at 'id', 'email'` (if you have array of fields
  you need to expand `user.attributes.values_at *fields`). NOTE that we
  use string instead of symbols.
  On Rails 5.2 you can directly use slice on AR model `user.slice :id, :name`
  (before that you can do on `user.attributes.slice :id`)

* safe navigation operator `&.` can be used instead of `.try` for example: `user
&& user.name` can be written as `user&.name`. It is usefull with find_by for
example `User.find_by(email: 'a@b.c')&.tap { |u| }`
* In ruby 2.3 there is also `Array#dig` and `Hash#dig` so instead of
  `params[:a].try(:[], :b)` you can pick `params.dig(:a, :b)`. when you need to
  take value and not sure if provided (usually in some json response)
  if you want to find specific object you can use `select{} || {}`
  ```
  (array_or_hash.dig("users", "accounts")&.select{|account| account["type"] ==
  "facebook"} || {}).dig(0, "link")
  ```
* rails has hash except `my_hash.except :my_key` to ignore only my_key value and
returns also the hash. Also oposite select is slice `my_hash.slice :my_key` to
pick `{my_key: 1}` (also returns hash)
* If you need only values for specific keys use `my_hash.values_at :my_key,
:my_other_key`. Also given some value you can find key `my_hash.key some_value`
so use that instead of select {}.keys.first
* rails has
[deep_transform_keys](https://apidock.com/rails/v4.0.2/Hash/deep_transform_keys)
that can transform keys for your select `hash.deep_transform_keys{ |key|
key.to_s.titleize }`
* find in array by object property and return array of selected items
  `array.select {|i| i.user == current_user }`. To grab only first one you can
  use `array.detect {|el| }` or alias `array.find {|el|}`.
* ruby has `Hash#invert` which will replace keys and values.
  Hash#invert `{a: 1, b: 2}.invert # {1: a, 2: b}`
* you can create hash with `Hash[]`.
  ```
  >> [['a','b'], [1, 2]].transpose #=> [["a", 1], ["b", 2]]
  >> Hash[[['a','b'], [1, 2]].transpose] #=> {"a"=>1, "b"=>2}
  ```
* `map` and `collect` are the same methods
* to check if string starts with or ends with some substring prefix sufix you
can use `s.start_with? prefix` or `s.end_with? suffix`
* get substring based on position `s[3..-4]` or `s[3, s.length - 3]`

# Url encode

* `URI.escape 'a b' # => 'a%20b'`
* `URI.unescape 'a%20b' # => 'a b'`
* `CGI::escape 'a b' # => 'a+b'`
* `CGI.unescapeHTML 'a+b'`
* `ERB::Util.url_encode`
* `WEBrick::HTTPUtils.escape 'a b' # => a%20b`

For url the best is to use WEBrick::HTTPUtils.escape since it will not convert
`/` slashes but will convert space `file name` and square brackets `[1]`
```
module CommonPdf
  def escape(link)
    # space and square brackets needs to be escaped
    WEBrick::HTTPUtils.escape link
  end
end

RSpec.describe CommonPdf do
  it '#escape' do
    object = Object.new
    object.extend CommonPdf
    sample_link = 'http://my.domain/file name [1].pdf'
    expect(object.escape(sample_link)).to eq 'http://my.domain/file%20name%20%5B1%5D.pdf'
  end
end
```

Rails params use double backslash instead of one, for example if user fill in
`\d+` it will be saved as `\\d+`. To use with regexp you need to interpolate,
like `"str".scan(/#{my_param}/).first`

# Retry from rescue

~~~
begin
  retries ||= 0
  puts "try ##{ retries }"
  raise "the roof"
rescue
  retry if (retries += 1) < 3
end

# output:
# try #0
# try #1
# try #2
~~~

# Pry

https://github.com/pry/pry
Instert a line `binding.pry` (or `binding.irb` if you want irb)

~~~
pry
u = User.last
cd u
# to jump back you can:
# cd ..
# nesting
# jump-to 1


ls # list all methods, constants and variables for this user object
show-method full_name # or aliases: show-source or $
find-method xpath Nokogiri
stat Nokogiri::CSS.xpath_for
edit full_name # edit the source and reload automatically
full_name

play -l 123 # run line 123

wtf? # show stack trace of last exception
wtf??? # more lines
cat --ex # show exact line of exception

.git diff # run linux commands, just add prefix .
shell-mode # $ run commands, use #{} to interpolate ruby
~~~

For rails use https://github.com/rweng/pry-rails `gem 'pry-rails'`
```
show-routes
show-models
```
todo https://www.youtube.com/watch?v=4hfMUP5iTq8

# Tips

* insert in strings with percentage `"Nums are %f %f" % [1, 2]`
* [101 ruby code factoids](http://6ftdan.com/allyourdev/2016/01/13/101-ruby-code-factoids/)
* you can return only from methods, but not from `do end` blocks
* ruby regex match will return
[matchData](https://ruby-doc.org/core-2.2.0/MatchData.html) or `nil` if there is
no match (so you need to check if not nil).
https://ruby-doc.org/core-3.1.2/Regexp.html#class-Regexp-label-Named+captures
```
/(\w)(\w)/.match("ab").captures # => ["a", "b"]
```

For MatchData you can call `captures` to get matched groups. You can use block
form syntax of `if`

  ~~~
  exception.message.match(/for column (.*) at row/) do |match_data|
    detail += " for the field #{match_data.captures.first}"
  end
  ~~~

  If you do not want to use block syntax, you can replace `if` with array syntax
  and return first matching group
  ```
  detail = message[/asd@asd/, 1]
  ```
  or you can cast match to string
  ```
  email = message.match(/\S+@\S+/).to_s
  ```
  You can enable multiline match with m
  ```
  email = message.match /\S+@\S+/m
  ```

  for error `Lint/AmbiguousRegexpLiteral: Ambiguous regexp literal. Parenthesize
  the method arguments if it's surely a regexp literal, or add a whitespace to
  the right of the / if it should be a division` you can use alternative forms
  for regular expressions

  ~~~
  a = '[\w\s]+'
  /#{a}/
  %r[#{a}]
  Regexp.new a


  Regexp.new(t('user_mailer.welcome'))

  # to add multiline modifier to Regexp.new
  Regexp.new("some reg", Regexp::MULTILINE)
  ~~~


* decorators poro presenters
[thoughtbot](https://robots.thoughtbot.com/evaluating-alternative-decorator-implementations-in)
example code [slides](http://nithinbekal.com/slides/decorator-pattern/#/)
[video railsconf](https://www.youtube.com/watch?v=bHpVdOzrvkE)
* you can count non nil values in array with `[nil, 1, 2].compact # => [1,2]`
  for empty string or space only string you can use presence and map
  ```
  [nil, "", " ", 1].map(&:presence).compact # => [1]
  ```
* you can call methods with dot but also with double colon `"a".size` or
`"a"::size`. You can put spaces or new lines anywhere `a   .   size`.
* if you see error `method_missing': undefined method this` than you need to
reinstall ruby `rvm reinstall 2.3.1`. But is related to rubygems
[1420](https://github.com/rubygems/rubygems/issues/1420) and best patch is
[tenderlove
comment](https://github.com/rubygems/rubygems/issues/1420#issuecomment-169178431)
* argument list length could be variable, there is
[splat](https://en.wikibooks.org/wiki/Ruby_Programming/Syntax/Method_Calls#Variable_Length_Argument_List.2C_Asterisk_Operator)
star/asterix/expand `*` operator, where all other parameters are collected in
array (note you can not use splat and last hash attribute) (splat is only for
assignment for function parameters)

  ~~~
  def f(x, y, *allOtherValues)
  end
  f(1, 2, 3, c: 4) # allOtherValues = [3, { c: 4} ]
  f(1, 2) # allOtherValues = []
  ~~~

  If it is used in method call than it is oposed:

  ~~~
  a = [1, 2]
  f(*a) # is the same as f(1, 2)
  ~~~

  Note that assigning variables by unpacking the array

  ~~~
  a = [1, 2, 3]
  b, *rest = *a

  b    # => 1
  rest # => [2, 3]
  a # => [1, 2, 3]
  ~~~

  Expand inside array or hash
  ```
  a = [3, 4]
  b = [1, 2, *a, 5]
  # => [1, 2, 3, 4, 5]
  # it works fine for empty array
  a = []
  b = [1, 2, *a, 5]
  # => [1, 2, 5]

  # for hash use double splat
  a = { a: 1 }
  b = { b: 2, **a, c: 3}
  # => { b: 2, a: 1, c: 3 }
  ```

  so you can use destructuring block arguments (params), for example

  ```
  a = [[1, 2], [3, 4]]
  a.each_with_index do |(first, last), i|
  end
  ```

  Last argument could be hash so it can grab all params

  ~~~
  def f(args)
    puts args.inspect
  end
  f(a: 1, b: 2) # { a: 1, b: 2 }
  ~~~

  method default parameters can be set using ruby 2.0 keyword arguments `def
  my_method(name:, position: 1);end` They looks better than position arguments.
  It will be an error if we call without required arguments, for example
  `my_method position: 2`. It will be an error if we call with non existing arg
  `my_method name: 'me', date: 3`. You can mix with position arguments. It does
  not work if you instead of hash use object `ActionController::Parameters.new
  name: 'me'` so you need to call with `my_method name:
  ActionController::Parameters.new(name: 'me').name`
  In ruby > 2.0 you can use keyword arguments (params are exploded decomposed),
  for which you can define default values or they need to be required (hash key
  is required)

  ~~~
  def f(x, y, c: , d: 1)
  end
  f(1, 2, c: 1, d: 2)
  f(1, 2) # ArgumentError: missing keyword: c
  ~~~

  There exists double splats (**) which is used for hashes
  [link](https://alexcastano.com/everything-about-ruby-splats/)
  Single splat convert to array, but double splat convert to hash. Note that it
  only takes symbol keys (and leave string keys). And double splat differs from
  hash last arg since hash can have default value, but it can be overwritten
  with some non hash. So use splat if you really need last param to be hash

  ~~~
  def f_with_hash(a, h = {})
  end
  def f_with_double(a, **d)
  end
  f_with_hash 1, 2 # => h = 2
  f_with_double 1, 2 # => ArgumentError: wrong number of arguments (given 2, expected 1)
  ~~~

  Another usage is when you require some keys, and other are optional. Also when
  using `class` optional value since you can not use variable name `class` so
  attribute class can be extracted
  ```
  def f(name:, **options)
    link_to 'h','a', class: options[:class]
  end

  f(bla: 'asd') # ArgumentError (missing keyword: name)
  f(name: 'asd', class: 'a') # options = { class: 2 }
  ```
  Double splat can be used for merging hashes
  ```
  h = {
    **some_hash,
    b: 1,
    **some_method_that_returns_hash,
  }
  ```

  To see parameters of some function you can use

  ~~~
  Object.instance_method(:f).parameters
  ~~~

  Similar to asteriks, last parameter can be prefixed with ampersand (&) which
  means function expect a code block. A Proc object will be created and assigned
  to parameter.

  ~~~
  def f(a, &p)
    p.call a
  end
  # parenthesis for method params are required when passing using {}
  f(1) { |i| puts i } # output: 1
  # parenthesis are not required for do end
  f 1 do |i| puts i; end # output: 1
  ~~~

  Similarly, using ampersand in method call for a Proc object, it will be
  extrapolated/replaced with block `f2 &p` for which you can use:
  `def f2; yield; end`. This `&p` exists inside method with code block
  param.

* sort hash by keys with `h.sort` and by values with `h.sort {|a,b|
a[1]<=>b[1]}` or using `sort_by` with method `o.sort_by &:published_at`

~~~
h = { "a" => 20, "b" => 30, "c" => 10  }
h.sort                       #=> [["a", 20], ["b", 30], ["c", 10]]
h.sort {|a,b| a[1]<=>b[1]}   #=> [["c", 10], ["a", 20], ["b", 30]]
# example of group count
MemberPicture.group(:member_profile_id).count.sort {|a,b| a[1]<=>b[1]}
~~~

in rails you can get from ActiveRecord some objects by array of ids and if you
want to keep order you can with `index_by(&:id)` that will create hash of `{3:
user3, 8: user8, 1: user1}` but with no order, so you can sort that or slice

~~~
Something.find(array_of_ids).index_by(&:id).slice(*array_of_ids).values

# or
people_by_id = Person.find(ids).index_by(&:id) # Gives you a hash indexed by ID
ids.map {|id| people_by_id[id] }
~~~

But also ordering and other filtering can be done in sql
https://stackoverflow.com/questions/10150152/find-model-records-by-id-in-the-order-the-array-of-ids-were-given/38378457#38378457

* count for array presence `[1,1,2].tally # => {1=>2, 2=>1}`
* switch case example

~~~
case a
when 1..5
  "between 1 and 5"
else
  "other"
end
~~~

When you need to match object class do not use `object.class` in case statement
http://batsov.com/articles/2012/10/14/ruby-tip-number-3-matching-on-an-objects-class-in-a-case-expression/

~~~
# case receiver.class # this is wrong
# better not to use receiver.class == Customer, but receiver.is_a? Customer
# triple equal is used for that (for a.is_a? b, I can put label b on a)
# for module it tests for `.is_a?`
# for range it test for `.includes?`
# for regexp it test for `.match`
case receiver
when Customer
end
~~~

New in  Ruby 2.5
* `Module#attr, attr_accessor, attr_reader, attr_writer, define_method,
alias_method, undef_method` and `remove_method` are now all public.

  ~~~
  class A
    def initialize
      @some_instance = 1
    end
  end

  a = A.new # <A: @some_instance=1>
  A.attr_accessor :some_instance
  a.some_instance = 2
  ~~~
* rescue/else/ensure are allowed inside do/end blocks (without begin/end)
* bundler is standard library
* `Hash#transform_keys`
* yield_self (similar to rails's `try`) (`tap` returns object instead of last
line of the block).

~~~
to_upper = -> (str) { str.upcase }
"string".yield_self(&to_upper)
        .yield_self(&add_greeting)`
~~~

* tripple equal `===` is operator that for ranges calls `.includes?`, for regexp
  calls `.match?`, for proc calls `.call`
* fake dummy objects could be generated from hash with:
  ```
  require "ostruct"
  d = Struct.new(:name, :type, :years).new 'Dule', :kayak, 35
  d = OpenStruct.new name: 'Dule'
  d.name # => "Dule"
  ```
  If you need recursively generated OpenStruct than you can use it again
  ```
   a=OpenStruct.new(user: OpenStruct.new(name: "Dule"))
   a.user.name # => "Dule"
  ```

  or you can convert to json and parse with Open struct class: `h = { a: 1, b: {
  c: 1 }}; o=JSON.parse(h.to_json, object_class: OpenStruct);`. So now you can
  use nested attributes  `o.b.c # => '1'`.
  Another way is to use rails `h = ActionController::Parameters.new a: { b: { c:
  1 }}` so you can `h.merge! b:1`
  Also you can define methods on OpenStruct properties, usefull when you need to
  call it with argument or if it is an array
  ```
  d = OpenStruct.new(
    users: [
      OpenStruct.new(name: "Dule")
    ].tap do |users|
      users.define_singleton_method(:active) {users}
    end
  ).tap do |d|
    d.define_singleton_method(:m1) do |param|
      param
    end
  end

  d.m1 123 # => 123
  d.users.active
  ```

* raise error when hash key does not exists (instead of returning `nil`).
  `default_proc` will be executed if key does not exists

~~~
# config/initializers/constants.rb
class Constant
  def self.hash_or_error_if_key_does_not_exists(h)
    # https://stackoverflow.com/questions/30528699/why-isnt-an-exception-thrown-when-the-hash-value-doesnt-exist
    # raise if key does not exists h[:non_exists] or h.values_at[:non_exists]
    h.default_proc = -> (_h, k) { raise KeyError, "#{k} not found!" }
    # raise when value not exists h.key 'non_exists'
    def h.key(value)
      k = super
      raise KeyError, "#{value} not found!" unless k
      k
    end
    h
  end

  def self.CUSTOMER_EVENTS
    hash_or_error_if_key_does_not_exists(
      EDIT: 'edit',
    )
  end
end
~~~

~~~
# spec/models/constant_spec.rb
RSpec.describe Constant do
  it 'store string values' do
    expect(Constant.CUSTOMER_EVENTS[:EDIT]).to eq 'edit'
    expect(Constant.CUSTOMER_EVENTS.key 'edit').to eq :EDIT
  end

  it 'raise error if key does not exists' do
    expect do
      Constant.CUSTOMER_EVENTS[:not_exists]
    end.to raise_error KeyError

    expect do
      Constant.CUSTOMER_EVENTS.values_at 'not_exists'
    end.to raise_error KeyError

    expect do
      Constant.CUSTOMER_EVENTS.key 'not_exists'
    end.to raise_error KeyError
  end
end
~~~

* use `enumerator.map do ... end` block when there is a side effects and curly
  braces than you need return value `enumerator.map { ... }`

* documentation using
 [rdoc](https://en.wikibooks.org/wiki/Ruby_Programming/RubyDoc) SimpleMarkup
 format which is pretty mature, example is on
 https://ruby-doc.org/core-2.4.0/String.html but also Rails uses it
 https://guides.rubyonrails.org/api_documentation_guidelines.html
  ```
  # test rdoc in command line
  echo "+:to_param+" | rdoc --pipe
  ```
* yard is newer and it supports links https://www.rubydoc.info/github/rails/rails
  syntax for yard is simple markdown using
  [@tags](https://www.rubydoc.info/gems/yard/file/docs/Tags.md#taglist). It supportds rdoc markup.
    https://devhints.io/rdoc
  ```
  # This is a doc
  #
  # @example
  #    code
  # @param
  # @return [String] name
  # @deprecated
  # @see https://asd.com
  # :section: Related methods
  # Those methods are related
  # def input # :nodoc:
  # class M # :nodoc: all
  ```

  To run use:
* convert cast single value and array value to array `Array(1)` and
  `Array([1])` https://stackoverflow.com/questions/18358717/ruby-elegantly-convert-variable-to-an-array-if-not-an-array-already
* hash with indifferent access (symbol or string) this is Rails ActiveSupport
  `h = HashWithIndifferentAccess.new a: 2` so you can use `h[:a]` or `h['a']`
* inserting key value in hash at specific position
  ```
  # https://stackoverflow.com/a/17251424/287166
  class Hash
  def insert_before(key, kvpair)
    arr = to_a
    pos = arr.index(arr.assoc(key))
    if pos
      arr.insert(pos, kvpair)
    else
      arr << kvpair
    end
    replace Hash[arr]
  end
end
  ```

* `string split(' ').join(' ')` can be replaced with `'  a   b'.strip.squeeze`
* if you need snake case from class name, you can
`described_class.name.underscore`. Of you need class from controller name
`users_controller` you can `controller_name.classify.constantize` which returns
`UsersController`. Classify uses camellize and singularize ie inflectors
https://github.com/duleorlovic/rails/blob/master/activesupport/lib/active_support/inflector/methods.rb#L203
To check if exists use `['my_table'].inject(Object) { |c, n| c.const_defined? n}
# => true false` source
https://github.com/duleorlovic/rails/blob/master/activesupport/lib/active_support/inflector/methods.rb#L289
not sure why this does not raise error, but `Object.const_defined? 'my_table'`
railse
* to create array of hashes or array of new objects you can not use `[{}]*2` or
  `[User.new]*2` since that will be the same object in both places. You have to
  use `Array.new(2) { User.new }`
* interate and generate hash can be done using inject and accumulator
  ```
  array = [['A', 'a'], ['B', 'b'], ['C', 'c']]
  hash = array.inject({}) do |memo, (key, value)|
    memo[key] = value
    memo
  hash # => {'A' => 'a', 'B' => 'b', 'C' => 'c'}

  # find the longest word
  longest = %w{ cat sheep bear }.inject do |memo,word|
     memo.length > word.length ? memo : word
  end
  longest                                         #=> "sheep"
  ```

  but better is to use `each_with_object({}) { |el, a|` accumulator is last
  param not the first. You can summarize using `.inject` or `.each_with_object`
  or synonim is `.reduce`

* write file save to file in single line
  ```
  File.write '/path/to/file', 'My Text', mode: 'a'
  ```

  To read one liner
  ```
  file_content = File.read("/path/to/file")
  ```
* for read from a file `file = File.open(file_name)` you can call `file.read`.
  But you can create String io `io =  StringIO.new 'asd'` and call `io.read`.
  That is usefull for testing when you read from standard input
  ```
  io = StringIO.new 'some content'
  $stdin = io

  gets # 'some content'
  ```

* to check if ruby is called from cli
  ```
  #!/usr/bin/env ruby
  if __FILE__==$0
    puts "ergs=#{ARGV}"
  end
  ```

* to reopen the class you can use class eval like https://github.com/apotonick/gemgem-trbrb/blob/7cc8c7a0de78ba00092957a32d8cd234f102c73f/test/test_helper.rb#L19
  ```
  Cell::TestCase.class_eval do
    include ::Capybara::DSL
  end
  ```
* `{}` and `do end` have [different
  precedence](https://github.com/bbatsov/rubocop/issues/1520) so when rubocop
  alerts for `Style/Lambda: Use the `lambda` method for multi-line lambdas` you
  need parenthesis for example `scope :active, (lambda do ... end)`

* `.map` two arguments is not easy
*
todo

http://norswap.com/ruby-dark-corners/
https://www.amazon.com/Eloquent-Ruby-Addison-Wesley-Professional/dp/0321584104
http://www.confidentruby.com/
http://www.poodr.com/
http://poignant.guide/book/chapter-5.html
https://prograils.com/posts/ruby-on-rails-books-experienced-level
