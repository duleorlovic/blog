---
layout: post
title: Api design
---

# REST and JSON

Just to note that [REST
api](https://en.wikipedia.org/wiki/Representational_state_transfer) means:
Statelessness (no data between requests), Resource identification per request
(update only one row, get could have more rows), Representational state transfer
(returned representation JSON is enough to identify and manipulate row). REST
enable cacheability so we can scale to large number of components.

Once API is exposed, you should not modify it, except for critical bugfixes. Use
namespace

~~~
namespace :api, default: { format: :json } do
  namespace :v1 do
    resources :expenses, only: [:index, :create, :update, :destroy]
  end
end
~~~

If you use jbuilder, than create folder `app/views/api/v1/expenses` for view and
controller `Api::V1::ExpensesController`

# JSONAPI Resources JR

[jsonapi-resources](https://github.com/cerebris/jsonapi-resources) give us
implementation of JSONAPI

~~~
rails g jsonapi:resource Api::V1::Post
rails g jsonapi:controller Api::V1::Post

namespace :api do
  namespace :v1 do
    jsonapi_resources :posts
  end
end

# config/initializers/jsonapi_resources.rb:
JSONAPI.configure do |config|
  # built in paginators are :none, :offset, :paged
  config.default_paginator = :paged
  config.default_page_size = 50
  config.maximum_page_size = 1000

  # Do this if you use UUID's instead of Integers for id's
  config.resource_key_type = :uuid
end
~~~

Each [resource](http://jsonapi-resources.com/v0.9/guide/resources.html) can be
configured with:

* `attributes` `attribute`
  * there are also `id` and `type`. Computed fields can use `@model` but also
  need to be defined with `attribute`
  * `fetchable_fields` all attributes are fetchable, and if you want to remove
  some than override method
  * `self.updatable_fields`, `self.creatable_fields` to override use class methods
  ~~~
  class ContactResource < JSONAPI::Resource
    attributes :name_first, :name_last, :full_name
    def full_name
      "#{@model.name_first}, #{@model.name_last}"
    end
    def self.updatable_fields(context)
      super - [:full_name]
    end
    # class method can be defines inside class <<
    class << self
      def creatable_fields(context)
        super - [:full_name]
      end
    end
  end
  ~~~

* `has_many`
* `filters`, `filter`


# Postman

You can use plugin
[Postman packaged
app](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en)
or [Postman REST
client](https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm?hl=en)
to send API requests.

If you want to send json request from Postman than use header `Accept:
application/json`. If server responds with json, it can set header
`Content-type: Application/json`.

Nice extension is [JSON
Formatter](https://chrome.google.com/webstore/detail/json-formatter/bcjindcccaagfpapjjmafapmmgkkhgoa)
that will render json response in readable way.


Curl commands:

[curl commands]({{ site.baseurl }} {% post_url 2016-02-01-bash %}#curl)

# POST vs PUT

When an user has one image (image is part of the user) than it is fine to do
`PUT /users/1/image` when you want to update or create image.
`PUT` should be idempotent (also `DELETE` so you can call it several times).

If user have multiple images, than do not use nested, but use `POST
/users/1/images` to create and `PUT /images/1` to update image.

# Use nouns

Only verb should be HTTP method (GET, POST, PUT, DELETE). Url should contain
only nouns that represent resource. So instead `POST /users/1/send-message` it's
better to use `POST /users/1/messages` with `Content-Type: application/json
{ "message": "Hello" }`.

If you need to send messages to multiple users, you can create new endpoint and
send data there `POST /group-messages; Content-Type: application/json; { [ {
"user" : { "id" : 1 }, "message" : "Hello 1" },...] }`

Use url params for search/filter `GET /places?lat=123&lon=123`.

For authorization use headers `POST /users/1/message; Authorization: Bearer
123`.

HTTP body can contain json data.

# Response status codes:

[w3](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)

Success:

* *200 OK* (for GET)
* *201 Created* successful POST, login
* *202 Accepted* Accepted but is being processed async
* *204 No Content* `:no_content` success delete, log out, sign out

For errors we could respond with `head :not_found` (and hide sensitive
information) but its better to send the reason and some info (`render json:
{ error: 'Some problem', error_status: 401, error_code: 1234 }, status: 401`)

* *303 See Other* when session create (login) email does not exists
* *400 Bad Request*  `:bad_request` when we have `ParameterMissing`
* *401 Unauthorized* `:unauthorized` used for wrong password on login form, or
when there is no current_user but it should exists.
  * note that it should not send `:unauthorized` when user is changing password
  and type incorect old one (he still should be logged in, and not automatically
  logged out since unauthorized request occurs).
* *403 Forbidden* `:forbidden` when current user exists but is forbidden from
accessing this data
* *404 Not Found* `:not_found` url is not valid router or resource does not
exist
* *410 Gone* data has been deleted, deactivated ...
* *422 Unprocessable Entity* `:unprocessable_entity` change password form but
current password is bad, or form validation errors `if ! @user.update() render
json: @user.errors.full_messages.join(','), status: :unprocessable_entity`, or
when creating new resource but uniqueness validation (already exists) prevents

Server errors

* `500 Internal Server Error` unexpected happened on server side
* `503 Service Unavailable` api is overloaded or in maintenance mode

Common responses:
Rails does not come with `:unauthorized` exception
https://stackoverflow.com/questions/25892194/does-rails-come-with-a-not-authorized-exception
so you can create one
```
# app/models/authorization_exception.rb
class AuthorizationException < Exception
end
```
and you can use for example in `login`
```
# app/controllers/api/v2/sessions_controller.rb
  def subscriber_login
    # this will raise ActiveRecord::RecordNotFound
    subscriber = Subscriber.find_by! username: params[:username]
    # instead of bang we could use find_by and manually raise, result is the same
    raise ActiveRecord::RecordNotFound.new("Couldn't find Subscriber") unless subscriber
    raise AuthorizationException.new('Password is not correct') unless subscriber.authenticate(params[:password])
    render json: { auth_token: JwtAuth.encode_subscriber_id(subscriber.id) }
```

and rescue from those exceptions and show json message

~~~
# app/controllers/application_controller.rb
  rescue_from JWT::DecodeError do |e|
    render json: { error_message: e.message, error_status: :bad_request}, status: :bad_request
  end

  rescue_from ActiveRecord::RecordNotFound do |e|
    render json: { error_message: e.message, error_status: :not_found }, status: :not_found
  end

  rescue_from ActiveRecord::RecordInvalid do |e|
    render json: { error_message: e.message, error_status: :unprocessable_entity }, status: :unprocessable_entity
  end

  # used with params.permit(:domain).fetch(:domain)
  rescue_from ActionController::ParameterMissing do |e|
    render json: { error_message: e.message, error_status: :bad_request }, status: :bad_request
  end

  rescue_from AuthorizationException do |e|
    render json: { error_message: e.message, error_status: :unauthorized }, status: :unauthorized
  end
~~~

In tests you can put `byebug` below `rescue_from ActiveRecord::RecordNotFound do |e|` so you can

```
Rails.application.class.parent.name.underscore # => myapp
puts exception.backtrace.select {|r| r.match Rails.application.class.parent.name.underscore }
```

Usually in case of validation error, I put only all error messages in one field
`error_message` for example :

~~~
# app/controllers/users_controller.rb

  def send_password
    alert = "Failed to send new password"
    format.json { render json: { error_message: alert, error_status: :unprocessable_entity }, status: :unprocessable_entity }
  end

  def create
    @user = User.new(user_params)
    if @user.save
      render json: @user, status: :created
    else
      render json: { error_message: @user.errors.full_messages.to_sentence },
             error_status: :unprocessable_entity
    end
  end

  def update
    unless @user.update(user_params)
      render json: { error_message: @user.errors.full_messages.to_sentence },
             error_status: :unprocessable_entity
    end
  end
~~~

In angular, I use `connectionInterceptor` to set that `data.error` in case of
other errors like: server offline.

In case of success, I use `message`

~~~
  notice = "Successfully updated"
  format.json { render json: { message: notice, data: user.to_json } }
~~~

# Generate JSON

[default ActiveModel JSON
Serializer](http://api.rubyonrails.org/classes/ActiveModel/Serializers/JSON.html)
can be customized with `render json: @phones.as_json(only: [:id], expect:
[:created_at], include: :posts)`
but for larger API you need to have centralized serializers.


# ActiveModelSerializer

To generate json you can use
[active_model_serializers](https://github.com/rails-api/active_model_serializers)
[documents](https://github.com/rails-api/active_model_serializers/tree/0-10-stable/docs)

~~~
cat >> Gemfile << HERE_DOC
# serializer for api
gem 'active_model_serializers', '~>0.10.0'
HERE_DOC
bundle

rails g serializer user
~~~

If you rely on rails convention (empty `def show;end`, implicit render json and
not specify `render @user`) than serializer will not be used. If you specify
`render json: @user` or `render json: @users` than it will use `UserSerialier`.
If you have some other structure like `render json: { user: @user }` than
serializer will not be used. You can manully call `render json: @user,
serializer: UserSerialier` or `render json: @users, each_serializer:
UserSerialier`

~~~
# app/serializers/application_serializer.rb
class ApplicationSerializer< ActiveModel::Serializer
  include Rails.application.routes.url_helpers
end
# app/serializers/user_serializer.rb
def UserSerializer < ApplicationSerializer
  attributes :id, :name
end
~~~

Associations are handled in
[reflections](http://www.rubydoc.info/gems/active_model_serializers/ActiveModel/Serializer/Reflection)

You can call `@users.active_model_serializer.new(@users, options).to_json`
For grape you can use
<https://github.com/ruby-grape/grape-active_model_serializers>

# json api

<http://jsonapi.org/>

~~~
cat >> config/initializers/active_model_serializers.rb << HERE_DOC
ActiveModelSerializers.config.adapter = :json_api
HERE_DOC
~~~

# JBuilder

jBuilder is already included in rails and is nice if you already have some
views.
https://github.com/rails/jbuilder


# API Documentation

If you use rspec then go with
[rspec_api_documentation](https://github.com/zipmark/rspec_api_documentation),
or swagger

## Test and Documentation from rspec

rspec_api_documentations/dsl and generate api docs `rake docs:generate` and
`gnome-open docs/`:
You need to put tests in `/spec/acceptance` in order to get [generate docs
working](https://github.com/zipmark/rspec_api_documentation#rake-task)
Configuration
```
# spec/support/rspec_api_documentation.rb
RspecApiDocumentation.configure do |config|
  config.docs_dir = Rails.root.join('public', 'api')
  config.format = :json
  # for json post or patch requests, request body needs to be encoded to json
  # so use this condif instead of let(:raw_post) { params.to_json }
  config.request_body_formatter = Proc.new { |params| params.to_json }

  config.curl_host = 'admin.my-domain.com'

  used_headers = ['Content-Type', 'Authentication']
  ignored_headers = ['Cookie', 'Host']
  config.curl_headers_to_filter = ignored_headers
  config.request_headers_to_include = used_headers
  config.response_headers_to_include = used_headers

  # By default examples and resources are ordered by description. Set to true
  # keep the source order.
  config.keep_source_order = true
end
```

Usage:

* `resource` synonim for describe, but you need to use `resource` if you want to
  generate docs and have
* `get`, `post`, `patch`, `delete`
  * you can validate `response_status` and `response_body` which is
  `JSON.parse(response.body)`
* `parameter` used to define params. You need `let(:param1) { 'my_email' }` in
  order to show it in example body or curl
* `example_request` is the same as calling `example` (synonim for `it`) and
`do_request`
* `send :name` to get value of `name`
* `no_doc`
* `explanation` can be inside resource or it block


If you got [415 error UNSUPPORTED_MEDIA_TYPE
](http://www.rubydoc.info/gems/jsonapi-resources/JSONAPI) than set header.
If you got `is not a valid resource` code 101, than you mispelled `type` top
level attribute (for example `users`) on [resource
object](http://jsonapi.org/format/#document-resource-objects).
If you got `"no implicit conversion of nil into Hash"` than problem is that we
need to declare `attribute` on the resource.
If you for `does not contain a key` code 109, than you need to add `:id` to both
"data" and "url" and use different name, for example this is duplication for
update resources:

~~~
  patch '/v1/organizations/:organization_id' do
    let(:organization) { create :organization, name: 'My Organization' }
    let(:organization_id) { organization.id }
    let(:id) { organization.id }
    parameter :id, 'Id of the organization.', required: true
  end
~~~


If you use parameter status than you need to disable dsl
https://github.com/zipmark/rspec_api_documentation/issues/329
https://github.com/zipmark/rspec_api_documentation/issues/215

```
# config/support/rspec_api_documentation.rb
RspecApiDocumentation.configure do |config|
  config.disable_dsl_status!
end
```

HTTP codes are standard (200 get, 201 created, 204 deleted).

You can debug with:

~~~
tail -f log/test.log
~~~

To change domain which is used in tests you can define full url in `get`,
`post`... like `get "http://#{API_DOMAN}/posts" do` (note that we need to use
http prefix so it does not add it after example.org
`http://example.org/www.api.domain/posts`)
Using `RspecApiDocumentation.configure do |config|` and
`config.curl_host = 'my.domain.com'` will change domain only on curl
example, not in test, so it will be `curl -g
"my.domain.comwww.api.domain/posts"`. Another way is to use `header
'Host', API_DOMAIN` so you do not need to use full url.
https://github.com/zipmark/rspec_api_documentation/issues/304 note that here we
do not need protocol http, we just need host domain

```
# spec/acceptance/api/v2/posts_spec.rb
require 'rspec_api_documentation/dsl'

API_DOMAIN = 'subdomain.my-domain.com'

resource 'Posts' do
  header 'Accept', 'application/json'
  header 'Authentication', :auth_token

  # use header
  header 'Host', API_DOMAIN
  # or use full http path
  get "#{API_DOMAN}/posts" do
  end

  let(:user) { create :user }
  let(:auth_token) { JwtAuth.encode_user_id user.id }
  let(:post) { create :post }
  before do
    user
    post
  end
end
```

# JWT Json web tokens

```
gem 'jwt'

t = JWT.encode( {a: 1}, 'secret')
=> "eyJhbGciOiJIUzI1NiJ9.eyJhIjoxfQ.LrlPmSL4FxrzAHJSYbKzsA997COXdYCeFKlt3zt5DIY"
>> JWT.decode t, 'secret' #=> [{"a"=>1}, {"alg"=>"HS256"}]
```

Testing application is in `~/rails/temp/rails_5.2.3/` branch `devise_jwt_token`.
You can add custom devise strategy so both devise http and jwt authentication
can work
http://blog.plataformatec.com.br/2019/01/custom-authentication-methods-with-devise/
api_token_strategy

```
# app/strategies/jwt_strategy.rb
class JwtStrategy < Warden::Strategies::Base
  def valid?
    # A copy of JwtStrategy has been removed from the module tree but is still active
    # JwtAuth.token_from_request_headers request.headers
    # so we need to copy same method here
    request.headers['Authentication'].to_s.split(' ').last
  end

  def authenticate!
    user = User.find_by(id: JwtAuth.decoded_user_id(request.headers))
    if user
      success! user
    else
      fail! 'Invalid email or password'
    end
  end
end

# app/services/jwt_auth.rb
class JwtAuth
  SECRET = Rails.application.secrets.secret_key_base

  def self.encode_user_id(user_id)
    payload = { user_id: user_id }
    JWT.encode(payload, SECRET)
  end

  def self.decoded_user_id(headers)
    token = token_from_request_headers headers
    payload = JWT.decode(token, SECRET)[0]
    payload['user_id']
  end

  def self.token_from_request_headers(headers)
    headers['Authentication'].to_s.split(' ').last
  end
end

# config/initializers/devise.rb
  config.warden do |manager|
    manager.default_strategies(scope: :user).unshift :jwt_strategy
  end

# config/initializers/warden.rb
Warden::Strategies.add(:jwt_strategy, JwtStrategy)
```

## Swagger ui grape

<https://github.com/kendrikat/grape-swagger-ui>

~~~
# Gemfile
gem 'grape-swagger'
gem 'grape-swagger-ui'


# app/api/v1/root.rb
require 'grape-swagger'
module API
  module V1
    class Root < Grape::API
      mount Events
      add_swagger_documentation
    end
  end
end

# config/initializers/assets.rb
config.assets.precompile += %w(swagger_ui.js swagger_ui.css swagger_ui_print.css swagger_ui_screen.css)
~~~

## Swagger docs

~~~
sed -i Gemfile -e '/group :development do/a  \
  # use github until it is merged https://github.com/richhollis/swagger-docs/pull/144
  gem 'swagger-docs', git: 'https://github.com/kwerle/swagger-docs'
o
~~~

Basepath is important since other json files will be looked there.

`param` first argument is: `:query`, `:path` or `:form`

~~~
  swagger_controller :customer_sessions, "Customer Sessions"

  swagger_api :create do
    summary "Customer sign in"
    notes "Sign in form"
    param :form, :email, :string, :required, :Email"
  end
~~~

To generate json, you need to run `rake swagger:docs` or overide precompile task

~~~
# lib/tasks/precompile_overrides.rake
namespace :assets do
  task :precompile do
    Rake::Task['assets:precompile'].invoke
    Rake::Task['swagger:docs'].invoke
  end
end
~~~

Usually for precompile is disabled for production, so you need to enable it in
`config/environments/production.rb`

Copy `dist` content to your `public/apidocs` and update all paths in
`public/apidocs/index.html` to match your `/apidocs`, and change js `url =
"/apidocs/api-docs.json";`

* default sort is alphabetical for path and methods (delete, get, path...). You
can provide a function for `apisSorter` `operationsSorter` but too complicated.
Order in server response is not supported.

## Appman swagger

TODO: https://www.loom.com/share/c922b8074b9443e899f388f1a19d095c
https://rubygems.org/gems/appmap_swagger

# Apipie rails

If you are using minitest there is not automatic way to generate docs, so you
can use [apipie-rails](https://github.com/Apipie/apipie-rails) for
documentation.

~~~
echo "gem 'apipie-rails'" >> Gemfile
bundle install
rails g apipie:install # this will generate config/initializers/apipie.rb and update routes
~~~

Jus add option `config.translate = false`

# Grape

If grape is used than we can rescue from all exceptions, but that needs to be
before `mount` methods and only for StandardError exceptions

~~~
# app/api/v1/root.rb
module API
  module V1
    class Root < Grape::API
      rescue_from Koala::Facebook::AuthenticationError, MyappExceptions::Base do |e|
        ExceptionNotifier.notify_exception(
                        Exception.new(e.message),
                        data: {
                          message: e.message,
                          stack: e.backtrace,
                          error: e
                        }
        )
        Rack::Response.new(["#{e.class} #{e.message}"], 500, 'Content-type' => 'text/error').finish
      end
      # now we can mount ...
      mount Users
      ....
~~~

~~~
# app/models/myapp_exceptions.rb
module MyappExceptions
  # use this base class so it is easier to rescue only that
  # grape rescue only from standard error
  class Base < StandardError
  end
  class UserNotFound < Base
  end
end
~~~

# Guide

<https://github.com/interagent/http-api-design>

# Tips

* model skill level as string, but consider using array. For example if someone
wants to cover all skill levels.

<https://scotch.io/tutorials/build-a-restful-json-api-with-rails-5-part-two>

https://medium.com/@stevenpetryk/providing-useful-error-responses-in-a-rails-api-24c004b31a2e#.lp6e0sjpf

https://github.com/tuwukee/jwt_sessions

In email search for Chris Kottom Lesson 9: Token-Based Authentication with JWTs
Accessing Google oauth to list my videos https://martinfowler.com/articles/command-line-google.html

