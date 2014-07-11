---
layout: post
title:  Ruby on Rails Layouts and rendering
date:   2014-07-01 14:06:33
categories: layout render ruby_on_rails
---

Short reminder how rails use rendering
---


[Guide](http://guides.rubyonrails.org/layouts_and_rendering.html) says that response from the server can be: render, redirect, or head. `ActionController::Base#render` designedly use convention over configuration, that means if we request *products#action_name as HTML* it will render *app/view/products/action_name.html* template (if there has not been a call to render something else). *ActionController::Base#render* method can set up **template** and **partial** to be rendered as **text, json, xml, js** (contenti-type) and set the response code (:status => :ok). 

For example `render "edit" #or :edit`  will change default template to *edit* that belons to the same controller, or we can use from *carts* controller with `render "carts/edit"`. It should be explicited with parameter `render template: "edit"`so we know what is going on (it will render edit action template not partial). Another usage examples of *ActionController::Base#render*:

{% highlight ruby %}
render plain: "OK" 
render html: "<strong>OK</strong>".html_safe 
render xml: @product # automatically calls .to_xml
render json: @product # no need for .to_json
render js: "alert();" # will set content-type MIME text/javascript
{% endhighlight %}

Four options for *render* are:

1. `:content_type => "text/html"` (or "application/json" or "application/xml" or "application/rss") this sets MIME content-type
1. `:layout => 'special_layout'` (or false) can be set to whole controller. Convention is to look first for the same name as controller
1. `:location => photo_url(@photo)` sets HTTP location header
1. `:status => :ok` is 200, [:unprocessable_entity](http://guides.rubyonrails.org/layouts_and_rendering.html#the-status-option) is 422 

Redirect example is `redirect_to :back`.

Head example is `head :created, location: photo_path(@photo)` and is much clearer then `format.js { render nothing: true }`.


There are 6 assets tag helpers:

{% highlight html %}
<%= auto_discovery_link_tag(:rss, {action: "feed"}, {title: "RSS Feed"}) %>
<%= javascript_include_tag "main", "/photos/columns" %>
<%= stylesheet_link_tag "main", "photos/columns" %>
<%= image_tag "icons/delete.gif" %>
<%= video_tag "movie.ogg" %>
<%= audio_tag "music/first_song.mp3" %>
{% endhighlight %}

Asset pipeline
---

All asset files should be inside of `app/assets`, `lib/assets` or `vendor/assets` and they are served with sprockets gem hereof three features: fingerprint, minification and precompilation of sass and coffeescript. That features are not used in development mode, but you can try it if you enable `config.serve_static_assets` in *config/environments.production.rb*, add `secret_key_base` in *config/secrets.yml*  and try:

{% highlight ruby %}
RAILS_ENV=production rake db:setup
RAILS_ENV=production rake assets:precompile
rails s -e production
{% endhighlight %}

[Rails Assets Pipeline](http://guides.rubyonrails.org/asset_pipeline.html) hides the first subfolder, for example *app/assets/javascrips/posts.js* will be overwritten by *app/assets/custom/posts.js* and served as *assets/posts.js* in development or included in *application-123.js* in production. It should be included in assets pipeline by `//= require posts` or `//= require tree .` (if not they will not be included, but can be accessible in development mode, but not in production mode). If we want to include whole library (with special index file for example *lib/assets/library_name/index.js*) we require just folder name `//= require library_name`.

If we want to use controller specific assets (that is loaded only when that controller responds) we should not use `*= require tree .` in *app/assets/stylesheets/application.css* or *app/assets/javascripts/application.js* . Since we are including another js or css asset (that is not included in application.js/css) we have to add it to assets pipeline, for example in *config/initializers/assets.rb* 

    Rails.application.config.assets.precompile += %w( posts.js )
    Rails.application.config.assets.precompile += %w( posts.css )

and include them in a view or layout file

    <%= javascript_include_tag params[:controller] %>
    <%= stylesheet_link_tag params[:controller] %>

All non js or css files are included automatically (images, text, pdf) along with applications.js and application.css by the default matcher for compiling:

    [ Proc.new { |path, fn| fn =~ /app\/assets/ && !%w(.js .css).include?(File.extname(path)) }, /application.(css|js)$/ ]

Erb for assets is used only for `background-image: url(<%= asset_path 'image.png' %>)}` or asset_data_uri (including data directly into css). Remeber that precompiling assets is done only once. Same hyphenated function exists in sass *assets-path("image.png")*. When including data direclty into css a dependency has to 

View
---

View `render` method has nothing to do with controller `render` method. `<%= render 'menu' %>` will render _menu.html. `<%= render 'product/edit' %>` will search for product/_edit.html. Partials can be rendered with its own [layout](http://guides.rubyonrails.org/layouts_and_rendering.html#partial-layouts) `<%= render partial: 'menu', layout: 'graybar' %>`


Each partial has local variable with the same name as partial and you can pass an object into it with :object `<%= render partial: "customer", object: @new_customer %>` or if it is an instance of Customer model shorthand is

    <%= render @customer %> 
    
which will use *_customer.html.erb* with local object `customer`. `<%= render @customers || "There is nothing" %>` for collections, each item will be rendered with _customer.html partial with local objects customer. Also the all `@instance_objects` are accessible but default local object should be enough. If it is not, then you can pass with `<%= render partial: @customers, locals: {title: "aa"} %>` (it needs parameter `partial:` when using `locals`). You can change the layout for collection also or use spacer_template beetween each pair `<%= render partial: @customers, spacer_template: 'product_ruler' %>`.


[Nested layouts](http://guides.rubyonrails.org/layouts_and_rendering.html#using-nested-layouts) is one way to provide different layouts for different controller. It is using `content_for :stylesheet` that will be provided in head section. Sublayout, besides those *content_for* is simply calling `<%= render template: "layouts/application" %>` that is replaced with application layout.

In layouts you can use `<body class="<%= params[:controller] %>">` if you have something specific for each controller, instead sending content for (btw content_for can be controller method with this [gist](https://gist.github.com/hiroshi/985457))

Better solution is to use [partial's layouts](http://railsguides.net/rails-nested-layouts/).

Some rules:

* indicate explicitly that what we are rendering `render template: 'controller/action'`, or `render partial: 'controller/partial'` 
