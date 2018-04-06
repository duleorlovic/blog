---
layout: post
title: Angular 1.x and Ruby on Rails
tags: angular testing rails
npToc: false
---

# Rails as asset pipeline

I like to have Rails doing all javascript assets so I choose
[angular-rails-templates](https://github.com/pitr/angular-rails-templates) to
cache all templates and [bower-rails](https://github.com/rharriso/bower-rails)
gem to install all scripts under vendor folder (nice
[tutorial](http://angular-rails.com/bootstrap.html)). When angular requests
templates, it needs url of that file. Since rails uses asset pipeline we need to
use erb `templateUrl: '<%= asset_path 'template' %>'` (when we store assets on
CDN, than we need to allow CORS since angular is loaded on our domain). Other
solution is
[angular-rails-templates](https://github.com/pitr/angular-rails-templates) gem
which create angular cache for all templates that found in
`app/assets/javascript/templates` (if you want to place templates outside of
that folder watch this
[issue](https://github.com/pitr/angular-rails-templates/issues/107)). Keep in
mind that you need latest gem version (1.4 does not work).

Since rails use minification, we need to write angular dependency inject
arguments inside '' like `'$scope'` so they survive minification that happens to
javascript. Those arguments are all except last one which is a function in
`app.controller("MyController",['$scope',...,myController])`

It is easier to follow callstack if we don't use anonymous function. On other
hand it's difficult to keep sync injection params with actual definition params,
so I use [ng-annotate](https://github.com/kikonen/ngannotate-rails).

# railsResourceFactory

There is a angular factory for resource
[angularjs-rails-resource](https://github.com/FineLinePrototyping/angularjs-rails-resource).

~~~
bower install angularjs-rails-resource --save
# resolve with latest angular, it works find with 1.5.3
~~~

You can start from [EXAMPLES](https://github.com/FineLinePrototyping/angularjs-rails-resource/blob/master/EXAMPLES.md) and search applications for it's usage
[saveIndicatorInterceptor](https://github.com/search?q=saveIndicatorInterceptor&type=Code&utf8=%E2%9C%93)

Here is simple example for ionic

~~~
# www/js/app.js
angular.module 'starter', ['ionic', 'rails']

# www/index.html
    <!-- Rails Resource
    https://github.com/FineLinePrototyping/angularjs-rails-resource -->
    <script src="lib/angularjs-rails-resource/angularjs-rails-resource.min.js"></script>

# www/js/resources/locationTiken.resource.coffee
angular.module 'starter'
  .factory 'LocationTicket', (railsResourceFactory, CONSTANT) ->
    railsResourceFactory
      url: CONSTANT.SERVER_URL + '/location_tickets'
      name: 'location_ticket'

# www/js/locationTicke/locationTicket.controller.coffee
~~~

Note that
[resource-url](https://github.com/FineLinePrototyping/angularjs-rails-resource#resource-urls)
could be a function that will be evaluated at runtime (usefull if you need to
change api url depending on which user is logged in).  You can set context with

~~~
Item.query({category: 'Software'}, {storeId: 123}) // would generate a GET to /stores/123/items?category=Software
Item.get({storeId: 123, id: 1}) // would generate a GET to /stores/123/items/1
~~~

I use that context to change url for my custom methods.

~~~
# some controller
LocationTicket.query({}, info: true).then (info) ->
  vm.info = info

# www/js/resources/locationTicket.resource.coffee
angular.module 'starter'
  .factory 'LocationTicket', (railsResourceFactory, CustomerAuth) ->
    railsResourceFactory
      url: (context) ->
        if context && context.id
          CustomerAuth.locationServerUrl + '/api/v1/location_tickets/' +
          context.id
        else if context.info
          CustomerAuth.locationServerUrl + '/api/v1/location_tickets/info'
        else
          CustomerAuth.locationServerUrl + '/api/v1/location_tickets'
      name: 'location_ticket'
~~~

I use interceptor for loader. Also show toast if flag is set like
`cart.showToastOnError = true; cart.delete()`. For some resource I show toast
for all errors by adding `interceptors: [notifyInterceptor]`

~~~
# src/app/resource/myResourceFactory.interceptor.coffee
angular.module 'myapp.resources'
  # https://github.com/FineLinePrototyping/angularjs-rails-resource/blob/master/vendor/assets/javascripts/angularjs/rails/resource/resource.js#L860
  .factory 'myResourceFactory', (RailsResource, saveIndicatorInterceptor) ->
    (config) ->
      Resource = () ->
        Resource.__super__.constructor.apply(this, arguments)
        return
      RailsResource.extendTo Resource
      config["interceptors"] ||= []
      config["interceptors"].push saveIndicatorInterceptor
      Resource.configure config
      Resource
  .factory 'saveIndicatorInterceptor', ($rootScope, $q, toastr) ->
    beforeRequest: (httpConfig, resourceConstructor, context) ->
      $rootScope.isLoading = true
      httpConfig
    afterResponse: (result, resourceConstructor, context) ->
      $rootScope.isLoading = false
      result
    afterResponseError: (rejection, resourceConstructor, context) ->
      $rootScope.isLoading = false
      if context.showToastOnError
        if rejection.data.error
          message = rejection.data.error
        else
          message = JSON.stringify rejection.data
        toastr.error message
      $q.reject rejection
  # https://github.com/FineLinePrototyping/angularjs-rails-resource#example-interceptor
  .factory 'notifyInterceptor', (toastr, $q) ->
    afterResponseError: (rejection, resourceConstructor, context) ->
      message = JSON.stringify rejection.data
      toastr.error message
      $q.reject rejection
~~~

So I use `myResourceFactory` as base for all my resources.

~~~
angular.module 'myapp.resources'
  .factory 'Restaurant', (myResourceFactory, railsSerializer, CONFIG) ->
    myResourceFactory
      url: (context) ->
        # https://github.com/FineLinePrototyping/angularjs-rails-resource/issues/141
        # https://github.com/FineLinePrototyping/angularjs-rails-resource#resource-urls
        if context && context.id
          CONFIG.API_URL + '/restaurants/'+context.id
        else if context && context.userId
          CONFIG.API_URL + '/users/'+context.userId+'/restaurants'
        else if context && context.link
          CONFIG.API_URL + '/restaurants/'+context.link+'/by-link'
        else
          CONFIG.API_URL + '/restaurants'
      name: 'restaurant'
      serializer: railsSerializer( ->
        this.resource 'menuSections', 'MenuSection'
      )
      interceptors: [notifyInterceptor]
~~~

If you need to add property to resorce object (to have custom url) you should
use finally to clean up that resource for later `.save()`

~~~
    vm.signInAs = (user) ->
      user.signInAs = true
      user.save().then (resp) ->
        toastr.info "Authenticated as #{user.firstName}. Please refresh"
      .finally ->
        user.signInAs = false
~~~

[Adding custom
methods](https://github.com/FineLinePrototyping/angularjs-rails-resource/blob/master/EXAMPLES.md#adding-custom-methods-to-a-resource) 

# Example apps for Angular Rails:

Tutorial videos (first are free) [egghead](https://egghead.io/technologies/angularjs?order=ASC)

* [RADD](https://github.com/jesalg/RADD): custom session with devise, api docs
  are generated
  [rspec_api_documentation](https://github.com/zipmark/rspec_api_documentation)
* [flapper-news](https://github.com/duleorlovic/flapper-news)
  [thinkster](https://thinkster.io/angular-rails)
  [angular-devise](https://github.com/cloudspace/angular_devise)
* [lunch_hub](https://github.com/jasonswett/lunch_hub)
  [post](https://www.airpair.com/ruby-on-rails/posts/authentication-with-angularjs-and-ruby-on-rails)
  [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth), grunt
* [todo-rails4-angularjs](https://github.com/mkwiatkowski/todo-rails4-angularjs)
  [shellycloud](https://shellycloud.com/blog/2013/10/how-to-integrate-angularjs-with-rails-4)
  turbolinks, html auth
* [receta](https://github.com/davetron5000/receta)
  [angular-rails](http://angular-rails.com/) CRUD (no auth), TDD, angular-flash
* [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth)
  [devise-token-auth-demo](https://github.com/lynndylanhurley/devise_token_auth_demo)
  devise-token-auth `cd ng-token-auth/test && bower install` update
  `config/default.yml` so `API_URL='//devise-token-auth-demo.local:3000'` and run
  `gulp dev` and go to [localhost:7777](http://localhost:7777). In another
  window `cd devise_token_auth_demo && bundle && rails s`.
  [ng-token-auth](https://github.com/search?utf8=%E2%9C%93&q=ng-token-auth&type=Code&ref=searchresults)
  is used ~2.5K times. On server use gem
  [devise_token_auth](https://github.com/lynndylanhurley/devise_token_auth), run
  application with `gulp dev` and run [demo
  server](https://github.com/lynndylanhurley/devise_token_auth_demo.git).
* [angularonrails.com](https://www.angularonrails.com/ruby-on-rails-angularjs-single-page-application/)
* [telegram.org](https://web.telegram.org/#/login) encrypted json

* for start from scratch using ng-token-auth look at

  [devise-oauth-angular]({{ site.baseurl }}{% post_url 2015-12-20-devise-oauth-angular %})

## Angular inside rails asset pipeline

~~~
echo '
# js package manager
gem "bower-rails"
# cache all templates
gem "angular-rails-templates"
' >> Gemfile
bundle
rails g bower_rails:initialize
git add . && git commit -m "rails g bower_rails:initialize"

echo '
Rails.application.config.assets.paths << Rails.root.join("vendor","assets","bower_components")
Rails.application.config.assets.paths << Rails.root.join("vendor","assets","bower_components","bootstrap-sass-official","assets","fonts")
Rails.application.config.assets.precompile << %r(.*.(?:eot|svg|ttf|woff|woff2)$)
' >> config/initializers/bower_rails.rb

echo '
asset "angular"
asset "bootstrap-sass-official"
# vim: ft-ruby
' >> Bowerfile

rake bower:install

git rm app/assets/stylesheets/application.css
echo '
@import "bootstrap-sass-official/assets/stylesheets/bootstrap-sprockets";
@import "bootstrap-sass-official/assets/stylesheets/bootstrap";
' >> app/assets/stylesheets/application.scss

sed -i '/jquery_ujs/a \
//= require angular/angular' app/assets/javascripts/application.js

git add . && git commit -am "rake bower:install angular & boostrap"
~~~

## Angular and Rails totally separated using gulp

Asset pipeline using gulp.
Good reference is [railsAngularTutorial](https://github.com/grantgeorge/railsAngularTutorial) 

~~~
# first create rails server side
rails new --database=postgresql air
cd air
git init && git add . && git commit -m "rails new air  --database=postgresql"

rails g scaffold articles title:string body:text
rake db:migrate
git add . && git commit -m "rails g scaffold articles title:string body:text"
echo "Article.create(title: 'Test Article', body: 'A test article. Cool')" >> db/seeds.rb
rake db:seed
mkdir app/controllers/v1
git mv app/controllers/articles_controller.rb app/controllers/v1/articles_controller.rb
sed -i "/articles/c \\\n\
  scope '/api' do\n\
    namespace :v1, defaults: { format: :json } do\n\
      resources :articles, except: [:new, :edit]\n\
    end\n\
  end\n\
" config/routes.rb
sed -i '/class ArticlesController/c class V1::ArticlesController < ApplicationController' app/controllers/v1/articles_controller.rb
mkdir app/views/v1
git mv app/views/articles/ app/views/v1/
sed -i '/article_url/c \  json.url v1_article_url(article, format: :json)' app/views/v1/articles/index.json.jbuilder
git add . && git commit -m "Move Articles to api/v1 namespace"

echo "gem 'rails_12factor', group: :production" >> Gemfile
bundle install
git add . && git commit -m "Heroku uses 12 factor"
heroku create
heroku addons:create heroku-postgresql:hobby-dev
git push heroku master --set-upstream
heroku run rake db:migrate db:seed
heroku open api/v1/articles
~~~

Yeoman yo [gulp-angular](https://github.com/Swiip/generator-gulp-angular) is great since it provides some initial tools.
I use `$log` for logging objects, for example `$log.debug loginController: 'submitLogin then', resp: resp`

~~~
# than creating angular client side
npm install -g yo generator-gulp-angular
mkdir client && cd $_
yo gulp-angular myappAngular # don't --default
# choose coffe cript and Angular material
git add . && git commit -m "yo gulp-angular with coffeescript and Material"
# config proxy for /api and /omniauth, add params -p and -o
sed -i gulp/server.js -f - <<HERE_DOC
/function browserSyncInit(baseDir, browser) {/c \
function browserSyncInit(baseDir, browser, port, open, livereload) {\n\
  port = port === undefined ? '3000' : port;\n\
  open = open === undefined ? false : open;\n\
  var snippetOptions = livereload === true ? {} : { rule: { match: /xxx/ } };

/routes: routes/i \    middleware: [\n\
      proxyMiddleware(["/api","/omniauth"], { target: "http://localhost:"+port }),\n\
    ],
/browserSync.instance/a \\
    port: 9000,\n\
    ui: {\n\
      port: 8080,\n\
    },\n\
    open: open,\n\
    snippetOptions: snippetOptions,
s/task('serve'/task('serve_original'/
HERE_DOC
cat >> gulp/server.js <<'HERE_DOC'
gulp.task('serve', ['watch'], function () {
  // http://stackoverflow.com/questions/28538918/pass-parameter-to-gulp-task
  var port, open, livereload, i;
  i = process.argv.indexOf("-p");
  if (i>-1) {
    port = process.argv[i+1];
    console.log("Find parameter -p " + port);
  }
  i = process.argv.indexOf("-o");
  if (i>-1) {
    open = true;
    console.log("Find parameter -o");
  }
  i = process.argv.indexOf("-l");
  if (i>-1) {
    livereload = true;
    console.log("Find parameter -l");
  }
  browserSyncInit([path.join(conf.paths.tmp, '/serve'), conf.paths.src], undefined, port, open, livereload);
});
HERE_DOC
git add . && git commit -m "Config middleware to 3000 or -p param"

# better formating
sed -i src/app/index.module.coffee \
  -e 's/, /,\n  /g' \
  -e 's/  \[/\[\n  /g' \
  -e 's/\]/,\n\]/'

# use latest angular material
sed -i bower.json -e '/angular-material/c\
    "angular-material": "*",'
bower update angular-material --save
~~~

Example articles factory:

~~~
mkdir src/app/components/articles
cat > src/app/components/articles/article.factory.js << \EOF
'use strict';

angular.module('myappAngular')
  .factory('Article', function ($resource) {
    return $resource('api/v1/articles/:articleId', {
      articleId: '@id'
    }, {
      update: {
        method: 'PUT'
      }
    });
  });
EOF

echo note that this works for javascript not for coffeescript
sed -i '/vm.showToastr/a \
    Article.query(function (res) {\
      vm.articles = res;\
    });' src/app/main/main.controller.js
sed -i '/function MainController/c \  function MainController($timeout, webDevTec, toastr, Article) {' src/app/main/main.controller.js
sed -i '/Allo/c \
    <h1>Articles</h1>\
    <div ng-repeat="article in main.articles">\
      <h2>{{ "{{ article.title" }} }}</h2>\
      <p>{{ "{{ article.body" }} }}</p>\
    </div>\
' src/app/main/main.html
cd .. # back to root
git add . && git commit -m "Adding angular Article resource and show on main page"
~~~

To run on linux/windows, install git (Use Git from the Windows Command Prompt)
and nodejs.  Run in windows command line cmd: `npm install -g gulp` and `npm
install` and `gulp serve`

If you want to completely split front end code, than use some [Gulp tasks]({% post_url 2016-03-02-gulp-tasks %})

# Angular httpProvider can't send json patch request

Can't set `'Content-Type': 'application/json'` for angular `$http(method:
'PATCH')`. I tried with both inline `header` and config
`$httpProvider.defaults.headers.patch = 'Content-Type': 'application/json'`but
not luck. It is always http request.

UPDATE: it can send, but url needs to be `.json`
Do not use `patch` because old browsers (if you use ionic, that means old
phones).
