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

Rails provides in-process queuing (it keeps in memory and is running with rails)
so if you put byebug it will stop rails process.
For production you need some
backend queuing library to store that to db and later execute. That process
should be started separately from rails.

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
cat >> Gemfile << HERE_DOC
# background job using redis
gem 'resque'
HERE_DOC
bundle

sed -i config/application.rb -e '/^  end$/i \
    config.active_job.queue_adapter = :resque\
'

cat >> lib/tasks/resque.rake << HERE_DOC
require 'resque/tasks'

# we need to load our rails environment
# without this, we can call: QUEUE=* rake environemt resque:work
task 'resque:setup' => :environment
HERE_DOC
~~~

Two configurations:

~~~
if ENV["REDISTOGO_URL"].present?
  uri = URI.parse(ENV["REDISTOGO_URL"])
  REDIS = Redis.new(host: uri.host, port: uri.port, password: uri.password)
else
  REDIS = Redis.new(host: "localhost")
end

Resque.redis = REDIS
~~~

or

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
`heroku addons:docs redistogo`

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
but on on heroku free hoby type, you can run both web and worker.

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

## Delayed Job

Configuration is in `config/application.rb` to set
`config.active_job.queue_adapter = :delayed_job`. Note that it is not fully
compatible with activejob (does not use default mailer queue name, starts
immediatelly and in background with rake jobs:work)

Running is using gem `daemons` (so put it in your Gemfile) and then `rails g
delayed_job` will generate script file `bin/delayed_job` which you can use

* `RAILS_ENV=production bin/delayed_job start`
* `RAILS_ENV=production bin/delayed_job start --queues=webapp,jobs`
* `RAILS_ENV=production bin/delayed_job start --exit-on-complete` exit when
  there are not more jobs
* `RAILS_ENV=production bin/delayed_job run` is to run in foreground
* `RAILS_ENV=production bin/delayed_job restart`
* `RAILS_ENV=production bin/delayed_job status`
* rails ways is `rake jobs:work`, also `rake jobs:workoff` to exit after is done
with all jobs, `rake jobs:clear` to delete all jobs, `QUEUES=webapp,jobs rake
jobs:work`

Note that email letter opener does not work when you run with `rake jobs:work`,
but works when `bin/delayed_job run` (Launchy works in both cases, this
difference is only for mailer).

Workers will check database every 5 seconds.

Jobs are objects with a method called `perform`. Also you can override
`Delayed::Worker.max_attempts` with your method `max_attempts` (you can find
[defaults](https://github.com/collectiveidea/delayed_job#gory-details) for other
methods like: `max_run_time`, `detroy_failed_jobs?`)

To see that is it actually working you need to enable log:

~~~
# config/initializers/delayed_job_config.rb
Delayed::Worker.logger = Logger.new(File.join(Rails.root, 'log', 'delayed_job.log'))
~~~

Also you can see all jobs in console:

~~~
Delayed::Job.all
~~~

Usage is simply with inserting `delay` method and it will run in background. So
instead `@user.activate(params)` call with `@user.delay.activate(params)`.
Another way is to define `handle_asynchronously :activate` on User class. It can
take these params:

* `priority: 10` lower numbers run first
* `run_at: 5.minutes.from_now` or `handle_asynchronously :activate, run_at:
Proc.new { 5.minutes.from_now }`
* `queue: 'important'` or `handle_asynchronously :ativate, queue: 'important'`
than you can assign priority for each queue:

  ~~~
  Delayed::Worker.queue_attributes = {
    important: { priority: -10 },
    low_priority: { priority: 10 }
  }
  ~~~

For mailer we need to remove `.deliver` method and use `.delay` in prefix, like
`MyMailer.delay(run_at: 5.minutes.from_now).welcome(user)`

Another way to run background jobs is with `Delayed::Job.enqueue CleanJob.new,
queue: 'import'`. Note that for ActiveJob instances it runs immediatelly
and also in background task. So advice is not to mix those two...

[best practices](https://www.sitepoint.com/delayed-jobs-best-practices/)

Sample worker

`User.find(1).with_lock do sleep(10); puts "worker 1 done" end`
`User.find(1).with_lock do sleep(1); puts "worker 2 done" end`
