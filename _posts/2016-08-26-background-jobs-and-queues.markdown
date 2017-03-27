---
layout: post
title: Background jobs and queues
---

# ActiveJob - Rails wrapper

[Using ActiveJob](http://guides.rubyonrails.org/active_job_basics.html) can
simplify writing jobs so you can change queuing backend.
`rails g job my_todo --queue critical` it will generate job for which we will
just log 'Hi'.

~~~
# app/jobs/my_todo_job.rb
class MyTodoJob < ActiveJob::Base
  queue_as :critical

  def perform(*args)
    puts "puts will show in QUEUE=* rake resque:work"
    puts "use VERBOSE=1 to see job name, queue and params"
    Rails.logger.info "This will show in log/development,log: start #{args}"
    sleep 5
    Rails.logger.info "MyTodoJob end"
  end
end
~~~

You can use that with `wait` and `wait_until`

~~~
MyTodoJob.perform_later args
MyTodoJob.set(wait: 1.week).perform_later args
MyTodoJob.set(wait_until: Date.tomorrow.noon).perform_later args
~~~

With ActiveJob you can pass entire ActiveRecord objects because GlobalID will
deserialize for us. But if you are using directly some jobs (not inherited from
ActiveJob::Base) than you should pass object_id.

Rails provides in-process queuing (it keeps in memory) and you need some backend
queuing library to store that to db and later execute. That process should be
started separately from rails.

# Sidekiq

~~~
# config/application.rb
    config.active_job.queue_adapter = :sidekiq
~~~

Sidekiq is faster but requires thread safe code.

You need to install redis server

# Resque

Add resque by adding it to Gemfile, and you need to add some config files.

~~~
echo "gem 'resque'" >> Gemfile
bundle

sed -i config/application.rb -e '/^  end$/i \
    config.active_job.queue_adapter = :resque\
'

cat >> lib/tasks/resque.rake << HERE_DOC
require "resque/tasks"

# we need to load our rails environment
# without this, we can call: QUEUE=* rake environemt resque:work
task "resque:setup" => :environment
HERE_DOC
~~~

You need to install and configure redis

~~~
cat >> config/redis.yml << HERE_DOC
development: &default localhost:6379
test: *default
production: <%= REDISTOGO_URL %>
HERE_DOC

cat >> config/initializers/resque.rb << HERE_DOC
config = YAML.load_file(Rails.root.join('config', 'redis.yml'))
# configure redis connection
Resque.redis = config[Rails.env]
HERE_DOC
~~~

You can use Rails ActiveJob or any class module that responds to `perform`

~~~
mkdir app/jobs
cat >> app/jobs/process.rb << HERE_DOC
module SimpleJob
  @queue = :default

  def self.perform(args)
    puts "puts will show in QUEUE=* rake resque:work"
    puts "use VERBOSE=1 to see job name, queue and params"
    Rails.logger.info "logger will show in log/development,log: start #{args}"
    sleep 3
    Rails.logger.info "SimpleJob end"
  end
end
HERE_DOC
~~~

You can enqueue from rails console (watch out that you do not already have some
background worker running, because it will eat those jobs).

~~~
3.times { |i| Resque.enqueue(SimpleJob,i) }
3.times { |i| MyTodoJob.perform_later i }
# note that SimpleJob will wait until critical tasks are done
~~~

Note that if you use `ActiveJob::Base` than you do not need to restart process
when changing job, but if you use plain class than you need to kill and start
again. In either case you do not need to reload rails console, since it merely
sends name of jobs.

You can list all queues with `Resque.queues` (`low, critical, default`)

To send notification in case of error use [exception notification for resque]( {{ site.baseurl }}
{% post_url 2015-01-11-good-rails-exception-notification-better-than-tests %}#resque)


Since you need to run separate process for background jobs, you need to write
`Procfile`. By default heroku will run with WEBRICK server and it is advisable
to use puma.

Always use `QUEUE` when calling `QUEUE=* rake reque:work`

* `QUEUES=comma,separated,list`
* `QUEUE=my_queue`
* `QUEUE=*` or `QUEUES=*`

~~~
// Procfile
web: bundle exec puma -C config/puma.rb
worker: env TERM_CHILD=1 QUEUE=* bundle exec rake resque:work
~~~

You can start both web and worker with `foreman start`
`heroku addons:create redistogo` is needed to enable redis on heroku.

# Resque scheduler for recurring tasks

You can create heroku scheduler (cron task) and call (for example every hour)
your custom rake task in which you can add new jobs.
Disadvantage of this approach is memory. Every time it starts new Rails
environment (that could be 450MB) and several could be at the same time.
Also I do not like to write the code outside of git, so I prefer to use plugins.

All plugins use [rufus-scheduler](https://github.com/jmettraux/rufus-scheduler).

resque-scheduler works in that way to check every 5 seconds if some of the tasks
should be processes. If it find them, they are pushed to jobs queue.

~~~
# lib/tasks/resque.rake
require 'resque/tasks'
require 'resque/scheduler/tasks'

task "resque:setup" => :environment

namespace :resque do
  task :setup_schedule => :setup do
    require 'resque-scheduler'
    Resque.schedule = {
      UpdateViews: {
        every: '5s',
      }
    }
    # Resque.schedule = YAML.load_file('config/scheduler.yml')
  end

  # somehow scheduler task need to be separated from setup_scheduler
  task :scheduler => :setup_schedule
end
~~~

Note that scheduler can be defined as hash or yml file. If you are using yml you
need to put inside strings all values `5s`, cron...

~~~
# config/scheduler.yml
MyJob:
  # cron: '* * * * *'
  every: '5s'
  queue: 'critical'
~~~

You need to start both `rake resque:work` and `rake resque:scheduler`.
(usually you need to export exception recipients only in `work` proccess if you
need notification).

If you want to run that on same dyno you can use
[foreman](http://stackoverflow.com/questions/18176043/does-anyone-run-more-than-one-resque-worker-in-a-heroku-dyno)

~~~
// Procfile
web: bundle exec puma -C config/puma.rb
resque: bundle exec foreman start -f Procfile.workers
~~~

~~~
// Procfile.workers
worker_1: QUEUE=* bundle exec rake resque:work
worker_2: QUEUE=* bundle exec rake resque:scheduler
~~~
