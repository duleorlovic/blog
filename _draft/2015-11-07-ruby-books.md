~~~
layout: post
~~~


Here is what I did not think about long time ago so I'm writting here to better memorize.

But first, more imporant is to list Resources:

* http://ruby-doc.com/docs/ProgrammingRuby/
* https://en.wikibooks.org/wiki/Ruby_Programming
* http://www.toptal.com/ruby-on-rails
* http://www.toptal.com/ruby
* [Ruby User's Guide](http://www.rubyist.net/~slagell/ruby/)
* [The Book Of Ruby](http://www.sapphiresteel.com/ruby-programming/The-Book-Of-Ruby.html) or shorter version *The Little Book Of Ruby*


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

As everything in Ruby is object, so `class A;end` is also object with type Class. It extends a class called Module `A.class.superclass #=> Module`. That is why classes has more features than modules (can be instantied, can be extended by other classes). On other side, module can be included (ie all module's instance methods become avaiable as instance methods in the class) very similar to inheritance. Module can also be extended (ie all module's instance methods become class methods). You can see all superclases and mixin modules with [A.ancestors](http://ruby-doc.com/docs/ProgrammingRuby/html/ospace.html).

Ruby gems use `self.included` hook to modify class that is including a module (Rails version of this is `ActiveSupport::Concern`). So when you include some module in your class, beside instance methods, it will extend and add some class methods as well. For example [Linkable](https://gist.github.com/duleorlovic/724b8ab1eb44d7f847ee)

Singleton mixin is used when do not allow multiple instance of some class, ie we set `private_class_method :new`, and create it in some other class method `def Logger.create;@@loger = new unless @logger;end`

Access:

* public methods can be called by anyone, all methods are public by default (except `initialize` which is private)
* protected methods can be invoked by anyone, but only within context of methods of defining class or its subclasses (`a=A.new;a.prot` does not work if the call is outside of instance methods, if works in some method `b=B.new;b.access(a)` and `class B<A;def access(a);a.prot;a=A.new;a.prot;end;end`)
* private methods can be called only for that object of defining class or subclasses (so `some_object.priv_meth` does not work, only exception is for private writter methods `self.value=1` that can we called within context. It works inside any method `class B<A;def p;priv_math;end;end`)

`load "filename.rb"` includes that resource every time method is executead (like copy paste code), but `require` only once and only when needed.

Variables are not objects. They just hold a reference to objects. You can use `var.dup` to create another object or you can `var.freeze` to prevent modifications.

Ruby look up for methods for current object metaclass - eigenclass (for methods defined only for that particular object ie singleton methods - common in GUI - different action for different buttons) than in ancestors classes.

Proc are objects that can be called (executed) `p = proc { puts 1 };p.call`

Ruby define scope of variable using its name, precisely, first char: `$` global, `@` instance, `[a-z]|_` local, `[A-Z]` constant (only exception is `nil` which is constant and `self` which is global maintaned by interpreter). `defined?` is operator that can say if variable is defined. Global are accessible everywhere, local only inside proc, loop, def ... end, class ... end, module ... end. But procedure local objects share the same scope with parent. That way we can create something similar to closures to simulate classes.

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


* [Matrix](http://docs.ruby-lang.org/en/2.2.0/Matrix.html#method-c-5B-5D) `[]=`
is private but we can
[open](http://stackoverflow.com/questions/12683772/how-to-modify-a-matrix-ruby-std-lib-matrix-class/21559458#21559458)

~~~
class Matrix
  public :"[]=", :set_element, :set_component
end
~~~

[puts
debugger](https://tenderlovemaking.com/2016/02/05/i-am-a-puts-debuggerer.html)
* print callstack `puts caller`
* `p method(:my_method).source_location` to find method implementation
* if `method` is implemented, we can unbind from kernel and rebind to request object

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
