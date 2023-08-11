---
layout: post
title: Rails cache
---

[Guide](http://guides.rubyonrails.org/caching_with_rails.html) tells that page
and action caching has been extracted to gems and currently we use fragment
caching only.

By default caching in rails development environment is disabled. To enable you
need to `touch tmp/caching-dev.txt` AND to restart the server. You can see in
config that you can also toggle with `rails dev:cache`
```
# config/environments/development.rb
  # Enable/disable caching. By default caching is disabled.
  # Run rails dev:cache to toggle caching.
  if Rails.root.join("tmp/caching-dev.txt").exist?
    config.action_controller.perform_caching = true
    config.action_controller.enable_fragment_cache_logging = true

    config.cache_store = :memory_store
    config.public_file_server.headers = {
      "Cache-Control" => "public, max-age=#{2.days.to_i}"
    }
  else
    config.action_controller.perform_caching = false

    config.cache_store = :null_store
  end
```
So by default is not enabled in development so in rails console we see
```
Rails.cache
#    @name="ActiveSupport::Cache::Strategy::LocalCache">

Rails.cache.class
 => ActiveSupport::Cache::NullStore
```
If we run `rails dev:cache` than it uses memory (persists untill you exit the
console)
```
Rails.cache
 => #<ActiveSupport::Cache::MemoryStore entries=0, size=0, options={:compress=>false}>

Rails.cache.class
 => ActiveSupport::Cache::MemoryStore

# before it was `ActiveSupport::Cache::FileStore`
```
On production we should switch to use `dalli+memcached` or `redis`  (you already
have redis when you use background jobs like Sidekiq).

Filestore will create files on `tmp/cache/:rand/:rand/cache_key` so you can find
them `ls -R tmp/cache/`. If cache_key is array, it is joined with `/`.
On heroku `heroku run bash` I do not see cache files `ls tmp/cache` probably
because heroku is read only system (only buildpack can add stuff) so it is
better to use memcached.

# Redis cache store

In Rails 5.2 you can use

~~~
# Gemfile
gem 'hiredis'
gem 'redis', require: ['redis', 'redis/connection/hiredis']
~~~

~~~
# config/application/environments/production.rb
# config/application/environments/development.rb
config.cache_store = :redis_cache_store
~~~

In console you can check keys and values

~~~
pp Rails.cache.redis.keys
~~~

# Install memcached

For ubuntu you need to install memcached service.
Note that memcached will **automatically** remove old cache files.

~~~
sudo apt-get install memcached
sudo service memcached status
~~~

For heroku you need addon
[memcachier](https://devcenter.heroku.com/articles/memcachier)

~~~
heroku addons:create memcachier:dev
~~~

This will create env variables which you can use in secrets

~~~
# sed -i config/secrets.yml -e '/^test:/i
  # caching server
  memcachier_servers: <%= ENV["MEMCACHIER_SERVERS"] %>
  memcachier_username: <%= ENV["MEMCACHIER_USERNAME"] %>
  memcachier_password: <%= ENV["MEMCACHIER_PASSWORD"] %>
'
~~~

Use [gem dalli](https://github.com/petergoldstein/dalli) in your Gemfile

~~~
cat >> Gemfile << HERE_DOC
# https://guides.rubyonrails.org/caching_with_rails.html#activesupport-cache-memcachestore
# cache store is in mem_cache_store which uses dalli
gem 'dalli'
HERE_DOC
~~~

~~~
cat >> config/initializers/dalli.rb << HERE_DOC
Rails.application.configure do
  # https://devcenter.heroku.com/articles/building-a-rails-3-application-with-memcache
  # https://devcenter.heroku.com/articles/memcachier
  config.cache_store = :mem_cache_store # this will use local memcached service
  if secrets.memcachier_servers.present?
    config.cache_store = :mem_cache_store,
      secrets.memcachier_servers.split(','),
      {
        username: secrets.memcachier_username,
        password: secrets.memcachier_password,
        failover: true,
        socket_timeout: 1.5,
        socket_failure_delay: 0.2,
        down_retry_delay: 60
      }
  end
end
HERE_DOC
~~~

Try with `heroku run rails c`
```
Rails.application.config.cache_store
```

## Install memcached using rubber

Read about [capistrano and rubber]({{ site.baseurl }}
{% post_url 2016-09-07-cap3-capistrano-deployment-rubber-provision %}) how to
set up, so you can add memcached with

~~~
# this will copy deploy-memcached.rb and rubber-memcached.yml and role files
bundle exec rubber vulcanize memcached
~~~

Add role `memcached` to Vagrantfile or your hosts.
Since vagrant machine do not have
[image_type](https://github.com/rubber/rubber/blob/master/templates/memcached/config/rubber/role/memcached/memcached.conf#L4)
which is used to determine max_mem, you need to define it

~~~
# config/rubber/rubber-memcached.yml
memcached_max_mem: 2048
memcached_max_mem_hash:
  ...
~~~

Also we need to enable listening on all required ports.

To use in rails you need to add

~~~
cat >> config/initializers/dalli.rb << HERE_DOC
Rails.application.configure do
  dalli_config = Rails.root.join('config','dalli.rb')
  if File.exists?(dalli_config)
    require dalli_config; include ::Rubber::Dalli::Config
  else
    config.cache_store = :dalli_store # this will use local memcached service
  end
end
HERE_DOC
~~~


## Install in plain irb

~~~
require "rubygems"
require "dalli"
CACHE = Dalli::Client.new("127.0.0.1", { :namespace => 'my_project', :expires_in => 3600, :socket_timeout => 3, :compress => true }) # Options are self-explanatory
CACHE.set("key", "value", 180) # last option is expiry time in seconds
CACHE.get("key")
~~~

# Manual check if it is working

You can check from bash if memcached service is running

~~~
netstat -ap   | grep 11211
# tcp        0      0 localhost:11211         *:*                     LISTEN      17897/memcached
# udp        0      0 localhost:11211         *:*                                 17897/memcached

# if listening on all ip addresses is enabled than you should see
tcp        0      0 *:11211                 *:*                     LISTEN      16065/memcached
udp        0      0 *:11211                 *:*                                 16065/memcached
~~~

You should be able to connect

~~~
telnet localhost 11211
stats

# another command is
/usr/share/memcached/scripts/memcached-tool localhost:11211 stats
~~~

If you use vagrant, than `config/rubber/role/memcached/dalli.rb` will use
`rubber_instance.full_name` which is `default.foo.com` but I'm not able to
connect using that domain name. So you need to change
`config/rubber/role/memcached/memcached.conf` to add below comment `# -l
127.0.0.1` add this option

~~~
# we want to listen on all IP addresses
-l 0.0.0.0
~~~

You can check in console if properly enabled

~~~
Rails.cache.class
~~~

And if check its stats

~~~
Rails.cache.stats
# {"127.0.0.1:11211"=>{"pid"=>"14592", "uptime"=>"8319", ...

# if service is not working you can expect something like
[DEBUG] Dalli::Server#connect mylocationsubdomain.my-domain.vagrant:11211
[WARNING] mylocationsubdomain.my-domain.vagrant:11211 failed (count: 0) Errno::ECONNREFUSED: Connection refused - connect(2) for "mylocationsubdomain.my-domain.vagrant" port 11211
~~~

We can ask ActiveSupport for that also

~~~
ActiveSupport::Cache.lookup_store(:mem_cache_store).stats
~~~

If cache is using filestore exception will be raised

~~~
Rails.cache.stats
NoMethodError: undefined method `stats' for #<ActiveSupport::Cache::FileStore:0x000000058a45d0>
~~~

You can see [more info about
stats](http://www.pal-blog.de/entwicklung/perl/memcached-statistics-stats-command.html)


# Fragment caching

Main usage is like this

~~~
<% cache item do %>
  bla
<% end %>
~~~

Key will contain template path, item id, item updated_at and template digest so
if you change anything, cache will be expired.

For index pages I use key to be string, that is maximum updated at (also
template digest will invalidate cache).

~~~
<% cache ['todos', @tasks.maximum(:updated_at)] do %>
  My todos list
<% end %>
~~~

Note that if you use translations on different subdomain than put default locale
in the key also.

```
<% cache ['moves-markers', I18n.locale, latest_move_timestamp] do %>
```

Note that you should not use same keys on multiple places on the page.
So if you have to use on same page, use different first string.

Note that cache key should include all items that happens to be inside `if`
statement, for example we show different content for different users. Also if we
should numbers that depends on current day, for example `Todays registered: <%=
User.where('created_at > ?', Time.zone.at_begging_of_day)` we should include
Date in cache keys since it should be updated (event we do not touch any user)

```
<% cache ['tickets', can?(:view, company), company.tickets.maxium(:updated_at)] do %>
  <% if can? :view, company %>
    Hi admin, here is the secret link
  <% else %>
    <%= company.tickets.all.size %>
  <% end %>
<% end %>
```

To use simple keys you can
```
<% cache "asd", skip_digest: true do %>
<% end %>
```
To find cache key you can use
```
old = Rails.cache.redis.keys
# make a request
new = Rails.cache.redis.keys

new - old
["views/asd"]
```

You can clear cache in rails console : `Rails.cache.clear`, also
`rake tmp:cache:clear` for rails clear cache

You can use conditional caching you can use `cache_if`

~~~
<% cache_if params[:cache], product do %>
  <%= render product %>
<% end %>
~~~

If you show some data from nested resources, for example product comment, you
need to add relation so any update will also trigger parent update, and
invalidate parent cache.

~~~
# app/models/comment.rb
  belongs_to :product, touch: true
~~~

Inside json jbuilder you need to call `json.cache!`

~~~
# app/views/users/index.json.jbuilder
json.cache! ['users', @users.maxium(:updated_at)] do
  ...
end
~~~

# Low level caching

In console or in code you can cache put, get

~~~
Rails.cache.write 'my_key', 123
Rails.cache.read 'my_key'
Rails.cache.fetch 'my_key' do
  123
end
~~~

`fetch` will read from cache if exists or write to cache
For any ActiveRecord instance, you have method `cache_key` which you can use
for example in `fetch` block.

~~~
class Product < ApplicationRecord
  def competing_price
    Rails.cache.fetch("#{cache_key}/competing_price", expires_in: 12.hours) do
      Competitor::API.find_price(id)
    end
  end
end
~~~

`cache key` uses `id` and `updated_at`

~~~
Task.new.cache_key
# => "tasks/new"
Task.last.cache_key
# => "tasks/2-20170511114115000000"
~~~

Note that [sql
caching](http://guides.rubyonrails.org/caching_with_rails.html#sql-caching) is
only inside one action. Usually rails do not perform sql query untill it is
needed. If you only read `current_user` in before actions, that query will be
cached. But if you update current_user, than following `current_user` is not
reading from cache (for example if you use current_user in view or in other
before actions).

You can use this [snippets
](http://vinsol.com/blog/2014/02/11/guide-to-caching-in-rails-using-memcache/)

# Test

Enable caching in test with
```
# config/environments/test.rb
  if Rails.root.join("tmp/caching-test.txt").exist?
    config.action_controller.perform_caching = true
    config.action_controller.enable_fragment_cache_logging = true

    config.cache_store = :memory_store
    config.public_file_server.headers = {
      "Cache-Control" => "public, max-age=#{2.days.to_i}"
    }
  else
    config.action_controller.perform_caching = false

    config.cache_store = :null_store
  end
```
you can use helpers
```
# test/a/rails_cache_helper.rb
# Add `include RailsCacheHelper` to test/test_helper.rb
#
# Credits https://kevinjalbert.com/testing-the-use-of-rails-caching/
module RailsCacheHelper
  # Enable cache with config/environments/test.rb and
  #   touch tmp/caching-test.txt
  def with_clean_caching
    Rails.cache.clear
    yield
  ensure
    Rails.cache.clear
  end

  def cache_has_value?(value)
    cache_data.values.map(&:value).any?(value)
  end

  def key_for_cached_value(value)
    cache_data.each_value do |key, entry|
      return key if entry&.value == value
    end
  end

  private

  def cache_data
    Rails.cache.instance_variable_get(:@data)
  end
end
```

and test for example
```
```
# Theory

[mnot cache_docs](https://www.mnot.net/cache_docs/)
