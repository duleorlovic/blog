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

Invoke jobs with attributes `wait` and `wait_until`

~~~
MyTodoJob.perform_later args
MyTodoJob.set(wait: 1.week).perform_later args
MyTodoJob.set(wait_until: Date.tomorrow.noon).perform_later args

# or
MyTodoJob.perform_now args
~~~

With ActiveJob you can pass entire ActiveRecord objects because GlobalID will
deserialize for us. But if you are using directly some jobs (not inherited from
ActiveJob::Base) than you should pass object_id.

Error `Unsupported argument type: Move` when using
`UserMailer.signup(move).deliver_later`. You need to pass id instead of object
when delivering later.

Rails provides in-process queuing (it keeps in memory and is running with rails)
so if you put byebug it will stop rails process.
By default `config.active_job.queue_adapter = :inline` better is to use
`:async`.  Both inline and async Active Job do not support priority, timeout and
retry. You can find all adapter http://api.rubyonrails.org/v5.1/classes/ActiveJob/QueueAdapters.html

# Sidekiq

Add `gem 'sidekiq'` to Gemfile.

~~~
# config/application.rb
    config.active_job.queue_adapter = :sidekiq
    config.active_job.queue_name_prefix = 'mysite'
~~~

Sidekiq is faster but requires thread safe code.

You need to install redis server, which is simply adding Heroku redis addon.
Since it allows only 20 connections you need to limit connections for sidekiq.
It requires usually concurrency + 5 connections to Redis.
https://github.com/mperham/sidekiq/issues/117

~~~
# Procfile
web: bundle exec puma -C config/puma.rb
worker: bundle exec sidekiq -C config/sidekiq.yml
~~~

You can run `bundle exec sidekiq -q default -q mailers` but better is in config:
~~~
# config/sidekiq.yml
:concurrency:  3
:queues:
  - default
  - mailers

  - my_site_default
  - my_site_mailers
~~~

Concurrency is a number which are used by sidekiq server to create redis
connections (=concurrency + 5 per one process).
Sidekiq client defaults to 5 connections per process but this can be limited to
1 per process (it does not use `concurrency`)
So for max 10 redis connections you can use:

~~~
# three unicorns = 3 connections
Sidekiq.configure_client do |config|
  config.redis = { :size => 1 }
end
# so one sidekiq can have 7 connections
Sidekiq.configure_server do |config|
  # only 2 connections are for workers
  config.redis = { :size => 7 }
end
~~~

You can limit connection pool to 3 so all threads will use only those
connections. Concurrency is less than (server pool size - 5)*2. At least
`concurrency` connections to database is also required.
Puma threads share client connections per process. so if size is 3, all threads
will use only those 3 connections.
Dyno count is important, so if you have 4 dynos of one sidekiq and 1 dyno of
another sidekig queue, that is 5 * (concurrency + 5) connections.

To see how many jobs is in default queue:

~~~
Sidekiq::Queue.new.size # default name is 'default'
Sidekiq::Queue.new('mailers').size
Sidekiq::Queue.new('my_app_mailers').size
~~~

To see dashboard add two lines to routes

~~~
require 'sidekiq/web'
Rails.application.routes.draw do
  mount Sidekiq::Web => '/sidekiq'
~~~

Testing sidekiq https://github.com/mperham/sidekiq/wiki/Testing

You can use block

~~~
require 'sidekiq/testing'

Sidekiq::Testing.inline! do
  HardWorker.perform_async
  assert_worked_hard
end
~~~

For emails https://github.com/mperham/sidekiq/issues/724

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

For web use

~~~
# Gemfile
gem 'resque-web', require: 'resque_web'
~~~

# Define jobs

You can use Rails `ActiveJob` (so you can use `perform_later`) or any class
module that responds to `perform` (but than you need to use
`Resque.enqueue(SimpleJob,i)`, or `Delayed::Job.enqueue SimpleJob.new` or
`SimpleJob.delay.perform` to enqueue)

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

Define jobs using Struct (or ApplicationJob or ActiveJob::Base, but that
is not needed). Put params in initialization.

~~~
FetchFeedJob = Struct.new(:feed_id) do
  def perform
    puts "called with #{feed_id}"
  end
end
# invoke with
Delayed::Job.enqueue FetchFeedJob.new(feed.id)
# you can still use rails version
rake jobs:work
# no need to restart for code changes and byebug
~~~

or you can pass params to `perform` and use rails `perform_later` to invoke

~~~
# app/jobs/send_sms_job.rb
class SendSmsJob < ActiveJob::Base
  queue_as :webapp

  rescue_from Net::ReadTimeout, SocketError do |e|
    e.ignore_please = true
    sleep 1
    # re-raise so job is retried
    raise e
  end
  def perform(location_sms_alert)
  end
end

# somewhere in the code
SendSmsJob.perform_later location_sms_alert
~~~

Install

~~~
# Gemfile
gem 'delayed_job_active_record'
rails generate delayed_job:active_record
rake db:migrate
# this will create bin/delayed_job
~~~

Configuration is in `config/application.rb` to set
`config.active_job.queue_adapter = :delayed_job`. Note that it is not fully
compatible with activejob (does not use default mailer queue name, starts
immediatelly and in background with rake jobs:work)

Running is using gem `daemons` (so put it in your Gemfile) and then `rails g
delayed_job` will generate script file `bin/delayed_job` which you can use

* `RAILS_ENV=production bin/delayed_job start` to run all queues
* `RAILS_ENV=production bin/delayed_job start --queues=webapp,jobs` set specific
queue (QUEUE=webapp env does not work for delayed job)
* `RAILS_ENV=production bin/delayed_job start --exit-on-complete` exit when
  there are not more jobs
* `RAILS_ENV=production bin/delayed_job run` is to run in foreground (sometimes
byebug does not work, so I use `rake jobs:work` when debugging)
* `RAILS_ENV=production bin/delayed_job restart`
* `RAILS_ENV=production bin/delayed_job status`
* `cat tmp/pids/delayed_job.pid` can give you process id

The Rails way is:
* `rake jobs:work`
* `rake jobs:workoff` to exit after is done with all jobs
* `rake jobs:clear` to delete all jobs
* `QUEUES=webapp,jobs rake jobs:work` to set specific queue (by default it runs
all)

Note that email letter opener does not work when you run with `rake jobs:work`,
but works when `bin/delayed_job run` (Launchy works in both cases, this
difference is only for mailer).

Workers will check database every 5 seconds.
Note that you do not need to refresh the runner when you update the code. Kill
and restart is only required if you update some of the initializers files.

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
job = Delayed::Job.find 10 # 10 is job.id

job.handler # to see how job will be called
job.last_error # to see backtrace of error
job.failed_at # time when last failed

# to rerun, re run, invoke, retry job you can
job.invoke_job # this does not remove job if successfully, so you need to do
job.destroy # that manually
# or
Delayed::Worker.new.run job # this runs in current process (not in
# bin/delayed_job run) and will remove if successfully
# or
job.update_attributes attempts: 0, run_at: Time.now, failed_at: nil
# locked_by: nil, locked_at: nil
# job.failed_at = nil; job.save!
~~~

To invoke jobs you can do that in three ways (all three ways support queue name)

~~~
# call for object method like user.activate
user.delay(queue: 'tracking').activate
# call in rake tasks
Delayed::Job.enqueue MyJob.new(user_id), queue: 'tracking'
# define it on User class
handle_asynchronously :activate, queue: 'tracking'
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

For mailer instead of `.deliver` method we can also use `.delay` in prefix, like
`MyMailer.delay(run_at: 5.minutes.from_now).welcome(user)`. Or use rails 5
`deliver_now` or `deliver_later`.
Mailer `queue` is by default `mailers`. For other jobs `queue` is `nil`.
In Rails 5 you can rename to `config.action_mailer.deliver_later_queue_name =
'default_mailer_queue'`.


Another way to run background jobs is with `Delayed::Job.enqueue CleanJob.new,
queue: 'import'`. Note that for ActiveJob instances it runs immediatelly
and also in background task. So advice is not to mix those two...

[best practices](https://www.sitepoint.com/delayed-jobs-best-practices/)

Sample worker

`User.find(1).with_lock do sleep(10); puts "worker 1 done" end`
`User.find(1).with_lock do sleep(1); puts "worker 2 done" end`

You can add web based monitoring https://github.com/ejschmitt/delayed_job_web

~~~
# Gemfile
gem 'delayed_job_web'

# config/routes.rb
  match "/delayed_job" => DelayedJobWeb, :anchor => false, :via => [:get, :post]

# config/initializers/delayed_job_web.rb
if Rails.env.production?
  DelayedJobWeb.use Rack::Auth::Basic do |username, password|
    ActiveSupport::SecurityUtils.variable_size_secure_compare(Rails.application.secrets.delayed_job_username, username) &&
      ActiveSupport::SecurityUtils.variable_size_secure_compare(Rails.application.secrets.delayed_job_password, password)
  end
end

# config/secrets.yml
  delayed_job_username: delayed_job
  delayed_job_password: delayed_job
~~~

When you want to cancel a delayed job you can find and destroy it... but better
is to create a job that is immune ie check at the begginning...

# Test background jobs spec

`ActiveJob::Base.queue_adapter = :test` will change queue adapter for all
following test.
You can see differences between queue adapters
<http://api.rubyonrails.org/v5.1.4/classes/ActiveJob/QueueAdapters.html>
There is test helpers like `assert_performed_with` http://api.rubyonrails.org/classes/ActiveJob/TestHelper.html#method-i-assert_performed_with
example of use is
<https://github.com/eliotsykes/rspec-rails-examples/blob/master/spec/jobs/headline_scraper_job_spec.rb>
For mailers Rails uses
[ActionMailer::DeliveryJob](https://blog.bigbinary.com/2018/01/15/rails-5-2-allows-mailers-to-use-custom-active-job-class.html)
so to test in minitest

~~~
require 'test_helper'

class ProductTest < ActiveJob::TestCase
  test 'billing job scheduling' do
    # if you want to test outcome ie how job is performed
    # perform_enqueued_jobs only: ActionMailer::DeliveryJob do
    # or you can assert how it is called `_with`
    # assert_enqueued_with job: ActionMailer::DeliveryJob, args: 1 do
    # or assert and perform
    assert_performed_jobs 2, only: ActionMailer::DeliveryJob do
      product.charge(account)
    end
  end
end
~~~

IF you use Delayed::Job you can test in three ways:
First is `Delayed::Worker.delay_jobs = true`

~~~
expect do
  post url, params
end.to change { Delayed::Job.count }.by(1)

expect do
  Delayed::Worker.new.work_off
end.to change(Delayed::Job, :count).by(-1)
~~~

Second is `Delayed::Worker.delay_jobs = false` so job is performed inline ie
invoked immediatelly.


# TIPS

* Always check if job is eligible to run , world changes, it could be already
run bu some error.
* Test if job is added and if it added only once.
