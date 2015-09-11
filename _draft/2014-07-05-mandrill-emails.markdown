I prefer Mandrill instead of sendgrid, since sendgrid can not automatically convert html to txt mails. Mandrill has nice API so you do not need background job to send a lot of emails quickly.

To set it on rails, use those changes:

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

authentication
http://www.openspf.org/SPF_Record_Syntax


feedback loop is in a header and some clients enable them http://www.list-unsubscribe.com/

