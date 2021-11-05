---
layout: post
title:  Ruby on Rails Layouts and rendering
tags: layout render ruby-on-rails
---

# Short reminder how rails use rendering

[Guide](http://guides.rubyonrails.org/layouts_and_rendering.html) says that response from the server can be: render, redirect, or just head. `ActionController::Base#render` designedly use convention over configuration, that means if we request *products#show as HTML* it will render *app/view/products/show.html.erb* template (if there has not been a call to render something else). `ActionController::Base#render` method can set up **template** and **partial** to be rendered as **text, json, xml, js** (that is content-type) and set the **response code** (:status => :ok). 

For example `render "edit" #or :edit`  will change default rendering from *show.html.erb* to *edit.html.erb* (in the same controller). We we can render template from another controller, for example *carts* controller with command `render "carts/edit"`. It should be explicited with parameter `render template: "edit"`so we know what is going on (it will render edit template not partial). Another examples of *ActionController::Base#render*:
`render text: 'OK'` is replaced with `render plain: 'OK'`

```
render plain: 'OK'
render html: "<strong>OK</strong>".html_safe 
render xml: @product # automatically calls .to_xml
render json: @product # no need for .to_json
render js: "alert('OK');" # will set content-type MIME text/javascript
# multiline respond in javascript with ruby code (espaced with j so we can see '
render js: %(
  flash_notice('#{view_context.j I18n.t('successfully_updated')}');
  window.location.assign('<%= isp_billing_profiles_path %>');
)

# you can use `head status`
head :created, location: photo_path(@photo)
# this is better than
format.js { render nothing: true }
```

Four options for *render* are:

1. `:content_type => "text/html"` (or "application/json" or "application/xml" or "application/rss") this sets MIME content-type
1. `:layout => 'special_layout'` (or false) can be set to whole controller. Convention is to look first for the same name as controller
1. `:location => photo_url(@photo)` sets HTTP location header
1. `:status => :ok` is 200,
   [:unprocessable_entity](http://guides.rubyonrails.org/layouts_and_rendering.html#the-status-option)
   is 422

Redirect example is `redirect_to :back`.


In Ruby on Rails there are 6 asset tag helpers:

    <%= auto_discovery_link_tag(:rss, {action: "feed"}, {title: "RSS Feed"}) %>
    <%= javascript_include_tag "main", "/photos/columns" %>
    <%= stylesheet_link_tag "main", "photos/columns" %>
    <%= image_tag "icons/delete.gif" %>
    <%= video_tag "movie.ogg" %>
    <%= audio_tag "music/first_song.mp3" %>

# Asset pipeline

All asset files should be inside of `app/assets`, `lib/assets` or
`vendor/assets` and they are served with
[sprockets](https://github.com/rails/sprockets) hereof three features:
fingerprint, minification and precompilation of sass and coffeescript. That
features are not used in development mode, but you can see it when you run in
production environment.

[Rails Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html) hides
(ignores) the first subfolder, for example *app/assets/javascrips/posts.js* will
be overwritten by *app/assets/custom/posts.js* and served as *assets/posts.js*
in development or included in *application-123.js* in production.
It is important to remember that is `another-folder/application.js` uses
`require main.js` that `main.js` file could be picked from wrong location (all
asset paths are searched. If you want to add some path

~~~
# config/initializers/assets.rb
Rails.application.config.assets.paths << R"#{ails.root.join('app', 'assets', 'adminlte', 'images')
~~~

All files  should be referenced to assets pipeline by `//= require posts` or
`//= require tree .` (if it was not referenced, it could be accessible, but only
in development mode). If we want to include whole library (with special index
file for example *lib/assets/library_name/index.js*) we require just folder name
`//= require library_name`.

If we want to use controller specific assets (that is loaded only when that
controller responds) we should not use `*= require tree .` in
*app/assets/stylesheets/application.css* or
*app/assets/javascripts/application.js* . Since we are including another js or
css asset (that is not included in application.js/css) we have to add it to
assets pipeline, for example in *config/initializers/assets.rb* 

    Rails.application.config.assets.precompile += %w( posts.js )
    Rails.application.config.assets.precompile += %w( posts.css )

and include them in a view or layout file

    <%= javascript_include_tag params[:controller] %>
    <%= stylesheet_link_tag params[:controller] %>

Note that is `posts.js` is inside another folder, for example
`app/assets/landing/posts,js` than you need to add that folder to asset paths
`Rails.application.config.assets.paths << Rails.root.join('app', 'assets',
'landing')`. If not found, it will be ignored and not precompiled.

All non js or css files are included automatically (images, text, pdf) along
with applications.js and application.css by the default matcher for compiling:

    [ Proc.new { |path, fn| fn =~ /app\/assets/ && !%w(.js .css).include?(File.extname(path)) }, /application.(css|js)$/ ]


If you use canonical host you need to disable it `export
DO_NOT_USE_CANONICAL_HOST=true` and remove eventual
`config.action_controller.asset_host = asdasd`

# Compile in production mode and Heroku

~~~
RAILS_ENV=production rake db:create
RAILS_ENV=production rake assets:precompile

export RAILS_SERVE_STATIC_FILES=true
rails s -b 0.0.0.0 -e production
~~~

[One
issue](https://devcenter.heroku.com/articles/rails-4-asset-pipeline#known-issues)
with assets precompile is when you use erb in coffee and + secret. If
you change secrets without touching javascript files, something old from tmp
cache will be used. So always change js files when you change secrets, for
example `echo " " >> app/assets/javascripts/a.coffee`.
Similar dependcies could be for stylesheets scss + other asset, and you can use
`depend_on_asset`

~~~
# config/secrets.yml
a: <%= ENV["A"] || 'a' %>

# app/assets/javascript/a.coffee
a = '<%= Rails.application.secrets.a %>'

# app/assets/stylesheets/common.scss
.logo {
  background-image:url('<%= asset_path("logo.png") %> ');
}

rake assets:precompile
export A=b
mv logo2.png app/assets/images/logo.png
rake assets:precompile
cat public/assets/*
~~~

For Heroku you can check assets

~~~
heroku run bash
ls public/assets
~~~

in console you can see exact file name that rails believes it should serve

~~~
heroku run rails console
puts helper.asset_path("application.js")
~~~

Sometimes on heroku still need to purge 50MB tmp cache

~~~
heroku plugins:install heroku-repo
heroku repo:purge_cache
~~~

# Scss

Sass is similar to scss just without brackets and with indent. I think scss is
better since any css code is also scss code.

You can use asset path helpers in scss, instead of erb style `background-image:
url(<%= asset_url 'logo.png' %>)` you can use `asset-url` (which replace `<%=%>`
and `url()` and file name should be inside quotes like `asset-url('logo.png')`.
Remember that it was underscored in erb, use hyphenated in sass
`background-image: asset-url("logo.png")`. You can not use normal
`url("logo.png")`. Note that asset path `asset-path` does not work, you need to
use assets url.
Note that `app/assets/images/some_folder/some_image.jpg` should be
`asset-url('some_folder/some_image.jpg')`

Note that for coffeescript you need to add extension `customers.coffee.erb` and
use this initialization file

~~~
config/initializers/sprockets.rb
Rails.application.config.assets.configure do |env|
  env.context_class.class_eval do
    # include MyAppHelper
    include Rails.application.routes.url_helpers
  end
end
~~~

Note that if you are using sass-rails than you should use `@import
"filename_without_extension";` instead of `require`. That way you can access
global namespace and you can use variables. Do not link filename with extension
since than it will use plain css [import
rule](https://developer.mozilla.org/en/docs/Web/CSS/@import) instead of scss
inserting content.

`@import` knows current folder so you can write only relative path.

Erb for assets could be used only for asset_data_uri (including data directly
into css). Remember that precompiling assets is done only once.

You can set dependency between assets using
[link](https://github.com/sstephenson/sprockets#the-link-directive) `link_tree`
and `link_directory` directive.

If you use `require_tree` than you can set dependencies between files with
`require` so they are included before this file, for example
```
# app/assets/javascripts/plugins/jquery-validation/localication.init.js
/*
* default is english https://github.com/jquery-validation/jquery-validation/blob/dd187b016a1b4eef22ae93500eb37a44bf2ecd0d/src/core.js#L346
*= require plugins/jquery-validation/localization.messages_sr.js
*= require plugins/jquery-validation/localization.messages_ar.js
*/
var lang = $('html').attr('lang');
if (lang == 'sr')
  setSerbian();
else if (lang == 'ar')
  setArabic();
```

# Less

You can use less and scss in same project but you can not use variables from one
to another since they are not compatible.

~~~
/*
 *= require scss_wrapper
 *= require less_wrapper
 */
~~~

# NPM

Add npm to rails app that will install dependecies in `/node_modules`

~~~
npm init -y # to create package.json, -y to accept defaults
npm install jquery-ui-sortable-npm

cat >> .gitignore << HERE_DOC
/node_modules
HERE_DOC
cat >> config/initializers/assets.rb << HERE_DOC
Rails.application.config.assets.paths << Rails.root.join('node_modules')
Rails.application.config.assets.precompile << /\.(?:svg|eot|woff|ttf)$/
HERE_DOC
git add . && git commit -m "Adding npm"
~~~

For Heroku you need to use two build packs. Follow this
[commit](https://github.com/duleorlovic/heroku-rails-bower/commit/ba7c78a1f4f641cfe5592aa75b471aa142dc855a).
It works for latest node and npm version. Older version could give errors:

* bower version `~1.2` gives me error `error Path must be a string. Received
   ...` so make sure you use latest bower.

Old approach with assets from gems still works, no worry.

~~~
heroku create myapp-with-bower
heroku addons:create heroku-postgresql:hobby-dev
heroku buildpacks:set https://github.com/heroku/heroku-buildpack-ruby
heroku buildpacks:add --index 1 https://github.com/heroku/heroku-buildpack-nodejs
heroku buildpacks # should return  1.nodejs  2.ruby (latest will run process)
# alternativelly, we can define then in file .buildpacks
# echo 'https://github.com/heroku/heroku-buildpack-ruby
# https://github.com/heroku/heroku-buildpack-nodejs ' > .buildpacks
# heroku config:add BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git
git push heroku master --set-upstream
~~~

More info on [heroku deploy]({{ site.baseurl }} {% post_url 2015-04-05-common-rails-bootstrap-snippets %}#heroku-deploy)


## Boostrap

You can include bootstrap (which is now written in scss instead of less)

~~~
yarn add bootstrap jquery popper.js

cat >> app/assets/stylesheets/application.scss << \HERE_DOC

HERE_DOC

sed -i app/assets/javascripts/application.js -e '/require_tree/i\
//  from node_modules\
//= require jquery/dist/jquery.js\
//= require popper.js/dist/umd/popper.js\
//= require bootstrap/dist/js/bootstrap'

git commit -am "Adding bootstrap"
~~~

## Adding icons

There is a problem when some image/font files are hardcoded in css files, or
in case of scss, you can define a path, but not a fingerprint digest sha.
One solution is to deploy files without fingerprint, for example
[non-stupid-digest-assets](https://github.com/alexspeller/non-stupid-digest-assets)
gem you can add non digest version. First your assets should be seen
(precompiled) with sprockets than they will be again copied without digest.

```
cat >> Gemfile << HERE_DOC
gem "non-stupid-digest-assets"
HERE_DOC

cat >> config/initializers/assets.rb << \HERE_DOC
Rails.application.config.assets.precompile << /\.(?:svg|eot|woff|woff2|ttf)$/

NonStupidDigestAssets.whitelist += [
  /\.(?:svg|eot|woff|woff2|ttf)$/
]
HERE_DOC
```

Another solution is to override `@font-face` which includes proper `asset-url`

Sprockets `require` concatenates after sass compilation. So it's advices to use
`@import` sass command instead of `require`. `@import` will work also in
`application.css` but variable definition won't (like `$var: 1;`), so we need
to move `css -> scss`.

## Fontawesome

Here is example adding font awesome from npm https://www.npmjs.com/package/@fortawesome/fontawesome-free

https://fontawesome.com/icons?d=gallery&m=free

~~~
yarn add @fortawesome/fontawesome-free

cat >> app/assets/stylesheets/application.scss << \HERE_DOC
// node_modules
@import '@fortawesome/fontawesome-free/css/all.css'
// plugins
@import 'plugins/font-awesome-font-face'
HERE_DOC

# copy @font-face definition from node_modules...all.css and replace url with asset-url
cat >> app/assets/stylesheets/plugins/font-awersome-font-face.css' << HERE_DOC
@font-face {
  font-family: 'Font Awesome 5 Brands';
  font-style: normal;
  font-weight: normal;
  src: asset-url("@fortawesome/fontawesome-free/webfonts/fa-brands-400.eot");
  src: asset-url("@fortawesome/fontawesome-free/webfonts/fa-brands-400.eot?#iefix") format("embedded-opentype"), asset-url("@fortawesome/fontawesome-free/webfonts/fa-brands-400.woff2") format("woff2"), asset-url("@fortawesome/fontawesome-free/webfonts/fa-brands-400.woff") format("woff"), asset-url("@fortawesome/fontawesome-free/webfonts/fa-brands-400.ttf") format("truetype"), asset-url("@fortawesome/fontawesome-free/webfonts/fa-brands-400.svg#fontawesome") format("svg");
}

@font-face {
  font-family: 'Font Awesome 5 Free';
  font-style: normal;
  font-weight: 400;
  src: asset-url("@fortawesome/fontawesome-free/webfonts/fa-regular-400.eot");
  src: asset-url("@fortawesome/fontawesome-free/webfonts/fa-regular-400.eot?#iefix") format("embedded-opentype"), asset-url("@fortawesome/fontawesome-free/webfonts/fa-regular-400.woff2") format("woff2"), asset-url("@fortawesome/fontawesome-free/webfonts/fa-regular-400.woff") format("woff"), asset-url("@fortawesome/fontawesome-free/webfonts/fa-regular-400.ttf") format("truetype"), asset-url("@fortawesome/fontawesome-free/webfonts/fa-regular-400.svg#fontawesome") format("svg");
}

@font-face {
  font-family: 'Font Awesome 5 Free';
  font-style: normal;
  font-weight: 900;
  src: asset-url("@fortawesome/fontawesome-free/webfonts/fa-solid-900.eot");
  src: asset-url("@fortawesome/fontawesome-free/webfonts/fa-solid-900.eot?#iefix") format("embedded-opentype"), asset-url("@fortawesome/fontawesome-free/webfonts/fa-solid-900.woff2") format("woff2"), asset-url("@fortawesome/fontawesome-free/webfonts/fa-solid-900.woff") format("woff"), asset-url("@fortawesome/fontawesome-free/webfonts/fa-solid-900.ttf") format("truetype"), asset-url("@fortawesome/fontawesome-free/webfonts/fa-solid-900.svg#fontawesome") format("svg");
}
HERE_DOC
~~~

## Simple line icons

If you need to include for example
[simple-line-icons](https://github.com/thesabbir/simple-line-icons) than you
can place it inside `app/assets/simple_line_icons` and you need to overwrite
font path, ie instead of `url('../fonts/Simple-Line-Icons.eot?v=2.4.0')` use
`asset-url('fonts/Simple-Line-Icons.eot?v=2.4.0');`

~~~
// app/assets/stylesheets/pages.scss
/*
 *= require css/simple-line-icons
 */
@font-face {
  font-family: 'simple-line-icons';
  src: asset-url('fonts/Simple-Line-Icons.eot?v=2.4.0');
  src: asset-url('fonts/Simple-Line-Icons.eot?v=2.4.0#iefix')
  format('embedded-opentype'),
  asset-url('fonts/Simple-Line-Icons.woff2?v=2.4.0') format('woff2'),
  asset-url('fonts/Simple-Line-Icons.ttf?v=2.4.0') format('truetype'),
  asset-url('fonts/Simple-Line-Icons.woff?v=2.4.0') format('woff'), asset-url('fonts/Simple-Line-Icons.svg?v=2.4.0#simple-line-icons') format('svg');
  font-weight: normal;
  font-style: normal;
}
~~~

If you are installing from node module, you can not just update the path since
all font files will be fingerprinted, so the best is to copy to and edit scss
file

~~~
npm install simple-line-icons

// we use asset-url to find a fingerprinted path of font files in
// node_modules/simple-line-icons/fonts
$simple-line-font-path: "simple-line-icons/fonts/"
@import 'plugins/simple-line-icons'

cp node_modules/simple-line-icons/scss/simple-line-icons.scss app/assets/stylesheets/plugins

# app/assets/stylesheets/plugins/simple-line-icons.scss
# replace url with asset-url, like
// Fonts
@if $simple-line-font-family == "simple-line-icons" {
  @font-face {
    font-family: '#{$simple-line-font-family}';
    src:    asset-url('#{$simple-line-font-path}Simple-Line-Icons.eot?v=2.4.0');
    src:    asset-url('#{$simple-line-font-path}Simple-Line-Icons.eot?v=2.4.0#iefix') format('embedded-opentype'),
            asset-url('#{$simple-line-font-path}Simple-Line-Icons.woff2?v=2.4.0') format('woff2'),
            asset-url('#{$simple-line-font-path}Simple-Line-Icons.ttf?v=2.4.0') format('truetype'),
            asset-url('#{$simple-line-font-path}Simple-Line-Icons.woff?v=2.4.0') format('woff'),
            asset-url('#{$simple-line-font-path}Simple-Line-Icons.svg?v=2.4.0#simple-line-icons') format('svg');
    font-weight: normal;
    font-style: normal;
  }
}
~~~
You can test with:

~~~
RAILS_ENV=production rake db:setup db:migrate
RAILS_ENV=production rake assets:precompile -v
RAILS_SERVE_STATIC_FILES=true rails s -e production
~~~

You can check all current asset paths and write relative to that

~~~
rails runner "puts Rails.application.config.assets.paths"
~~~

# View

View `render` method has nothing to do with controller `render` method. `<%= render 'menu' %>` will render *_menu.html.erb* partial. `<%= render 'product/edit' %>` will search for product/_edit.html. Partials can be rendered with its own [layout](http://guides.rubyonrails.org/layouts_and_rendering.html#partial-layouts) `<%= render partial: 'menu', layout: 'graybar' %>`


Each partial has local variable with the same name as partial and you can pass an object into it with :object `<%= render partial: "customer", object: @new_customer %>` or if it is an instance of Customer model shorthand is `<%= render @customer %>` which will use *_customer.html.erb* with local object `customer`. I do not recomend this shorthand. It is more self-explanatory when full parameters are used.


It is prefered to use local variables when passing data to partial (instead of *@instance* variables). This is because partials can be used from different controllers, where some @instance variable is not set. For example, `<%= render 'users/customer', customer: @customer %>`.
Note that is you use `render partial: 'name', locals: { customer: @customer
}` than you need to use `locals`.
Locals of partial should be explained in a comment block:

    <%# customer partial uses locals
       - customer (required, instance of Customer)
       - contact_form (true/false, default true)
     %>
     <%# (do not use contact_form||=true since it will override contact_form=false %>
     <%
       unless defined? contact_form
         contact_form = true
       end 
     %> 

In layouts you can use `<body class="controller-<%= controller_name %>">`.
`controller_name` is `params[:controller]`. In you scss you can use:
`.controller-home .header ...` if you have something specific for each action
you can add `action_name` class.

*content_for* can be controller method with this
[gist](https://gist.github.com/hiroshi/985457)

# Errors

If you see error `ActionView::Template::Error (Unrecognised input):`  than you
might forget semicolon `;` in your less file.
