---
layout: post
title: Api design
---

# REST and JSON

Just to note that [REST
api](https://en.wikipedia.org/wiki/Representational_state_transfer) means:
Statelessness (no data between requests), Resource identification per request
(update only one row, get could have more rows), Representational state transfer
(returned representation JSON is enough to identify and manipulate row).

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

You can use plugin
[POSTMAN](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en)
to send API requests.

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
`PUT` should be idempotent.
But if user have multiple images, than use `POST /users/1/images` to create and
`PUT /images/1` to update image.

# Use nouns

Only verb should be HTTP method (GET, POST, PUT, DELETE). Url should contain
only nouns that represent resource. So instead `POST /users/1/send-message` it's
better to use `POST /users/1/messages` with `Content-Type: application/json
{ "message": "Hello" }`.

If you need to send messages to multiple users, you can create new endpoint and
send data there `POST /group-messages; Content-Type: application/json; { [ {
"user" : { "id" : 1 }, "message" : "Hello 1" },...] }`

Use url params for search/filter `GET /places?lat=123&lon=123`, but for
authorization use headers `POST /users/1/message; Authorization: Bearer 123`.
HTTP body can contain json data.

# Response status codes:

[w3](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)

* success
  * *200 OK* (for GET)
  * *210 Created* (successful POST)
  * *204 No Content* (success update or delete `head :no_content`)
  * *202 Accepted* Accepted but is being processed async
* errors give us more info but for sensitive info we could send only `head
  :not_found` but its better to send the reason:
  * *303 See Other* when session create email does not exists
  * *400 Bad Request*  `if ! @user.update() render json: @user.errors, status:
    :bad_request` or `ParameterMissing`
  * *401 Unauthorized* used for wrong password on login form, or when there is
    no current_user but it should exists. Note that it should not send when user
    is changing password and type incorect old one (he still should be logged
    in, and not automatically logged out since unauthorized request occurs).
  * *403 Forbidden* when current user exists but is forbidden from accessing
    this data
  * *404 Not Found* url is not valid router or resource does not exist
  * *410 Gone* data has been deleted, deactivated ...
  * *422 Unprocessable Entity* change password form but current password is bad
* server
  * `500 Internal Server Error` unexpected happened on server side
  * `503 Service Unavailable` api is overloaded or in maintenance mode

~~~
# app/controllers/application_controller.rb
  rescue_from ActiveRecord::RecordNotFound do
    head :not_found
  end

  rescue_from ActionController::ParameterMissing do
    head :bad_request
  end

  rescue_from Pundit::NotAuthorizedError do
    head :unauthorized
  end
~~~

Format JSON:

Usually in case of error, I put only all error messages in one field `error` for
example :

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
