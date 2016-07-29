---
layout: post
title: Send and receive emails in rails
tags: rails emails
---

# Email providers

There is nice table of [main
providers](http://socialcompare.com/en/comparison/transactional-emailing-providers-mailjet-sendgrid-critsend)

* Gmail smtp is the most easiest way to start

  ~~~
  # config/application.rb
    config.action_mailer.smtp_settings = {
        address: "smtp.gmail.com",
        port: 587,
        domain: "gmail.com",
        authentication: "plain",
        enable_starttls_auto: true,
        user_name: Rails.application.secrets.gmail_email,
        password: Rails.application.secrets.gmail_password
    }

  # config/secrets.yml
  ~~~


* Sendgrid is simple to start on heroku. Just add new add-on free plan with
  commands `heroku addons:create sendgrid` and that will set up env keys.
  `heroku config` you can find the keys and copy them to `heroku config:set
  SMTP_USERNAME=asdasdasd SMTP_PASSWORD=asdasdasd`. It allows sending with `from`
  field any domain, but in gmail it shows that message is from: `My Company
  support@example.com via sendgrid.me`.

  ~~~
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

  ~~~

  ~~~

* Mandrill is better than Sendgrid, since Sendgrid can not automatically convert
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

* Sparkpost offer a lot of free usage (mandrill requires subscription) so
  currenlty it is my best option. You need first to validate your domain, so you
  can send with `from` field with that domain. You need also to 

  ~~~
  echo "gem 'sparkpost_rails'" >> Gemfile

  sed -i config/environments/production.rb -e '/^end$/i \
    config.action_mailer.delivery_method = :sparkpost'

  cat > config/initializers/sparkpostrails.rb << HERE_DOC
  SparkPostRails.configure do |c|
    c.sandbox = false
    c.api_key = Rails.application.secrets.sparkpost_api_key
  end
  HERE_DOC

  sed -i config/secrets.yml -e '/^test:/i \
    # email provider\
    sparkpost_api_key: <%= ENV["SPARKPOST_API_KEY"] %>'

  vi config/secrets.yml # update default_mailer_sender to match your domain
  ~~~

# Interceptor

When you need to test production emails localy, than you can set up interceptor
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
:letter_opener`. Works whenever email is sent (even from ajax response or console).

# Style

For easier styling, you should use *roadie* gem, put in Gemfile

~~~
# Attach css classes to emails
gem 'roadie'
gem 'roadie-rails'
~~~

To preview emails create *app/mailer_previews/application_mailer_preview.rb*
with content

~~~
class ApplicationMailerPreview < ActionMailer::Preview
  def new_message_from_client
    message = Message.last
    ApplicationMailer.new_message_from_client message
  end
end
~~~

add a line `config.action_mailer.preview_path =
"#{Rails.root}/app/mailer_previews"` to *config/environments/development.rb* and
go to [rails/mailers](http://localhost:3000/rails/mailers).


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

Feedback Loop is in a header and some clients enable them http://www.list-unsubscribe.com/


# Internal Notification

Those are usefull admin or devops notifications

~~~
# app/mailers/application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  layout 'mailer'

  INTERNAL_EMAIL = Rails.application.secrets.internal_notification_email

  def internal_notification(subject, item)
    mail to: INTERNAL_EMAIL,
         subject: subject,
         body: "<h1>#{subject}</h1><strong>Details:</strong>" +
           item.inspect
             .gsub(', ', ",<br>")
             .gsub('{', '<br>{<br>')
             .gsub('}', '<br>}<br>'),
         content_type: "text/html"
  end
end
~~~

You can send notification in model (see that first arg is string, other are
joined to hash)

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

# ActionMailer

[Here](http://guides.rubyonrails.org/action_mailer_basics.html#complete-list-of-action-mailer-methods)
is what we can do with ActionMailer:

* headers
* attachments
* mail
