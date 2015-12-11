I prefer Mandrill instead of sendgrid, since sendgrid can not automatically convert html to txt mails. Mandrill has nice API so you do not need background job to send a lot of emails quickly.

To set sending just use those changes:

~~~
# Gemfile
gem 'mandrill_dm'

# config/application.rb
config.action_mailer.delivery_method = :mandrill

# config/initializers/mandrill.rb
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

# config/secrets.yml
development:
  mandrill_api_key: <%= ENV["MANDRILL_API_KEY"] %>
  mail_interceptop_email: <%= ENV["MAIL_INTERCEPTOR_EMAIL"] %>
~~~

When you need to preview a lot of emails, its faster to use letter_opener gem. Just put in your Gemfile under development `gem "letter_opener"` and in *config/environments/development.rb* `config.action_mailer.delivery_method = :letter_opener`. Works even for ajax requests.

For easier styling, you should use *roadie* gem, put in Gemfile

~~~
# Attach css classes to emails
gem 'roadie'
gem 'roadie-rails'
~~~

To preview emails create *app/mailer_previews/application_mailer_preview.rb* with content

~~~
class ApplicationMailerPreview < ActionMailer::Preview
 
  def new_message_from_client
    message = Message.last
    ApplicationMailer.new_message_from_client message
  end
end

~~~
add a line `config.action_mailer.preview_path = "#{Rails.root}/app/mailer_previews"` to *config/environments/development.rb* and go to [rails/mailers](http://localhost:3000/rails/mailers).


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



~~~
# app/mailers/application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  default from: "support@eatfresheveryday.com"
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

You can send notification in model (see that first arg is string, other are joined to hash)

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
