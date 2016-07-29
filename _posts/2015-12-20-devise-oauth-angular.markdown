---
layout: post
title: From simple Devise to Oauth on Angular
tags: angular devise oauth
---

# Devise

Basic example application with [Devise](https://github.com/plataformatec/devise) default signup/login views.

~~~
echo "gem 'devise'" >> Gemfile
bundle
rails generate devise:install
git add . && git commit -m "rails g devise:install"
rails g devise User
rake db:migrate
git add . && git commit -m "rails g devise user"

# optional
# rails g devise:views && git add . && git commit -m "rails g devise:views"
sed -i app/views/layouts/application.html.erb -e '/<body>/a \
  <% if current_user %>\
    <strong><%= current_user.email %></strong>\
    <a href="<%= destroy_user_session_path %>" data-method="delete">Sign out<a>\
  <% else %>\
    <a href="<%= new_user_registration_path %>">Sign up</a>\
    <a href="<%= new_user_session_path %>">Log in</a>\
  <% end %>'
git add . && git commit -m "Adding login/logout header in layout"
~~~

Read *config/initializers/devise.rb* about default configuration.

You can use `before_action :authenticate_user!` in controllers that will
redirect to `/users/sign_in`. You need to [set up
emails]({{ site.baseurl }}{% post_url 2015-04-05-common-rails-bootstrap-snippets %})
to actually receive registration email.


# Devise and Omniauth

Read [wiki](https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview) to add facebook and google authentication. It's easy installation.

~~~
cat >> Gemfile <<HERE_DOC
gem 'omniauth-facebook'
gem 'omniauth-google-oauth2', '0.2.5'
gem "omniauth-oauth2", '1.2.0' # skip_jwt issue
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

Customization:

~~~
sed -i app/views/layouts/application.html.erb -e '/<body>/a \
<%= link_to "Sign in with Facebook", user_omniauth_authorize_path(:facebook) %>'

sed -i app/models/user.rb -e '/end/i \
\
  def self.from_omniauth(auth)\
    user = find_by(email: auth.info.email)\
    return user if user\
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
# no need for , :omniauth_providers => [:facebook]

sed -i config/routes.rb -e '/devise_for/c \
  devise_for :users,\
             controllers: {\
               omniauth_callbacks: "omniauth_callbacks",\
             }'

cat > app/controllers/omniauth_callbacks_controller.rb << HERE_DOC
class OmniauthCallbacksController < Devise::OmniauthCallbacksController
  # https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview
  # data is in request.env["omniauth.auth"]
  [:facebook, :google_oauth2, :twitter].each do |provider| # yaho_oauth2
    define_method provider do
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
end
HERE_DOC
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


# Facebook app

1. Create app on [developers.facebook.com](http://developers.facebook.com). By default
   fb app is in development mode and can only be used by app admins, developers
   and testers.

1. Add Contact Email on
   https://developers.facebook.com/apps/FACEBOOK_APP_ID/settings/ and toggle
   switch on https://developers.facebook.com/apps/FACEBOOK_APP_ID/review-status/

1. Add https://developers.facebook.com/apps/FACEBOOK_APP_ID/settings/advanced
   *Valid OAuth redirect URIs* for all links that are redirected. You can find
   the link on facebook error page url
   `www.facebook.com/dialog/oauth?client_id=...&redirect_uri=...`. You can add
   just domain (`/omniauth/facebook/callback` is not needed)
   Note that this are server url (not frontend url):
   `http://localhost.dev:3003`,
   `http://localhost:9000` and for https also
   `https://localhost.dev:3003`

Changes are visible immediatelly.
More on blog facebook share buttons.

# Google console

Create a project in google console and enable *Google+ API*. Create *OAuth 2.0
client ID* (or edit exists one clicking on its name) and set *Authorized
redirect URIs* to all urls that will be used (not just domain like for facebook,
we need whole url path), like

* `http://localhost:3000/users/auth/google_oauth2/callback` this is default
  `devise_for :users`
* `http://localhost:9000/omniauth/google_oauth2/callback` and also for https
* `https://localhost.dev/omniauth/google_oauth2/callback` 
* also fill the *Authorized JavaScript origins* with domains and port like
`http://localhost.dev`.

Dont forget to save **Changes needs 5 min to propagate**

# Twitter app

On <https://apps.twitter.com/> you can create application.
`app/{{APP_ID}}/keys` will give you TWITTER_API_KEY and TWITTER_API_SECRET

# Angular ng-token-auth demo

Angular [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth) is used with [devise-token-auth](https://github.com/lynndylanhurley/devise_token_auth).
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

For server side you can see [devise_token_auth_demo compare](https://github.com/lynndylanhurley/devise_token_auth_demo/compare/dbf0a69b1e67e34862906c3d24747bc98ffff815...master)

~~~
cd devise_token_auth_demo
vi config/database.yml # change to your username
echo -e 'gem "letter_opener", :group => :development' >> Gemfile
sed -i '/end$/i \  config.action_mailer.delivery_method = :letter_opener' config/environments/development.rb

# hosts should be the same as in ng-token-auth/test/config/default.html
# config/environments/development.rb redirection url
# OmniAuth.config.full_host = "http://localhost.dev:3003"

export GITHUB_KEY=asd GITHUB_SECRET=asd GOOGLE_KEY=$GOOGLE_CLIENT_ID GOOGLE_SECRET=$GOOGLE_CLIENT_SECRET FACEBOOK_KEY=$FACEBOOK_KEY FACEBOOK_SECRET=$FACEBOOK_SECRET
rails s
~~~

# Angular and ng-token-auth from scratch with Yeoman

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

# change route from auth to api/v1/auth
sed -i config/routes.rb -e 's:\bauth\b:api/v1/auth:'

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

Perform some [common bootstrap stuff for secrets and emails]({{ site.baseurl }}{% post_url 2015-04-05-common-rails-bootstrap-snippets %}) and add [oauth installation as above](#tocAnchor-1-2).

For client start with steps at [2015-11-26-angular-and-ruby-on-rails]({{ site.baseurl }}
{% post_url 2015-11-26-angular-and-ruby-on-rails %}).


We will use [md-dialog](https://material.angularjs.org/latest/demo/dialog) to show
signup buttons or login form in another
[tab](https://material.angularjs.org/latest/demo/tabs).

I have some problems to stay logged in immediatelly after log in and than hit refresh (while some params in url). We need to remove those params.

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
    <div ng-show="!!$root.user.id">Hello {{ $root.user.email }}</div>\
    <md-button href="#" ng-show="!!$root.user.id" ng-click="vm.signOut()">Sign Out</md-button>'

# config routes since default /api/auth/facebook does not match rails /auth/:provider
sed -i src/app/index.config.coffee \
-e 's/config (/config ($authProvider, /' \
-e '$a\
    $authProvider.configure\
      apiUrl: "api/v1"\
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
                  {{ vm.serverErrors.login.email }}.
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
                  {{ vm.serverErrors.login.join(', ') }}.
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
                <div ng-message="email">Must look like email.</div>
                <div ng-message="server">
                  {{ vm.serverErrors.registration.email.join(', ') }}.
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
                  {{ vm.serverErrors.registration.password.join(', ') }}.
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
                <div>{{ vm.serverErrors.registration.password_confirmation.join(', ') }}</div>
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

# Devise-angular


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

# Admin sign in as another user

If admin wants to become some other user he can use `sign_in(:user, @user, {
:bypass => true })`.  With ng-token is similar

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


# Testing

