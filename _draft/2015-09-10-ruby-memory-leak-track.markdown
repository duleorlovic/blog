Grab logs:

On heroku you can use `heroku logs -t | tee log/production.log` or just download if you some log service enabled (like Logentries).
Grep only GET requests.

~~~
grep -o 'Started GET ".*" for' log/production.log | # -o is only
sed 's/Started GET "/http:\/\/localhost:3000/' | # remove Starteg GET
sed 's/" for$//' | # remote trailing for
grep -v auth | # don't use auth page and callback
tee urls.txt 
~~~

For ajax requests, you need to set `-H 'Content-Type:application/javascript;'` 
(I don't know why this does not work on rails)
For logged in users you need to set `-H 'Cookie: asd=asd'` where *asd* is keys which you can find in Chrome -> Chrome developer tools -> tab Resources -> submenu Cookies -> localhost -> very big *name* (~127 chars) and *value* (~542 chars)

In any ruby code you can use this snippet
~~~
# you can see memory with
# tail log/production.log -f | grep -o MEMORY.* --line-buffered | tee log/memory.log
# you need to remove *rails_12factor* heroku gem to see *log/production.log'
# and set config.serve_static_files = true
Thread.new do
  while true
    pid = Process.pid
    rss = `ps -eo pid,rss | grep #{pid} | awk '{print $2}'`.to_i
    Rails.logger.info "MEMORY[#{pid}]: time: #{Time.now} rss: #{rss}, live_objects: #{GC.stat[:heap_live_slots]}"
    sleep 2
  end
end
~~~

*GC.stat[:heap_live_slots]* is not available in ruby 2.1

It's good to have similar env as on production, so we will download database anduse it localy

~~~
sudo su postgres -c 'pg_restore -d my_app_production --clean --no-acl --no-owner /home/orlovic/Downloads/a.d'
~~~

To plot data from *memory.log*, run `gnuplot memory_profile.gp`

~~~
#!/usr/bin/gnuplot

# download production log from heroku and create urls.txt
# grep -o 'Started GET ".*" for' log/heroku_production.log | # -o is only
# sed 's/Started GET "/http:\/\/localhost:3000/' | # remove Started GET
# sed 's/" for$//' | # remote trailing for
# grep -v auth | # don't use auth page and callback
# tee urls.txt 

# For logged in users you need to set `-H 'Cookie: asd=asd'`
# you need to remove *rails_12factor* heroku gem to see *log/production.log'
# and set config.serve_static_files = true

# get the data with:
# tail log/production.log -f | grep -o MEMORY.* --line-buffered | tee log/memory_profile.log

# to see image, run with:
# gnuplot docs/memory_profile.gp
# gnome-open memory_profile.png

reset
set terminal png
set output "memory_profile.png"

set xdata time
set timefmt "%Y-%m-%d %H:%M:%S"
set format x "%H:%M"
set xlabel "time"

set ylabel "memory"
set y2label "heap_live_objects"
#set yrange [0:1000000]
set y2tics

set title "rss"
set grid
set style data linespoints

plot "log/memory_profile.log" using 3:7 with lines title "rss", \
  "" using 3:9 with lines lc rgb 'blue' title "heap_live_objects" axes x1y2

pause 5
reread
~~~


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


http://www.linuxatemyram.com/play.html
https://tunemygc.com/configs/f4179cc82e52ba59155dc285802e0a89
