---
layout: post
title:  Common Rails bootstrap snippets
tags: ruby-on-rails devise carrierwave
---

Paste this code to create some basic starting application *myapp.com* with 
authentication and other funny tools.

> give a man a fish and you feed him for a day. teach a man to fish and you feed
him for a lifetime

Hint: `echo -e "\n" >> filename` will add new line (that's why `-e` to the
filename). You can use single quotes so you do not need to write `-e` and `\n`.
Or you can use `cat > filename << HERE_DOC ... some lines with ' or " ... HERE_DOC` for multiline. First `\HERE_DOC` when no parametar expanded.

`sed -i '/haus/a home' filename` will inplace (`-i`) search for *haus* and 
append *home* after that line (beside insert before `i`, this could be `a`
append and `c` change matched line). Multiple lines need to have `\` at the end of line (multiline *echo ' ...'* does not need that trailing backslash). You can use `sed -i "// a" file.txt` but then you need `\n\` at the end of each line. Remember that no char (even space) could be after last `\`.

`sed '$aTEXT_AT_END` where `$` (or `"\$a"` if `"` is used) match last line, `a`
is append. Some commands accept address `3,6aTEXT`

`sed 's/find/replace/g` will replace word `find` with `replace`.


Brackets need to be escaped like `\(`, `\1` means first group, `&` means matched
text...
[tutorial](http://www.thegeekstuff.com/2009/10/unix-sed-tutorial-advanced-sed-substitution-examples/)
Adding `,` and new `text` after `match` could be with `sed
's/\(match.*\)/\1,\ntext/`

Sed has something different regular expressions so follow this [link](http://www.gnu.org/software/sed/manual/html_node/Regular-Expressions.html)

When you want to change chars (and not whole line) than you can use `s/me/you/g', or several regexp in same command for example

~~~
sed src/app/index.module.coffee \
  -e 's/, /,\n  /g' \
  -e 's/  \[/\[\n  /g' \
  -e 's/\]/,\n\]/'
~~~


If you have a lot regexp, than its better to use [here-doc](http://tldp.org/LDP/abs/html/here-docs.html)
and read from standard input `-` [link](https://unix.stackexchange.com/questions/45591/using-a-here-doc-for-sed-and-a-file/45592#45592?newreg=39c715cb752f44a9bba9b3f3f74f2015)

~~~
sed src/app/index.module.coffee -f - <<HERE_DOC
s/, /,\n  /g
s/  \[/\[\n  /g
s/\]/,\n\]/
HERE_DOC
~~~

Note that all commands one per line. As with `sed ''` you need backslash for each line of multiline command.

 When you really need to add multiline template
and don't want to escape `'` and to add `\` to the end of each line, than you
can try following command [inspiration](https://stackoverflow.com/questions/26770426/use-sed-in-bash-to-replace-string-with-heredoc/26770678#26770678)
which will replace new lines with `ћ` (`N` means multiline match, `a` label... multiline match `:a;N;$!ba;s/\n/ћ/g`, note that `$` needs to be escaped if inside `""`) and than return back new line.

Quote in `'HERE_DOC'` will not [substitute params](http://tldp.org/LDP/abs/html/here-docs.html#EX71C) so `$` `/` or `\` will remain in here doc. Instead of `<<` you can use `<<-` and closing HERE_DOC can be indented with *tab* character.
For regexp we need
to escape `/` and `\` with `\/` and `\\` (`s:\\:\\\\:g;s:/:\\/:g`)

~~~
sed src/app/index.module.coffee -e "s/client/$(sed ':a;N;$!ba;s/\n/ћ/g;s:\\:\\\\:g;s:/:\\/:g;' <<'HERE_DOC'
    I'm "$just"
    some long /'\ multiline <\template>.
HERE_DOC
)/g;s/ћ/\n/g"
~~~

Sometimes we need to add/replace line with template (not replace regexp)
than we need to move replacing `s/#/\n/g` outside or sed since it will
not be applied to new template.
Note than we can't use inline replacement but we can move tmp file.

~~~
sed src/app/index.module.coffee -e "/client/a $(sed ':a;N;$!ba;s/\n/ћ/g;s:\\:\\\\:g;s:/:\\/:g;' <<'HERE_DOC'
    I'm "$just"
    some long /'\ multiline <\template>.
HERE_DOC
)" | sed "s/ћ/\n/g" > tmp && mv tmp src/app/index.module.coffee
~~~


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
sed -i '/group :development do/a  \
  # to detect N+1 sql queries\
  gem "bullet"\
  # do not show assets in log\
  gem "quiet_assets"\
  # irbtools includes interactive_editor gem (vim inside irb)\
  # just create ~/.irbrc with\
  # require "rubygems"\
  # require "irbtools"\
  gem "irbtools", require: "irbtools/binding"
' Gemfile

echo '
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

For various ways of integrating Angular look at
[angular-and-ruby-on-rails]({{ site.baseurl }}
{% post_url 2015-11-26-angular-and-ruby-on-rails %})

# Simplify secrets

~~~
echo -e '# export keys in your .profile file
development: &default
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] # rake secret %>
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
#sed -i '/  end$/i \\n \   config.action_mailer.default_url_options = { host: "localhost", port: 3000 }' config/environments/development.rb 
sed -i '/  end$/i \\n \   config.action_mailer.default_url_options = Rails.application.secrets.default_url.symbolize_keys' config/application.rb

# devise
sed -i '/mailer_sender/c \  config.mailer_sender =
Rails.application.secrets.default_mailer_sender' config/initializers/devise.rb
~~~

## Letter opener for development 

~~~
sed -i '/group :development do/a  \
  # open emails in browser\
  gem "letter_opener"' Gemfile
sed -i '/^end$/i \  config.action_mailer.delivery_method = :letter_opener' config/environments/development.rb 
bundle && git add . && git commit -m "Letter opener to see emails in browser"
~~~

## Mandrill for production and local interceptor

We need two environment variables: `MANDRILL_API_KEY` and `MAIL_INTERCEPTOR_EMAIL` which you should set up
 in your *~/.bashrc* file with this `export MANDRILL_API_KEY=123123` and `export MAIL_INTERCEPTOR_EMAIL=you@youremail.com`

~~~
echo '# sending emails
gem "mandrill_dm"' >> Gemfile && bundle
sed -i '/^  end$/i \\n    config.action_mailer.delivery_method = :mandrill' config/application.rb

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

See blog post [devise-oauth-angular]({{ site.baseurl }}
{% post_url 2015-12-20-devise-oauth-angular %})

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
export MYAPP_NAME=air
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
heroku addons:create heroku-postgresql:hobby-dev
git push heroku master --set-upstream
~~~

On heroku add *Papertrail* add-on and go to the [https://papertrailapp.com/events](https://papertrailapp.com/events) and search for "Started GET" , save and create email alert.

