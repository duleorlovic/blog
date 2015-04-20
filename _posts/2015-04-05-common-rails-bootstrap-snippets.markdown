---
layout: post
title:  Rails bootstrap snippets
categories: ruby-on-rails devise carrierwave
---

Paste this code to create some basic starting application *myapp.com* with authentication and other funny tools.

> give a man a fish and you feed him for a day. teach a man to fish and you feed him for a lifetime

Initial commit

~~~
rails new myapp
cd myapp
git init . && git add . && git commit -m "rails new myapp"
echo -e "# vim temp files\\n*.swp\\n*.swo" >> .gitignore
git add . && git commit -m "Customization"
~~~

#### Devise authentication

Default [devise](https://github.com/plataformatec/devise) auth. You can use `before_action :authenticate_user!` in controllers that will redirect to `/users/sign_in`. You need to set up emails to actually receive registration email.

~~~
echo "gem 'devise'" >> Gemfile && bundle
rails g devise:install && rails g devise user && rake db:migrate
git add . && git commit -m "rails g devise:install && rails g devise user"
sed -i '/<body>/a \\n\n<p class="notice"><%= notice %></p>\n<p class="alert"><%= alert %></p>' app/views/layouts/application.html.erb 
sed -i '/end$/i \\n  config.action_mailer.default_url_options = { host: "localhost", port: 3000 }' config/environments/development.rb 
sed -i '/end$/i \\n  config.action_mailer.default_url_options = { host: "myapp.com" }' config/environments/production.rb 
# rails g devise:views && git add . && git commit -m "rails g devise:views"
git add . && git commit -m "Finishing configuration that devise gem suggests"
sed -i '/<body>/a \\n<% if current_user %>\n<strong><%= current_user.email %></strong> <a href="<%= destroy_user_session_path %>" data-method="delete">Sign out<a>\n<% else %>\n<a href="<%= new_user_registration_path %>">Sign up</a> <a href="<%= new_user_session_path %>">Log in</a><% end %>'  app/views/layouts/application.html.erb 
git add . && git commit -m "Adding login/logout header in layout"
~~~

#### Mandrill for sending emails

It will use two environment variables: `MANDRILL_API_KEY` and `MAIL_INTERCEPTOR_EMAIL` which you should set up in your *~/.bashrc* file with this `export MANDRILL_API_KEY=123123` and `export MAIL_INTERCEPTOR_EMAIL=you@youremail.com`

~~~
echo "gem 'mandrill_dm'" >> Gemfile && bundle
sed -i '/end$/i \\n  config.action_mailer.delivery_method = :mandrill' config/applications.rb

echo '# mandrill initializer
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
end' > config/initializers/mandrill.rb

sed -i '/\(development:\|test:\|production:\)/a \  mandrill_api_key: <%= ENV["MANDRILL_API_KEY"] %>\n  mail_interceptop_email: <%= ENV["MAIL_INTERCEPTOR_EMAIL"] %>' config/secrets.yml
git add . && git commit -am "Adding mandrill for sending emails"
~~~

#### Company scaffold with skipped unused files
~~~
rails g scaffold company name:string user:references --no-stylesheets --no-fixture --no-test-framework --no-helper --no-assets --no-jbuilder
sed -i '/companies/a \  root "companies#index"' config/routes.rb
rake db:migrate && git add . && git commit -m "rails g scaffold company name:string user:references"
~~~

#### Carrierwave for uploading
~~~
echo -e "gem 'carrierwave'\\ngem 'fog'" >> Gemfile && bundle
rails generate uploader Document
rails g migration add_document_to_users document:string
sed -i '/class User/a \  mount_uploader'  app/models/user_model.rb
rake db:migrate
~~~

