---
layout: post
title: Rails cache
---

[Guide](http://guides.rubyonrails.org/caching_with_rails.html) tells that page
and action caching has been extracted to gems. Now we can use fragment caching

You can enable with

~~~
  # config/environments/development.rb
  config.action_controller.perform_caching = true
~~~

# Install memcached

For ubuntu you need to install memcached service.
Memcached will automatically remove old cache files

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
sed -i config/secrets.yml -e '/^test:/i \
  # caching server\
  memcachier_servers: <%= ENV["MEMCACHIER_SERVERS"] %>\
  memcachier_username: <%= ENV["MEMCACHIER_USERNAME"] %>\
  memcachier_password: <%= ENV["MEMCACHIER_PASSWORD"] %>\
'
~~~

Use [gem dalli](https://github.com/petergoldstein/dalli)

~~~
cat >> config/initializers/dalli.rb << HERE_DOC
Rails.application.configure do
  config.cache_store = :dalli_store # this will use local memcached service
  if secrets.memcachier_servers.present?
    config.cache_store = :dalli_store,
                          secrets.memcachier_servers.split(","),
                         {
                           username: secrets.memcachier_username,
                           password: secrets.memcachier_password,
                           failover: true,
                           socket_timeout: 1.5,
                           socket_failure_delay: 0.2,
                           down_retry_delay: 60,
                         }
  end
end
HERE_DOC
~~~

You can test is console

~~~
Rails.cache.stats
# {"127.0.0.1:11211"=>{"pid"=>"14592", "uptime"=>"8319", ...
~~~

# Fragment caching

~~~
<% cache item do %>
  bla
<% end %>
~~~

Key will contain template path, item id, item updated_at and template digest so
if you change anything, cache will be expired.

If you show some data from nested resources, for example product comment, you
need to add relation so any update will also trigger parent update.

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

# Theory

[mnot cache_docs](https://www.mnot.net/cache_docs/)
