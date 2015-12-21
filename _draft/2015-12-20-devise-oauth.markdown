# Devise

[Devise](https://github.com/plataformatec/devise)
Basic example application with devise default signup/login views.

You can use `before_action :authenticate_user!` in controllers that will redirect to `/users/sign_in`. You need to set up emails to actually receive registration email.



~~~
echo "gem 'devise'" >> Gemfile
bundle
rails generate devise:install
git add . && git commit -m "rails g devise:install"
rails g devise User
git add . && git commit -m "rails g devise user"
# optional
# rails g devise:views && git add . && git commit -m "rails g devise:views"
# sed -i '/<body>/a \\n<% if current_user %>\n  <strong><%= current_user.email %></strong> <a href="<%= destroy_user_session_path %>" data-method="delete">Sign out<a>\n<% else %>\n  <a href="<%= new_user_registration_path %>">Sign up</a> <a href="<%= new_user_session_path %>">Log in</a>\n<% end %>'  app/views/layouts/application.html.erb 
# git add . && git commit -m "Adding login/logout header in layout"
~~~

Read *config/initializers/devise.rb* about default configuration.

## Oauth

Read [wiki](https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview) to add facebook, google authentication.

~~~
echo -e "\n\
gem 'omniauth-facebook'\n\
gem 'omniauth-google-oauth2'\n\
" >> Gemfile
bundle
rails g migration AddOmniauthToUsers provider:index uid:index
rake db:migrate
sed -i '/APP_ID/a \
  config.omniauth :facebook,\
                  Rails.application.secrets.facebook_key,\
                  Rails.application.secrets.facebook_secret,\
                  scope: "public_profile,email",\
                  info_fields: "email,name"\
  config.omniauth :google_oauth2,\
                  Rails.application.secrets.google_client_id,\
                  Rails.application.secrets.google_client_secret,\
                  {}\
' config/initializers/devise.rb

sed -i '/test:/i \
  # Facebook Autentication\
  facebook_key: <%= ENV["FACEBOOK_KEY"] %>\
  facebook_secret: <%= ENV["FACEBOOK_SECRET"] %>\
  # Google signup\
  google_client_id: <%= ENV["GOOGLE_CLIENT_ID"] %>\
  google_client_secret: <%= ENV["GOOGLE_CLIENT_SECRET"] %>\
' config/secrets.yml

sed -i '/<body>/a \
<%= link_to "Sign in with Facebook", user_omniauth_authorize_path(:facebook) %>\
' app/views/layouts/application.html.erb
echo 'class OmniauthCallbacksController < Devise::OmniauthCallbacksController
  # https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview
  # data is in request.env["omniauth.auth"]
  [:facebook, :google_oauth2, :twitter ].each do |provider| # yaho_oauth2
    define_method provider do
      @user = User.from_omniauth(request.env["omniauth.auth"])

      if @user.persisted?
        sign_in_and_redirect @user, :event => :authentication #this will throw if @user is not activated
        set_flash_message(:notice, :success, :kind => provider) if is_navigational_format?
      else
        session["devise.facebook_data"] = request.env["omniauth.auth"]
        redirect_to new_user_registration_url
      end
    end
  end
end ' > app/controllers/omniauth_callbacks_controller.rb

sed -i '/end/i \
  def self.from_omniauth(auth)\
    user = where(email: auth.info.email).first\
    return user if user\
    user = where(provider: auth.provider, uid: auth.uid).first\
    return user if user\
    # create new user with some password\
    user = User.create!(\
      email: auth.info.email,\
      password: Devise.friendly_token[0, 20],\
      provider: auth.provider,\
      uid: auth.uid,\
    )\
    # user.name = auth.info.name   # assuming the user model has a name\
    # user.image = auth.info.image # assuming the user model has an image\
    user\
  end\
' app/models/user.rb
vi app/models/user.rb # add :omniauthable
# no need for , :omniauth_providers => [:facebook]


sed -i '/devise_for/c \
  devise_for :users,\
    controllers: {\
      omniauth_callbacks: "omniauth_callbacks",\
    }\
' config/routes.rb
~~~

If you need multiple identities per account than create separate table for identity.

# Facebook app

Create app. By default fb app is in development mode and can only be used by app admins, developers and testers.

Add Contact Email on https://developers.facebook.com/apps/FACEBOOK_APP_ID/settings/
and toggle switch on https://developers.facebook.com/apps/FACEBOOK_APP_ID/review-status/

Add https://developers.facebook.com/apps/FACEBOOK_APP_ID/settings/advanced *Valid OAuth redirect URIs* for all sites like: `http://www.lvh.me:3001/` and `http://localhost:3004`

More on blog facebook share buttons.
 
# Google console

Create a project in google console and enable Google+ API. Create OAuth 2.0 client ID (or edit exists one clicking on its name) and set *Authorized redirect URIs* to all urls that will be used, like `http://devise-token-auth-demo.dev/omniauth/google_oauth2/callback` `http://www.lvh.me:3001/users/auth/google_oauth2/callback` and don't forget to save.
Also fill the *Authorized JavaScript origins* with domains and port like `http://localhost:3004`.

# Angular

Angular [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth) is used with [devise-token-auth](https://github.com/lynndylanhurley/devise_token_auth).
Run [client](https://github.com/lynndylanhurley/ng-token-auth#development) with:

~~~
cd ng-token-auth
npm install
cd test && bower install
vi config/default.yml # edi API_URL to match server port
cd ..
gem install sass
gulp dev
gnome-open http://localhost:7777/
~~~

# Testing

