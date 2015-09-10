videos
Goruco 2010 Aman Gupta: memprof https://vimeo.com/12748731
Goruco 2014 Aaron Quint: ruby perfomance tooling https://www.youtube.com/watch?v=cOaVIeX6qGg

~~~
types = Hash.new(0)
ObjectSpace.each_object do |obj|
types[obj.class] += 1
end
pp types.sort_by { |klass,num| num }
~~~

