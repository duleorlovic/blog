---
layout: post
title:  Common Rails bootstrap snippets
categories: ruby-on-rails devise carrierwave
noToc: false
---

Paste this code to create some basic starting application *myapp.com* with 
authentication and other funny tools.

> give a man a fish and you feed him for a day. teach a man to fish and you feed
him for a lifetime

Hint: `echo -e "\n" >> filename` will add new line (that's why `-e` to the
filename). You can use single quotes so you do not need to write `-e` and `\n`.

`sed -i '/haus/a home' filename` will inplace (`-i`) search for *haus* and 
append *home* after that line (beside insert before `i`, this could be `a`
append and `c` change matched line). Multiple lines need to have `\` at the end of line (multiline *echo ' ...'* does not need that trailing backslash).

Sed has something different regular expressions so follow this [link](http://www.gnu.org/software/sed/manual/html_node/Regular-Expressions.html)


# Initial commit

~~~
rails new myapp
cd myapp
git init . && git add . && git commit -m "rails new myapp"
~~~

# Gitignore 

~~~
echo -e '# vim temp files
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
' >> .gitignore
git commit -am "Update .gitignore"
~~~

# Gemfile development & production tools

~~~
echo '
group :development do
  # to detect N+1 sql queries
  gem "bullet"
  # do not show assets in log
  gem "quiet_assets"
  # irbtools includes interactive_editor gem (vim inside irb)
  # just create ~/.irbrc with 
  # require 'rubygems'
  # require 'irbtools'
  gem 'irbtools' 
end

# adding vendor prefixes to css rules
gem "autoprefixer-rails"

# sets timezone based on browser timezone for each request
# gem "browser-timezone-rails"
'>> Gemfile

bundle
git commit -am "Adding useful development & production gems"
~~~

# Front-end

## Adding flash

~~~
sed -i '/yield/c \
  <% flash.each do |key, value| -%>\
    <div class="flash-<%= key %>"><%= value %></div>\
  <% end %>\
\
  <article>\
    <%= yield %>\
  </article>\
' app/views/layouts/application.html.erb
git add . && git commit -m "Adding flash to layout"
~~~

## Bourbon css mixins

~~~
echo '
# css mixin library http://bourbon.io/
gem "bourbon"
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

## Twitter bootstrap

~~~
echo '
# twitter boostrap sass https://github.com/twbs/bootstrap-sass
gem "bootstrap-sass" #, :git => "https://github.com/twbs/bootstrap-sass.git", :branch => "next"
' >> Gemfile
bundle
echo '
// "bootstrap-sprockets" must be imported before "bootstrap" and "bootstrap/variables"
@import "bootstrap-sprockets";
@import "bootstrap";
' > app/assets/stylesheets/application.scss
git rm app/assets/stylesheets/application.css
sed -i '/jquery_ujs/a \
//= require bootstrap-sprockets' app/assets/javascripts/application.js
git commit -am "Adding boostrap"
~~~

## Angular

~~~
echo '
# js package manager
gem "bower-rails"
# cache all templates
gem "angular-rails-templates"
' >> Gemfile
bundle
rails g bower_rails:initialize
git add . && git commit -m "rails g bower_rails:initialize"

echo '
Rails.application.config.assets.paths << Rails.root.join("vendor","assets","bower_components")
Rails.application.config.assets.paths << Rails.root.join("vendor","assets","bower_components","bootstrap-sass-official","assets","fonts")
Rails.application.config.assets.precompile << %r(.*.(?:eot|svg|ttf|woff|woff2)$)
' >> config/initializers/bower_rails.rb

echo '
asset "angular"
asset "bootstrap-sass-official"
# vim: ft-ruby
' >> Bowerfile

rake bower:install

git rm app/assets/stylesheets/application.css
echo '
@import "bootstrap-sass-official/assets/stylesheets/bootstrap-sprockets";
@import "bootstrap-sass-official/assets/stylesheets/bootstrap";
' >> app/assets/stylesheets/application.scss

sed -i '/jquery_ujs/a \
//= require angular/angular' app/assets/javascripts/application.js

git add . && git commit -am "rake bower:install angular & boostrap"
~~~

# Simplify secrets

~~~
echo -e '# export keys in your .profile file
development: &default
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] || "`rake secret`" %>
  # sending emails
  mandrill_api_key: <%= ENV["MANDRILL_API_KEY"] %>
  mail_interceptor_email: <%= ENV["MAIL_INTERCEPTOR_EMAIL"] %>
  default_mailer_sender: <%= ENV["DEFAULT_MAILER_SENDER"] || "support@example.com" %>

  default_url:
    host: <%= ENV["DEFAULT_URL_HOST"] || "example.com" %>
    port: <%= ENV["DEFAULT_URL_PORT"] || 80 %>

test: *default
production: *default
' > config/secrets.yml

git add . && git commit -m "Simplify secrets"
~~~

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

# Sample page

~~~
rails g controller pages index # --skip-helper --skip-assets --skip-controller-specs --skip-view-specs
sed -i "/root 'welcome#index'/c \  root 'pages#index'" config/routes.rb
git add . && git commit -m "Adding sample index page"
~~~

# Sending Email

## Common mail settings

~~~
#sed -i '/end$/i \\n  config.action_mailer.default_url_options = { host: "localhost", port: 3000 }' config/environments/development.rb 
sed -i '/end$/i \\n  config.action_mailer.default_url_options = Rails.application.secrets.default_url.symbolize_keys' config/environments/production.rb 
~~~

## Letter opener for development 
~~~
echo -e 'gem "letter_opener", :group => :development' >> Gemfile
sed -i '/end$/i \  config.action_mailer.delivery_method = :letter_opener' config/environments/development.rb 
bundle && git add . && git commit -m "Letter opener to see emails in browser"
~~~

## Mandrill for production and local interceptor

We need two environment variables: `MANDRILL_API_KEY` and `MAIL_INTERCEPTOR_EMAIL` which you should set up
 in your *~/.bashrc* file with this `export MANDRILL_API_KEY=123123` and `export MAIL_INTERCEPTOR_EMAIL=you@youremail.com`

~~~
echo "gem 'mandrill_dm'" >> Gemfile && bundle
sed -i '/end$/i \\n  config.action_mailer.delivery_method = :mandrill' config/applications.rb

echo '# mandrill initializer
require "yaml" # this is needed for Rails secrets
MandrillDm.configure do |config|
  config.api_key = Rails.application.secrets.mandrill_api_key
end
class DevelopmentMailInterceptor
  def self.delivering_email(message)
    message.subject = "#{message.to} #{message.subject}"
    message.to = Rails.application.secrets.mail_interceptor_email
  end
end
if Rails.env.development?
  ActionMailer::Base.register_interceptor(DevelopmentMailInterceptor)
end
' > config/initializers/mandrill.rb

#sed -i '/\(development:\|test:\|production:\)/a \  mandrill_api_key: <%= ENV["MANDRILL_API_KEY"] %>\n  mail_interceptop_email: <%= ENV["MAIL_INTERCEPTOR_EMAIL"] %>' config/secrets.yml
git add . && git commit -am "Adding mandrill for sending emails"
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

## Devise gem

See blog post

# Company scaffold with skipped unused files
~~~
rails g scaffold company name:string user:references --no-stylesheets --no-fixture --no-test-framework --no-helper --no-assets --no-jbuilder
sed -i '/companies/a \  root "companies#index"' config/routes.rb
rake db:migrate && git add . && git commit -m "rails g scaffold company name:string user:references"
~~~


# Carrierwave for uploading

You should set up your AWS keys in *~/.bashrc* `export AWS_ACCESS_KEY_ID=123123` and `export AWS_SECRET_ACCESS_KEY=123123`. Bucket should be created as standard USA bucket. You can use single table field for multiple files (field type json) but than you need postgres database, we will keep it simple.

~~~
echo -e "gem 'carrierwave'\\ngem 'fog'" >> Gemfile && bundle
rails generate uploader Document
rails g migration add_document_to_companies document:string
sed -i '/class Company/a \  mount_uploader :document, DocumentUploader'  app/models/company.rb
rake db:migrate
git add . && git commit -m "Adding carrierwave gem, document uploader and mount on company"


~~~

# Heroku deploy

~~~
export MYAPP_NAME=myapp
# postgresql on production
sed -i "/gem 'sqlite3/c \
gem 'pg'\
" Gemfile
echo '
# heroku uses this 12 factor gem
gem "rails_12factor", group: :production
' >> Gemfile
bundle

sed -i '/adapter: sqlite3/c \  adapter: postgresql' config/database.yml
sed -i "/development.sqlite3/c \  database: ${MYAPP_NAME}_dev" config/database.yml
sed -i "/test.sqlite3/c \  database: ${MYAPP_NAME}_test" config/database.yml
sed -i "/production.sqlite3/c \  database: ${MYAPP_NAME}_prod" config/database.yml

git commit -am "Heroku uses pg and 12factor gem"
heroku apps:create $MYAPP_NAME
git push heroku master --set-upstream
~~~

On heroku add *Papertrail* add-on and go to the [https://papertrailapp.com/events](https://papertrailapp.com/events) and search for "Started GET" , save and create email alert.

