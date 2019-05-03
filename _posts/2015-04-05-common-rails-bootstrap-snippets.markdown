---
layout: post
title:  Common Rails bootstrap snippets
tags: ruby-on-rails devise carrierwave
---

Paste this code to create some basic starting application *myapp.com* with
authentication and other funny tools. I extensively use [echo sed grep](
{{ site.baseurl }} {% post_url 2016-05-18-echo-sed-grep-command-line-editing %})
commands here.

> give a man a fish and you feed him for a day. teach a man to fish and you feed
him for a lifetime


# RVM and other tools

If you do not have `rails` or `git` you need to install

~~~
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
\curl -sSL https://get.rvm.io | bash
rvm install 2.3.0
gem install bundle

sudo apt install git postgresql libpq-dev nodejs
~~~

# Initial commit

You can choose database with rails param `rails new myapp --database=postgresql`
but it is better to create `.railsrc` so it is used for all rails

```
cat >> ~/.railsrc << HERE_DOC
-d postgresql # use postgresql
HERE_DOC
```

Create new app

```
rails new myapp
# to specify version you can use
rails _5.2.1_ new myapp
cd myapp
rails db:create
git init . && git add . && git commit -m "rails new myapp"
```

For error
```
Webpacker can't find application in /home/orlovic/Downloads/rails_6_beta2_stimulus/public/packs/manifest.json. Possible causes:
```
you need to install webpack and run initial compilation
```
yarn add @rails/webpacker
bin/webpack
```

# UUID

https://github.com/trkin/contact_form/commit/52532f3378604f59fb16c1e6432cc567542c391d

~~~
bin/rails g migration enable_extension_for_uuid
last_migration
# add following line to migration
  def change
    enable_extension 'pgcrypto' unless extension_enabled?('pgcrypto')
  end

cat >> config/initializers/generators.rb << HERE_DOC
Rails.application.config.generators do |g|
  g.orm :active_record, primary_key_type: :uuid
end
HERE_DOC

rake db:create db:migrate
git add .
git commit -m 'Enable uuid in postgresql'
~~~

Later, if you add some references, you need to specify `type: :uuid` like:

~~~
  t.references :post, type: :uuid, index: true
~~~

or error will be raised:

~~~
DETAIL:  Key columns "post_id" and "id" are of incompatible types: bigint and uuid.
~~~

Note that ordering by id, uuid is not possible. It is hard to implement on
existing projects and on MySQL db.

# Sample page

~~~
rails g controller pages home --skip-helper --skip-assets
sed -i config/routes.rb -e "/^end$/i \\
  # root page\n\
  root 'pages#home'\
"
~~~
Rubocop cli
~~~
rubocop --auto-correct # to autocorrent correct some files (except lineLength)
~~~

~~~
# initial scale on mobile devices
sed -i app/views/layouts/application.html.erb -e '/title/a \
    <meta name="viewport" content="width=device-width, initial-scale=1">'
~~~

# Font icons

Fontello provides a icons which you can include in your project using
https://github.com/railslove/fontello_rails_converter
```
# Gemfile
# pick icons
gem 'fontello_rails_converter'

# select some fonts http://fontello.com/ and download zip to `tmp/fontello.zip`
bundle exec fontello convert --no-download
gnome-open http://localhost:3001/fontello-demo.html

# when you want to update you can
fontello open
# select new icons
bundle exec fontello convert
```
To configure in rails you need to import
```
# app/assets/stylesheets/application.sass
// vendor
@import 'fontello'
```
And use with classes
```
# app/layouts/application.html.erb
<i class="demo-icon icon-mobile"></i>
```

Prepare icons https://github.com/fontello/fontello/wiki/How-to-use-custom-images#preparing-images-in-inkscape


# Gitignore

~~~
cat >> .gitignore << 'HERE_DOC'
# vim temp files
*.swp
*.swo
# carrierwave upload files
/public/uploads
# gedit files
*~
# vagrant files
.vagrant
# byebug
.byebug_history
# rspec temporary file
spec/examples.txt
HERE_DOC
git commit -am "Update .gitignore"
~~~

# Gemfile development & production tools

Some of the gems could be found on [thoughtbot
suspenders](https://github.com/thoughtbot/suspenders)
Paid 3th party service alternative for `bullet` is scoutapp, for
`exception_notification` is sentry, informantapp.

~~~
cat >> Gemfile << HERE_DOC
group :development do
  # pretty print Ruby objects in full color
  gem 'awesome_print', require: 'ap'
  # static analysis security vulnerability scanner, run: `brakeman`
  gem 'brakeman', '~> 3.5.0', require: false
  # detect N+1 sql queries
  gem 'bullet'
  # detect out-of-date or vulnerable gems, run: `gemsurance`
  gem 'gemsurance'
  # automatic reload
  gem 'guard-livereload', require: false
  # open emails in browser
  gem 'letter_opener'
  # detect file changes
  gem 'listen', '~> 3.0.5'
  # support for Rails Panel - chrome extension
  gem 'meta_request'
  # debugger
  gem 'pry'
  gem 'pry-rails'
  # ruby static code analyzer
  gem 'rubocop', require: false
  # spring speeds up development by keeping your application running in the
  # background. Read more: https://github.com/rails/spring
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'
HERE_DOC
~~~

Some not used gems
~~~
  # do not show assets in log\
  # gem "quiet_assets # can not find rails 5 version"\

  # irbtools includes interactive_editor gem (vim inside irb)\
  # just create ~/.irbrc with\
  # require "rubygems"\
  # require "irbtools"\
  gem "irbtools", require: "irbtools/binding"\

  # is not needed since rails will show line on which there is
  # exception (no need to insert console and use on web page).
  gem better_errors

  # Access an interactive console on exception pages or by calling 'console' anywhere in the code.
  gem 'web-console', '>= 3.3.0'

  # config/application.rb
  # run with rails s -p b 0.0.0.0 to allow local network, allow remote requests\
  config.web_console.whiny_requests = false'
~~~

Production gems and configurations

~~~
cat >> Gemfile << HERE_DOC
# adding vendor prefixes to css rules
gem "autoprefixer-rails"

# sets timezone based on browser timezone for each request
gem "browser-timezone-rails"
HERE_DOC

# need some js for timezone
cat >> app/assets/javascripts/application.js << HERE_DOC
//= require js.cookie
//= require jstz
//= require browser_timezone_rails/set_time_zone
HERE_DOC

bundle
guard init livereload
# gem https://github.com/guard/guard-livereload suggest some old extension
# https://addons.mozilla.org/en-US/firefox/addon/livereload/ but there is new
# https://addons.mozilla.org/en-US/firefox/addon/livereload-web-extension
# or you can use rack-livereload
# I receive error No such middleware to insert before: ActionDispatch::Static
# so better is to use browser plugin instead of gem 'rack-livereload'
# sed -i config/environments/development.rb -e '/^end/i \
#   # livereload\
#   # use rack-livereload or browser extension\
#   # if guard is not running, there is an error in js console:\
#   # Cross-origin plugin content from  must have a visible size larger than 400 x\
#   # 300 pixels, or it will be blocked. Invisible content is always blocked.\
#   config.middleware.insert_after ActionDispatch::Static, Rack::LiveReload\
# '

When I get error with command RAILS_ENV=production rails assets:precompile
 No such middleware to insert before: ActionDispatch::Static
Than solution is to add `gem 'rails_12factor`, group: :production`
https://github.com/AssetSync/asset_sync/issues/221#issuecomment-75492905


cat > config/initializers/bullet.rb << HERE_DOC
if defined? Bullet
  Bullet.enable = true
  Bullet.alert = true
  Bullet.rails_logger = true
end
HERE_DOC

git add . && git commit -m "Adding guard and other useful gems"

# to run guard live reload, first increase max watches than just run guard
# https://github.com/guard/listen/wiki/Increasing-the-amount-of-inotify-watchers
# echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo
sysctl -p
guard -d
# livereload debug with `guard -d`. Error if you also run it on another project
# there should not be eventmachine.rb:530:in `start_tcp_server': no acceptor (port is in use or requires root privileges) (RuntimeError)
~~~

Customize log output of rails logger in production with
https://github.com/roidrage/lograge#handle-actioncontrollerroutingerror

## Fonts

Copy definition from node_modules css files where font-face is defined, and
change from `url` to `asset-url`

```
npm install typeface-roboto
mkdir app/assets/stylesheets/plugins
cat >> app/assets/stylesheets/plugins << HERE_DOC
@font-face {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 400;
  src: local('Roboto Regular'), local('Roboto-Regular'), asset-url('typeface-roboto/files/roboto-latin-400.woff') format('woff'), asset-url('typeface-roboto/files/roboto-latin-400.woff2') format('woff2');
}
@font-face {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 500;
  src: local('Roboto Medium'), local('Roboto-Medium'), asset-url('typeface-roboto/files/roboto-latin-500.woff') format('woff'), asset-url('typeface-roboto/files/roboto-latin-500.woff2') format('woff2');
}
@font-face {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 600;
  src: local('Roboto SemiBold'), local('Roboto-SemiBold'), asset-url('typeface-roboto/files/roboto-latin-500.woff') format('woff'), asset-url('typeface-roboto/files/roboto-latin-500.woff2') format('woff2');
}
@font-face {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 700;
  src: local('Roboto Bold'), local('Roboto-Bold'), asset-url('typeface-roboto/files/roboto-latin-700.woff') format('woff'), asset-url('typeface-roboto/files/roboto-latin-700.woff2') format('woff2');
}
HERE_DOC

cat >> app/assets/stylesheets/application.sass << HERE_DOC
// plugins
@import 'plugins/roboto_font_face'
HERE_DOC

cat >> app/assets/stylesheets/common/body.sass << HERE_DOC
body
  font-family: 'Roboto', sans-serif
HERE_DOC
```

## Twitter bootstrap Gem

## Adding flash (both from server and client)

You can use [text-center](http://getbootstrap.com/css/#type-alignment) bootstrap
helper (ie `text-align: center;`). Also the collors

~~~
sed -i app/views/layouts/application.html.erb -e '/yield/c \
  <div class="text-center">\
    <span class="notice text-success" id="notice"></span>\
    <span class="alert text-danger" id="alert"></span>\
  </div>\
  <script>\
    <%=raw "flash_message(document.getElementById('notice'), '#{j notice}');" if notice %>\
    <%=raw "flash_message(document.getElementById('alert'), '#{j alert}');" if alert %>\
  </script>\
\
  <article>\
    <%= yield %>\
  </article>'

cat > app/assets/javascripts/main.js.erb << 'HERE_DOC'
var FLASH_LETTER_STEP = 10;
var FLASH_DURATION = 5000;
function flash_appear(element, message, i) {
  if (i == undefined)
    i = 0;
  element.innerText = message.substring(0,i);
   setTimeout(function(){ 
    if (i<message.length)
      flash_appear(element, message, i+1);
  }, FLASH_LETTER_STEP);
}
function flash_dissapear(element, message, i) {
  if (message == undefined)
    message = element.innerText;
  if (i == undefined)
    i = message.length-1;
  element.innerText = message.substring(0,i);
  setTimeout(function(){ 
    if (i>0)
      flash_dissapear(element, message,i-1);
  }, FLASH_LETTER_STEP);
}

function flash_message(element, message) {
  flash_appear(element, message);
  setTimeout(function(){ flash_dissapear(element); }, FLASH_DURATION);
}
HERE_DOC
git add . && git commit -m "Adding flash to layout"
~~~

## Bourbon css mixins

~~~
echo '
# css mixin library http://bourbon.io/
gem "bourbon", '~> 5.0.0.beta' # bourbon 5 is requred by bitters 1.3
# https://github.com/thoughtbot/bitters/issues/235
gem "neat"
' >> Gemfile
echo '
@import "bourbon";
@import "base/base"; // this is http://bitters.bourbon.io/
@import "neat";
' > app/assets/stylesheets/application.scss
git rm app/assets/stylesheets/application.css

bundle
gem install bitters
cd app/assets/stylesheets && bitters install
sed -i '/grid-settings/c @import "grid-settings";' base/_base.scss
cd -

git add . && git commit -m "Adding bourbon, neat and bitters scss"
~~~

## Slim

~~~
sed -i Gemfile -e '/group :development do/a  \
  # slim templating\
  gem "slim-rails"'

gem install html2slim
erb2slim app/views/leads/index.html.erb
~~~

## Angular

For various ways of integrating Angular look at
[angular-and-ruby-on-rails]({{ site.baseurl }}
{% post_url 2015-11-26-angular-and-ruby-on-rails %})

# Simplify secrets and add smtp credentials

YML file can use anchor (`&`) and reference (`*`) so you do not repeat the code.
When you use reference `*` (as for testing) you can not add or update keys, but
with `<<` you can (as for production) (more on [get syntax right](
{{ site.baseurl }} {% post_url 2017-01-10-get-syntax-right-in-jade-yaml-haml %}#yaml))
Note that you can use default values for env variables but only for those
strings. Do not use boolean since `<%= ENV['MY_VAR'} || true %>` will always
resolve to true.
Instead of this
~~~
development: &default
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] || 'some_secret' %>

test: *default
production:
  <<: *default
  monitor_mode: true
~~~

In rails 5 there are `shared` key which will use all stuff.

~~~
cat > config/secrets.yml << HERE_DOC
shared:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] || 'some_secret' %>

  # sending emails
  smtp_username: <%= ENV["SMTP_USERNAME"] %>
  smtp_password: <%= ENV["SMTP_PASSWORD"] %>

  # for all outgoing emails
  mailer_sender: <%= ENV["MAILER_SENDER"] || "My Company <support@example.com>" %>

  # default_url is required for links in email body or in links in controller
  # when url host is not available (for example rails console)
  default_url:
    host: <%= ENV["DEFAULT_URL_HOST"] || (Rails.env.production? ? "premesti.se" : "www.myapp.localhost") %>
    port: <%= ENV["DEFAULT_URL_PORT"] || (Rails.env.development? ? Rack::Server.new.options[:Port] : nil) %>

# `shared` is automatically included, no need to write
# shared: &default
# development: *default
# or
# development:
#   <<: *default
HERE_DOC

git add . && git commit -m "Simplify secrets"
~~~

# Basic mail settings

We can generate application mailer with: 

~~~
rails generate mailer UserMailer hello
git add . && git commit -am "rails generate mailer UserMailer hello"
~~~

Change default from address:

~~~
sed -i app/mailers/application_mailer.rb -e '/default/c \
  default from: Rails.application.secrets.mailer_sender'
~~~

Set default url option, that is domain for `root_url`:

~~~
# if you need to get from rails 4 use { port: Rails::Server.new.options[:Port] }
# in config/environments/development.rb and also include
# require 'rails/commands/server` on the top of the file

# for rails 5 you can use Rack::Server.new.options[:Port] and no need to
# require anything, but it works only if you provide -p param to rails s

# config/application.rb
    link = Rails.application.secrets.default_url.symbolize_keys
    # for link urls in emails
    config.action_mailer.default_url_options = link
    # for link urls in rails console
    config.after_initialize do
      Rails.application.routes.default_url_options = link
    end
    # for asset-url or img_tag in emails
    config.action_mailer.asset_host = "http://#{link[:host]}:#{link[:port]}"
~~~

For [devise]({{ site.baseurl }}{% post_url 2015-12-20-devise-oauth-angular %})

For local development use Letter opener:

~~~
sed -i '/group :development do/a  \
  # open emails in browser\
  gem "letter_opener"' Gemfile
sed -i '/^end$/i \  config.action_mailer.delivery_method = :letter_opener' config/environments/development.rb 
bundle && git add . && git commit -m "Letter opener to see emails in browser"
rails runner UserMailer.hello.deliver
~~~

For production see
[send and receive emails in rails]({{ site.baseurl }}{% post_url 2016-05-17-send-and-receive-emails-in-rails %})

# Skip generators

~~~
sed -i '/Rails::Application/a \
    config.i18n.enforce_available_locales = true\
    config.generators do |generate|\
      generate.helper false\
      generate.javascript_engine false\
      generate.request_specs false\
      generate.routing_specs false\
      generate.stylesheets false\
      generate.test_framework :rspec\
      generate.view_specs false\
    end\
    config.action_controller.action_on_unpermitted_parameters = :raise\
' config/application.rb
git add . && git commit -m "Skip generators"
~~~


# Authentication

## Cleareance gem

[Clearance](https://github.com/thoughtbot/clearance) is simple email authorization. It could work with [facebook auth](https://gist.github.com/stevebourne/2394427) but it's designed to be only email auth.

~~~
echo "gem 'clearance'" >> Gemfile && bundle
rails generate clearance:install
rake db:migrate
rails generate clearance:routes
git add . && git commit -m "rails g clearance:install"

sed -i '/<body>/a \\n\
  <nav>\
    <% if signed_in? %>\
      <%= current_user.email %>\
      <%= button_to "Sign out", sign_out_path, method: :delete %>\
    <% else %>\
      <%= link_to "Sign in", sign_in_path %>\
    <% end %>\
  </nav>\
' app/views/layouts/application.html.erb
git add . && git commit -m "Adding sign signout path"
~~~


# Company scaffold with skipped unused files
~~~
rails g scaffold company name:string user:references --no-stylesheets --no-fixture --no-test-framework --no-helper --no-assets --no-jbuilder
sed -i '/companies/a \  root "companies#index"' config/routes.rb
rake db:migrate && git add . && git commit -m "rails g scaffold company name:string user:references"
~~~

# Puma

Puma is now default webserver on rails. But by default it runs in single mode
WEB_CONCURRENCY=1 so on heroku it will allow RAILS_MAX_THREADS connections if GIL is
not trigered.
You can simulate slow connection with `sleep 10`, it does not triggel GIL.

~~~
export RAILS_MAX_THREADS=1
curl $u/action_with_sleep_10 &
curl $u/action_with_sleep_10 &
# real 10s
# real 20s


export RAILS_MAX_THREADS=2
curl $u/action_with_sleep_10 &
curl $u/action_with_sleep_10 &
# real 10s
# real 10s
~~~


You can follow [heroku article](https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server)

~~~
cat >> Gemfile <<HERE_DOC
gem 'puma'
HERE_DOC
bundle

# Rails 5 puma is default, and config/puma.rb is used, so do not need Procfile
# but is it advisable to write it and put rakek db:migrate so you do not need to
# run migration after you deploy. Restart is not needed as it was needed after
# heroku run rake db:migrate (there are no exception, but update was not
# actually saved in new columns, so restart was needed)
cat >> Procfile <<HERE_DOC
web: bundle exec puma -C config/puma.rb
release: rake db:migrate
HERE_DOC

# default puma config is fine, but on production should include those lines
cat >> config/puma.rb <<HERE_DOC
workers Integer(ENV['WEB_CONCURRENCY'] || 2)
threads_count = Integer(ENV['RAILS_MAX_THREADS'] || 5)
threads threads_count, threads_count

preload_app!

rackup      DefaultRackup
port        ENV['PORT']     || 3000
environment ENV['RACK_ENV'] || 'development'

on_worker_boot do
  # Worker specific setup for Rails 4.1+
  # See: https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#on-worker-boot
  ActiveRecord::Base.establish_connection
end
HERE_DOC

# on development always use single mode
mkdir config/puma
cat >> config/puma/development.rb <<HERE_DOC
workers 0
HERE_DOC
~~~

# Heroku deploy

If you need to run separate command for background jobs, you need to write
`Procfile`.

~~~
export MYAPP_NAME=my-app # only dash, not underscore
# # postgresql on production
# sed -i Gemfile -e "/gem 'sqlite3/c \
# gem 'pg'"
# sed -i '/adapter: sqlite3/c \  adapter: postgresql' config/database.yml
# sed -i "/development.sqlite3/c \  database: ${MYAPP_NAME}_dev" config/database.yml
# sed -i "/test.sqlite3/c \  database: ${MYAPP_NAME}_test" config/database.yml
# sed -i "/production.sqlite3/c \  database: ${MYAPP_NAME}_prod" config/database.yml
#
# rails in production use: production: url: <%= ENV['DATABASE_URL'] %>

cat >> Gemfile << HERE_DOC
# heroku uses this 12 factor gem
gem 'rails_12factor', group: :production
HERE_DOC
bundle
git commit -am "Heroku uses 12factor gem"

heroku apps:create $MYAPP_NAME
heroku addons:create heroku-postgresql:hobby-dev # this will set DATABASE_URL
git push heroku master --set-upstream

heroku run rake db:migrate db:seed
# heroku run rake db:setup will raise error:
# PG::ConnectionBad: FATAL:  permission denied for database "postgres"
# DETAIL:  User does not have CONNECT privilege
# https://kb.heroku.com/why-am-i-seeing-user-does-not-have-connect-privilege-error-with-heroku-postgres-on-review-apps try
# heroku pg:reset --confirm $MYAPP_NAME
# heroku restart
#
# I do not know how to drop db since db:migrate:reset does not work either

# sometimes you need to recompile assets when you change secrets but assets are
# not changed, and you need to purge cache, install plugin
# https://github.com/heroku/heroku-repo
# heroku plugins:install heroku-repo
# heroku repo:purge_cache

heroku open
~~~

Heroku use ubuntu 16 or Ubuntu 18, you can change

~~~
heroku apps:info
heroku stack
heroku stack:set heroku-16
~~~

You can pull from heroku databse https://devcenter.heroku.com/articles/heroku-postgresql#pg-push-and-pg-pull
You need to find the name of heroku database by clicking on Postgresl plugin

~~~
rails db:drop
heroku pg:pull postgresql-name-on-heroku my_rails_app_development

# also pushing
heroku pg:reset
heroku pg:push buyers_development postgresql-animate-86842
~~~

If you need to provision new database (upgrade from free to hobby) than you can
use pg copy
https://devcenter.heroku.com/articles/upgrading-heroku-postgres-databases#upgrading-with-pg-copy
~~~
heroku pg:copy DATABASE_URL HEROKU_POSTGRESQL_TEAL_URL
heroku pg:promote HEROKU_POSTGRESQL_TEAL_URL
heroku config # find color of old database
heroku addons:destroy HEROKU_POSTGRESQL_NAVY_URL
~~~

If you need to compile node packages than you need need custom
[buildpack](https://devcenter.heroku.com/articles/nodejs-support#customizing-the-build-process).
We could use 3th party gulp buildpacks but default
[node](https://docs.npmjs.com/misc/config)
[buildpack](https://github.com/heroku/heroku-buildpack-nodejs) works fine.

There is also for mysql <https://github.com/Shopify/heroku-buildpack-mysql> or
<https://github.com/din-co/heroku-buildpack-mysql>


`devDependencies` need to be renamed to `dependecies` since it runs node
production mode (I tried to disable production mode using NPM_CONFIG_ONLY but
than other thinks does not work). Imporant is
[NODE_MODULES_CACHE](https://devcenter.heroku.com/articles/nodejs-support#cache-behavior).
If we (by default) cache `/node_modules` it will not build new version to public
folder.

~~~
# deploy to heroku
heroku buildpacks:set https://github.com/heroku/heroku-buildpack-ruby
heroku buildpacks:add --index 1 https://github.com/heroku/heroku-buildpack-nodejs

heroku buildpacks # should return  1. nodejs  2. ruby (latest wins :)
# alternativelly, we can define then in file .buildpacks
# echo 'https://github.com/heroku/heroku-buildpack-ruby
# https://github.com/heroku/heroku-buildpack-nodejs ' > .buildpacks
# heroku config:add BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git
~~~

~~~
git rm -rf public
echo '/public
/node_modules' >> .gitignore
git add .gitignore
git commit -m "Remove public folder from git repo"

echo '{
  "name": "rootApp",
  "scripts": {
  },
  "dependencies": {
    "myappAngular": "file:./client"
  },
  "cacheDirectories": [
    "client/node_modules",
    "client/bower_components"
  ]
}
' > package.json
git add package.json && git commit -m "Add package.json for nodejs buildpack detect"

cd client
sed -i "/dist: 'dist/c \  dist: '../../public'," gulp/conf.js
sed -i '/"scripts":/a \    "postinstall": "bower install && gulp build",' package.json
sed -i '/"devDependencies":/c \  "dependencies": {\
    "bower": "*",' package.json
git add . && git commit -m "Configure postinstall gulp build"
cd ..

git push heroku
heroku config # NPM_CONFIG_PRODUCTION=true NODE_ENV=production NODE_MODULES_CACHE=true
heroku open
# usefull command to test is `npm install`
# git commit --amend --allow-empty --no-edit && echo "output is saved: cat log/heroku.log" && git push heroku -f > log/heroku.log 2>&1 && heroku run bash -c 'mysql -v'
~~~

On heroku add *Papertrail* add-on and go to the
<https://papertrailapp.com/events> and search
for "Started GET" , save and create email alert or [add internal
notification]({{ site.baseurl }}{% post_url 2016-05-17-send-and-receive-emails-in-rails %})

<https://devcenter.heroku.com/articles/custom-domains>
For custom domain just add on settings your domain name and create CNAME record
for `www` with value `myapp.herokuapp.com`.
If you need to match all subdomains (wildcard), you can put *Domain Name*
`*.kontakt.in.rs` and CNAME record for `*` with same value
`myapp.herokuapp.com`.
If you need to add root domain (naked, bare, zone apex) than you can't use
loopia but some other suggested domain name providers.

## Deploy javascript npm required tasks

## Heroku dyno puma settings

<https://youtu.be/itbExaPqNAE>
Some Things in system = arrival rate X time spent in system so for server it is:
hequests_in_system = requests per second X average_response_time (115 req/s *
0.147s = 16 request in system, so we need at leat 16 workers in same time).
utilization = requests_in_system / how_many_workers
for example = 115 req/s X 147ms response / 45 workers = 37%

Use 3 WEB_CONCURRENCY workers and 3-5 RAILS_MAX_THREADS (not more since each thread
need connection to database, and use some on memory).
https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#workers suggests 2-4 workers
`sleep` also do not lock GIL (so all threads are working).
> On MRI, there is a Global Interpreter Lock (GIL) that ensures only one thread
> can be run at any time. IO operations such as database calls, interacting with
> the file system, or making external http calls will not lock the GIL. Most
> Rails applications heavily use IO, so adding additional threads will allow
> Puma to process multiple threads, gaining you more throughput. `

https://devcenter.heroku.com/articles/scaling#autoscaling

Also increase database, redis and memcached connections.
Heroku postgresql hobby-basic has limit of 20 connections (enought for 3x5=15)

# Google app engine

It is not hard to deploy to [google ap engine
](https://cloud.google.com/ruby/rails/appengine)
