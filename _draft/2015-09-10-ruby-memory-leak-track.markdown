videos
Goruco 2010 Aman Gupta: memprof https://vimeo.com/12748731
Goruco 2014 Aaron Quint: ruby perfomance tooling https://www.youtube.com/watch?v=cOaVIeX6qGg
Rocky Mountain Ruby 2014, Hemant Kumar, RBKit https://www.youtube.com/watch?v=hcaYjiAIres

blog post from github https://github.com/blog/1475-escape-velocity
~~~
types = Hash.new(0)
ObjectSpace.each_object do |obj|
types[obj.class] += 1
end
pp types.sort_by { |klass,num| num }
~~~

~~~
./bin.rbtrace -p 8963 -e 'GC.start(full_mark: true); require "objspace";\
ObjectSpace.dump_all( output: File.open("heap.json","w"))"
~~~

## Discourse

* http://samsaffron.com/archive/2013/11/22/demystifying-the-ruby-gc
* benchmarking localy https://meta.discourse.org/t/benchmarking-discourse-locally/9070
* 

# About GC
http://thorstenball.com/blog/2014/03/12/watching-understanding-ruby-2.1-garbage-collector/

## TTM1

* http://tmm1.net/
* http://tmm1.net/ruby21-rgengc/


## How rails show fixes in memory

* https://github.com/rails/rails/pull/21523


# Simulate load

`ab -n 500 -c 5 -C 'asd123=asd123' http://yourapp.com/`

* [rblineprof](https://github.com/tmm1/rblineprof)
* [stackprof](https://github.com/tmm1/stackprof) replacing [perftools](https://github.com/tmm1/perftools.rb)
* [stackprof-remote](https://github.com/quirkey/stackprof-remote)
allocation_tracer

[ruby-prof](https://github.com/ruby-prof/ruby-prof)
https://github.com/tmm1/rbtrace

http://blog.skylight.io/hunting-for-leaks-in-ruby/
http://www.slideshare.net/engine_yard/debugging-ruby
