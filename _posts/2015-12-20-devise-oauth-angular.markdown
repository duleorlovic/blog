---
layout: post
title: From simple Devise to Oauth on Angular
tags: angular devise oauth
---

# Devise

Basic example application with [Devise](https://github.com/plataformatec/devise)
default signup/login views.

~~~
cat >> Gemfile <<HERE_DOC

# user authentication
gem 'devise', '~> 4.5.0'
HERE_DOC
bundle
rails generate devise:install
git add . && git commit -m "rails g devise:install"
rails g devise User
rake db:migrate
git add . && git commit -m "rails g devise user"

# optional generate views
rails g devise:views && git add . && git commit -m "rails g devise:views"
# optional create home page and navbar links
rails g controller pages home
sed -i config/routes.rb -e "/^end$/i \\
  # root page\n\
  root 'pages#home'\
"
sed -i app/views/layouts/application.html.erb -e "/<body>/a \\
    <%= link_to 'Home', root_path %>\n\
    <% if current_user %>\n\
      <strong><%= current_user.email %></strong>\n\
      <%= link_to 'Sign out', destroy_user_session_path, method: :delete %>\n\
    <% else %>\n\
      <%= link_to 'Sign up', new_user_registration_path %>\n\
      <%= link_to 'Log in', new_user_session_path %>\n\
    <% end %>\
"
git add . && git commit -m "Adding login/logout header in layout"
~~~

You can add facebook auth below.

Read *config/initializers/devise.rb* about default configuration.

You can use `before_action :authenticate_user!` in controllers that will
redirect to `/users/sign_in`. You need to [set up
emails]({{ site.baseurl }}{% post_url 2015-04-05-common-rails-bootstrap-snippets %})
to actually receive registration email.

If you enable `lockable` than user will be locked when number of failed login
attempts reaches 20. You can send internal notification when that happens by
overriding devise mailer and method `def unlock_instructions(record, token,
opts={})`

When user is logged in, than in session there is a id of current_user
`session['warden.user.user.key'] # => [[9], "$2a$10$TUfyHaPAWV.A1/6JLuCTGO"]`

Errors like `ActionView::Template::Error (undefined method new_confirmation_path
for Did you mean?  new_user_confirmation_path user_confirmation_path):` occurs
when you add, for example `confirmable` in model, but gem are already loaded in
spring, so you need to `spring stop` and restart rails server.

To enable additional fields to be permited attributes for new columns, for
example `:username`

~~~
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:username])
  end
end
~~~

Errors like `NoMethodError (undefined method 'users_url' for
#<DeviseRegistrationsController:0x007ff6068d92b8>):`

IF you need to check user.authenticate_with password you can use valid password

~~~
user.valid_password? 'new_password'
~~~

There are some modules which you can use

~~~
# app/models/user.rb
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable, :confirmable,
         :omniauthable
~~~



# Devise and Omniauth

Read [wiki](https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview) to
add facebook and google authentication. It's easy installation.
For facebook we use
[omniauth-facebook](https://github.com/mkdynamic/omniauth-facebook)
and excellent [rails casts #360](https://www.youtube.com/watch?v=E_XACDrZSiI)

~~~
cat >> Gemfile <<HERE_DOC
gem 'omniauth-facebook'
gem 'omniauth-google-oauth2', '0.2.5'
# https://github.com/zquestz/omniauth-google-oauth2/issues/204
# fix to 1.3.1 because of redirect_uri_mismatch
gem "omniauth-oauth2", '1.3.1'
# 1.2.1 to skip_jwt issue
# Could not find a valid mapping for path "/omniauth/google_oauth2/callback"
HERE_DOC
bundle
rails g migration AddOmniauthToUsers provider:index uid:index
rake db:migrate
sed -i config/initializers/devise.rb -e '/APP_ID/a \
  config.omniauth :facebook,\
                  Rails.application.secrets.facebook_key,\
                  Rails.application.secrets.facebook_secret,\
                  scope: "public_profile,email",\
                  info_fields: "email,name"\
  config.omniauth :google_oauth2,\
                  Rails.application.secrets.google_client_id,\
                  Rails.application.secrets.google_client_secret,\
                  {} # skip_jwt: true'
# skip_jwt only needed for omniauth > 1.2.2
# https://github.com/zquestz/omniauth-google-oauth2/issues/197

sed -i config/secrets.yml -e '/^test:/i \
  # Facebook Autentication\
  facebook_key: <%= ENV["FACEBOOK_KEY"] %>\
  facebook_secret: <%= ENV["FACEBOOK_SECRET"] %>\
  # Google signup\
  google_client_id: <%= ENV["GOOGLE_CLIENT_ID"] %>\
  google_client_secret: <%= ENV["GOOGLE_CLIENT_SECRET"] %>\
'
~~~

If you need to send some params to callback (for example current user, or some
other state) you can do it and access using `params['omniauth.params']` env
field.  There are also: `["omniauth.strategy", "omniauth.origin",
"omniauth.params", "omniauth.auth"]`. `omniauth.origin` is usefull to redirect
back to the page on which he logs in.

~~~
sed -i app/views/layouts/application.html.erb -e '/<body>/a \
<%= link_to "Sign in with Facebook",
user_facebook_omniauth_authorize_path(my_param: 1) %>'

sed -i app/models/user.rb -e '/end/i \
\
  def self.from_omniauth(auth)\
    user = find_by(email: auth.info.email)\
    return user if user\
    # user changed his email on facebook\
    user = find_by(provider: auth.provider, uid: auth.uid)\
    return user if user\
    # create new user with some password
    user = User.create!(\
      email: auth.info.email,\
      password: Devise.friendly_token[0, 20],\
      provider: auth.provider,\
      uid: auth.uid,\
    )\
    user.skip_confirmation! # this will just add confirmed_at = Time.now\
    # user.name = auth.info.name # assuming the user model has a name\
    # user.image = auth.info.image # assuming the user model has an image\
    user\
  end'

vi app/models/user.rb # add :omniauthable
# no need for , :omniauth_providers => [:facebook], but :omniauthable is needed

# config/routes.rb
  devise_for :users, controllers: {
    omniauth_callbacks: 'devise/my_omniauth_callbacks',
    confirmations: 'devise/my_confirmations',
    registrations: 'devise/my_registrations',
  }

# app/controllers/devise/my_omniauth_callbacks_controller.rb
module Devise
  class MyOmniauthCallbacksController < OmniauthCallbacksController
    # https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview
    # data is in request.env["omniauth.auth"]
    [:facebook, :google_oauth2, :twitter].each do |provider| # yaho_oauth2
      define_method provider do
        # use request.env["omniauth.params"]["my_param"]
        @user = User.from_omniauth(request.env["omniauth.auth"])

        if @user.persisted?
          sign_in_and_redirect @user, event: :authentication
          # this will throw if @user is not activated
          set_flash_message(:notice, :success, kind: provider) if is_navigational_format?
        else
          session["devise.facebook_data"] = request.env["omniauth.auth"]
          redirect_to new_user_registration_url
        end
      end
    end
    def failure
      # this could be: no_authorization_code # when we did not whitelisted domain
      # on facebook app settings
      redirect_to root_path, alert: "Can't sign in. #{request.env['omniauth.error'].try(:message)} #{request.env['omniauth.error.type']}"
    end
  end
end
~~~

If you need multiple identities per account than create separate table for
identity.

Note this situations:

* user changed email address (lost password for old one) and updated on
  facebook profile. He should be able to add new email address, not just to
  change old one. Maybe he used the same password which he can not recover. If
  user can recover password only using original email than they are blocked.
  Hackerrank has a button to "Add another email".
  So we should check if his facebook email address was updated and add add that
  email to our database as well.


* user click forgot password, it change password, than it is asked to login,
  than he should not see again forgot password page
* when user change the password it should stay logged in
  ```
    current_user.password='asdfasdf'
    current_user.password_confirmation='asdfasdf'
    current_user.save!
    bypass_sign_in current_user
  ```

# Facebook app

Is you use `omniauth-facebook` alone than there are problems with devise (it
double redirect and raise csrf exception). So do not use with devise, or
configure devise to use facebook auth.

1. Create app on [developers.facebook.com](http://developers.facebook.com). By
   default fb app is in development mode and can only be used by app admins,
   developers and testers. Add Contact Email if not already there on
   https://developers.facebook.com/apps/FACEBOOK_APP_ID/settings/

1. For production you need to go *App Review*
   https://developers.facebook.com/apps/FACEBOOK_APP_ID/review-status/ and
   toggle switch on

1. Add *Facebook Login* product and go to settings
   https://developers.facebook.com/apps/FACEBOOK_APP_ID/fb-login/ (before it was
   inside advance settings
   https://developers.facebook.com/apps/FACEBOOK_APP_ID/settings/advanced)
   Find input for *Valid OAuth redirect URIs* and fill with all links that are
   redirected. You can find the link on facebook error page url
   `www.facebook.com/dialog/oauth?client_id=...&redirect_uri=...`. You can add
   just domain (`/omniauth/facebook/callback` is not needed)
   Note that this are server url (not frontend url):

   * `http://localhost.local:3003`
   * `http://localhost:9000` and for https also
   * `https://localhost.local:3003`

Changes are visible immediatelly.
More on blog facebook share buttons.

Notes:

* no need to check if email is verified, since facebook will not allow
unverified accounts to use oauth

# Google console

Create a project in <https://console.developers.google.com/apis/> and enable
*Google+ API*.
Create *OAuth 2.0 client ID* (or edit exists one clicking on its
name) and set *Authorized redirect URIs* to all urls that will be used (not just
domain like for facebook, we need whole url path), like

* `http://localhost:3000/users/auth/google_oauth2/callback` this is default
  `devise_for :users`
* `http://localhost:9000/omniauth/google_oauth2/callback` and also for https
* `https://localhost.local/omniauth/google_oauth2/callback` 
* also fill the *Authorized JavaScript origins* with domains and port like
`http://localhost.local`.

Dont forget to save **Changes needs 5 min to propagate**

# Twitter app

On <https://apps.twitter.com/> you can create application.
Setup callbacks urls to point to you server, but this is not required with
twitter.
`app/{{APP_ID}}/keys` will give you TWITTER_API_KEY and TWITTER_API_SECRET

Twitter
[response](https://github.com/arunagw/omniauth-twitter#authentication-hash) do
not provide user's email address.

# Linkedin

Create app on <https://www.linkedin.com/developer/apps> and save to
LINKEDIN_CLIENT_ID and LINKEDIN_CLIENT_SECRET.
Add `gem 'omniauth-linkedin'`

# Angular authentication

I tried two approaches for authentication

* [angular_devise](https://github.com/cloudspace/angular_devise)
* [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth)

For demo usage checkout my repository
[angular-devise-ng-token-auth](https://github.com/duleorlovic/angular-devise-ng-token-auth)

## ng-token-auth demo example

Angular [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth) is
used with
[devise-token-auth](https://github.com/lynndylanhurley/devise_token_auth).

Run [client](https://github.com/lynndylanhurley/ng-token-auth#development) with:

~~~
cd ng-token-auth
npm install
cd test && bower install
vi config/default.yml # edit API_URL to match server port
# SITE_DOMAIN is only required for sitemap
cd ..
gem install sass
gulp dev
gnome-open http://localhost:7777/
~~~

For server side you can see [devise_token_auth_demo
compare](https://github.com/lynndylanhurley/devise_token_auth_demo/compare/dbf0a69b1e67e34862906c3d24747bc98ffff815...master)

~~~
cd devise_token_auth_demo
vi config/database.yml # change to your username
echo -e 'gem "letter_opener", :group => :development' >> Gemfile
sed -i '/end$/i \  config.action_mailer.delivery_method = :letter_opener' config/environments/development.rb

# hosts should be the same as in ng-token-auth/test/config/default.html
# config/environments/development.rb redirection url
# OmniAuth.config.full_host = "http://localhost.local:3003"

export GITHUB_KEY=asd GITHUB_SECRET=asd GOOGLE_KEY=$GOOGLE_CLIENT_ID GOOGLE_SECRET=$GOOGLE_CLIENT_SECRET FACEBOOK_KEY=$FACEBOOK_KEY FACEBOOK_SECRET=$FACEBOOK_SECRET
rails s
~~~

## ng-token-auth from scratch with Yeoman

When use signin, he get `access-token` in Response Header. Than he uses that
`access-token` for next request (Request Header), and for it he gets new
`access-token` in response [Token Header
Format](https://github.com/lynndylanhurley/devise_token_auth#token-header-format)

IMPORTANT:

Mount path is important. `mount_devise_token_auth_for 'User', at: '/auth'` so if
you access `api/v1` and `api/v2/...` it will send headers. If you mount under
`api/v1/auth` than headers will not be send for `api/v2/articles`

Rails 4.2.5 use uppercase header names (Access-Token), but that does not affect
the app. Also the method you fetch the resource does not matter, it could be
$http, angular-rails-resource... headers will be send with ng-token-auth

~~~
rails new my_app
cd my_app
cat >> Gemfile <<HERE_DOC
# user auth with devise and ng-token-auth
gem 'devise_token_auth', '=0.1.37.beta4' # fix version because of url localhost:3000//api/ issue
# github: 'lynndylanhurley/devise_token_auth'
gem 'omniauth'
HERE_DOC
bundle
rails g devise_token_auth:install User auth
rake db:migrate
git add . && git commit -m "rails g devise_token_auth:install User auth"
rails g devise:install
git add . && git commit -m "rails g devise:install"

# leave original route auth for mount_devise_token_auth_for

# config/environments/development.rb
# OmniAuth.config.full_host = "http://localhost:9000/" # this is url for google callback, not needed since it will read from request

# on registration, do not raise exception
sed -i app/controllers/application_controller.rb \
-e '/end$/i\
  protect_from_forgery with: :null_session\
  respond_to :json'

# allow unconfirmed access
sed -i config/initializers/devise.rb -e '/config.allow_unconfirmed_access_for/a \
  config.allow_unconfirmed_access_for = 7.days'

~~~

Perform some [common bootstrap stuff for secrets and emails]({{ site.baseurl }}
{% post_url 2015-04-05-common-rails-bootstrap-snippets %}) and add [oauth
installation as above](#devise-and-omniauth).

For client start with steps at [2015-11-26-angular-and-ruby-on-rails](
{{ site.baseurl }} {% post_url 2015-11-26-angular-and-ruby-on-rails %}).


We will use [md-dialog](https://material.angularjs.org/latest/demo/dialog) to
show signup buttons or login form in another
[tab](https://material.angularjs.org/latest/demo/tabs).

I have some problems to stay logged in immediatelly after log in and than hit
refresh (while some params in url). We need to remove those params.

~~~
cd client
bower install ng-token-auth --save

# add module dependency
sed -i src/app/index.module.coffee  -e "/]/i \  'ng-token-auth',"

# inject dependencies
sed -i src/app/components/navbar/navbar.directive.coffee -e ":a;N;\$!ba;s/    NavbarController.*return/$(sed ':a;N;$!ba;s/\n/ћ/g;s:\\:\\\\:g;s:/:\\/:g;' <<'MY_TEXT'
    NavbarController = (moment, $mdDialog, $auth, $rootScope) ->
      'ngInject'
      dialog = null
      vm = this
      # "vm.creation" is avaible by directive option "bindToController: true"
      vm.relativeDate = moment(vm.creationDate).fromNow()

      vm.showLoginDialog = (ev) ->
        dialog = $mdDialog.show
          controller: 'LoginController'
          controllerAs: 'vm'
          templateUrl: 'app/login/login.html'
          parent: angular.element(document.body)
          targetEvent: ev
          clickOutsideToClose: true

      vm.signOut = ->
        $auth.signOut()

      $rootScope.$on 'auth:login-success', (ev, user) ->
        $mdDialog.hide dialog
      return
MY_TEXT
)/g;s/ћ/\n/g"

# add links
sed -i src/app/components/navbar/navbar.html \
-e '/Contact/a \
    <md-button href="#" ng-show="!$root.user.id" ng-click="vm.showLoginDialog()">Try For Free</md-button>\
    <div ng-show="!!$root.user.id">Hello { { $root.user.email }}</div>\
    <md-button href="#" ng-show="!!$root.user.id" ng-click="vm.signOut()">Sign Out</md-button>'

# config routes since default /api/auth/facebook does not match rails /auth/:provider
sed -i src/app/index.config.coffee \
-e 's/config (/config ($authProvider, /' \
-e '$a\
    $authProvider.configure\
      apiUrl: "/"\
      authProviderPaths:\
        google:   "/auth/google_oauth2"\
        facebook: "/auth/facebook"'
mkdir src/app/login
cat > src/app/login/login.html <<\HERE_DOC
<md-dialog aria-label="login" ng-cloak>
  <md-toolbar>
    <div class="md-toolbar-tools">
      <h2>Try for free!</h2>
    </div>
  </md-toolbar>
  <md-dialog-content>
    <md-tabs md-selected="vm.tabsData.selectedIndex" md-align-tabs="bottom"
    md-dynamic-height>
      <md-tab label="defalt">
        <md-content class="md-padding">
          <md-button ng-click="$root.authenticate('google')">Sign in with Gmail</md-button>
          <md-button ng-click="$root.authenticate('facebook')">Sign in with Facebook</md-button>
          <md-button ng-click="vm.tabsData.selectedIndex=1">
            Sign in with email
          </md-button>
        </md-content>
      </md-tab>

      <md-tab label="Email">
        <md-content class="md-padding">
          <form ng-submit="vm.submitLogin(vm.login, loginForm)" name="loginForm" role="form">
            <md-input-container class="md-block">
              <label>Email</label>
              <input ng-model="vm.login.email" ng-required="true" name="email" placeholder="Email" type="email">
              <div ng-messages="loginForm.email.$error">
                <div ng-message="required">Email is required.</div>
                <div ng-message="server">
                  { { vm.serverErrors.login.email }}.
                </div>
              </div>
            </md-input-container>
            <md-input-container class="md-block">
              <label>Password</label>
              <input ng-model="vm.login.password" name="password"
              placeholder="Password" type="password" ng-required="true">
              <div ng-messages="loginForm.password.$error">
                <div ng-message="required">Password is required.</div>
                <div ng-message="server">
                  { { vm.serverErrors.login.join(', ') }}.
                </div>
              </div>
            </md-input-container>
            <md-button type="submit" class="md-raised md-primary">Sign in</md-button>
            or
            <md-button ng-click="vm.tabsData.selectedIndex=2">
              Register with email
            </md-button>
          </form>
        </md-content>
      </md-tab>

      <md-tab label="Registration">
        <md-content>
          <form name="registrationForm"
            ng-submit="vm.handleRegBtnClick(vm.registration, registrationForm)" role="form">
            <md-input-container class="md-block">
              <md-icon>email</md-icon>
              <input ng-model="vm.registration.email" type="email"
              placeholder="Email (required)" ng-required="true" name="email"
              ng-email="true">
              <div ng-messages="registrationForm.email.$error">
                <div ng-message="required">Email is required.</div>
                <div  ng-message="email">Must look like email.</div>
                <div ng-message="server">
                  { { vm.serverErrors.registration.email.join(', ') }}.
                </div>
              </div>
            </md-input-container>
            <md-input-container md-no-float class="md-block">
              <input ng-model="vm.registration.password" type="password"
              placeholder="password" ng-required="true" ng-minlength=6
              ng-maxlength=20 name="password">
              <div ng-messages="registrationForm.password.$error">
                <div ng-message="required">Password is required.</div>
                <div ng-message="minlength,maxlength">Password should be between 6 and 20
                  chars.</div>
                <div ng-message="server">
                  { { vm.serverErrors.registration.password.join(', ') }}.
                </div>
              </div>
            </md-input-container>
            <md-input-container md-no-float class="md-block">
              <input ng-model="vm.registration.password_confirmation"
              type="password" placeholder="Password confirmation"
              name="password_confirmation" ng-required="true">
              <div ng-messages="registrationForm.password_confirmation.$error">
                <div ng-message="required">Password confirmation is required.</div>
                <div ng-message="password_match">Passwords don't match.</div>
              </div>
              <div
            ng-messages="vm.serverErrors.registration.password_confirmation">
                <div>{ { vm.serverErrors.registration.password_confirmation.join(', ') }}</div>
              </div>
            </md-input-container>

            <md-button type="submit" class="md-raised md-primary"
            >Register</md-button>
          </form>
        </md-content>
      </md-tab>
    </md-tabs>
  </md-dialog-content>
</md-dialog>
HERE_DOC

cat > src/app/login/login.controller.coffee <<\HERE_DOC
angular.module 'client'
  .controller 'LoginController', ($mdDialog, $auth, $log, $scope) ->
    'ngInject'
    vm = this
    vm.login = {}
    vm.registration = {}
    # vm.registration =
    #   email: 'asd@asd.asd'
    #   password: 'asdasd'
    #   password_confirmation: 'asdasd'
    # vm.login =
    #   email: 'asd@asd.asd'
    #   password: 'asdasd'

    vm.tabsData = {
      selectedIndex: 0,
    }

    vm.submitLogin = (login, loginForm) ->
      handleError = (resp) ->
        loginForm.password.$setValidity('server',false)
        $scope.vm.serverErrors =
          login: resp.errors
        $log.debug loginController: 'submitLogin handleError', resp_errors: resp.errors
      $auth.submitLogin(login)
        .then (resp) ->
          $log.debug  loginController: 'submitLogin then', resp: resp
        .catch handleError
      return

    vm.handleRegBtnClick = (registration, registrationForm) ->
      handleError = (resp) ->
        for field of resp.data.errors
          if registrationForm[field]
            registrationForm[field].$setValidity('server',false)
        $scope.vm.serverErrors =
          registration: resp.data.errors
        $log.debug 'handleError'
      $auth.submitRegistration(registration)
        .then (resp) ->
          $auth.submitLogin
            email: $scope.vm.registration.email
            password: $scope.vm.registration.password
          $log.debug 'submitRegistration then'
        .catch handleError
    return
HERE_DOC
~~~

When you want to override default initial behavior, for example create another
table that will hold identities, you need to override a lot of things
[#23](https://github.com/lynndylanhurley/devise_token_auth/issues/23)
[#453](https://github.com/lynndylanhurley/devise_token_auth/pull/453) Another
approach (plan B) is to put your business logic in another model let say,
`participant` thas has many `users`. This has a problem of assigning oauth
account to the current email-user participant (we need to pass current
email-user token to oauth session so we know that we can connect those two users
to the same participant). Also adds new unnecessary model.

Easiest solution is to add uniqueness to email field and rescue with signing in.
Then we just repeat whole proccess from `omniauth_success` from
devise_token_auth gem.
Similar approach is in this opensource rails angular app
[manshar](https://github.com/manshar/manshar/pull/177/files)

~~~
rails g migration add_uniq_index_to_users
sed -i db/migrate/*add_uniq_index_to_users* -e '/change/a\
    remove_index :users, :email\
    add_index :users, :email, unique: true'
rake db:migrate

sed -i config/routes.rb -e "s^\(mount_devise_token_auth_for.*$\)^\1, controllers: {\n\
    omniauth_callbacks: 'users/omniauth_callbacks',\n  }^"
mkdir app/controllers/users
cat > app/controllers/users/omniauth_callbacks_controller.rb <<\HERE_DOC
module Users
  class OmniauthCallbacksController <
    DeviseTokenAuth::OmniauthCallbacksController

    rescue_from ActiveRecord::RecordNotUnique,
                with: :user_already_registered_with_this_email

    def user_already_registered_with_this_email
      # repeat whole omniauth_success method for this user
      @resource = User.find_by_email(@resource.email)
      # get_resource_from_auth_hash
      create_token_info
      set_token_on_resource
      create_auth_params

      if resource_class.devise_modules.include?(:confirmable)
        # don't send confirmation email!!!
        @resource.skip_confirmation!
      end

      sign_in(:user, @resource, store: false, bypass: false)

      unless @resource.image.present?
        @resource.image = auth_hash['info']['image']
      end
      @resource.save!

      render_data_or_redirect(
        'deliverCredentials',
        @auth_params.as_json,
        @resource.as_json
      )
    end
  end
end
HERE_DOC
~~~

Also override session and password controller to use
`email` field instead of `uid` field. That way oauth users can use email login
and email users can use oauth login without rescue exceptions (only first time)


Some links for Ionic authentication:

* Ionic does not like ipCookies [#222](https://github.com/lynndylanhurley/ng-token-auth/issues/222) [#90](https://github.com/lynndylanhurley/ng-token-auth/issues/90)
* [ionic-rails-sample](https://github.com/jwako/ionic_rails_sample)
* [ionic
ng-token-auth](https://github.com/search?q=ionic+ng-token-auth&ref=reposearch&type=Code&utf8=%E2%9C%93)
* [ionic rails](https://github.com/search?utf8=%E2%9C%93&q=ionic+rails)

## Angular-devise

IMPORTANT:

For angular_devise, path could be default `users/sign_in.json` (no need to place
it on the root).
In order to send headers and cookies, you need to add
[$httpProvider.defaults.withCredentials =
true](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS#Requests_with_credentials)
to the `index.config.coffee`.

If protect_from_forgery is enabled, you need to pass `XSRF-TOKEN` token as
cookie, and it will be returned later. We will read from that cookie (not from
hidden input fields as rails does).

Sometime rails responds multiple times, and last cookie is used.

<del>Note that `Set-Cookie` is only of GET request</del> Set-cookie could be
missing if you use `protect_from_forgery with: :null_session`. Best way is to
always rise exception, so you know when xsrf happens.

Note that cookies are stored per domain. In ajax, if you request two different
domains a.local and b.local, they will receive different sessions cookies (in
rails for example `session[:customer_id]` will show different values)
Chrome Developer Tools hides cookies for other domains...
I do not know how to clear cookies for other domain since I can not see them in
Developer Tools Resources...
So it is imporant to have same AuthProvider loginPath and logoutPath (login
at a.local and logout at b.local will not work).

Protect from forgery is for all requests except HEAD and GET
[link](http://api.rubyonrails.org/classes/ActionController/RequestForgeryProtection/ClassMethods.html).
[link2](http://api.rubyonrails.org/classes/ActionController/RequestForgeryProtection.html)

It is important if you `Set-Cookie` on before OR after action. If it is
`after_action` than if request is not authorized (`before_action
:authenticate_user!`) than that after_action will not be done, so NO XSRF-TOKEN
will be set. So better is to use `before_action`.

`request.xhr?` is strange. It returns nil for angular requests. Also, when there
are not cookies and login POST is sent, it passes `before_action
:set_csrf_cookie_for_ng, if: -> { request.xhr? }` but `puts request.xhr?
'XHR=true' : 'XHR=nil'` shows nil.

Example application
[angular-devise-ng-token-auth](https://github.com/duleorlovic/angular-devise-ng-token-auth)

~~~
# app/controllers/application_controller.rb

  protect_from_forgery with: :exception

  # http://stackoverflow.com/questions/14734243/rails-csrf-protection-angular-js-protect-from-forgery-makes-me-to-log-out-on
  after_action :set_csrf_cookie_for_ng # , if: -> { request.xhr? }

  def set_csrf_cookie_for_ng
    cookies['XSRF-TOKEN'] = form_authenticity_token if protect_against_forgery?
  end

  rescue_from ActionController::InvalidAuthenticityToken do |exception|
    respond_to do |format|
      format.html { raise exception }
      format.json do
        set_csrf_cookie_for_ng
        render json: { error: 'Invalid authenticity token' }, status: :unprocessable_entity
      end
    end
  end

protected

  def verified_request?
    super || valid_authenticity_token?(session, cookies['XSRF-TOKEN'])
  end
~~~

Here is example login controller for ionic

~~~
bower install --save angular-devise

# www/index.html
    <!-- Devise https://github.com/cloudspace/angular_devise -->
    <script src="lib/AngularDevise/lib/devise-min.js"></script>

# www/js/app.js
angular.module('starter', ['ionic', 'Devise'])

# www/js/app.config.cofee
angular.module 'starter'
  .config (AuthProvider, $httpProvider, CONSTANT, AuthInterceptProvider) ->
    AuthProvider.loginPath CONSTANT.SERVER_URL + '/users/sign_in.json'
    AuthProvider.logoutPath CONSTANT.SERVER_URL + '/users/sign_out.json'
    $httpProvider.defaults.withCredentials = true
    $httpProvider.interceptors.unshift 'csrfInterceptor'
    AuthInterceptProvider.interceptAuth true
    console.log 'config'

# www/js/interceptors/csrf.interceptor.coffee
# http://stackoverflow.com/questions/14734243/rails-csrf-protection-angular-js-protect-from-forgery-makes-me-to-log-out-on
window.csrfRepeated = false
angular.module 'starter'
  .factory 'csrfInterceptor', ($q, $injector) ->
    responseError: (rejection) ->
      if rejection.status == 422 &&
      rejection.data.error == 'Invalid authenticity token'
        console.log "CSRF error so try again and only one time"
        if ! window.csrfRepeated
          window.csrfRepeated = true
          deferred = $q.defer()

          successCallback = (resp) ->
            deferred.resolve(resp)
          errorCallback = (resp) ->
            deferred.reject(resp)

          $http = $http || $injector.get('$http')
          $http(rejection.config).then(successCallback, errorCallback)
          return deferred.promise

      $q.reject(rejection)

# www/js/login/login.jade
ion-view(view-title="Sign In")
  ion-content
    .list
      form(ng-submit='vm.handleSubmitLogin(vm.login)')
        label.item.item-input.item-stacked-label
          span.input-label Username
          input(type="text" placeholder="Your username"
          ng-model="vm.login.email")
        label.item.item-input.item-stacked-label
          span.input-label Password
          input(type="password" placeholder="Your password"
          ng-model="vm.login.password")
        .padding
          button.button.button-block.button-positive Sign In

# www/js/login/login.controller.coffee
angular.module 'starter'
  .controller 'LoginController', (Auth, $state) ->
    vm = this
    vm.login =
      email: 'asd@asd.asd'
      password: 'asdfasdf'

    init = ->
      Auth.currentUser().then(
        (user) ->
          console.log user
          $state.go 'tab.dashboard'
        (error) ->
          console.log error
      )

    vm.handleSubmitLogin = (login) ->
      Auth.login(login).then(
        (user) ->
          console.log user
          $state.go 'tab.dashboard'
        (error) ->
          console.log error
      )

    init()
    return


# www/js/app.router.coffee
angular.module 'starter'
  .config ($stateProvider, $urlRouterProvider) ->
    $stateProvider
      .state 'login',
        url: '/login'
        templateUrl: 'jade_build/js/login/login.html'
        controller: 'LoginController'
        controllerAs: 'vm'
      .state 'tab',
        url: '/tab'
        abstract: true
        templateUrl: 'jade_build/js/tabs/tabs.html'
        controller: 'TabsController'
      .state 'tab.dashboard',
        url: '/dashboard'
        views:
          'tab-dashboard':
            templateUrl: 'jade_build/js/dashboard/dashboard.html'
            controller: 'DashboardController'
            controllerAs: 'vm'
      .state 'tab.account',
        url: '/account'
        views:
          'tab-account':
            templateUrl: 'jade_build/js/account/account.html'
            controller: 'AccountController'
            controllerAs: 'vm'

    $urlRouterProvider.otherwise '/login'

# www/js/app.run.coffee
angular.module 'starter'
  .run ($rootScope, NotifyService, $state) ->
    $rootScope.$on 'devise:login', (event, currentUser) ->
      console.log 'devise:login'

    $rootScope.$on 'devise:new-session', (event, currentUser) ->
      console.log 'devise:new-session user logs in with Auth.login'

    $rootScope.$on 'devise:unauthorized', (event, xhr, deferred) ->
      NotifyService.toast "Unauthorized. Please log in again"
      $state.go 'login'

    return

# www/js/tabs/tabs.controller.coffee
angular.module 'starter'
  .controller 'TabsController', (Auth, $state, $rootScope) ->
    init = ->
      Auth.currentUser().then(
        (user) ->
          $rootScope.user = user
        (error) ->
          console.log error
          $state.go 'login'
      )
    init()
    console.log 'TabsController'
    return

~~~

# Resend confirmation email on login

When user has not confirmed email and `config.allow_unconfirmed_access_for =
3.days` has expired, and when he login there will be an error:

> Failed to login because A confirmation email was sent to your account at
> asd@asd.asd. You must follow the instructions in the email before your account
> can be activated

We can resend confirmation in this failed login attempt (don't resend email in
case `allow_unconfirmed_access_for` is not set or zero and you automatically log
in user after registration). You can also check if confirmation is send more
than 1.day ago and update `@resource.confirmation_sent_at = Time.now.utc -
Devise.allow_unconfirmed_access_for` (this substraction is needed because
someone can try to login right after confirmation email is send).

~~~
# config/routes.rb
#   mount_devise_token_auth_for( 'User', controllers: { sessions: 'users/sessions' })
# config/initializers/devise_token_auth.rb define url or use from secrets
#  config.default_confirm_success_url = 'http://localhost:9000'
#  config.default_confirm_success_url = \
#    "http://#{Rails.application.secrets.default_url['host']}" \
#    ":#{Rails.application.secrets.default_url['port']}"

cat > app/controllers/users/sessions_controller.rb <<\HERE_DOC
module Users
  class SessionsController < DeviseTokenAuth::SessionsController
    def render_create_error_not_confirmed
      @redirect_url = DeviseTokenAuth.default_confirm_success_url
      @resource.send_confirmation_instructions(
        redirect_url: @redirect_url,
      )
      super
    end
  end
end
HERE_DOC
~~~

Problem with `ng-token` is that is returns user object with snake
`user.first_name` instead of camelCase `user.firstName` as it is for Rails
Resources

# Sign in after user click on confirmation link

~~~
# app/controllers/confirmations_controller.rb
class ConfirmationsController < Devise::ConfirmationsController
  def show
    super
    sign_in resource if resource.confirmed?
  end
end
~~~

~~~
# config/routes.rb
  devise_for :users, controllers: {
    confirmations: :confirmations
  }
~~~

# Admin sign in as another user

If admin wants to become some other user `login_as` he can use `sign_in(:user,
@user, { :bypass => true })`.

~~~
# app/views/layouts/application.html.erb
<% if Rails.env.development? %>
  <small>
    only_on_development
    <% User.first(10).each do |user| %>
      <%= link_to user.email, sign_in_as_path(user_id: user.id) %>
    <% end %>
  </small>
<% end %>

# config/routes.rb
  get 'sign_in_as', to: 'application#sign_in_as'

# app/controllers/application_controller.rb
  before_action :authenticate_user!, except: [:sign_in_as]
  def sign_in_as
    return unless Rails.env.development?
    user = User.find params[:user_id]
    request.env['devise.skip_trackable'] = true
    sign_in :user, user, byepass: true
    redirect_to root_path
  end
~~~

With ng-token is similar
~~~
def sign_in_as
  @resource = @user
  # copy original code
  # https://github.com/lynndylanhurley/devise_token_auth/blob/master/app/controllers/devise_token_auth/sessions_controller.rb#L33
  # create client id
  @client_id = SecureRandom.urlsafe_base64(nil, false)
  @token     = SecureRandom.urlsafe_base64(nil, false)

  @resource.tokens[@client_id] = {
    token: BCrypt::Password.create(@token),
    expiry: (Time.zone.now + DeviseTokenAuth.token_lifespan).to_i
  }
  @resource.save

  sign_in(:user, @resource, store: false, bypass: false)
  render json: @user
end
~~~

just note that user need to be `user.active_for_authentication?`. In seed you
need to have `user.skip_confirmation!` since you will not be able to log in as.

# Serialization

Devise will show only id, email, create_at and updated_at. To add more fields
override user as_json

~~~
# app/models/user.rb
  def as_json(options={})
    r = super(options)
    r["domains"] = 'trk.in.rs'
    r
  end
~~~

# Redirection after sign in

Stored location for resource is usually kept in `session[:user_return_to]`. Yo
enable it you have to add before action
https://github.com/plataformatec/devise/wiki/How-To:-Redirect-back-to-current-page-after-sign-in,-sign-out,-sign-up,-update
~~~
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action :_store_user_location!, if: :_storable_location?
  # The callback which stores the current location must be added before you authenticate the user
  # as `authenticate_user!` (or whatever your resource is) will halt the filter chain and redirect
  # before the location can be stored.

  # Its important that the location is NOT stored if:
  # - The request method is not GET (non idempotent)
  # - The request is handled by a Devise controller such as Devise::SessionsController as that could cause an 
  #    infinite redirect loop.
  # - The request is an Ajax request as this can lead to very unexpected behaviour.
  def _storable_location?
    request.get? && is_navigational_format? && !devise_controller? && !request.xhr?
  end

  def _store_user_location!
    # :user is the scope we are authenticating
    # this path can be fetched with path = stored_location_for(resource)
    store_location_for(:user, request.path)
  end

  # https://www.rubydoc.info/github/plataformatec/devise/Devise/Controllers/Helpers:after_sign_in_path_for
  # override in ApplicationController since if you override in
  # SessionsController and it wont be used in RegistrationController (for example
  # user already signed in and needs to be redirected)
  def after_sign_in_path_for(resource)
    # we need save stored location in variable since it is not indenpotent
    # if we call twice stored_location_for(:user) than second will be nil
    # so do not use stored_location_for in views or anywhere else... you can use
    # session[:user_return_to] if you really need
    redirect_url = stored_location_for(resource)
    return redirect_url if redirect_url.present?
    # calculate default paths for user
  end


# app/controllers/users/registrations_controller.rb
note that inside Devise controllers and also in application controller you can
use `stored_location_for :user`

~~~

If you need to store landing page for future analitics or you need to redirect
users after specific actions, you can implement your own

~~~
# app/controllers/application_controller.rb
  before_action :set_back_variable_into_session_if_exists
  before_action :save_landing_page, unless: :current_user

  def set_back_variable_into_session_if_exists
    session[:_back] = params[:_back] if params[:_back].present?
    # facebook login overwrite params and querystring so we need to use request.env to get _back param
    # https://github.com/intridea/omniauth-oauth2/issues/28
    session[:_back] = request.env['omniauth.params'].try(:[],'_back') if request.env['omniauth.params'].try(:[],'_back').present?
  end
  def save_landing_page
    if ! session[:landing_page].present?
      session[:landing_page] = request.fullpath
    end
  end

  # this overrides devise default
  def after_sign_in_path_for(resource)
    view_context.after_log_in_path_for(resource)
  end

# app/helpets/application_helper.rb
  def after_log_in_path_for(user)
    track_sign_in(user) if user.customer?
    origin = request.env['omniauth.origin'] unless [new_user_session_url, signup_jobseeker_url, signup_employer_url, user_omniauth_authorize_url(:linkedin), user_omniauth_authorize_url(:facebook)].include? "#{request.env['omniauth.origin']}".split('?').first
    session[:_back] || origin || controller.stored_location_for(user) || if user.user?
      job_seeker_first_step_path
    elsif user.customer?
      employer_after_signup_path(:subscribe)
    elsif user.admin?
      users_path
    elsif user.superadmin?
      users_path
    else
      root_path
    end
  end
~~~

# Testing

You can use [test login helpers]( {{ site.baseurl }}
{% post_url 2015-11-09-rails-testing %}
#login-helper)
In `spec/rails_helper.rb` uncomment line that require all `spec/support/**/*.rb`
files and create that login helper.

And you can use in your acceptance tests `login_as user` where `user =
User.create email: 'asd@.asd.asd', password: 'asdasd'`

To suppress error in tests `ERROR -- omniauth: (facebook) Authentication
failure! invalid_credentials encountered.  ` you can use this helper

~~~
  def silence_omniauth
    previous_logger = OmniAuth.config.logger
    OmniAuth.config.logger = Logger.new("/dev/null")
    yield
  ensure
    OmniAuth.config.logger = previous_logger
  end

  silence_omniauth { click_link 'Sign in using Facebook' }
~~~


# HTTP Basic auth

You can use basic http auth but you should `config.force_ssl = true` for
production env.

~~~
# app/controllers/admin/admin_controller.rb
module Admin
  class AdminController < ApplicationController
    http_basic_authenticate_with(
      name: Rails.application.secrets.admin_username,
      password: Rails.application.secrets.admin_password
    )
  end
end

# app/controllers/admin/users_controller.rb
module Admin
  class UsersController < Admin::AdminController
  end
end
~~~

If you want different password for different request type

~~~
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
  skip_before_action :verify_authenticity_token, if: :json_request?

  before_action :authenticate

  def authenticate
    return true if Rails.env.test?
    authenticate_or_request_with_http_basic do |username, password|
      return true if request.format.html? && username == Rails.application.secrets.admin_username && password == Rails.application.secrets.admin_password
      return true if request.format.json? && username == Rails.application.secrets.admin_json_username && password == Rails.application.secrets.admin_json_password
      false
    end
  end
end
~~~

# Sorcery

<https://github.com/Sorcery/sorcery>

# Devise send email in background

Read this gem
https://github.com/plataformatec/devise/wiki/How-To:-Send-devise-emails-in-background-(Resque,-Sidekiq-and-Delayed::Job)
or you can add manually (just put lines in User model, so it overrides devise's)

~~~
# app/models/user.rb
  devise :database_authenticatable, :device_async

# config/initializers/devise.rb
# Register devise-async model in Devise
Devise.add_module(:devise_async, model: 'devise_async')

# app/models/concerns/devise_async.rb
module DeviseAsync
  extend ActiveSupport::Concern

  included do
    # This method overwrites devise's own `send_devise_notification`
    # message = devise_mailer.send(notification, self, *args)
    # message.deliver_now
    # also need to fetch user in MyDeviseMailer
    # protected is required, or ActionView::Template::Error: undefined method `main_app'
    protected
    def send_devise_notification(notification, *args)
      message = devise_mailer.send(notification, self, *args)
      message.deliver_later
    end
  end
end
~~~

or oneliner instead of all above

~~~
# app/models/user.rb
  # https://github.com/plataformatec/devise#activejob-integration
  def send_devise_notification(notification, *args)
    devise_mailer.send(notification, self, *args).deliver_later
  end
~~~

Look below if you have problem with serilization

# Custom devise mailer

https://github.com/plataformatec/devise/wiki/How-To:-Use-custom-mailer
I override becaus I wante dto use delive_later but have a problem with
serilization (Neo4j objects)

~~~
ActiveJob::SerializationError: Unsupported argument type: User
~~~

so I send `id` instead of `self`

~~~
# app/models/user.rb
  # This method overwrites devise's own `send_devise_notification`
  # message = devise_mailer.send(notification, self, *args)
  # message.deliver_now
  # also need to fetch user in MyDeviseMailer
  # protected is required, or ActionView::Template::Error: undefined method `main_app'

  protected

  def send_devise_notification(notification, *args)
    message = devise_mailer.send(notification, id, *args)
    message.deliver_later
  end
~~~

So My devise mailer just call super with same arguments

~~~
# app/mailers/my_devise_mailer.rb
class MyDeviseMailer < Devise::Mailer
  helper :application # gives access to all helpers defined within `application_helper`.
  include Devise::Controllers::UrlHelpers # Optional. eg. `confirmation_url`
  default template_path: 'devise/mailer' # to make sure that your mailer uses the devise views

  def confirmation_instructions(record_id, token, opts = {})
    record = User.find record_id
    super record, token, opts
  end

  def reset_password_instructions(record_id, token, opts = {})
    record = User.find record_id
    super record, token, opts
  end

  def unlock_instructions(record_id, token, opts = {})
    record = User.find record_id
    super record, token, opts
  end

  def email_changed(record_id, opts = {})
    record = User.find record_id
    super record, opts
  end

  def password_change(record_id, opts = {})
    record = User.find record_id
    super record, opts
  end
end
~~~

# Session expired

For ajax requests when session is expired we need to redirect
https://coderwall.com/p/zxe6dq/devise-redirect-ajax-request-when-session-timed-out-timeoutable-module

# Detect if already signed in to social sites

http://www.tomanthony.co.uk/blog/detect-visitor-social-networks/
