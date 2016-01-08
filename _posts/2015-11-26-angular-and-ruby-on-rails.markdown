---
layout: post
title: Angular 1.x and Ruby on Rails
tags: angular testing rails
---

# Rails as asset pipeline

I like to have Rails doing all javascript assets so I choose [angular-rails-templates](https://github.com/pitr/angular-rails-templates) to cache all templates and [bower-rails](https://github.com/rharriso/bower-rails) gem to install all scripts under vendor folder (nice [tutorial](http://angular-rails.com/bootstrap.html)). When angular requests templates, it needs url of that file. Since rails uses asset pipeline we need to use erb `templateUrl: '<%= asset_path 'template' %>'` (when we store assets on CDN, than we need to allow CORS since angular is loaded on our domain). Other solution is [angular-rails-templates](https://github.com/pitr/angular-rails-templates) gem which create angular cache for all templates that found in `app/assets/javascript/templates` (if you want to place templates outside of that folder watch this [issue](https://github.com/pitr/angular-rails-templates/issues/107)). Keep in mind that you need latest gem version (1.4 does not work).

Since rails use minification, we need to write angular dependency inject arguments inside '' like `'$scope'` so they survive minification that happens to javascript. Those arguments are all except last one which is a function in `app.controller("MyController",['$scope',...,myController])`

It is easier to follow callstack if we don't use anonymous function. On other hand it's difficult to keep sync injection params with actual definition params, so I use [ng-annotate](https://github.com/kikonen/ngannotate-rails).

There is a angular factory for resource [angularjs-rails-resource](https://github.com/FineLinePrototyping/angularjs-rails-resource). For example application search [saveIndicatorInterceptor](https://github.com/search?q=saveIndicatorInterceptor&type=Code&utf8=%E2%9C%93)


# Coffescript

* no need `;` at the end of line
* block {...} is replaced with `->` and proper indend (two spaces) or could be inline
* parantheses (...) in one line can be ommited, in multiple lines they need
* object definition {...} braces can be ommited, name/value pairs could be on new lines (if object is only argument than parentheses can be ommited)
* No need to write return command
* [try coffeescript](http://coffeescript.org/) and convertor [js2coffe](http://js2.coffee/)

# JSON

Response status codes:

* *OK 200* (for GET), *Created 201* (successful POST), *No Content 204* (success update or delete `head :no_content`).
* errors give us more info but for sensitive info we could send only `head :not_found` but its better to send the reason:
  * *Not Found 303* `rescue_from ActiveRecord::RecordNotFound do head(:not_found) end`
  * *Bad Request 400*  `if ! @user.update() render json: @user.errors, status: :bad_request`
  * *Unauthorized 401* `rescue_from Pundit::NotAuthorizedError do head(:unauthorized) end `
  * *Forbidden 403*
  * *Unprocessable Entity 422*

# REST

Just to note that [REST api](https://en.wikipedia.org/wiki/Representational_state_transfer) means: Statelessness (no data between requests), Resource identification per request (update only one row, get could have more rows), Representational state transfer (returned representation JSON is enough to identify and manipulate row).

Once API is exposed, you should not modify it, except for critical bugfixes. Use namespace

~~~
namespace :api, default: { format: :json } do
  namespace :v1 do
    resources :expenses, only: [:index, :create, :update, :destroy]
  end
end
~~~

You can use plugin [POSTMAN](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en) to send API requests.

Curl commands:

~~~
export U=http://localhost:3000/api/v1/
curl ${U}/expenses
curl ${U}/expenses -I # or --head show only header
curl ${U}/expenses -H 'Authorization: Token token="c576f0136149a2e2d9127b3901015545"'
~~~

Tutorial videos (first are free) [egghead](https://egghead.io/technologies/angularjs?order=ASC)

# Example apps for Angular Rails:

* [RADD](https://github.com/jesalg/RADD): custom session with devise, api docs are generated [rspec_api_documentation](https://github.com/zipmark/rspec_api_documentation)
* [flapper-news](https://github.com/duleorlovic/flapper-news) [thinkster](https://thinkster.io/angular-rails) [angular-devise](https://github.com/cloudspace/angular_devise)
* [lunch_hub](https://github.com/jasonswett/lunch_hub) [post](https://www.airpair.com/ruby-on-rails/posts/authentication-with-angularjs-and-ruby-on-rails)
  [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth), grunt
* [todo-rails4-angularjs](https://github.com/mkwiatkowski/todo-rails4-angularjs) [shellycloud](https://shellycloud.com/blog/2013/10/how-to-integrate-angularjs-with-rails-4) turbolinks, html auth
* [receta](https://github.com/davetron5000/receta) [angular-rails](http://angular-rails.com/) CRUD (no auth), TDD, angular-flash
* [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth) [devise-token-auth-demo](https://github.com/lynndylanhurley/devise_token_auth_demo) devise-token-auth `cd ng-token-auth/test && bower install` update `config/default.yml` so `API_URL='//devise-token-auth-demo.dev:3000'` and run `gulp dev` and go to http://localhost:7777/. In another window `cd devise_token_auth_demo && bundle && rails s`

[ng-token-auth](https://github.com/search?utf8=%E2%9C%93&q=ng-token-auth&type=Code&ref=searchresults) is used more than [angular-rails]https://github.com/lynndylanhurley/ng-token-auth.git/) is used more than angular-rails. On server use gem [devise_token_auth](https://github.com/lynndylanhurley/devise_token_auth), run application with `gulp dev` and run [demo server](https://github.com/lynndylanhurley/devise_token_auth_demo.git).

# Start from scratch using yeoman angular-gulp

~~~
rails new my_app
cd my_app
echo '# user auth with devise and ng-token-auth
gem "devise_token_auth"
gem "omniauth"
' >> Gemfile
bundle
rails g devise_token_auth:install User auth
rake db:migrate
git add . && git commit -m "rails g devise_token_auth:install User auth"

mkdir client && cd $_
yo gulp-angular

# change dist to ../public

~~~

https://medium.com/opinionated-angularjs/techniques-for-authentication-in-angularjs-applications-7bbf0346acec

https://scotch.io/tutorials/token-based-authentication-for-angularjs-and-laravel-apps
Task://www.sitepoint.com/implementing-authentication-angular-applications/

http://tylermcginnis.com/angularjs-factory-vs-service-vs-provider/

https://www.youtube.com/watch?v=A494WFSi6HU
://www.sitepoint.com/implementing-authentication-angular-applications/

https://www.youtube.com/watch?v=q8ausBZTrxU
https://www.youtube.com/watch?v=SvL_aZt3zyU
https://www.youtube.com/watch?v=qTNGtpfxPtg


rails generate teaspoon:install --coffee

~~~
# spec/javascripts/spec_helper.coffee
#= require support/bind-poly
#= require application
#= require angular-mocks/angular-mocks
~~~

~~~
# Bowerfile
asset "angular-mocks"
~~~


rails generate teaspoon:install --coffee

~~~
# spec/javascripts/spec_helper.coffee
#= require support/bind-poly
#= require application
#= require angular-mocks/angular-mocks
~~~

~~~
# Bowerfile
asset "angular-mocks"
~~~

