---
layout: post
title: Bootstrap front-end framerwork
---

# Bootstrap 4

# Bootstrap

Bootstrap grid is mobile first, so three `.col-md-4` will be stacked until
desktop and large desktop, where it will be three equal width columns.
You do not need to define all, `.col` will occupy rest of place.
Bootstrap Media queries:

* xs - phones (no need for media queary since this is default)
* sm - tablets `@media (min-width: @screen-sm-min) { // 576px (Bv3 768px) and up`
* md - desktops `@media (min-width: @screen-md-min) { // 768px (Bv3 992px)`
* lg - larger desktops `@media (min-width: @screen-lg-min) { // 992px (Bv3 1200px)`
* xl - extra lage `@media (min-width: @screen-xl-min) { // 1200pxa`

So when you specify for sm, you do not need to specify for md or lg.

With bootstrap 4 you can use mixins for media queries:
~~~
@include media-breakpoint-up(sm) {
  .some-class {
    display: block;
  }
}

// or other direction (given and smaller)
@include media-breakpoint-down(sm) {}

// only given size
@include media-breakpoint-only(sm) {}

// multiple sizes
@include media-breakpoint-between(md, xl) {}
@media (min-width: 768px) and (max-width: 1199.98px) { ... }
~~~

With Bootstrap3 you can use their predefined media queries with `min-width`
(mobile first), for example

~~~
@media(min-width:$screen-sm){
  .alert {
    position: absolute;
    bottom: 0px;
    right: 20px;
  }
}

// or in other direction (given and smaller)
@media(max-width: $screen-sm) { }
// or better do not use boundary
@media(max-width: 575.98px) { }
~~~

Bootstrap `container` has fixed width for big devices.

Boostrap 3 you can use [responsive available
classes](http://getbootstrap.com/css/#responsive-utilities-classes) to toggle
between `display: block, inline, inline-block` or to hide, for specific viewport
size `.visibe-sm-block .hidden-md .hidden-sm-up`

In Bootstrap 4 you can use `d-*-none` class
<https://getbootstrap.com/docs/4.0/utilities/display/#hiding-elements>
For example to hide on smaller than md use `d-none d-md-block`


You can write your own to match all greater (or smaller) sizes:

~~~
# when it is max-width than it is Non-Mobile First Method
# so we use le (less or equal then)
# (min-width is Mobile First and we use hide-ge-)
/* Large Devices, Wide Screens */
@media only screen and (max-width : 1200px) {
  .hide-le-lg {
    display: none;
  }
}
/* Medium Devices, Desktops */
@media only screen and (max-width : 992px) {
  .hide-le-md {
    display: none;
  }
}
/* Small Devices, Tablets */
@media only screen and (max-width : 768px) {
  .hide-le-sm {
    display: none;
  }
}
/* Extra Small Devices, Phones */
@media only screen and (max-width : 480px) {
  .hide-le-xs {
    display: none;
  }
}
~~~

* you can center bootstrap column with offset
* you can set icons on inputs, easy with generators, just add `prepend:
'password'`

## Bootstrap modal

You can use ajax, but than `render layout: false` and provide only inner html of
`<div class="modal"><div class="modal-dialog">...` ie only `.modal-content` will
be replaced. So you should use same header and footer in initial and ajax
version.

To close modal with escape key you need to add tabindex on modal elemenent:
`<div class="modal" tabindex="-1">`. Note that on Firefox ESC closing modal
works without this tabindex attribute. Note also that select2 dows 

On activation button you do not need `data-keyboard="true"` since that is
default. But remove `data-keyboard="false"` if exists and you want to close
modal with escape key.

You can use `autofocus="true"` option to focus input field when modal shows up.

For [tooltips](http://getbootstrap.com/javascript/#tooltips) you can use html
version `data-html="true" data-toggle="tooltip" title="<h2>Hi</h2>"`.

If you use ajax than modal content is cached, you can clear that cache on shown
or hidden event (or in ajax response).

~~~
<script>
$('#assign-location-modal').on('shown.bs.modal', function () {
  $(this).removeData('bs.modal');
});
</script>
# or for all, for example in document_on.coffee
$(document).on 'hidden.bs.modal', '.modal', ->
  $(this).removeData('bs.modal')
  console.log "remove boostrap modal"
~~~

You can hide close modal with `$('#my-modal').modal('toggle');`

Usually `fade` makes some problem with positioning so you should not use fadding
for custom modals.

## Bootstrap tooltip

To inspect `data-toggle="tooltip"` you need to find it's selector and
manually call `$('.myel').tooltip("show");` or in developer tools when you
select `<a title=` element you can reference it with `$0` so command is
`$($0).tooltip("show")` or you can increase delay for hide

~~~
  $('[data-toggle="tooltip"]').tooltip(
    delay:
      hide: 100000
  )
~~~

If it is inside element with `overflow-y: scroll;` than it won't be seen if it
goes over borders. You need to change container of the tooltip. You can do it
simply for all tooptips by overwriting DEFAULTS (if you need more customization,
you can overwrite any prototype of the class)

~~~
$.fn.tooltip.Constructor.DEFAULTS.container = 'body'
$.fn.tooltip.Constructor.DEFAULTS.placement = 'right'
~~~

## Bootstrap input group and  button group

Buttons and inputs can be grouped, but [input
group](http://getbootstrap.com/components/#btn-groups) is not advised to group
with select. [Here](http://jsfiddle.net/MansukhKhandhar/4309n31p/1/) is example
of input with select and button.

# Color helpers

Primary colors are [link](https://getbootstrap.com/docs/4.0/utilities/colors/)
`primary`, `secondary`, `success`, `danger`, `warning`, `info`, `dark`, `light`,
`white`.  For text there is `muted` and also need background `text-light
bg-dark` and `text-white bg-dark`.

* text `.text-success`
* background `.bg-warning`
* buttons `.btn-primary`

Other helpers are:

* `.clearfix`
* `.text-center`, `.text-left`, `.text-nowrap`
* `.show` and `.hidden`
* I usually create my own `.hide-not-important` since bootstrap's `hide` and
`hidden` are with `important`. Do not use jQuery `$el.show()` (hide) or
`$el.slideDown()` (slideUp) since they use `display: block` which does not work
well with `display: flex` elements, better is to use
`$(el).addClass('hide-not-important')`. There is also
[hidden](https://getbootstrap.com/docs/4.0/content/reboot/#html5-hidden-attribute)

  ~~~
  .hide-not-important {
    display: none;
    &.active {
      display: initial;
    }
  }
  ~~~
* margin and padding helpers in format `<property><sides>-<breakpoint>-<size>`
  <https://getbootstrap.com/docs/4.0/utilities/spacing/#notation> 
  property is `m` margin or `p` padding
  sides `t` top, `b` bottom, `l` left, `r` right, `x` both l and r, `y` t and b
  size `0`, `1` (`$spacer * 0.25`), `2` (`$spacer * .5`), `3` (`$spacer`), `4`
  (`$spacer * 1.5`), `5` (`$spacer * 3`), `auto` for margin auto. Spacer is 1rem

  sides or breakpoint not required: `p-0` padding none, `mt-5` margin top,
  `ml-auto` to align to the right, `mx-auto` for horizontal centering of fixed
  width block elements.  If you put inside `d-flex` than it will be scretched so
  use `d-flex align-items-center` to center verticaly.

* `d-flex` class to convert to flexbox container.  Other flex helpers
  https://getbootstrap.com/docs/4.0/utilities/flex/
  for example to enable wrap use `d-flex flex-wrap`

* `dl-horizontal` can be replaces with
  ~~~
  <dl class='row justify-content-start'>
    <dt class='col-sm-2'>Sign In Count</dt>
    <dd class='col'><%= @user.sign_in_count %></dd>
    <dt class='w-100'></dt>
    <dt class='col-sm-2'>Last Sign In at</dt>
    <dd class='col'><%= @user.last_sign_in_at.to_s :short %></dd>
    <dt class='w-100'></dt>
  </dl>
  ~~~

# Rails

## Bootstrap generators

This provides scaffold files

~~~
cat >> Gemfile << HERE_DOC
gem 'bootstrap-generators', '~> 3.3.4'
HERE_DOC
bundle
rails generate bootstrap:install --force # this will overwrite application.html.erb and
# generate lib/templates and stylesheets/boostrapp-generatora/variables.scss

git rm app/assets/stylesheets/application.css
cat > app/assets/stylesheets/application.scss << HERE_DOC
@import "bootstrap-generators";
HERE_DOC
git add app/assets/stylesheets/application.scss
~~~

## Sass installation

This is original twitter bootstrap sass port

~~~
cat >> Gemfile << HERE_DOC
# twitter boostrap sass https://github.com/twbs/bootstrap-sass
gem "bootstrap-sass", '~> 3.3.7'
HERE_DOC
bundle

# cat > app/assets/stylesheets/application.scss << HERE_DOC
# // "bootstrap-sprockets" must be imported before "bootstrap" and "bootstrap/variables"
# @import "bootstrap-sprockets";
# @import "bootstrap";
# HERE_DOC

# sed -i app/assets/javascripts/application.js -e '/jquery_ujs/a \
# //= require bootstrap-sprockets' # this is not needed since previous command
# will insert //= require bootstrap

# cat >> app/assets/stylesheets/application.scss << HERE_DOC
# // fixed navbar needs padding http://getbootstrap.com/components/#navbar-fixed-top
# body {
#   padding-top: 60px;
# }
# HERE_DOC

git add . && git commit -m "Adding boostrap"
~~~

~~~
# add devise links
cat > /tmp/template <<\HERE_DOC
        </ul>
        <ul class="nav navbar-nav">
          <% if current_user %>
            <li class="<%= 'active' if %(forms questions).include? params[:controller] %>">
              <%= link_to "My Forms", forms_path %>
            </li>
          <% end %>
        </ul>
        <ul class="nav navbar-nav navbar-right">
          <% if current_user %>
            <li class="dropdown">
              <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false"><%= current_user.email %> <span class="caret"></span></a>
              <ul class="dropdown-menu">
                <li><a href="#">Action</a></li>
                <li><a href="#">Another action</a></li>
                <li><a href="<%= edit_user_registration_path %>">Change password</a></li>
                <li role="separator" class="divider"></li>
                <li><a href="<%= destroy_user_session_path %>" data-method="delete">Sign out</a></li>
              </ul>
          <% else %>
            <li><a href="<%= new_user_registration_path %>">Sign up</a></li>
            <li><a href="<%= new_user_session_path %>">Log in</a></li>
            <li><%= link_to "Sign in with Facebook", user_omniauth_authorize_path(:facebook) %></li>
            <li><%= link_to "Sign in with Google", user_omniauth_authorize_path(:google_oauth2) %></li>
          <% end %>
        </ul>
HERE_DOC

sed -i app/views/layouts/application.html.erb -e '/<.ul>/ {
  r /tmp/template
  d
}'
~~~

## Nabar

https://getbootstrap.com/docs/4.0/examples/navbars/ shows different styles

Here is an example adding style for specific media for alert

~~~
// app/assets/stylesheets/common.scss
@import "bootstrap-variables";

.hidden-left {
  position: absolute;
  left: -1000px;
}

@media(min-width:$screen-sm){
  .alert {
    position: absolute;
    top: 8px;
    padding: 8px;
    padding-right: 30px; // close sign
    z-index: 9999;
    left: 480px; // enought to see nav buttons
  }
}
~~~

Still, default rails form build will render `field_with_errors` on label and
input wrap, so if you need to change for bootstrap error classes.
Note that with Bootstrap 3, you have to change `control-group` to `form-group`,
add `form-control` to `<input>` elements, `help-inline` to `help-block`, and
`warning` to `has-warning` or `error` to `has-error`.
The easiest approach is with **bootstrap form**.

## Bootstrap form

[bootstrap_form gem rails-bootstrap-forms
git](https://github.com/bootstrap-ruby/rails-bootstrap-forms) (not old pluralize
[bootstrap_forms](https://github.com/sethvargo/bootstrap_forms)). Just add two
lines

~~~
cat >> Gemfile << HERE_DOC
# adding bootstrap_form_for
gem 'bootstrap_form'
HERE_DOC

sed -i app/assets/stylesheets/application.scss -e '1i\
/*\
 *= require rails_bootstrap_forms\
 */'
~~~

and use your `bootstrap_form_for`

You can use [horizontal
forms](https://github.com/bootstrap-ruby/rails-bootstrap-forms#horizontal-forms)
with `layout: :horizontal` (note that it won't work for string `layout:
'horizontal'`). You can add options `label_col: 'col-sm-4', control_col:
'col-sm-8'`. Also you can override `control_class`, `wrapper_class` (if it is a
hash you can override any option).

you can make checkbox inline with button

~~~
  <%= f.check_box :remember_me, inline: true %>
  <%= f.submit t "sign_in" %>
~~~

You can use form to have horizontal layout, and some input group can be inline,
so you can group input and select (note that is you have horizontal layout you
can not use this since col-sm-2 and col-sm-10 will be added, look below for
custom form builder).

~~~
<%# app/views/todos/_form.html.erb %>
<%= bootstrap_form_for @task, url: todos_path do |f| %>
  <div class="form-inline">
    <%= f.label :name %>
    <div class="input-group input-group__with-select">
      <%= f.text_field :name, input_group_class: 'input-group-under-group-with-select', skip_label: true %>
      <%= f.select :id, 1..5, skip_label: true %>
    </div>
  </div>
<% end %>
~~~

~~~
# app/assets/stylesheets/application.scss
.input-group__with-select {
  .input-group-under-group-with-select,.form-group {
    position: initial;
    display: initial;
    border-collapse: initial;
  }
  .form-control {
    width: 50%!important;
  }
}
~~~

You can prepend or append strings.

~~~
  <%= f.text_field :price, append: '$' %>
~~~

Checkboxes should be inside a `form-group`

~~~
  <%= f.form_group label: { text: 'Admin' } do %>
    <%= f.check_box :admin, label: '' %>
  <% end %>
~~~

Static test can be displayed with

~~~
<%= f.static_control :email %>
<%= f.static_control labe: 'Custom Static Control' do %>
  Content here
<% end %>
~~~

You can use original rails form builder appending `_without_bootstrap`

~~~
  <%= f.text_field_without_bootstrap :name %>
~~~

## Form builder

You can write your own form builder that extends for example
[rails-bootstrap-forms](https://github.com/bootstrap-ruby/rails-bootstrap-forms)
But since `bootstrap_form_for` (which is defined inside helper) calculate layout
class (`:inline` or `:horizontal`) we need to use same helper
(`bootstrap_form_for ... builder: MyFormBuilder` instead of `form_for ...
builder: MyFormBuilder`). I noticed huge chrome memory problems (memory chrome
is increasing when there is no `form-horizontal` class).

Here is example of input with select base od
[example](http://jsfiddle.net/MansukhKhandhar/4309n31p/1/)

~~~
# app/form_builders/my_form_builder.rb
class MyFormBuilder < BootstrapForm::FormBuilder
  def my_text_field(method, options = {})
    content_tag :div, class: "my-wrapper" do
      super method, options
    end
  end
  def text_field_with_select(method_for_text_input, options_for_text_input, method_for_select, choices, options_for_select = {}, html_options = {})
    content_tag :div, class: "from-group form-group__with_select" do
      form_group_builder method_for_text_input, options_for_text_input do
        select_without_group = form_group_builder_without_group(method_for_select, options_for_select, html_options) do
          select_without_bootstrap(method_for_select, choices, options_for_select, html_options)
        end
        text_field_without_bootstrap(method_for_text_input, options_for_text_input) +
          select_without_group
      end
    end
  end

  # this will overwrite all f.submit form helpers. data-disable-with does not
  # work well when responding with csv so there you should explicitly disable:
  # <%= f.submit "Generate CSV", 'data-disable-with': nil %>
  def submit(name, options = {})
    options.reverse_merge! 'data-disable-with': "<i class='fa fa-spinner fa-spin'></i> #{name}"
    hidden_field_tag(:commit, name) +
      button(name, options)
  end

  def single_image_upload(method, options = {})
    
  end

  private

  def form_group_builder_without_group(method, options, html_options = nil)
    # copy from original form_group_builder
    # bootstrap_form-2.5.2/lib/bootstrap_form/form_builder.rb
    options.symbolize_keys!
    html_options.symbolize_keys! if html_options

    # Add control_class; allow it to be overridden by :control_class option
    css_options = html_options || options
    control_classes = css_options.delete(:control_class) { control_class }
    css_options[:class] = [control_classes, css_options[:class]].compact.join(" ")

    options = convert_form_tag_options(method, options) if acts_like_form_tag

    wrapper_class = css_options.delete(:wrapper_class)
    wrapper_options = css_options.delete(:wrapper)
    help = options.delete(:help)
    icon = options.delete(:icon)
    label_col = options.delete(:label_col)
    control_col = options.delete(:control_col)
    layout = get_group_layout(options.delete(:layout))
    form_group_options = {
      id: options[:id],
      help: help,
      icon: icon,
      label_col: label_col,
      control_col: control_col,
      layout: layout,
      class: wrapper_class
    }

    if wrapper_options.is_a?(Hash)
      form_group_options.merge!(wrapper_options)
    end

    unless options.delete(:skip_label)
      if options[:label].is_a?(Hash)
        label_text  = options[:label].delete(:text)
        label_class = options[:label].delete(:class)
        options.delete(:label)
      end
      label_class ||= options.delete(:label_class)
      label_class = hide_class if options.delete(:hide_label)

      if options[:label].is_a?(String)
        label_text ||= options.delete(:label)
      end

      form_group_options.merge!(label: {
        text: label_text,
        class: label_class,
        skip_required: options.delete(:skip_required)
      })
    end
    form_group_without_group(method, form_group_options) do
      yield
    end
  end

  def form_group_without_group(*args, &block)
    options = args.extract_options!
    name = args.first

    options[:class] = ["form-group", options[:class]].compact.join(' ')
    options[:class] << " #{error_class}" if has_error?(name)
    options[:class] << " #{feedback_class}" if options[:icon]

    control = capture(&block).to_s
    control.concat(generate_help(name, options[:help]).to_s)
    control.concat(generate_icon(options[:icon])) if options[:icon]

    control
  end
end
end
~~~

## Data confirm modal

When rails use `data-confirm='Are you sure?'` it opens default browser's builtin
`confirm()`. Instead of that, you can open bootstrap 3 or 4 modal with
<https://github.com/ifad/data-confirm-modal>
