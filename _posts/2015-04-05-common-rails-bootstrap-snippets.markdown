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


# Initial commit

~~~
rails new myapp --database=postgresql
cd myapp
git init . && git add . && git commit -m "rails new myapp --database=postgresql"
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
  gem "quiet_assets"\
  # irbtools includes interactive_editor gem (vim inside irb)\
  # just create ~/.irbrc with\
  # require "rubygems"\
  # require "irbtools"\
  gem "irbtools", require: "irbtools/binding"\
\
  # automatic reload\
  gem "guard-livereload", "~> 2.5", require: false\
  gem "rack-livereload"'

cat >> Gemfile << HERE_DOC
# adding vendor prefixes to css rules
gem "autoprefixer-rails"

# sets timezone based on browser timezone for each request
# gem "browser-timezone-rails"
HERE_DOC

bundle
guard init livereload
sed -i config/environments/development.rb -e '/^end/i \
  # livereload\
  config.middleware.insert_after ActionDispatch::Static, Rack::LiveReload\
  # run with rails s -p b 0.0.0.0 to allow local network, allow remote requests\
  config.web_console.whiny_requests = false'

git add . && git commit -m "Adding useful development & production gems"
~~~

~~~
# initial scale on mobile devices
sed -i app/views/layouts/application.html.erb -e '/title/a \
  <meta name="viewport" content="width=device-width, initial-scale=1">'

~~~

# Sample page

~~~
rails g controller pages index --skip-helper --skip-assets --skip-controller-specs --skip-view-specs
echo '<button class="btn btn-primary">
  <i class="fa fa-camera-retro" aria-hidden="true"></i> fa-camera-retro</i>
</button>
' >> app/views/pages/index.html.erb
sed -i "/root 'welcome#index'/c \  root 'pages#index'" config/routes.rb
git add . && git commit -m "Adding sample index page"
~~~

# Front-end

## Bower

I would not add node_modules to git as it was suggested <https://coderwall.com/p/6bmygq/heroku-rails-bower>

You need to use two build packs. Follow this
[commit](https://github.com/duleorlovic/heroku-rails-bower/commit/ba7c78a1f4f641cfe5592aa75b471aa142dc855a).
It works for latest node and npm version. Older version could give errors:

* bower version `~1.2` gives me error `error Path must be a string. Received
   ...` so make sure you use latest bower.

Old approach with assets from gems still works, no worry.
First, we need to initialize bower.

~~~
# do not name your package as bower or other existing package
npm init -y # to create package.json, -y to accept defaults
npm install bower --save
sed -i package.json -e '/scripts/a \
    "postinstall": "./node_modules/bower/bin/bower install",'
yes '' | bower init # to create bower.json, 'yes' is to choose default options
echo '{
  "directory": "vendor/assets/bower_components"
}' > .bowerrc
echo '
# npm and bower packages
node_modules
vendor/assets/bower_components' >> .gitignore
cat >> config/initializers/assets.rb << HERE_DOC
Rails.application.config.assets.paths << Rails.root.join('vendor', 'assets', 'components')
Rails.application.config.assets.precompile << /\.(?:svg|eot|woff|ttf)$/
HERE_DOC
git add . && git commit -m "Adding bower"

heroku create myapp-with-bower
heroku addons:create heroku-postgresql:hobby-dev
heroku buildpacks:set https://github.com/heroku/heroku-buildpack-ruby
heroku buildpacks:add --index 1 https://github.com/heroku/heroku-buildpack-nodejs
heroku buildpacks # should return  1.nodejs  2.ruby (latest will run process)
# alternativelly, we can define then in file .buildpacks
# echo 'https://github.com/heroku/heroku-buildpack-ruby
# https://github.com/heroku/heroku-buildpack-nodejs ' > .buildpacks
# heroku config:add BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git
git push heroku master --set-upstream
~~~

Example adding bootstrap:

~~~
bower install bootstrap --save
sed -i app/assets/stylesheets/application.css -e '/require_tree/i \
 *= require bootstrap/dist/css/bootstrap'
sed -i app/assets/javascripts/application.js -e '/require_tree/i\
//= require bootstrap/dist/js/bootstrap'

git commit -am "Adding bootstrap"
~~~

Adding css and js files is working fine. There is a problem when some image/font files are hardcoded in css files.
Hopefully there is scss verion of your library and you can override some
variables. When filename is fixed and you can not include digest sha
than you need to deploy files without fingerprint. With help of
[non-stupid-digest-assets](https://github.com/alexspeller/non-stupid-digest-assets)
gem you can add non digest version. First your assets should be seen
(precompiled) with sprockets than they will be again copied without digest.

Sprockets `require` concatenates after sass compilation. So it's advices to use
`@import` sass command instead of `require`. `@import` will work also in
`application.css` but variable definition won't (like `$var: 1;`), so we need
to move `css -> scss`.

Here is example adding [fontawesome](http://fontawesome.io/examples/)

~~~
bower install fontawesome --save
mv app/assets/stylesheets/application.css app/assets/stylesheets/application.scss
cat >> app/assets/stylesheets/application.scss << \HERE_DOC
$fa-font-path: 'font-awesome/fonts';
@import 'font-awesome/scss/font-awesome';
HERE_DOC
cat >> Gemfile << HERE_DOC
gem "non-stupid-digest-assets"
HERE_DOC

cat >> config/initializers/assets.rb << \HERE_DOC
Rails.application.config.assets.precompile << /\.(?:svg|eot|woff|ttf)$/
HERE_DOC

cat >> config/initializers/non_digest_assets.rb << \HERE_DOC
NonStupidDigestAssets.whitelist += [
  /\.(?:svg|eot|woff|ttf)$/
]
HERE_DOC
~~~

You can test with:

~~~
RAILS_ENV=production rake db:setup db:migrate
RAILS_ENV=production rake assets:precompile -v
RAILS_SERVE_STATIC_FILES=true rails s -e production
~~~

## Twitter bootstrap Gem

~~~
cat >> Gemfile << HERE_DOC
# twitter boostrap sass https://github.com/twbs/bootstrap-sass
# gem "bootstrap-sass" #, :git => "https://github.com/twbs/bootstrap-sass.git", :branch => "next"
# scaffolding generators and layout with rails generate bootstrap:install
gem 'bootstrap-generators', '~> 3.3.4'
HERE_DOC
bundle

# cat > app/assets/stylesheets/application.scss << HERE_DOC
# // "bootstrap-sprockets" must be imported before "bootstrap" and "bootstrap/variables"
# @import "bootstrap-sprockets";
# @import "bootstrap";
# HERE_DOC

rails generate bootstrap:install --force # this will overwrite application.html.erb and
# generate lib/templates and stylesheets/boostrapp-generatora/variables.scss

cat > app/assets/stylesheets/application.scss << HERE_DOC
@import "bootstrap-generators";
HERE_DOC

git add app/assets/stylesheets/application.scss
git rm app/assets/stylesheets/application.css


# sed -i app/assets/javascripts/application.js -e '/jquery_ujs/a \
# //= require bootstrap-sprockets' # this is not needed since previous command
# will insert //= require bootstrap

# cat >> app/assets/stylesheets/application.scss << HERE_DOC
# // fixed navbar needs padding http://getbootstrap.com/components/#navbar-fixed-top
# body {
#   padding-top: 60px;
# }
# HERE_DOC

git add . && git commit -m "Adding boostrap"
~~~

~~~
# add devise links
cat > /tmp/template <<\HERE_DOC
        </ul>
        <ul class="nav navbar-nav">
          <% if current_user %>
            <li class="<%= 'active' if %(forms questions).include? params[:controller] %>">
              <%= link_to "My Forms", forms_path %>
            </li>
          <% end %>
        </ul>
        <ul class="nav navbar-nav navbar-right">
          <% if current_user %>
            <li class="dropdown">
              <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false"><%= current_user.email %> <span class="caret"></span></a>
              <ul class="dropdown-menu">
                <li><a href="#">Action</a></li>
                <li><a href="#">Another action</a></li>
                <li><a href="#">Something else here</a></li>
                <li role="separator" class="divider"></li>
                <li><a href="<%= destroy_user_session_path %>" data-method="delete">Sign out</a></li>
              </ul>
          <% else %>
            <li><a href="<%= new_user_registration_path %>">Sign up</a></li>
            <li><a href="<%= new_user_session_path %>">Log in</a></li>
            <li><%= link_to "Sign in with Facebook", user_omniauth_authorize_path(:facebook) %></li>
            <li><%= link_to "Sign in with Google", user_omniauth_authorize_path(:google_oauth2) %></li>
          <% end %>
        </ul>
HERE_DOC

sed -i app/views/layouts/application.html.erb -e '/<.ul>/ {
  r /tmp/template
  d
}'
~~~

Here is an example adding style for specific media for alert

~~~
// app/assets/stylesheets/common.scss
@import "bootstrap-variables";

.hidden-left {
  position: absolute;
  left: -1000px;
}

@media(min-width:$screen-sm){
  .alert {
    position: absolute;
    top: 8px;
    padding: 8px;
    padding-right: 30px; // close sign
    z-index: 9999;
    left: 480px; // enought to see nav buttons
  }
}
~~~

Still, default rails form build will render `field_with_errors` on label and
input wrap, so if you need to change for bootstrap error classes.
Note that with Bootstrap 3, you have to change `control-group` to `form-group`,
add `form-control` to `<input>` elements, `help-inline` to `help-block`, and
`warning` to `has-warning`.
The easiest approach is with **bootstrap form**.

## Bootstrap form


[bootstrap form](https://github.com/bootstrap-ruby/rails-bootstrap-forms) gem
(not old [bootstrap_forms](https://github.com/sethvargo/bootstrap_forms)). Just
add two lines

~~~
cat >> Gemfile << HERE_DOC
# adding bootstrap_form_for
gem 'bootstrap_form'
HERE_DOC

sed -i app/assets/stylesheets/application.scss -e '1i\
/*\
 *= require rails_bootstrap_forms\
 */'
~~~

and use your `bootstrap_form_for`

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

~~~
cat > config/secrets.yml << HERE_DOC
# export keys in your .profile file
development: &default
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] || 'some_secret' %>

  # sending emails
  smtp_username: <%= ENV["SMTP_USERNAME"] %>
  smtp_password: <%= ENV["SMTP_PASSWORD"] %>

  default_mailer_sender: <%= ENV["DEFAULT_MAILER_SENDER"] || "My Company <support@example.com>" %>

  default_url:
    host: <%= ENV["DEFAULT_URL_HOST"] || "example.com" %>
    port: <%= ENV["DEFAULT_URL_PORT"] || 80 %>

test: << *default
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
#sed -i '/  end$/i \\n \   config.action_mailer.default_url_options = { host: "localhost", port: 3000 }' config/environments/development.rb 
sed -i config/application.rb -e '/  end$/i \
    config.action_mailer.default_url_options = Rails.application.secrets.default_url.symbolize_keys'
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
gem 'fog'
HERE_DOC
bundle
rails generate uploader Document
# we need just one field type string to store file url
rails g migration add_document_to_companies document:string
rake db:migrate
sed -i '/class Company/a \  mount_uploader :document, DocumentUploader'  app/models/company.rb
git add . && git commit -m "Adding carrierwave gem, document uploader and mount on company"
~~~

Replace `f.text_field :document` with `f.file_field :document` in your form. In
view you can use `company.document.url`.
You can use single table field for multiple files (field type json) but
than you need postgres database.

## Store on AWS S3

For [Amazon
S3](https://github.com/carrierwaveuploader/carrierwave#using-amazon-s3) you need
to set up your AWS keys in *~/.bashrc* `export AWS_ACCESS_KEY_ID=123123` and
`export AWS_SECRET_ACCESS_KEY=123123`. Bucket should be created as standard USA
bucket.

~~~
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
  aws_region: <%= ENV["AWS_REGION"] || "us-east-1" %>\n'

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

## Uploading using jQuery-File-Upload

[jQuery-File-Upload](https://github.com/blueimp/jQuery-File-Upload) 
is much cleaner way of uploading
[heroku tutorial](https://devcenter.heroku.com/articles/direct-to-s3-image-uploads-in-rails)
that uses
[PresignedPost](http://docs.aws.amazon.com/sdkforruby/api/Aws/S3/PresignedPost.html)

~~~
cat >> Gemfile << HERE_DOC
# direct S3 uploading
gem 'aws-sdk', '~> 2'
HERE_DOC

cat >> config/initializers/aws.rb << HERE_DOC
Aws.config.update({
  region: Rails.application.secrets.aws_region,
  credentials: Aws::Credentials.new(
    Rails.application.secrets.aws_access_key_id,
    Rails.application.secrets.aws_secret_access_key
  ),
})

S3_BUCKET = Aws::S3::Resource.new.bucket(
  Rails.application.secrets.aws_bucket_name
)
HERE_DOC
~~~

~~~
# in controller
before_action :set_s3_direct_post, only: [:new, :edit, :create, :update]

def set_s3_direct_post
  @s3_direct_post = S3_BUCKET.presigned_post(
    key: "uploads/#{SecureRandom.uuid}/${filename}",
    success_action_status: '201', # Aws will respond with XML
    acl: 'public-read'
  )
end
~~~

You can save one or many urls in `documents` field (type text) and use
`serialize :documents, Array` but if you have more logic:

~~~
rails g model document name key documentable:references{polymorphic}

~~~

# Production Heroku deploy

You can follow [heroku article](https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server)

~~~
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
~~~

On heroku add *Papertrail* add-on and go to the
[https://papertrailapp.com/events](https://papertrailapp.com/events) and search
for "Started GET" , save and create email alert or [add internal
notification]({{ site.baseurl }}{% post_url 2016-05-17-send-and-receive-emails-in-rails %})
