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

~~~
cat >> ~/.railsrc << HERE_DOC
-T # skip minitests
-d postgresql # use postgresql
HERE_DOC

rails new myapp
cd myapp
git init . && git add . && git commit -m "rails new myapp"
~~~

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

~~~
rubocop --auto-correct # to correct some files if LineLength max is 115
sed -i Gemfile -e '/group :development do/a  \
  # to detect N+1 sql queries\
  gem "bullet"\
  # do not show assets in log\
  # gem "quiet_assets # can not find rails 5 version"\
  # irbtools includes interactive_editor gem (vim inside irb)\
  # just create ~/.irbrc with\
  # require "rubygems"\
  # require "irbtools"\
  gem "irbtools", require: "irbtools/binding"\
\
  # automatic reload\
  gem "guard-livereload", require: false\
  gem "rack-livereload"\
\
  # detects sql and gems vulnerability\
  gem "brakeman", require: false\
'

cat >> Gemfile << HERE_DOC
# adding vendor prefixes to css rules
gem "autoprefixer-rails"

# sets timezone based on browser timezone for each request
gem "browser-timezone-rails"
HERE_DOC

bundle
guard init livereload
sed -i config/environments/development.rb -e '/^end/i \
  # livereload\
  config.middleware.insert_after ActionDispatch::Static, Rack::LiveReload\
  # run with rails s -p b 0.0.0.0 to allow local network, allow remote requests\
  config.web_console.whiny_requests = false'

cat > config/initializers/bullet.rb << HERE_DOC
if defined? Bullet
  Bullet.enable = true
  Bullet.alert = true
  Bullet.rails_logger = true
end
HERE_DOC

git add . && git commit -m "Adding useful development & production gems"

# to run guard live reload, first increase max watches than just run guard
# https://github.com/guard/listen/wiki/Increasing-the-amount-of-inotify-watchers
# echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo
sysctl -p
# guard
~~~

## Pronto

You can create another github user bot (and invite him to collaborate on
your project) and use [pronto](https://github.com/mmozuras/pronto) to post
comments on commit, pull request or status

~~~
sed -i Gemfile -e '/group :development do/a  \
  gem "pronto", require: false\
  gem "pronto-rubocop", require: false\
  gem "pronto-brakeman", require: false\
  gem "pronto-eslint", require: false\
  gem "pronto-jshint", require: false\
  gem "pronto-poper", require: false\
  gem "pronto-rails_best_practices", require: false\
  gem "pronto-reek", require: false\
  gem "pronto-scss", require: false\
  gem "pronto-flay", require: false\
'
~~~

Generate [Personal Access
Token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)
for the bot user and export in your env file so pronto command can post comments

~~~
pronto run -f github
pronto run -f github_status
pronto run -f github_pr
~~~

You need to set up target commit (default is master) to which it needs to
compare current HEAD. It compare only changes that occurs between those two
(changes on master are ignored).

<https://christoph.luppri.ch/articles/2017/03/05/how-to-automatically-review-your-prs-for-style-violations-with-pronto-and-rubocop/?utm_source=rubyweekly&utm_medium=email>
to find last pull request id

# Sample page

~~~
rails g controller pages index --skip-helper --skip-assets --skip-controller-specs --skip-view-specs
echo '<button class="btn btn-primary">
  <i class="fa fa-camera-retro" aria-hidden="true"></i> fa-camera-retro</i>
</button>
' >> app/views/pages/index.html.erb
sed -i config/routes.rb -e '/draw/a \
  root "pages#index"'
git add . && git commit -m "Adding sample index page"
~~~

~~~
# initial scale on mobile devices
sed -i app/views/layouts/application.html.erb -e '/title/a \
  <meta name="viewport" content="width=device-width, initial-scale=1">'
~~~

# Front-end

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
    <%=raw "flash_notice('"'#{j notice}'"');" if notice %>\
    <%=raw "flash_notice('"'#{j alert}'"');" if alert %>\
  </script>\
\
  <article>\
    <%= yield %>\
  </article>'

cat > app/assets/javascripts/main.js.erb << 'HERE_DOC'
var FLASH_LETTER_STEP = 5;
var FLASH_DURATION = 5000;
function flash_appear($element, message, i) {
  if (i == undefined)
    i = 0;
  $element.text(message.substring(0,i));
  setTimeout(function(){ 
    if (i<message.length)
      flash_appear($element,message,i+1);
  }, FLASH_LETTER_STEP);
}
function flash_dissapear($element, message, i) {
  if (message == undefined)
    message = $element.text();
  if (i == undefined)
    i = message.length-1;
  $element.text(message.substring(0,i));
  setTimeout(function(){ 
    if (i>0)
      flash_dissapear($element,message,i-1);
  }, FLASH_LETTER_STEP);
}
function flash_alert(message) {
  flash_appear($('#alert'),message);
  setTimeout(function(){ flash_dissapear($('#alert')); }, FLASH_DURATION);
}
function flash_notice(message) {
  flash_appear($('#notice'),message);
  setTimeout(function(){ flash_dissapear($('#notice')); }, FLASH_DURATION);
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
{{ site.baseurl }} {% post_url 2017-01-10-get-syntax-right-in-jade-yaml %}#yaml))
Note that you can use default values for env variables but only for those
strings. Do not use boolean since `<%= ENV['MY_VAR'} || true %>` will always
resolve to true.

~~~
cat > config/secrets.yml << HERE_DOC
# export keys in your .profile file
development: &default
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] || 'some_secret' %>

  # sending emails
  smtp_username: <%= ENV["SMTP_USERNAME"] %>
  smtp_password: <%= ENV["SMTP_PASSWORD"] %>

  # for all outgoing emails
  default_mailer_sender: <%= ENV["DEFAULT_MAILER_SENDER"] || "My Company <support@example.com>" %>

  # default_url is required for links in email body or in links in controller
  # when url host is not available (for example rails console)
  default_url:
    host: <%= ENV["DEFAULT_URL_HOST"] || "example.com" %>
    port: <%= ENV["DEFAULT_URL_PORT"] || 80 %>

test: *default
production:
  <<: *default
  monitor_mode: true
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
  default from: Rails.application.secrets.default_mailer_sender'
~~~

Set default url option, that is domain for `root_url`:

~~~
#sed -i '/^  end$/i \\n \   config.action_mailer.default_url_options = { host: "localhost", port: 3000 }' config/environments/development.rb 
sed -i config/application.rb -e '/^  end$/i \
    # for link urls in emails\
    config.action_mailer.default_url_options = Rails.application.secrets.default_url.symbolize_keys\
    # for link urls in rails console\
    config.after_initialize do\
      Rails.application.routes.default_url_options = Rails.application.config.action_mailer.default_url_options\
    end'
~~~

For [devise]({{ site.baseurl }}{% post_url 2015-12-20-devise-oauth-angular %})
you need to set default sender:

~~~
# devise
sed -i config/initializers/devise.rb -e '/mailer_sender/c \
  config.mailer_sender = Rails.application.secrets.default_mailer_sender'
~~~

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


# Carrierwave for uploading

## Store on server

~~~
cat >> Gemfile << HERE_DOC
gem 'carrierwave'
HERE_DOC
bundle
rails generate uploader Document
# we need just one field type string to store file url
rails g migration add_document_to_companies document:string
rake db:migrate
sed -i app/models/company.rb -e '/class Company/a \
  mount_uploader :document, DocumentUploader'
git add . && git commit -m "Adding carrierwave gem document uploader"
~~~

Replace `f.text_field :document` with `f.file_field :document` in your form. In
view you can use `company.document.url`.

~~~
<%# app/views/companies/_form.html.erb %>
  <%  if @company.document.present?  %>
    <%= image_tag @company.document, class: 'image-small'%>
    <%= f.check_box :remove_document %>
  <% end %>
  <%= f.file_field :document %>

# app/controllers/companies_controller.rb
  def company_params
    params.require(:company).permit(:document, :remove_document)
  end
~~~

It is straightforward to use uploader in multiple fields. Also you can use
single table field for multiple files (field type json) but than you need
postgres database.

When you rendering json, than
[carrierwave will add nested
url](http://stackoverflow.com/questions/28184975/carrierwave-causing-json-output-to-become-nested-on-photo-key).
Solution is render json manually with `json.document_url company.document.url`
or to override uploader serilization with

~~~
# app/uploaders/document_uploader.rb
  def serializable_hash
    url
  end
~~~

Resizing is by adding mini magick and configure uploader. It works on Heroku
too. You can process files, 
create new varsions based on
[condtition](https://github.com/carrierwaveuploader/carrierwave#conditional-versions) or process based on [condition](http://stackoverflow.com/questions/11778464/conditional-versions-process-with-carrierwave)

~~~
echo "gem 'mini_magick'" >> Gemfile
bundle

# app/uploaders/document_uploader.rb
  include CarrierWave::MiniMagick
  process resize_to_limit: [200, 300]
  process resize_to_limit: [300, 300], if: :logo?
  def logo?(picture)
    # check if we mount_uploader :logo_url or something else
    picture.file.headers.match(/logo_url/)
  end
  version :thumb do
    process resize_to_fill: [200, 300]
  end
~~~

## Store on AWS S3

For [Amazon
S3](https://github.com/carrierwaveuploader/carrierwave#using-amazon-s3) you need
to set up your AWS keys in *~/.bashrc* `export AWS_ACCESS_KEY_ID=123123` and
`export AWS_SECRET_ACCESS_KEY=123123`. Bucket should be created as standard USA
bucket.

~~~
cat >> Gemfile << HERE_DOC
gem 'fog'
HERE_DOC
cat > config/initializers/carrierwave.rb << 'HERE_DOC'
# https://github.com/jnicklas/carrierwave#using-amazon-s3
CarrierWave.configure do |config|
  config.fog_credentials = {
    :provider               => 'AWS',
    :aws_access_key_id      => Rails.application.secrets.aws_access_key_id,
    :aws_secret_access_key  => Rails.application.secrets.aws_secret_access_key,
    :region                 => Rails.application.secrets.aws_region # us-east-1
  }
  config.fog_directory  = Rails.application.secrets.aws_bucket_name
end
HERE_DOC

sed -i config/secrets.yml -e '/^test:/i \
  # aws s3\
  aws_bucket_name: <%= ENV["AWS_BUCKET_NAME"] %>\
  aws_access_key_id: <%= ENV["AWS_ACCESS_KEY_ID"] %>\
  aws_secret_access_key: <%= ENV["AWS_SECRET_ACCESS_KEY"] %>\
  # region is important for all non us-east-1 regions\
  aws_region: <%= ENV["AWS_REGION"] || "us-east-1" %>\
'

sed -i app/uploaders/document_uploader.rb -e '/storage :file/r \
  # storage :file\
  storage :fog'

git add . && git commit -m "Configure AWS S3"
~~~

## Store directly on AWS S3 and upload the key to the server

You can put the `direct_upload_form_for` on any page, let's use show:

~~~
cat >> Gemfile << HERE_DOC
# direct upload to S3
gem 'carrierwave_direct'
HERE_DOC
bundle

sed -i app/uploaders/document_uploader.rb -e '/DocumentUploader/a \
  include CarrierWaveDirect::Uploader'

sed -i app/uploaders/document_uploader.rb -e '/store_dir/c \
  # we do not use store_dir because of dirrect carrierwave\
  def store_dir_origin'

cat >> app/views/companies/show.html.erb << 'HERE_DOC'
<%= direct_upload_form_for @uploader do |f| %>
  <%= f.file_field :document %>
  <%= f.submit %>
<% end %>
HERE_DOC

sed -i app/controllers/companies_controller.rb -e '/def show/a \
   # @uploader = @company.document # do not use old since key will remain\
   @uploader = DocumentUploader.new\
   # default key is /uploads/<unique_guid>/foo.png\
   # you can change, but use ONLY ONE folder ie "1/2/a.txt" -> "2/a.txt"\
   # it always adds prefix "uploads" so it does not need to be written\
   @uploader.key = "uploads/#{@company.id}-#{request.ip}/${filename}"\
   @uploader.success_action_redirect = company_url(@company)\
   if params[:key]\
     @company.document.key = params[:key]\
     @company.save!\
     # we need to reload since old key is there\
     @company = Company.find(@company.id)\
     # or to redirect\
     redirect_to company_path(@company)\
   end\
   # you can call @company.remove_document! to remove from aws, but please\
   # reload after that with @company = Company.find(@company.id)'

sed -i config/initializers/carrierwave.rb -e '/^end/i \
  # max_file_size is not originally on carrierwave, but is added on CWDirect\
  # if file is greater than allowed than error is from Amazon EntityTooLarge\
  config.max_file_size = 20.megabytes  # defaults to 5.megabytes'
~~~


# Production Heroku deploy

Since you need to run separate command for background jobs, you need to write
`Procfile`. By default heroku will run with WEBRICK, so it is advisable to use
puma.
You can follow [heroku article](https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server)

~~~
cat >> Gemfile <<HERE_DOC
gem 'puma'
HERE_DOC
bundle

cat >> Procfile <<HERE_DOC
web: bundle exec puma -C config/puma.rb
HERE_DOC

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
~~~

~~~
export MYAPP_NAME=air
# # postgresql on production
# sed -i Gemfile -e "/gem 'sqlite3/c \
# gem 'pg'"
# sed -i '/adapter: sqlite3/c \  adapter: postgresql' config/database.yml
# sed -i "/development.sqlite3/c \  database: ${MYAPP_NAME}_dev" config/database.yml
# sed -i "/test.sqlite3/c \  database: ${MYAPP_NAME}_test" config/database.yml
# sed -i "/production.sqlite3/c \  database: ${MYAPP_NAME}_prod" config/database.yml
#
# rails in production use: production: url: <%= ENV['DATABASE_URL'] %>

echo '
# heroku uses this 12 factor gem
gem "rails_12factor", group: :production
' >> Gemfile
bundle

git commit -am "Heroku uses pg and 12factor gem"
heroku apps:create $MYAPP_NAME
heroku addons:create heroku-postgresql:hobby-dev
git push heroku master --set-upstream
# if you receive an error An error occurred while installing Ruby ruby-2.2.4
# just try again
git push heroku master --set-upstream
heroku open
heroku run rake db:setup
~~~

On heroku add *Papertrail* add-on and go to the
[https://papertrailapp.com/events](https://papertrailapp.com/events) and search
for "Started GET" , save and create email alert or [add internal
notification]({{ site.baseurl }}{% post_url 2016-05-17-send-and-receive-emails-in-rails %})

For custom domain just add on settings your domain name and create CNAME record
for `www` with value `myapp.herokuapp.com`. If you need to match all subdomains,
you can put *Domain Name* `*.kontakt.in.rs` and CNAME record for `*` with same
value `myapp.herokuapp.com`.
