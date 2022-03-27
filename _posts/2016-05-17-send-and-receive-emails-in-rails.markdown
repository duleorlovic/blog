---
layout: post
title: Send and receive emails in rails
tags: rails emails
---

# Email providers

There is nice table of [main
providers](http://socialcompare.com/en/comparison/transactional-emailing-providers-mailjet-sendgrid-critsend)

## Check SMTP

If you need to check smtp use <https://debugmail.io/> free service, just use
port 9025 instead 25 since ISP is blocking 25.
For command line you can use `swaks` like `swaks --to duleorlovic@gmail.com
--server $SERVER --port $PORT --auth-user $AUTH_USER --auth-password
$AUTH_PASSWORD --auth-plaintext --auth-hide-password`
so in autout you can see all telnet communications:

~~~
# generate base64 encoding
# special characters need to have \ in front
perl -MMIME::Base64 -e 'print encode_base64("duleorlovic\@gmx.com");'
perl -MMIME::Base64 -e 'print encode_base64("password");'

telnet debugmail.io 9025
EHLO main
AUTH LOGIN
<paste encoded username>
<paste encoded password>

ctrl + ]
ctrl + d
~~~

If you want to inspect how rails action_mailer sends and receive tcp messages
than put byebug in net smtp class on line 940 `get_response` `recv_response`
`/home/orlovic/.rvm/rubies/ruby-2.3.3/lib/ruby/2.3.0/net/smtp.rb`


## Gmail

Gmail smtp is the most easiest way to start

~~~
# config/application.rb
    config.action_mailer.smtp_settings = {
      address: 'smtp.gmail.com',
      port: 587,
      domain: 'gmail.com',
      authentication: 'plain',
      enable_starttls_auto: true,
      user_name: Rails.application.secrets.smtp_username,
      password: Rails.application.secrets.smtp_password
    }
    config.action_mailer.delivery_method = :smtp

# config/secrets.yml
  smtp_username: <%= ENV["SMTP_USERNAME"] %>
  smtp_password: <%= ENV["SMTP_PASSWORD"] %>
~~~

If you receive error `SocketError: getaddrinfo: Name or service not known` than
you probably miss the `address` field.
If there is error `with EOFError: end of file reached` than you need to change
`domain` field (should not be `localhost`, but the domain part of the sender
email, for example `gmail.com`).

The best way is to enable 2 step verification and create App Password https://support.google.com/accounts/answer/185833
App password can be used instead of password and does not require enable less
secure apps.

If you see error in logs:

```
2018-06-18T09:13:29.371621+00:00 app[web.1]: An error occurred when sending a notification using 'email' notifier. Net::SMTPAuthenticationError: 534-5.7.14 <https://accounts.google.com/signin/continue?sarp=1&scc=1&plt=AKgnsbu5

Username and Password not accepted. Learn more
```

**Note that google does not allow less secure app any more** https://support.google.com/accounts/answer/6010255
You need to Allow less secure apps https://support.google.com/accounts/answer/6010255

gmail send email as sometimes stops since google disable allow less secure app.
port 587 and Secured connection using TLS (this is default recommended) but
enable https://myaccount.google.com/u/7/lesssecureapps? or you will get error
```
Authentication failed. Please check your username/password and Less Secure Apps access for
```

Sometimes you can send from your IP but not from Heroku IP address.

```
For error  Errno::ECONNREFUSED (Connection refused - connect(2) for "localhost" port 25
```
the problem occurs when you in initializers (for example
config/initializers/devise.rb or config/initializers/exception_notification.rb)
use ApplicationMailer::MAILER_SENDER or some other constant from Rails classes
Note that this occurs only on production. So use only constants from
initializers.
```
Rails.application.config.action_mailer.smtp_settings
# it is the same as
Rails.configuration.action_mailer.smtp_settings
=> {:address=>"smtp.gmail.com", :port=>587, :authentication=>"plain", :enable_starttls_auto=>true, :user_name=>...
Rails.configuration.action_mailer.delivery_method
=> :smtp
```

## AWS Workmail

```
# config/application.rb
    config.action_mailer.smtp_settings = {
      address: 'smtp.mail.us-east-1.awsapps.com',
      port: 465,
      domain: 'mydomain.com',
      user_name: Rails.application.credentials.smtp_username,
      password: Rails.application.credentials.smtp_password,
      authentication: 'login',
      enable_starttls_auto: false,
      tls: true,
      ssl: true,
    }
    config.action_mailer.delivery_method = :smtp
```

aws ses simple email service
https://www.sitepoint.com/deliver-the-mail-with-amazon-ses-and-rails/

```
config.action_mailer.smtp_settings = {
  :address => "email-smtp.us-east-1.amazonaws.com",
  :port => 587,
  :user_name => ENV["SES_SMTP_USERNAME"], #Your SMTP user
  :password => ENV["SES_SMTP_PASSWORD"], #Your SMTP password
  :authentication => :login,
  :enable_starttls_auto => true
}
```
## Sendgrid

Sendgrid is simple to start on heroku. Just add new add-on free plan with
commands `heroku addons:create sendgrid` and that will set up env keys.
`heroku config` you can find the keys and copy them to `heroku config:set
SMTP_USERNAME=asdasdasd SMTP_PASSWORD=asdasdasd`. It allows sending with `from`
field any domain, but in gmail it shows that message is from: `My Company
support@example.com via sendgrid.me`.

~~~
# do not use Rails.application.config.action_mailer.smtp_settings
cat > config/initializers/smtp.rb << \HERE_DOC
ActionMailer::Base.smtp_settings = {
  :user_name => Rails.application.secrets.smtp_username,
  :password => Rails.application.secrets.smtp_password,
  :domain => 'yourdomain.com',
  :address => 'smtp.sendgrid.net',
  :port => 587,
  :authentication => :plain,
  :enable_starttls_auto => true
}
HERE_DOC
~~~

Another way is to use API

## Mandrill

Mandrill is better than Sendgrid, since Sendgrid can not automatically convert
html to txt mails. Also mandrill has nice API so you do not need background
job to send a lot of emails quickly. To setup sending using API just run:

~~~
# Gemfile
gem 'mandrill_dm'

# config/application.rb
config.action_mailer.delivery_method = :mandrill

# config/initializers/mandrill.rb
MandrillDm.configure do |config|
  config.api_key = Rails.application.secrets.mandrill_api_key
end

# config/secrets.yml
development:
  mandrill_api_key: <%= ENV["MANDRILL_API_KEY"] %>
~~~

If you are using `mandril_delivery` for ExceptionNotification than emails will
look scrambled, because generated html version will join all lines. Note that it
will trigger any webhooks that you have set up.

## Sparkpost

Sparkpost offer a lot of free usage (mandrill requires subscription) so
currently it is my best option. You need first to validate your domain, so you
can send with `from` field with that domain. You need also to

~~~
echo "gem 'sparkpost_rails'" >> Gemfile

sed -i config/environments/production.rb -e '/^end$/i \
  config.action_mailer.delivery_method = :sparkpost'

cat > config/initializers/sparkpostrails.rb << HERE_DOC
# https://github.com/the-refinery/sparkpost_rails#additional-configuration
SparkPostRails.configure do |c|
  c.api_key = Rails.application.secrets.sparkpost_api_key
end
HERE_DOC

sed -i config/secrets.yml -e '/^test:/i \
  # email provider\
  sparkpost_api_key: <%= ENV["SPARKPOST_API_KEY"] %>'

vi config/secrets.yml # update mailer_sender to match your domain
~~~

You can also use smtp with SPARK_POST but it is two times slower

~~~
# config/application.rb
    config.action_mailer.smtp_settings = {
      address: 'smtp.sparkpostmail.com',
      port: 587,

      enable_starttls_auto: true,
      user_name: 'SMTP_Injection',
      password: Rails.application.secrets.sparkpost_api_key,
    }
    config.action_mailer.delivery_method = :smtp # sparkpost

# check local configuration
rails runner "puts Rails.application.config.action_mailer.smtp_settings"
time rails runner 'UserMailer.signup.deliver_now!' # ~5sec with smtp
time rails runner 'UserMailer.signup.deliver_now!' # ~2.5sec with sparkpost
~~~

# Letter opener for local preview

~~~
sed -i '/group :development do/a  \
  # open emails in browser\
  gem "letter_opener"' Gemfile
sed -i '/^end$/i \  config.action_mailer.delivery_method = :letter_opener' config/environments/development.rb 
~~~

Note that email letter opener does not work when you run with `rake jobs:work`,
but works when `bin/delayed_job run` (Launchy works in both cases, this
difference is only for mailer).

# Interceptor

When you need to check production emails localy, than you can set up interceptor
so you receive all emails (and not real customer emails).

~~~
# config/initializers/interceptor.rb
class DevelopmentMailInterceptor
  def self.delivering_email(message)
    message.subject = "#{message.to} #{message.subject}"
    message.to = Rails.application.secrets.mail_interceptor_email
  end
end
if Rails.env.development?
  ActionMailer::Base.register_interceptor(DevelopmentMailInterceptor)
end

# config/secrets.yml
  mail_interceptop_email: <%= ENV["MAIL_INTERCEPTOR_EMAIL"] %>
~~~

When you need to preview a lot of emails, its faster to use letter_opener gem.
Just put in your Gemfile under development `gem "letter_opener"` and in
*config/environments/development.rb* `config.action_mailer.delivery_method =
:letter_opener`. Works when email is sent (even from ajax response or console).

# Style

[Official gmail styles](https://developers.google.com/gmail/design/css) supports
`<style>` in head and media queries but when you forward email than css styles
will be gone. Better is to use gem which will copy and duplicate all styles from
head to inline styles and that will support more clients (not just gmail).

For easier styling, you should use *roadie* gem that will generate all inline
style from your head styles.

~~~
# Attach css classes to emails
gem 'roadie'
gem 'roadie-rails'
~~~

You need to include mixing to each mailer (including in ApplicationMailer does
not help)

~~~
# app/mailers/my_mailer.rb
class MyMailer < ActionMailer::Base
  include Roadie::Rails::Automatic
end

# or include in ApplicaitionMailer and use roadie_mail

class ApplicationMailer < ActionMailer::Base
  include Roadie::Rails::Mailer
end

class MyMailer < ActionMailer::Base
  def welcome(user)
    roadie_mail to: user.email
  end
end
~~~

You can change template with `mail to: 'my@email.com', template_name:
'contact_form'`

To send without template you can
```
mail to: 'me@email.com' do |format|
  format.html { render text: 'a' }
end
```

Another solution is `gem 'premailer-rails'`
<https://github.com/fphilipe/premailer-rails> which can also generate text part
so you do not need to maintain it. Just add the gem and you are good to go.
I notice that in test I need to replace
```
mail = ActionMailer::Base.deliveries.last
# instead of using: mail.body use
mail.body.encoded
# or
mail.to_s
```

https://github.com/fphilipe/premailer-rails#how-it-works
You can use external styles (from public or from cdn) and it will be converted
to inline. Can not use font awesome since it requires custom font which is not
supported in gmail
https://stackoverflow.com/questions/40030954/how-to-use-custom-font-in-email-template

Gmail Android App will also parse media queries and apply that to inline styles
(it will override inline styles).
Gmail in the browser will parse styles (but not media queries) and apply to the
elements when presenting to the user.

To preview emails use generated preview files in
`test/mailers/previews/my_mailer_preview.rb` or create new file:

~~~
# app/mailer_previews/application_mailer_preview.rb
class ApplicationMailerPreview < ActionMailer::Preview
  def new_message_from_client
    message = Message.first || FactoryBot.create :message
    ApplicationMailer.new_message_from_client message
  end
end
~~~

add a line `config.action_mailer.preview_path =
"#{Rails.root}/app/mailer_previews"` to *config/environments/development.rb* and
go to [rails/mailers](http://localhost:3000/rails/mailers).
If you are using catch all route than add those lines
```
# config/routes.rb
  # https://stackoverflow.com/questions/26130130/what-are-the-routes-i-need-to-set-up-to-preview-emails-using-rails-4-1-actionmai
  get '/rails/mailers' => "rails/mailers#index"
  get '/rails/mailers/*path' => "rails/mailers#preview"
  # https://stackoverflow.com/a/6047561/287166
  match '*a', to: 'home#routing_error', via: [:get, :post]
```

Add authentication
```
# config/initializers/mailer_preview.rb
# https://stackoverflow.com/questions/60934362/rails-6-actionmailer-previews-and-http-basic-authentication
# https://stackoverflow.com/a/39399116/287166
class ::Rails::MailersController
  before_filter :_authenticate_admin!
  def _authenticate_admin!
    redirect_to root_path, alert: 'Only admin' unless current_admin_user.present?
  end
end
```

Another gem to preview emails <https://github.com/markets/maily>

Here is example of style:

~~~
# app/mailers/applicaion_mailer.rb
class ApplicationMailer < ActionMailer::Base
  default from: 'from@example.com'
  layout 'mailer'
  add_template_helper MailerHelper
end
~~~

~~~
# app/views/layouts/mailer.html.erb
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <style>
      .email-container {
        max-width: 500px;
      }
      .pre-header {
        display: none;
      }
      .bordered {
        border: 2px solid #ccc;
        border-radius: 5px;
        padding: 5px;
        background: #e5f1ff;
      }
    </style>
  </head>

  <body>
    <div class="email-container">
      <div class="pre-header">
        <%= yield :subject_line %>
      </div>
      <%= yield %>
    </div>
  </body>
</html>
~~~

~~~
# app/helpers/mailer_helper.rb
module MailerHelper
  def subject_line(message)
    content_for :subject_line, message
  end
end
~~~

~~~
# app/mailers/user_mailer/contact.html.erb
<%
  subject_line @user.name
%>
<h1><%= t "user_mailer.landing_signup.title", name: @user.email %></h1>
~~~

To change layout for devise mailer you can use
https://github.com/heartcombo/devise/wiki/How-To:-Create-custom-layouts#application--devise-config
```
  Devise::Mailer.layout "email"
```
or better is to change parent email
```
# config/initializers/devise.rb
  config.parent_mailer = 'ApplicationMailer'
```

# Receiving emails

When you want to receive, use [mandrill-rails](https://github.com/evendis/mandrill-rails).

~~~
echo '
# receiving emails and webhooks
gem "mandrill-rails" ' >> Gemfile

sed 
resource :inbox, :controller => 'inbox', :only => [:show,:create]
config/routes.rb

echo 'class InboxController < ApplicationController
  include Mandrill::Rails::WebHookProcessor

  def handle_inbound(event_payload)
    # do something with payload
  end
end ' > app/controllers/inbox_controller.rb
~~~

Mandrill:

* create api key for prod and test
* validate inbound domains for prod and test
* create routes for validated domains (this will create one webhook)
* create webhooks
* create rules that match api and hooktype and send it to webhook

authentication
http://www.openspf.org/SPF_Record_Syntax

Feedback Loop is in a header and some clients enable them
http://www.list-unsubscribe.com/


# Internal Notification

Those are usefull admin or devops notifications

~~~
# config/secrets.yml
shared:
  mailer_sender: <%= ENV["MAILER_SENDER"] || "My Company <support@example.com>" %>
  internal_notification_email: <%= ENV["INTERNAL_NOTIFICATION_EMAIL"] || "internal@example.com" %>
~~~

~~~
# mailers/application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  layout 'mailer'
  default from: Rails.application.secrets.mailer_sender

  INTERNAL_NOTIFICATION_EMAIL = Rails.application.secrets.internal_notification_email

  def internal_notification(subject, item = {})
    return unless INTERNAL_NOTIFICATION_EMAIL
    email_subject = "[MyApp#{' staging' if Rails.application.secrets.is_staging}] #{subject}"
    email_body = "<h1>#{subject}</h1><strong>Details:</strong>" +
                 item.inspect.
                   gsub(', ', ",<br>").
                   gsub('{', '<br>{<br>').
                   gsub('}', '<br>}<br>')
    mail to: INTERNAL_NOTIFICATION_EMAIL,
         subject: email_subject,
         body: email_body,
         content_type: "text/html"
  end
end
~~~

You can send notification in any class. Note that first param is string, and
ohers are hash.

~~~
# app/models/user.rb
  after_save :send_notification_geocode_failed

  def send_notification_geocode_failed
    if address_changed? && !city.present?
      ApplicationMailer.internal_notification(
        "geocode city is not present #{name}",
        name: name,
        url: Rails.application.routes.url_helpers.menu_url(link),
        address: address,
      ).deliver_now
    end
  end
~~~

Note that you should not send email in before blocks since when validation fails
it will rollback and even background job is rollbacked.

# ActionMailer

[Here](http://guides.rubyonrails.org/action_mailer_basics.html#complete-list-of-action-mailer-methods)
is what we can do with ActionMailer:

* headers
* attachments
* mail

You can use `before_action` and `after_action` and access to `params`
http://guides.rubyonrails.org/action_mailer_basics.html#action-mailer-callbacks

For example `prevent_delivery_to_guests`

~~~
class UserMailer < ApplicationMailer
  before_action { @business, @user = params[:business], params[:user] }

  after_action :prevent_delivery_to_guests

  def feedback_message
  end

    def prevent_delivery_to_guests
      if @user && @user.guest?
        mail.perform_deliveries = false
      end
    end
  end
~~~

# Dynamic smtp settings at runtime

~~~
class DeviseMailer < Devise::Mailer
  after_action :set_smtp

  def set_smtp
    # determine smtp settings form @receiver or other
    if isp.use_my_smtp_server && @_mail_was_called # spam could ignore mail
      mail.from = "#{receiver.smtp_from_name} <#{receiver.smtp_from_email}>"
      mail.reply_to = "#{receiver.smtp_from_name} <#{receiver.smtp_from_email}>"
      mail.delivery_method.settings.merge!(
        address: receiver.smtp_host,
        port: (receiver.smtp_port.present? ? receiver.smtp_port : 587),
        user_name: receiver.smtp_username,
        password: receiver.smtp_password,
      )
    end
  end
~~~

# Save emails in database


```
# app/mailers/application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  default from: Rails.application.credentials.mailer_sender
  layout 'mailer'

  after_action :save_email

  def save_email
    return if @user.blank?

    # here we have access to `mail` object
    @user.member_profile.emails.create(
      to: @user.email,
      subject: mail.subject,
      body: mail.body,
    )
  end
end
```
Test
```
# test/mailers/application_mailer_test.rb
require 'test_helper'

class ApplicationMailerTest < ActionMailer::TestCase
  test '#save_email' do
    user = users(:user)
    assert_difference 'Email.count', 1 do
      AdminMailer.add_photos(user).deliver_now
    end
    email = Email.last
    assert_equal 'Add your photo to improve responses', email.subject
  end
end
```

# Interesting

You can include small giff that looks like screencast. Image should be less than
1MB and included inline.

Email gems <http://awesome-ruby.com/#-email> and
[gmail](https://github.com/gmailgem/gmail)

You can use img tags and css background image, but if it is run in background
(it does not know on which request.host) than you need to set asset host (look
in [common rails bootstrap snippets]( {{ site.baseurl }} 
{% post_url 2015-04-05-common-rails-bootstrap-snippets %})


~~~
# app/views/layouts/mailer.html.erb
background-image: url('<%= asset_url 'cute-small.jpg' %>');
<%= image_tag 'premesti_se.gif' %>

# config/application.rb
config.action_mailer.asset_host = "http://my_host"
~~~

# Gmail Go To Action

Using some header json you can set button in gmail subject line "Quick Actions".
<https://stackoverflow.com/questions/22318432/how-do-i-add-go-to-action-in-gmail-subject-line-using-schema-org>

# Spam detection

You can disable registering specific email domains using this list
https://github.com/FGRibreau/mailchecker
Using this gem https://github.com/rubygarage/truemail you can check if actual
email account exists on smtp server.
Fake emails are detected using: whitelist/blacklist, regex, mx validation, smtp
validation
```
email_address = EmailAddress.new 'asd@asd.asd'
email_address.valid?
 => false
email_address.error
 => "Domain name not registered"
```

Format of emails can be validated using https://github.com/afair/email_address
```
Email
```

# Prevent spam

https://support.google.com/mail/thread/3973530?hl=en


# Testing emails

https://www.engineyard.com/blog/testing-async-emails-rails-42

~~~
# test/support/mailer_helpers.rb
module MailerHelpers
  def clear_mails
    ActionMailer::Base.deliveries = []
  end

  # if you deliver_now you can
  # assert_difference 'all_mails.count', 1 do
  # and for background deliver_later you need to assert perform or enqueue
  # inherit from ActiveJob::TestCase
  # or include ActiveJob::TestHelper
  # assert_performed_jobs 1, only: ActionMailer::MailDeliveryJob do
  def all_mails
    ActionMailer::Base.deliveries
  end

  # last_email is renamed to last_mail
  def last_mail
    raise 'you_should_use_give_me_last_mail_and_clear_mails'
    # ActionMailer::Base.deliveries.last
  end

  # some usage is like
  # mail = give_me_last_mail_and_clear_mails
  # assert_equal [email], mail.to
  # assert_match t('user_mailer.landing_signup.confirmation_text'), mail.html_part.decoded # mail.body.to_s when it is not multipart (devise) when there is not txt.erb template
  # confirmation_link = mail.html_part.decoded.match(
  #   /(http:.*)">#{t("confirm_email")}/
  # )[1]
  # visit confirmation_link
  def give_me_last_mail_and_clear_mails
    mail = ActionMailer::Base.deliveries.last
    clear_mails
    mail
  end
end
class ActiveSupport::TestCase
  include MailerHelpers
  # for assert_performed_jobs
  include ActiveJob::TestHelper
end
class ActionDispatch::IntegrationTest
  include MailerHelpers
  # for assert_performed_jobs
  include ActiveJob::TestHelper
end
~~~

# Click link tracking in emails

https://github.com/ankane/ahoy_email

# US Phones

You can use any number like `+1555,,,,` or `+1202555....` https://fakenumber.org/

* gmail shows download button for images, but you can prevent that by wrapping
  the image with link, or use style: `img + div { display:none; }`
* get local configuration development ActiveRecord
