---
layout: post
title: AdminLTE Free Template on Rails
---

# Installation

You can start by cloning template from github
[adminLTE](https://github.com/almasaeed2010/AdminLTE) and preview localy (it
should look the same as [preview](https://almsaeedstudio.com/preview)). You can
open documentation localy or
[online](https://almsaeedstudio.com/themes/AdminLTE/documentation/index.html)

~~~
git clone https://github.com/almasaeed2010/AdminLTE.git
cd AdminLTE
gnome-open index.html
gnome-open documentations/index.html
~~~

To install to existing rails project you can use [bower inside rails](
{{ site.baseurl }} {% post_url 2015-04-05-common-rails-bootstrap-snippets %}#bower
) and than add admin-lte

~~~
bower install admin-lte --save
~~~

To add new template while keeping existing you can use different layout base on
locale. You can set new layout with option `?layout=true`

~~~
# app/controllers/application_controller.rb
  layout proc { |controller| controller.new_layout? ? 'adminlte' : 'application' }
  before_filter :set_locale_for_layout

  def new_layout?
    I18n.locale == :rs
  end

  def set_locale_for_layout
    if session[:layout] == true
      if params[:layout] == "false"
        # disable new layout
        session[:layout] = false
        I18n.locale = 'en'
        logger.debug "layout became false"
      else
        logger.debug "layout still true"
        I18n.locale = 'rs'
      end
    else
      # session[:layout] == false
      if params[:layout] == "true"
        # enable new layout
        session[:layout] = true
        logger.debug "layout became true"
        I18n.locale = 'rs'
      else
        I18n.locale = 'rs'
        logger.debug "layout still false"
      end
    end
  end
~~~

Also we need to make changes on other places

~~~
# config/application.rb
    config.i18n.available_locales = [:en, :rs]

# config/locales/en.yml
# config/locales/devise.en.yml
en: &default
rs:
  <<: *default

# app/views/pages/sample_page.in.html.erb
I'm from adminlte layout


<!-- above Main content in app/views/layouts/adminlte.html.erb -->
    <div class="new-layout-switch">
      <%= link_to "Go Back To Old Layout", params.merge(layout: false) %>
    </div>

// app/assets/stylesheets/application_adminlte.scss
.m-b-10 {
  margin-bottom: 10px;
}

// similar to pull-right just without float
.text-align-right {
  text-align: right;
}

.new-layout-switch {
  position: absolute;
  padding-left: 4px;
  top: 50px;
  right: 0px;
  border-bottom-left-radius: 5px;
  background: #3C8DBC;
  a {
    color: white;
  }
}
~~~

We will use `starter.html` for initial layout.
If do not use turbolinks you can link assets in your
`vendor/assets/bower_components/` and precompile them in
`config/initializers/assets.rb` with `Rails.application.config.assets.precompile
<< /AdminLTE\/.*/`

Note that you need to precompile only files that are not used in your
application.js/css (but you link them separatelly).

~~~
# app/layouts/adminlte.html
  <link rel="stylesheet" href="<%= asset_path 'AdminLTE/bootstrap/css/bootstrap.min.css' %>">
  <link rel="stylesheet" href="<%= asset_path 'AdminLTE/dist/css/AdminLTE.min.css' %>">
  <link rel="stylesheet" href="<%= asset_path 'AdminLTE/dist/css/skins/skin-blue.min.css' %>">

<!-- use the same version of jquery that you found on vendor/assets/bower_components/AdminLTE/... -->
<script src="<%= asset_path 'AdminLTE/plugins/jQuery/jquery-2.2.3.min.js' %>"></script>
<script src="<%= asset_path 'AdminLTE/bootstrap/js/bootstrap.min.js' %>"></script>
<script src="<%= asset_path 'AdminLTE/dist/js/app.min.js' %>"></script>
~~~

## With turbolinks

You can use assets pipeline to combine all files to one .css and .js file

~~~
// app/assets/stylesheets/application_adminlte.scss
/*
 *= require rails_bootstrap_forms
 *= require AdminLTE/bootstrap/css/bootstrap.min
 *= require AdminLTE/dist/css/AdminLTE.min
 *= require AdminLTE/dist/css/skins/skin-blue.min
 */
~~~

~~~
// app/assets/javascript/application_adminlte.js
//  AdminLTE stuff
//= require AdminLTE/plugins/jQuery/jquery-2.2.3.min
//= require AdminLTE/bootstrap/js/bootstrap.min
//= require AdminLTE/dist/js/app.min
//
// Rails stuff
//= require jquery_ujs
//
// All layout stuff
//
// Our adminlte related stuff (filename ends with _adminlte)
~~~

Also you need to exclude those `_adminlte` files in original sprockets:

~~~
// app/assets/stylesheets/application.scss
 *= stub application_adminlte

// app/assets/javascript/application.js
//= stub application_adminlte
~~~

Stubing in sprockets is recursive so if you need to include libraries (for
example `jquery_ujs`) in both layouts it will [stub that
libraries also](https://github.com/sstephenson/sprockets/issues/467). Also if
use use `require_tree .` than you can inadvertently add some libs to some
layout.

Better approach is to create new folder `app/assets/adminlte`.
Add it to asset pipeline paths:

~~~
# config/initializers/assets.rb
Rails.application.config.assets.paths << Rails.root.join('app', 'assets', 'adminlte')
~~~

Since asset pipeline [ignore first subfolders]( {{ site.baseurl }}
{% post_url 2014-07-01-ruby-on-rails-layouts-and-rendering %}) you need to take
care to name files differently when you create new scss and coffee files. I
usually I add suffix `_adminlte`.

# Tunning

## Plugins

AdminLTE is based on Bootstrap 3 and jQuery 1.11+. It contains a lot of plugins
as you can see in their
[documentation](https://almsaeedstudio.com/themes/AdminLTE/documentation/index.html#plugins)

* [iCheck](https://github.com/fronteed/icheck) for checkboxes

  ~~~
  bower install icheck --save

  /* app/assets/adminlte/application_adminlte.scss
  *= require iCheck/skins/square/blue

  // app/assets/adminlte/application_adminlte.js
  //= require iCheck/icheck.min
  ~~~

  Note that when you run `bower install` than some version of jquery will be
  installed and it will overwrite existing jquery if you have used it (in old or
  new application.scss layout file).

  If you receive error similar to `Refused to execute script from
  because its MIME type ('text/html') is not executable, and strict MIME type
  checking is enabled` that means that asset path is bad, and rails can not find
  that precompiled asset. Sometime this happens only on production env.

## Options

It contains nice features (as stated in their [blog](https://almsaeedstudio.com/blog/features-of-adminlte-2.1))

* toolbar can be autoexpand using js options

You can use colors with background `bg-red` `bg-blue` ... but also for
components and it's status class. For example
[box](https://almsaeedstudio.com/themes/AdminLTE/documentation/index.html#component-box)
could be `box-default`, `box-primary`, `box-info`, `box-success`, `box-danger`
and `box-warning`

You can [collapse
box](https://github.com/almasaeed2010/AdminLTE/blob/master/dist/js/app.js#L562)
in javascript by calling `$.AdminLTE.boxWidget.collapse($('#box-id'));` but it
is better to write custom code.


