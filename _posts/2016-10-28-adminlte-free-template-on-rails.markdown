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

Latest version is 2.4.2 uses bootstrap 3 and jquery 3.
To install to existing rails project you can use [bower inside rails](
{{ site.baseurl }} {% post_url 2014-07-01-ruby-on-rails-layouts-and-rendering %}
#bower) and than add admin-lte

~~~
bower install --save admin-lte boostrap font-awesome ionicons
~~~

I use separated bootstrap (not from admin-lte package) because "Glyphicons" are
properly linked. Also I use latest font awesome.
There is [admin-lte.scss](https://github.com/aguegu/AdminLTE) version but not
updated.

We can use [starter.html](https://adminlte.io/themes/AdminLTE/starter.html) for
testing, just copy from `vendor/assets/bower_components/admin-lte/starter.html`
select `<div class="wrapper">` and put on your page. CSS and js will we using
sprockets and compiled in one big application.css/.js file. Since
adminlte is using less we will use two wrappers.

~~~
// app/assets/application.css
/*
 * load bootstrap here since importing in less will use css import file command
 *= require bootstrap/dist/css/bootstrap.css
 *= require application_less_wrapper
 *= require application_scss_wrapper
 */
~~~

~~~
# Gemfile, belowe sass-rails
# Also need less for adminlte
gem 'less-rails', '~> 3.0.0'
# See https://github.com/sstephenson/execjs#readme for more supported runtimes
gem 'therubyracer'
~~~

~~~
# config/initializers/assets.rb

# We need to add also subpaths admin-lte and admin-lte/skins since there is
# relative reference for bootstrap in skins less files
Rails.application.config.assets.paths << Rails.root.join('vendor', 'assets', 'bower_components')
Rails.application.config.assets.paths << Rails.root.join('vendor', 'assets', 'bower_components', 'admin-lte')
Rails.application.config.assets.paths << Rails.root.join('vendor', 'assets', 'bower_components', 'admin-lte', 'skins')
Rails.application.config.assets.precompile << /\.(?:svg|eot|woff|ttf)$/
~~~

Copy some header from starter.html to your layout file

~~~
# app/views/layouts/applicantion.html.erb
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title><%= content_for?(:page_title) ? yield(:page_title) : default_page_title %></title>
    <!-- Tell the browser to be responsive to screen width -->
    <meta content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" name="viewport">
    <%= csrf_meta_tags %>

    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
    <!-- HTML5 Shim and Respond.js IE8 support of HTML5 elements and media queries -->
    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
    <!--[if lt IE 9]>
    <script src="https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
    <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->

    <!-- Google Font -->
    <link rel="stylesheet"
          href="https://fonts.googleapis.com/css?family=Source+Sans+Pro:300,400,600,700,300italic,400italic,600italic">
  </head>

  <body class="hold-transition skin-blue sidebar-mini <%= sidebar_collapse %>">
    <div class="wrapper">

      <!-- Main Header -->
      <header class="main-header">
        <!-- Logo -->
        <a href="/" class="logo">
          <!-- mini logo for sidebar mini 50x50 pixels -->
          <span class="logo-mini">
            <b>ST</b>
          </span>
          <!-- logo for regular state and mobile devices -->
          <span class="logo-lg">
            <b>Stuff</b>Timesheet
          </span>
        </a>

        <!-- Header Navbar -->
        <nav class="navbar navbar-static-top" role="navigation">
          <!-- Sidebar toggle button-->
          <a href="#" class="sidebar-toggle" data-toggle="push-menu" role="button" data-server-url="<%= preferences_path %>">
            <span class="sr-only">Toggle navigation</span>
          </a>
          <!-- Navbar Right Menu -->
          <div class="navbar-custom-menu">
            <ul class="nav navbar-nav">
              <!-- Messages: style can be found in dropdown.less-->
              <li class="dropdown messages-menu">
                <!-- Menu toggle button -->
                <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                  <i class="fa fa-envelope-o"></i>
                  <span class="label label-success">4</span>
                </a>
                <ul class="dropdown-menu">
                  <li class="header">You have 4 messages</li>
                  <li>
                    <!-- inner menu: contains the messages -->
                    <ul class="menu">
                      <li><!-- start message -->
                        <a href="#">
                          <div class="pull-left">
                            <!-- User Image -->
                            <img src="dist/img/user2-160x160.jpg" class="img-circle" alt="User Image">
                          </div>
                          <!-- Message title and timestamp -->
                          <h4>
                            Support Team
                            <small><i class="fa fa-clock-o"></i> 5 mins</small>
                          </h4>
                          <!-- The message -->
                          <p>Why not buy a new awesome theme?</p>
                        </a>
                      </li>
                      <!-- end message -->
                    </ul>
                    <!-- /.menu -->
                  </li>
                  <li class="footer"><a href="#">See All Messages</a></li>
                </ul>
              </li>
              <!-- /.messages-menu -->

              <!-- Notifications Menu -->
              <li class="dropdown notifications-menu">
                <!-- Menu toggle button -->
                <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                  <i class="fa fa-bell-o"></i>
                  <span class="label label-warning">10</span>
                </a>
                <ul class="dropdown-menu">
                  <li class="header">You have 10 notifications</li>
                  <li>
                    <!-- Inner Menu: contains the notifications -->
                    <ul class="menu">
                      <li><!-- start notification -->
                        <a href="#">
                          <i class="fa fa-users text-aqua"></i> 5 new members joined today
                        </a>
                      </li>
                      <!-- end notification -->
                    </ul>
                  </li>
                  <li class="footer"><a href="#">View all</a></li>
                </ul>
              </li>
              <!-- Tasks Menu -->
              <li class="dropdown tasks-menu">
                <!-- Menu Toggle Button -->
                <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                  <i class="fa fa-flag-o"></i>
                  <span class="label label-danger">9</span>
                </a>
                <ul class="dropdown-menu">
                  <li class="header">You have 9 tasks</li>
                  <li>
                    <!-- Inner menu: contains the tasks -->
                    <ul class="menu">
                      <li><!-- Task item -->
                        <a href="#">
                          <!-- Task title and progress text -->
                          <h3>
                            Design some buttons
                            <small class="pull-right">20%</small>
                          </h3>
                          <!-- The progress bar -->
                          <div class="progress xs">
                            <!-- Change the css width attribute to simulate progress -->
                            <div class="progress-bar progress-bar-aqua" style="width: 20%" role="progressbar"
                                 aria-valuenow="20" aria-valuemin="0" aria-valuemax="100">
                              <span class="sr-only">20% Complete</span>
                            </div>
                          </div>
                        </a>
                      </li>
                      <!-- end task item -->
                    </ul>
                  </li>
                  <li class="footer">
                    <a href="#">View all tasks</a>
                  </li>
                </ul>
              </li>
              <!-- User Account Menu -->
              <li class="dropdown user user-menu">
                <!-- Menu Toggle Button -->
                <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                  <!-- The user image in the navbar-->
                  <img src="dist/img/user2-160x160.jpg" class="user-image" alt="User Image">
                  <!-- hidden-xs hides the username on small devices so only the image appears. -->
                  <span class="hidden-xs">Alexander Pierce</span>
                </a>
                <ul class="dropdown-menu">
                  <!-- The user image in the menu -->
                  <li class="user-header">
                    <img src="dist/img/user2-160x160.jpg" class="img-circle" alt="User Image">

                    <p>
                      Alexander Pierce - Web Developer
                      <small>Member since Nov. 2012</small>
                    </p>
                  </li>
                  <!-- Menu Body -->
                  <li class="user-body">
                    <div class="row">
                      <div class="col-xs-4 text-center">
                        <a href="#">Followers</a>
                      </div>
                      <div class="col-xs-4 text-center">
                        <a href="#">Sales</a>
                      </div>
                      <div class="col-xs-4 text-center">
                        <a href="#">Friends</a>
                      </div>
                    </div>
                    <!-- /.row -->
                  </li>
                  <!-- Menu Footer-->
                  <li class="user-footer">
                    <div class="pull-left">
                      <a href="#" class="btn btn-default btn-flat">Profile</a>
                    </div>
                    <div class="pull-right">
                      <a href="#" class="btn btn-default btn-flat">Sign out</a>
                    </div>
                  </li>
                </ul>
              </li>
              <!-- Control Sidebar Toggle Button -->
              <li>
                <a href="#" data-toggle="control-sidebar"><i class="fa fa-gears"></i></a>
              </li>
            </ul>
          </div>
        </nav>
      </header>
      <!-- Left side column. contains the logo and sidebar -->
      <aside class="main-sidebar">

        <!-- sidebar: style can be found in sidebar.less -->
        <section class="sidebar">

          <!-- Sidebar user panel (optional) -->
          <div class="user-panel">
            <div class="pull-left image">
              <img src="dist/img/user2-160x160.jpg" class="img-circle" alt="User Image">
            </div>
            <div class="pull-left info">
              <p>Alexander Pierce</p>
              <!-- Status -->
              <a href="#"><i class="fa fa-circle text-success"></i> Online</a>
            </div>
          </div>

          <!-- search form (Optional) -->
          <form action="#" method="get" class="sidebar-form">
            <div class="input-group">
              <input type="text" name="q" class="form-control" placeholder="Search...">
              <span class="input-group-btn">
                  <button type="submit" name="search" id="search-btn" class="btn btn-flat"><i class="fa fa-search"></i>
                  </button>
                </span>
            </div>
          </form>
          <!-- /.search form -->

          <!-- Sidebar Menu -->
          <ul class="sidebar-menu" data-widget="tree">
            <li class="header">HEADER</li>
            <!-- Optionally, you can add icons to the links -->
            <li class="active"><a href="#"><i class="fa fa-link"></i> <span>Link</span></a></li>
            <li><a href="#"><i class="fa fa-link"></i> <span>Another Link</span></a></li>
            <li class="treeview">
              <a href="#"><i class="fa fa-link"></i> <span>Multilevel</span>
                <span class="pull-right-container">
                    <i class="fa fa-angle-left pull-right"></i>
                  </span>
              </a>
              <ul class="treeview-menu">
                <li><a href="#">Link in level 2</a></li>
                <li><a href="#">Link in level 2</a></li>
              </ul>
            </li>
          </ul>
        </section>
      </aside>

      <!-- Content Wrapper. Contains page content -->
      <div class="content-wrapper">
        <!-- Content Header (Page header) -->
        <section class="content-header">
          <h1>
            <%= yield :page_header %>
            <small><%= yield :page_description %></small>
          </h1>
          <ol class="breadcrumb">
            <% get_breadcrumb_list.each_with_index do |(text, link), i| %>
              <li class="<%= 'active' if i == get_breadcrumb_list.length - 1 %>">
                <% if link.present? %>
                  <%= link_to link do %>
                    <% if i == 0 %>
                      <i class="fa fa-dashboard"></i>
                    <% end %>
                    <%= text %>
                  <% end %>
                <% else %>
                  <% if i == 0 %>
                    <i class="fa fa-dashboard"></i>
                  <% end %>
                  <%= text %>
                <% end %>
              </li>
            <% end %>

          </ol>
        </section>

        <!-- Main content -->
        <section class="content">

          <% if notice %>
            <div class="hide-to-up"><%= notice %></div>
            <script>
              flash_notice('<%= j notice %>');
            </script>
          <% end %>

          <% if alert %>
            <div class="hide-to-up"><%= alert %></div>
            <script>
              flash_alert('<%= j alert %>');
            </script>
          <% end %>

          <!-- Your Page Content Here -->
          <%= yield %>

        </section>
        <!-- /.content -->
      </div>
      <!-- /.content-wrapper -->

      <!-- Main Footer -->
      <footer class="main-footer">
        <!-- To the right -->
        <div class="pull-right hidden-xs">
          Anything you want
        </div>
        <!-- Default to the left -->
        <strong>Copyright &copy; 2016 <a href="#">Company</a>.</strong> All rights reserved.
      </footer>

      <!-- Control Sidebar -->
      <aside class="control-sidebar control-sidebar-dark">
      </aside>
      <!-- /.control-sidebar -->
      <!-- Add the sidebar's background. This div must be placed
           immediately after the control sidebar -->
      <div class="control-sidebar-bg"></div>
    </div>
    <script>
      <%= yield :javascript %>
    </script>

  </body>
</html>
~~~

Some helpers

~~~
# app/helpers/page_helper.rb
module PageHelper
  def sidebar_collapse
    if current_user&.sidebar_collapse
      'sidebar-collapse'
    end
  end

  def preferences_path
    '/'
  end

  # use in view: page_title "Home"
  def page_title(title)
    # this will add both page title and header below topnav
    content_for(:page_title) { title }
    content_for(:page_header) { title }
  end

  def default_page_title
    add_location_to_title(
      "Internet Subscriber Management"
    )
  end

  # use in view: page_description "Dashboard"
  def page_description(description)
    content_for(:page_description) { description.html_safe }
  end

  # use in view: breadcrumb "Home": root_path, "Dashboard": nil
  def breadcrumb(list)
    @breadcrumb = list
  end

  def get_breadcrumb_list
    @breadcrumb || []
  end
end
~~~

~~~
// app/assets/stylesheets/application_less_wrapper.less
@import "Ionicons/less/ionicons";
// AdminLTE.less stuff need to be after bootstrap which includes normalize.scss
@import "AdminLTE/build/less/AdminLTE";
@import "AdminLTE/build/less/skins/skin-blue";

// default sidebar width is 230px
@sidebar-width: 170px;
~~~

~~~
// app/assets/stylesheets/application_scss_wrapper.scss
// on production all assets gets fingerprint, but here we can define only path
// $fa-font-path: 'font-awesome/fonts';
// so use Bootstrap CDN for font files
$fa-font-path: "//netdna.bootstrapcdn.com/font-awesome/4.7.0/fonts" !default;
@import 'font-awesome/scss/font-awesome';
~~~

~~~
// app/assets/javascript/application_adminlte.js
//= require rails-ujs
//= require turbolinks
//= require_tree .
//= require js.cookie
//= require jstz
//= require browser_timezone_rails/set_time_zone
//
// AdminLTE stuff
//= require jquery/dist/jquery.min
//= require bootstrap/dist/js/bootstrap.min
//= require admin-lte/dist/js/adminlte
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
layout which you have not consider in that moment.

Better approach is to create new folder `app/assets/adminlte` and add it to
asset pipeline paths.

Since asset pipeline [ignore first subfolders]( {{ site.baseurl }}
{% post_url 2014-07-01-ruby-on-rails-layouts-and-rendering %}) you need to take
care to name files differently when you create new scss and coffee files. I
solved that by adding suffix `_adminlte`.

## Notify if template is missing

You can notify if some of the translated templates does not exists using [get
template name]( {{ site.baseurl }} 
{% post_url 2016-04-12-rails-tips %}#get-template-name) and add notification

~~~
# config/initializers/notify_if_adminlte_template_is_not_present.rb
# getting current template is not possible in rails
# https://github.com/rails/rails/issues/4876
# patch taken from
# http://stackoverflow.com/questions/4973699/rails-3-find-current-view-while-in-the-layout/8310881#8310881
class ActionView::TemplateRenderer
  alias_method :_render_template_original, :render_template

  def render_template(template, layout_name = nil, locals = {})
    if I18n.locale == :in &&
      template.inspect.end_with?(".in.html.erb") &&
      template.inspect.end_with?(".in.js.erb") &&
      template.inspect.end_with?(".in.json.jbuilder") &&
      template.inspect.start_with?("app/views/devise/mailer") &&
      template.inspect.start_with?("app/views/mailers") &&
      template.inspect.start_with?("app/views/api") &&
      template.inspect.start_with?("rails/mailers/email") &&
      template.inspect != "text template".freeze && # this is when send_data
      template.inspect != "html template".freeze # this is when render html
      cache_key = "adminlte_#{template.virtual_path}"
      if Rails.cache.fetch cache_key
        # already notified
      else
        Rails.cache.write cache_key, Time.now
        XceednetMailer.internal_notification(
          "Adminlte Template does not exists for #{template.virtual_path}",
          template
        ).deliver_now
      end
    end
    result = _render_template_original( template, layout_name, locals)
    result
  end
end
~~~

# Turbolinks

<https://github.com/almasaeed2010/AdminLTE/issues/563>

~~~
$(document).on 'turbolinks:load', ->
  console.log 'turbolinks:load'
  $(window).trigger('resize')
~~~

# Configuration

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

* [datatables]({{ site.baseurl }}
{% post_url 2016-07-26-usefull-plugins-services-and-opensource-stuff %}
  #datatables) is used for search and ordering on client side
* [bootstrap-datepicker](https://github.com/uxsolutions/bootstrap-datepicker)
* [bootstrap-daterangepicker](https://github.com/dangrossman/bootstrap-daterangepicker)

## Premium plugins

You can check premium version:
[adminlte](http://wrapbootstrap.com/preview/WB0N89JMK) which usually use free
plugins which you can download separatelly. It looks more professional.

* [wrapboostrap inspinia](https://wrapbootstrap.com/theme/inspinia-responsive-admin-theme-WB0R5L90S)
  * checkbox [abpetkov/switchery](https://github.com/abpetkov/switchery)
* [themeforest ucm theme crm](https://themeforest.net/item/ucm-theme-adminlte-crm/8565409?sso=1&_ga=2.183038054.1513538055.1496854657-1559534847.1496854657)

## Options

It contains nice features (as stated in their [blog](https://almsaeedstudio.com/blog/features-of-adminlte-2.1))

* toolbar can be autoexpand using js options

You can use colors with background `bg-red` `bg-blue` ... or for text
`text-red`, `text-blue`...
Colors could be also for components. For example
[box](https://almsaeedstudio.com/themes/AdminLTE/documentation/index.html#component-box)
could be `box-default`, `box-primary`, `box-info`, `box-success`, `box-danger`
and `box-warning`.

You can [collapse
box](https://github.com/almasaeed2010/AdminLTE/blob/master/dist/js/app.js#L562)
in javascript by calling `$.AdminLTE.boxWidget.collapse($('#box-id'));` but it
is better to write custom code.

[rails bootstrap forms](https://github.com/bootstrap-ruby/rails-bootstrap-forms)
works nice with template, for example registration form:

~~~
<%# app/views/devise/registrations/new.html.erb %>
<%= bootstrap_form_for resource, as: resource_name, url: registration_path(resource_name) do |f| %>
  <%= f.email_field :email, autofocus: true, placeholder: "Email", skip_label: true, icon: 'user' %>
  <%= f.password_field :password, class: 'form-control', placeholder: 'Password', skip_label: true, icon: 'lock' %>
  <%= f.password_field :password_confirmation, class: 'form-control', placeholder: 'Password Confirmation', skip_label: true, icon: 'lock' %>
  <button type="submit" class="btn btn-primary btn-block btn-flat" data-disable-with="Processing...">Sign up</button>
<% end %>
<%= render "devise/shared/links" %>
~~~

# Less source

Since AdminLTE is build using less, you can set variables to customize colors
and sizes.

~~~
// app/assets/adminlte/stylesheets/less_wrapper_adminlte.less
// AdminLTE stuff need to be after bootstrap (which includes normalize.scss)
@import "AdminLTE/build/less/AdminLTE";
@import "AdminLTE/build/less/skins/skin-blue";

@sidebar-width: 170px;
~~~

# CRUD and ajax

My prefered setup is to use ajax for edit/update form, use modal for create
(with prefilled form, without ajax, since in case of success it should
redirect).

# Using with existing template

To add new template while keeping existing you can use different layout base on
locale. You can set new layout with option `?layout=true`
So if you add param `?layout=true` it will render `index.in.html.erb` variant
using new layout and put that in session (so all form submittions also use new
layout). You can disable new_layout with `?layout=false`. If you have locale
files (probably `devise.en.yml`) than you need to copy to `:in`

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

