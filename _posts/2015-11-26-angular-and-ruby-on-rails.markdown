---
layout: post
title: Angular 1.x and Ruby on Rails
tags: angular testing rails
npToc: false
---

# Rails as asset pipeline

I like to have Rails doing all javascript assets so I choose [angular-rails-templates](https://github.com/pitr/angular-rails-templates) to cache all templates and [bower-rails](https://github.com/rharriso/bower-rails) gem to install all scripts under vendor folder (nice [tutorial](http://angular-rails.com/bootstrap.html)). When angular requests templates, it needs url of that file. Since rails uses asset pipeline we need to use erb `templateUrl: '<%= asset_path 'template' %>'` (when we store assets on CDN, than we need to allow CORS since angular is loaded on our domain). Other solution is [angular-rails-templates](https://github.com/pitr/angular-rails-templates) gem which create angular cache for all templates that found in `app/assets/javascript/templates` (if you want to place templates outside of that folder watch this [issue](https://github.com/pitr/angular-rails-templates/issues/107)). Keep in mind that you need latest gem version (1.4 does not work).

Since rails use minification, we need to write angular dependency inject arguments inside '' like `'$scope'` so they survive minification that happens to javascript. Those arguments are all except last one which is a function in `app.controller("MyController",['$scope',...,myController])`

It is easier to follow callstack if we don't use anonymous function. On other hand it's difficult to keep sync injection params with actual definition params, so I use [ng-annotate](https://github.com/kikonen/ngannotate-rails).

There is a angular factory for resource [angularjs-rails-resource](https://github.com/FineLinePrototyping/angularjs-rails-resource). For example application search [saveIndicatorInterceptor](https://github.com/search?q=saveIndicatorInterceptor&type=Code&utf8=%E2%9C%93)


# Coffeescript

* no need `;` at the end of line
* block {...} is replaced with `->` and proper indend (two spaces) or could be inline
* parantheses (...) in one line can be ommited, in multiple lines they need
* object definition {...} braces can be ommited, name/value pairs could be on new lines (if object is only argument than parentheses can be ommited)
* No need to write return command. Since last line is returning, put empty
  `return` in angular constructor functions
* [try coffeescript](http://coffeescript.org/) and convertor [js2coffe](http://js2.coffee/)

# JSON

Response status codes:

* *OK 200* (for GET), *Created 201* (successful POST), *No Content 204* (success update or delete `head :no_content`).
* errors give us more info but for sensitive info we could send only `head :not_found` but its better to send the reason:
  * *See Other 303* when session create email does not exists
  * *Bad Request 400*  `if ! @user.update() render json: @user.errors, status: :bad_request`
  * *Unauthorized 401*
  * *Forbidden 403*
  * *Not Found 404*
  * *Unprocessable Entity 422*

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
npm install generator-gulp-angular
mkdir client && cd $_
yo gulp-angular myappAngular # don't --default
# choose coffe cript and Angular material
git add . && git commit -m "yo gulp-angular with coffescrip and Material"
# config proxy for /api and /omniauth, add params -p and -o
sed -i gulp/server.js -f - <<HERE_DOC
/use strict/a var exec = require('child_process').exec;
/function browserSyncInit(baseDir, browser) {/c \
function browserSyncInit(baseDir, browser, port, open) {\n\
  port = port === undefined ? '3000' : port;\n\
  open = open === undefined ? false : open;

/routes: routes/i \    middleware: [\n\
      proxyMiddleware(["/api","/omniauth"], { target: "http://localhost:"+port }),\n\
    ],
/browserSync.instance/a \\
    port: 9000,\n\
    ui: {\n\
      port: 8080,\n\
    },\n\
    open: open,
s/task('serve'/task('serve_original'/
HERE_DOC
cat >> gulp/server.js <<'HERE_DOC'
gulp.task('serve', ['watch'], function () {
  // http://stackoverflow.com/questions/28538918/pass-parameter-to-gulp-task
  var port, open, i;
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
  browserSyncInit([path.join(conf.paths.tmp, '/serve'), conf.paths.src], undefined, port, open);

gulp.task('rails', function() {
  exec("rails server");
});
gulp.task('serve:rails', ['rails', 'serve']);
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

  To deploy on heroku we need custom [buildpack](https://devcenter.heroku.com/articles/nodejs-support#customizing-the-build-process). We could use 3th party gulp buildpacks but default [node](https://docs.npmjs.com/misc/config) [buildpack](https://github.com/heroku/heroku-buildpack-nodejs) works fine.
  
  `devDependencies` need to be renamed to `dependecies` since it runs node production mode (I tried to disable production mode using NPM_CONFIG_ONLY but than other thinks does not work). Imporant is [NODE_MODULES_CACHE](https://devcenter.heroku.com/articles/nodejs-support#cache-behavior). If we (by default) cache `/node_modules` it will not build new version to public folder.

~~~
# deploy to heroku
heroku buildpacks:set https://github.com/heroku/heroku-buildpack-ruby
heroku buildpacks:add --index 1 https://github.com/heroku/heroku-buildpack-nodejs

heroku buildpacks # should return  1. nodejs  2. ruby (latest wins :)
# alternativelly, we can define then in file .buildpacks
# echo 'https://github.com/heroku/heroku-buildpack-ruby
# https://github.com/heroku/heroku-buildpack-nodejs ' > .buildpacks
# heroku config:add BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git

git rm -rf public
echo '/public
/node_modules' >> .gitignore
git add .gitignore
git commit -m "Remove public folder from git repo"

echo '{
  "name": "rootApp",
  "scripts": {
  },
  "dependencies": {
    "myappAngular": "file:./client"
  },
  "cacheDirectories": [
    "client/node_modules",
    "client/bower_components"
  ]
}
' > package.json
git add package.json && git commit -m "Add package.json for nodejs buildpack detect"

cd client
sed -i "/dist: 'dist/c \  dist: '../../public'," gulp/conf.js
sed -i '/"scripts":/a \    "postinstall": "bower install && gulp build",' package.json
sed -i '/"devDependencies":/c \  "dependencies": {\
    "bower": "*",' package.json
git add . && git commit -m "Configure postinstall gulp build"
cd ..

git push heroku
heroku config # NPM_CONFIG_PRODUCTION=true NODE_ENV=production NODE_MODULES_CACHE=true
heroku open
# usefull command to test is `npm install` or `
# git add . && git commit --amend --no-edit && git push heroku -f > o 2>&1 && heroku run bash -c 'ls public'
~~~

To run on linux/windows, install git and nodejs, clone repository to `air`, than run in command prompt `cd air` `npm install` `npm install -g gulp` `gulp serve`


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

