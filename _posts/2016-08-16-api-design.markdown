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

# JSONAPI

[jsonapi-resources or jr](https://github.com/cerebris/jsonapi-resources) give us
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

Each resource can be configured with:

* `attributes`
* `has_many`
* `filters`, `filter`

# Postman

You can use plugin
[Postman packaged
app](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en)
or [Postman REST
client](https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm?hl=en)
to send API requests.

Nice extension is [JSON
Formatter](https://chrome.google.com/webstore/detail/json-formatter/bcjindcccaagfpapjjmafapmmgkkhgoa)
that will render json response in readable way.

Curl commands:

~~~
export U=http://localhost:3000/api/v1/
curl ${U}/expenses
curl ${U}/expenses -I # or --head show only header
curl ${U}/expenses -H 'Authorization: Token token="c576f0136149a2e2d9127b3901015545"'
~~~

# Current user or me

Instead of `/users/123/settings` you can create `/me/settings` endpoint.

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
* *204 No Content* `:no_content` success update or delete, log out, sign out

For errors we could respond with `head :not_found` (and hide sensitive
information) but its better to send the reason and some info:

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
json: @user.errors.full_messages.join(','), status: :unprocessable_entity`

Server errors

* `500 Internal Server Error` unexpected happened on server side
* `503 Service Unavailable` api is overloaded or in maintenance mode

Common responses:

~~~
# app/controllers/application_controller.rb
  rescue_from ActiveRecord::RecordNotFound do
    render json: { error: "Not found" }, status: :not_found
  end

  rescue_from ActionController::ParameterMissing do
    render json: { error: "Bad requests" }, status: :bad_request
  end

  rescue_from Pundit::NotAuthorizedError do
    render json: { error: "Unauthorized" }, status: :unauthorized
  end
~~~

Usually in case of validation error, I put only all error messages in one field
`error` for example :

~~~
  alert = "Failed to send new password"
  format.json { render json: { error: alert }, status: :unprocessable_entity }
~~~

In angular, I use `connectionInterceptor` to set that `data.error` in case of
other errors like: server offline.

In case of success, I use `message`

~~~
  message = "Successfully updated"
  format.json { render json: { message: message } }
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

~~~
~~~

# JBuilder

jBuilder is already included in rails and is nice if you already have some
views.

# API Documentation

If you use rspec then go with
[rspec_api_documentation](https://github.com/zipmark/rspec_api_documentation),
minitest try [apipie-rails](https://github.com/Apipie/apipie-rails)
or swagger

## Swagger

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
Order in server response if not supported.

https://medium.com/@stevenpetryk/providing-useful-error-responses-in-a-rails-api-24c004b31a2e#.lp6e0sjpf
<https://medium.com/statuscode/introducing-webpacker-7136d66cddfb#.edrnqbrj6>
