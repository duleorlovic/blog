---
layout: post
---
https://github.com/duleorlovic/sidekiq_tips/#readme

# Sidekiq tips

Docs on wiki
https://github.com/mperham/sidekiq/wiki

We could use sidekiq as queuing backend for ActiveJob
https://github.com/mperham/sidekiq/wiki/Active-Job
but since it is slower and sidekiq extension is hard to use, I prefer to use
basic sidekiq (outside of ActiveJob).

To install
```
bundle add sidekiq
```
and run the main process
```
sidekiq
```

You do not need to restart the process when you make changes in jobs. Restart is
needed when you make change in sidekiq.yml configuration.
You can start both rails and sidekiq in one command using Procfile
```
# Procfile
web: bundle exec rails s -p 3000
worker: bundle exec sidekiq -C config/sidekiq.yml
```
and start with
```
foreman start
```

Let's create plain sidekiq job
```
rails g sidekiq:job a
vi app/sidekiq/a_job.rb
```

Sidekiq uses syntax https://github.com/mperham/sidekiq/wiki/Scheduled-Jobs
similar to ActiveJob `perform_later`
```
AJob.perform_later args
AJob.set(wait: 1.week).perform_later args
AJob.set(wait_until: Date.tomorrow.noon).perform_later args
AJob.perform_now args
```
we have

```
AJob.perform_async 'duke'
AJob.perform_in 3.hours, 'duke'
AJob.perform_at 3.hours.from_now, 'duke'

# to invoke perform_now use
AJob.new.perform 'duke'
```

With ActiveJob you can pass entire ActiveRecord objects because GlobalID will
deserialize for us. But if you are using directly sidekiq jobs (not inherited
from ActiveJob::Base) than you should pass object_id.

# Queues

Since the main process will run default queue, I prefer to name the queue based
on the app and use that name in configuration. If you run jobs from another app
you will get `NameError: uninitialized constant ...`
https://github.com/mperham/sidekiq/wiki/Advanced-Options
```
# config/sidekiq.yml
---
:concurrency: 2
:queues:
  - my_app_critical
  - my_app_default
```

In this example `my_app_default` jobs will be executed only if there is no 2
`my_app_critical` enqueued jobs. Also note if there are 2 `my_app_default`
running jobs `my_app_critical` will have to wait them to complete.

Sidekiq supports ordered (like previos example) and weighted modes but you can
not mix those to modes. If, for example, you need a critical queue that is
processed first, and would like other queues to be weighted, you would dedicate
a Sidekiq process exclusively to the critical queue, and other Sidekiq processes
to service the rest of the queues.

```
sidekiq -q critical # Only handles jobs on the "critical" queue
sidekiq -q default -q low -q critical # Handles critical jobs only after checking for other jobs
```

# Redis

You need to install redis server, which is simply adding Heroku redis addon.
https://elements.heroku.com/addons/heroku-redis Or on AWS you can install
https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04
Since it allows only 10 connections you need to limit connections for sidekiq.

Sidekiq server uses two connections, so if `:concurrency: 3` than server uses 5
connections so you can use puma threads 5 to utilize all 10 connections.
https://github.com/mperham/sidekiq/issues/117
https://manuelvanrijn.nl/sidekiq-heroku-redis-calc/

For error
```
A Redis::CommandError occurred in background at 2022-07-29 05:55:22 UTC :

  MISCONF Redis is configured to save RDB snapshots, but it's currently unable to persist to disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.
  /Users/dule/.rvm/gems/ruby-3.0.1/gems/redis-4.2.5/lib/redis/client.rb:132:in `call'
```
you need to restart redis
```
brew services restart redis
```

# Web UI

https://github.com/mperham/sidekiq/wiki/Monitoring
```
# config/routes.rb
require 'sidekiq/web'

  mount Sidekiq::Web => '/sidekiq'

# app/views/pages/index.html.erb
<%= link_to "Sidekiq", "/sidekiq" %>
```

Add back to app button
```
# config/initializers/sidekiq.rb
require 'sidekiq/web'
Sidekiq::Web.app_url = '/' # show "Back to App" button
```
https://github.com/mperham/sidekiq/wiki/Best-Practices#4-use-precise-terminology
Do not use `worker` term
Job is created when you enqueue a job instance.
Process is the main sidekiq command that you started.
To see current processes and number of threads http://localhost:3000/sidekiq/busy
Busy Thread is created when proccess start executing the job

Job lifecycle https://github.com/mperham/sidekiq/wiki/Job-Lifecycle
Final state is either Processed (only counter) or Dead, other states are
transitive states (temporary states)

We can use sidekiq_options
```
# app/sidekiq/a_job.rb
  # Default retry is 25, do not use if job is not idempotent
  sidekiq_options retry: 25
  # will be completely ephemeral, not in Retry or Dead, just increment counters
  sidekiq_options retry: false
  # will go immediately to the Dead tab upon first failure
  sidekiq_options retry: 0
```

# Scheduler

https://github.com/moove-it/sidekiq-scheduler

```
bundle add sidekiq-scheduler
```

Create sample scheduler
```
# app/sidekiq_worker/weekly_remainder_worker.rb
require 'sidekiq-scheduler'

class WeeklyRemainderWorker
  include Sidekiq::Worker
  sidekiq_options queue: 'my_app_default'

  def perform
    puts "WeeklyRemainderWorker finished"
  end
end
```
add to configuration

```
# config/sidekiq.yml

:schedule:
  weekly_remainder:
    cron: '0 * * * * *'   # https://crontab.guru/
    class: WeeklyRemainderWorker
```

add to routes

```
# config/routes.rb
require 'sidekiq-scheduler/web'
```

# Clear of find a job

To clear see https://gist.github.com/wbotelhos/fb865fba2b4f3518c8e533c7487d5354
To clear all jobs from a current queue named my_app_

```
# this will not remove scheduled jobs, only enqueued jobs
Sidekiq::Queue.new.clear # this will clear `default` queue
Sidekiq::Queue.new('my_app_default').clear

Sidekiq::ScheduledSet.new.clear

# Reset counters
Sidekiq::Stats.new.reset
```

To find specific existing jobs you can use scan or map.compact
```
queue = Sidekiq::Queue.new("default")
# https://github.com/mperham/sidekiq/wiki/API
jobs = queue.map do |job|
  if job.klass == '[JOB_CLASS]'
    {job_id: job.jid, job_klass: job.klass, arguments: job.args}
  end
end.compact
```

# Test background jobs spec

You can not use ActiveJob assertions since we do not use ActiveJob
https://api.rubyonrails.org/classes/ActiveJob/TestHelper.html#method-i-assert_enqueued_jobs
like `assert_enqueued_with job: MyJob, args: [user]` `perform_enqueued_jobs` `assert_performed_jobs 1`

But you can test using https://github.com/mperham/sidekiq/wiki/Testing
```
# test/test_helper.rb
require "sidekiq/testing"
```
and
```
# test/sidekiq_workers/weekly_remainder_worker_test.rb
    assert_difference "AJob.jobs.size", 1 do
      ScheduleScraperPageWorker.new.perform
    end
```

