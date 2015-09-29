---
layout: post
title:  Rails bootstrap snippets
categories: ruby-on-rails devise carrierwave
---

Paste this code to create some basic starting application *myapp.com* with 
authentication and other funny tools.

> give a man a fish and you feed him for a day. teach a man to fish and you feed
him for a lifetime

Hint: `echo -e "\n" >> filename` will add new line (that's why `-e` to the
filename). You can use single quotes so you do not need to write `-e` and `\n`.
With sed, put `\` before jumping to new line.

`sed -i '/haus/a home' filename` will inplace (`-i`) search for *haus* and 
append *home* after that line (beside insert before `i`, this could be `a`
append and `c` change matched line)
Sed has something different regular expressions so follow this [link](http://www.gnu.org/software/sed/manual/html_node/Regular-Expressions.html)


Initial commit

~~~
rails new myapp
cd myapp
git init . && git add . && git commit -m "rails new myapp"
~~~

Gitignore 

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
' >> .gitignore
git commit -am "Update .gitignore"
~~~

Gemfile defaults

~~~
sed -i "/gem 'sqlite3/c gem 'sqlite3', group: :development\
\ngem 'pg', group: :production" Gemfile
echo -e "\\ngem 'rails_12factor', group: :production" >> Gemfile
echo -e "\\ngroup :development do\n\
  # to detect N+1 sql queries\n\
  gem 'bullet'\n\
  # do not show assets in log\n\
  gem 'quiet_assets'\n\
  # for using vim inside console, just add to .irbrc\n\
  # require 'irbtools'\n\
  #gem 'irbtools'\n\
  gem 'interactive_editor'\n\
end\n" >> Gemfile

git commit -am "Adding useful gems"
~~~

Simplify secrets

~~~
sed -i "/^development:/c development: &default" config/secrets.yml
sed -i "/^test:/c test: *default" config/secrets.yml

sed -i '/^development:\|^production:/a \  # Facebook Autentication\
  facebook_key: <%= ENV["FACEBOOK_KEY"] %>\
  facebook_secret: <%= ENV["FACEBOOK_SECRET"] %>' config/secrets.yml
~~~

### User model and Devise authentication

Default [devise](https://github.com/plataformatec/devise) auth. You can use `before_action :authenticate_user!` in controllers that will redirect to `/users/sign_in`. You need to set up emails to actually receive registration email.

~~~
echo "gem 'devise'" >> Gemfile && bundle
rails g devise:install && rails g devise user && rake db:migrate
git add . && git commit -m "rails g devise:install && rails g devise user"

sed -i '/<body>/a \\n\n  <p class="notice"><%= notice %></p>\n  <p class="alert"><%= alert %></p>' app/views/layouts/application.html.erb 
sed -i '/end$/i \\n  config.action_mailer.default_url_options = { host: "localhost", port: 3000 }' config/environments/development.rb 
sed -i '/end$/i \\n  config.action_mailer.default_url_options = { host: "myapp.com" }' config/environments/production.rb 
git add . && git commit -m "Finishing configuration that devise gem suggests"
# optional
# rails g devise:views && git add . && git commit -m "rails g devise:views"

sed -i '/<body>/a \\n<% if current_user %>\n  <strong><%= current_user.email %></strong> <a href="<%= destroy_user_session_path %>" data-method="delete">Sign out<a>\n<% else %>\n  <a href="<%= new_user_registration_path %>">Sign up</a> <a href="<%= new_user_session_path %>">Log in</a>\n<% end %>'  app/views/layouts/application.html.erb 
git add . && git commit -m "Adding login/logout header in layout"

# go and edit config/initializers/devise.rb
# to add facebook login folow https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview
~~~

### Sample page

~~~
rails g controller pages index --skip-helper --skip-assets --skip-controller-specs --skip-view-specs
sed -i "/root 'welcome#index'/c \  root 'pages#index'" config/routes.rb
git add . && git commit -m "Adding sample index page"
~~~

### Sending Email: letter opener for development and mandrill for production

~~~
echo -e 'gem "letter_opener", :group => :development' >> Gemfile
sed -i '/end$/i \  config.action_mailer.delivery_method = :letter_opener' config/environments/development.rb 
bundle && git add . && git commit -m "Letter opener to see emails in browser"
~~~

We need two environment variables: `MANDRILL_API_KEY` and `MAIL_INTERCEPTOR_EMAIL` which you should set up in your *~/.bashrc* file with this `export MANDRILL_API_KEY=123123` and `export MAIL_INTERCEPTOR_EMAIL=you@youremail.com`

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

### Twitter bootstrap

Source is http://railscasts.com/episodes/328-twitter-bootstrap-basics

#### Carrierwave for uploading

You should set up your AWS keys in *~/.bashrc* `export AWS_ACCESS_KEY_ID=123123` and `export AWS_SECRET_ACCESS_KEY=123123`. Bucket should be created as standard USA bucket. You can use single table field for multiple files (field type json) but than you need postgres database, we will keep it simple.

~~~
echo -e "gem 'carrierwave'\\ngem 'fog'" >> Gemfile && bundle
rails generate uploader Document
rails g migration add_document_to_companies document:string
sed -i '/class Company/a \  mount_uploader :document, DocumentUploader'  app/models/company.rb
rake db:migrate
git add . && git commit -m "Adding carrierwave gem, document uploader and mount on company"


~~~

