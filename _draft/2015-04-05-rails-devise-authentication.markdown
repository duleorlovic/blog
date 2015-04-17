Here are steps that will take you to the app with basic auth for example.com application.

~~~
rails new example
cd example
git init . && git add . && git commit -m "rails new example"
echo "gem 'devise'" >> Gemfile
bundle
rails g devise:install && rails g devise user && rake db:migrate
git add . && git commit -m "rails g devise:install && rails g devise user"
sed -i '/<body>/a \\n\n<p class="notice"><%= notice %></p>\n<p class="alert"><%= alert %></p>' app/views/layouts/application.html.erb 
sed -i '/end$/i \\n  config.action_mailer.default_url_options = { host: "localhost", port: 3000 }' config/environments/development.rb 
sed -i '/end$/i \\n  config.action_mailer.default_url_options = { host: "example.com" }' config/environments/production.rb 
# rails g devise:views
git add . && git commit -m "Finishing devise configuration"


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

http://sourcey.com/rails-4-omniauth-using-devise-with-twitter-facebook-and-linkedin/

If you want to override some devise controllers, you should write your own controllers that inherits from devise controllers.

For example when user click on confirmation link, they can be logged in automatically:
~~~
# app/controllers/confirmations_controller.rb

# GET /resource/confirmation?confirmation_token=abcdef
  def show
    super do
      sign_in resource if resource.errors.empty?
    end
  end

  def send_first_deal
    flash[:notice] = "First deal sent."
    redirect_to refer_path
  end

# config/routes.rb
  devise_for :users, controllers: { confirmations: 'confirmations' }
  devise_scope :user do
    # devise looks for user_root after confirmation and signedin
    get 'user_root', to: 'confirmations#send_first_deal'
  end

~~~

