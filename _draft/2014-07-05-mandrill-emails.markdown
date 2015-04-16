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

authentication
http://www.openspf.org/SPF_Record_Syntax


feedback loop is in a header and some clients enable them http://www.list-unsubscribe.com/

