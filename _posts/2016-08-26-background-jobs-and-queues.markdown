---
layout: post
title: Background jobs and queues
---

# ActiveJob

[Using ActiveJob](http://guides.rubyonrails.org/active_job_basics.html) can
simplify writing jobs so you can change queuing backend.
`rails g job my_todo --queue critical` it will generate job for which we will
just log 'Hi'.

~~~
# app/jobs/my_todo_job.rb
class MyTodoJob < ActiveJob::Base
  queue_as :critical

  def perform(*args)
    # puts also works fine, but this is more rails way
    Rails.logger.info "Hi"
    Rails.logger.flush
  end
end
~~~

You can use that with `wait` and `wailt_until`

~~~
MyTodoJob.perform_later args
MyTodoJob.set(wait: 1.week).perform_later args
MyTodoJob.set(wait_until: Date.tomorrow.noon).perform_later args
~~~

~~~~
# config/application.rb
    config.active_job.queue_adapter = :sidekiq
~~~

Sidekiq is faster but requires thread safe code.
With ActiveJob you can pass entire ActiveRecord objects because GlobalID will
deserialize for us. But if you are using directly some jobs (not inherited from
ActiveJob::Base) than you should pass object_id.

You need to install redis server

# Resque

Adding resque is simple, just add to Gemfile, create simple config and rake
task.

Since you need to run separate command for background jobs, you need to write
`Procfile`. By default heroku will run with WEBRICK, so it is advisable to use
puma.

Always use `QUEUE` when calling `QUEUE=* rake reque:work`

* `QUEUES=comma,separated,list`
* `QUEUE=my_queue`
* `QUEUE=*` or `QUEUES=*`

~~~
// Procfile
web: bundle exec puma -C config/puma.rb
worker: env TERM_CHILD=1 QUEUE=* bundle exec rake resque:work
~~~

~~~
# config/redis.yml
development: &default localhost:6379
test: *default
staging: *default
production: <%= REDISTOGO_URL %>
~~~

~~~
# config/initializers/resque.rb
config = YAML.load_file(Rails.root.join('config', 'redis.yml'))
# configure redis connection
Resque.redis = config[Rails.env]
~~~

~~~
# config/application.rb
    config.active_job.queue_adapter = :resque
~~~

~~~
# lib/tasks/resque.rake
require "resque/tasks"

task "resque:setup" => :environment
~~~

You can start both web and worker with `foreman start`
`heroku addons:create redistogo` is needed to enable redis on heroku.

# Recurring

You can create heroku scheduler (cron task) and call (for example every hour)
your custom rake task in which you can add new jobs.
Disadvantage of this approach it memory. Every time it starts new Rails
environment (that could be 450MB) and several could be at the same time.
Also I do not like to write the code outside of git, so I prefer to use plugins.

They all use [rufus-scheduler](https://github.com/jmettraux/rufus-scheduler).

resque-scheduler works in that way to check every 5 seconds if some of the tasks
should be processes. If it find them, they are pushed to jobs queue.

~~~
# lib/tasks/resque.rake
# Resque tasks
require 'resque/tasks'
require 'resque/scheduler/tasks'

namespace :resque do
  task :setup do
    require 'resque'
    Resque.redis = 'localhost:6379'
  end

  task :setup_schedule => :setup do
    require 'resque-scheduler'
    Resque.schedule = {
      UpdateViews: {
        every: '5s',
      }
    }
    # Resque.schedule = YAML.load_file('config/scheduler.yml')
  end

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


