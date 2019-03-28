---
layout: post
title: Bootstrap front-end framerwork
---

# Bootstrap 4

# Bootstrap

`div` is block element and by default it will be stacked. Bootstrap grid is
mobile first, so three `.col-md-4` will be stacked until desktop, so on desktop,
large and extra large desktop it will be like three equal width columns.  You do
not need to define all 12, `.col` will occupy rest of place. `.col-auto` will
occupy only that is needed (based on it's contents, it uses `max-width: 100%`).
When you specify for sm, you do not need to specify for md or lg. `col-xs-12` is
`width: 100%` which is default so no need to write that. Write only when you
need columns.

Bootstrap Media queries:

* xs - phones (no need for media queary since this is default)
* sm - tablets `@media (min-width: map-get($grid-breakpoints, 'sm') { // 576px (Bv3 768px) and up`
* md - desktops `@media (min-width: @screen-md-min) { // 768px (Bv3 992px)`
* lg - larger desktops `@media (min-width: @screen-lg-min) { // 992px (Bv3 1200px)`
* xl - extra lage `@media (min-width: @screen-xl-min) { // 1200pxa`

With bootstrap 4 you can use mixins for media queries:
~~~
// given (sm) and larger
@include media-breakpoint-up(sm) {
  .some-class {
    display: block;

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

Difference with bootstrap 3 is that there is new grid tier `xl` that was `lg` in
B3 (`sm` is 576px instead of 768px in B3) and all are bumped up one level (so
`.col-md-6` in v3 is now `.col-lg-6` in v4)
Boostrap 3 you can use [responsive available
classes](http://getbootstrap.com/css/#responsive-utilities-classes) to toggle
between `display: block, inline, inline-block` or to hide, for specific viewport
size `.visibe-sm-block .hidden-md .hidden-sm-up`

In Bootstrap 4 you can use `d-*-none` class
https://getbootstrap.com/docs/4.1/utilities/display/
To hide on smaller than md (ie show on md and larger) use `d-none d-md-block`
To show only on md use `.d-none .d-md-block .d-lg-none`

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

On B3 you can use ajax and `render layout: false` and provide only inner html of
`<div class="modal"><div class="modal-dialog">...` ie only `.modal-content` will
be replaced. So you should use same header and footer in initial and ajax
version.

On B4 we need to replace the modal content with custom js, so I use jBox modal
in this case.

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

## Bootstrap input group and button group

Buttons and inputs can be grouped, but [input
group](http://getbootstrap.com/components/#btn-groups) is not advised to group
with select. [Here](http://jsfiddle.net/MansukhKhandhar/4309n31p/1/) is example
of input with select and button.

# Helpers

Color helpers
Primary colors are [link](https://getbootstrap.com/docs/4.0/utilities/colors/)
`primary`, `secondary`, `success`, `danger`, `warning`, `info`, `dark`, `light`,
`white`.  For text there is `muted` and also need background `text-light
bg-dark` and `text-white bg-dark`. You can set default body background by
defining variable `$body-bg: #bcdee3` before importing boostrap.
You can use css `darken` for example for variable
`$link-hover-color: darken($link-color, 15%) !default;`
https://github.com/twbs/bootstrap/blob/e7e43edf65306efaf46a16ffc9fe35ef623bffef/scss/_variables.scss#L171

* text and links `.text-success`
* background `.bg-warning`
* buttons `.btn-primary`

Other helpers are:

* position helpers `position-relative`, `position-absolute` https://github.com/twbs/bootstrap/blob/e57a2f244ba8446fffe71847e6a58b18f7b2d541/scss/utilities/_position.scss
* `.clearfix`
* text size `.h1`(2.5rem), `.h2`(2rem), `.h3`(1.75rem), `.h4`(1.5rem), `.h5`(1.25rem), `.h6`(1rem) so you can use `h2` instead defining `font-size-2`
* `font-weight-bold` to use bold font
* text position`.text-center`, `.text-left`, `.text-nowrap`
 https://www.w3schools.com/bootstrap/bootstrap_ref_css_text.asp
* `text-truncate` with `max-width: 100px`
  https://getbootstrap.com/docs/4.1/utilities/text/#text-wrapping-and-overflow
* `.show` and `.hidden`
* `.d-none` to hide something. But when I want to show/hide in js I usually
  create my own `.d-none-not-important` since bootstrap's is with `important`.
  I used `hide-not-important` with jQuery `$el.show()` (hide) or
`$el.slideDown()` (slideUp) but they use `display: block` which does not work
well with `display: flex` elements, better is to use
`$(el).addClass('d-none-not-important')`. There is also
[hidden](https://getbootstrap.com/docs/4.0/content/reboot/#html5-hidden-attribute)

~~~
# app/assets/stylesheets/common/show_hide.sass
.d-none-not-important,.hide-not-important
  visibility: hidden
  opacity: 0
  transition: visibility 0.5s ease, opacity 0.5s ease
  // display: none
  &.active
    visibility: visible
    opacity: 1
    // display: initial
~~~
* margin and padding helpers in format `<property><sides>-<breakpoint>-<size>`
  <https://getbootstrap.com/docs/4.0/utilities/spacing/#notation> 
  property is `m` margin or `p` padding
  sides `t` top, `b` bottom, `l` left, `r` right, `x` both l and r, `y` t and b
  size `0`, `1` (`$spacer * 0.25`), `2` (`$spacer * .5`), `3` (`$spacer`), `4`
  (`$spacer * 1.5`), `5` (`$spacer * 3`), By default `$spacer: 1rem` 1rem is
  default font size ie 16px on B4 (1.5 line height) and 14px in B3 (1.428 line
  height)

  `mx-auto` for `margin: 0px auto`.
  sides or breakpoint not required: `p-0` padding none, `mt-5` margin top,

  For inline elements you can use `float-left` (instead of `pull-left`).
  For block elements `ml-auto` to align to the right (same as
  pull-right in B3), `mx-auto` for horizontal centering (need fixed width).
  For block elements you can wrap inside `d-flex w-100 justify-content-between`
  width 100% is important because you want right element to go right.
  https://getbootstrap.com/docs/4.0/components/list-group/#custom-content
  (also `justify-content-around justify-content-center justify-content-start`).
  For cross axis it will be scretched so use `d-flex align-items-center` to
  center verticaly (or horizontaly if it is column based).
  You can also align for particular item `align-self-center`.
  Grow item with `flex-grow-1` or `flex-shrink-1`.

* `d-flex` class to convert to flexbox container. You can change display with
  `d-inline`, `d-inline-block`.  Other flex helpers
  https://getbootstrap.com/docs/4.0/utilities/flex/
  for example to enable wrap use `d-flex flex-wrap`. You can use those classes
  instead of row col. For example
  ```
  <div class='d-flex flex-column flex-md-row'>
    <div></div>
    <div></div>
  </div>
  is the same as
  <div class='row'>
    <div class='col-md-6'></div>
    <div class='col-md-6'></div>
  </div>
  ```

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

## Overrides

There are two files that configure and override bootstrap

```
# app/stylesheets/application.sass
// default
@import 'common/variable_default_values'

// node_modules
@import 'bootstrap/scss/bootstrap'

// common
@import 'common/variables'
@import 'common/bootstrap_overrides'
```

```
# app/assets/stylesheets/common/bootstrap_overrides
// it is not actually overrides, but additional classes that modify existing

// this is used if you want btn inside h1
.btn.btn__font-size-inherit
  font-size: inherit
```

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

sed -i app/assets/stylesheets/application.css -e '1i\
/*\
 *= require rails_bootstrap_forms\
 */'
# or sass
cat >> app/assets/stylesheets/application.sass << HERE_DOC
// gems
@import 'rails_bootstrap_forms'
HERE_DOC
~~~

and use your `bootstrap_form_for`

You can use [horizontal
forms](https://github.com/bootstrap-ruby/rails-bootstrap-forms#horizontal-forms)
with `layout: :horizontal` (note that it won't work for string `layout:
'horizontal'`). You can add options `label_col: 'col-sm-4', control_col:
'col-sm-8'`. Also you can override `control_class`, `wrapper_class` (if it is a
hash you can override any option).

If you want all controlls and submit button to be in one line, you can use
`layout: :inline`.

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

You can prepend or append strings or buttons

~~~
  <%= f.text_field :price, append: '$' %>
  <%= f.text_field :body, append: f.primary(t('send')) %>
~~~

Checkboxes or radio buttons should be inside a `form-group` if you want to show
them indented (inside wrapper class) and to show label (control class).

~~~
  <%= f.form_group label: { text: 'Admin' }, help: 'This option enables admin' do %>
    <%= f.check_box :admin, label: '' %>
  <% end %>
~~~

To hide label instead of `label: false` you need to use `hide_label: true`

Static test can be displayed with

~~~
<%= f.static_control :email %>
<%= f.static_control label: 'Custom Static Control' do %>
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

Oneliner form is using button_to with: input label, target url, form class...
```
<%= button_to t('notify'), notify_device_path(device), class: 'btn btn-sm btn-secondary', title: t('send_notification_to_this_device'), form_class: 'd-inline' %>
```

You can change default `col-sm-2` control col class

```
# config/initializers/bootstrap_form.rb
module BootstrapForm
  class FormBuilder
    def default_label_col
      'col-sm-3'
    end
    def default_control_col
      'col-sm-9'
    end
  end
end
```

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

## Navbar

* `navbar` is starting class which `display: flex; flex-wrap: wrap; align-items:
  center; justify-content: space-between` (so if you have two childs like
  `navbar-brand` and `navbar-toggler` they will be on left and right side, but
  `navbar-collapse` has `flex-grow: 1; flex-basis: 100%` so that's why it looks
  like it's on left, also `navbar-expand-lg` has `justify-content: flex-start`).
* `navbar-light bg-light` to add colors (it will change color of nav-links)
* `navbar-collapse` contains all navbar. add `navbar-expand-lg` on navbar so on
  lg `navbar-toggler` dissapears, `navbar-collapse` expands and `navbar-nav`
  change from `column` to `flex-direction: row`. Inside `navbar-collapse` beside
  `navbar-nav` you can also nest one more `navbar-nav ml-auto` if you want to
  pull some links to right.

## Cards

```
<div class="card" style="width: 18rem;">
  <img class="card-img-top" src=".../100px180/" alt="Card image cap">
  <div class="card-body">
    <h5 class="card-title">Card title</h5>
    <p class="card-text">Some quick example text to build on the card title and make up the bulk of the card's content.</p>
    <a href="#" class="btn btn-primary">Go somewhere</a>
  </div>
</div>
```

Cards occupy full width, so you need to set width manually.
You can use `card-group` or `card-deck` (it has padding), or you can simply add
`d-flex flex-wrap` to wrapped element.
`card-body` is used for padding.

```
# app/assets/stylesheets/components/sizes.sass
.card--max-width
  @include media-breakpoint-up(md)
    // single card on sm and start with two on a md(minus two margins m-2)
    max-width: map-get($grid-breakpoints, 'md') / 2 - to-px(2 * map-get($spacers, 2))

# app/views/cards/index.html
<div class='d-flex flex-wrap justify-content-center'>
  <% @tasks.all.each do |task| %>
    <div class='card m-2 card--max-width'>
      <div class='card-body'>
```

To add buttons to card using list group, use flush to remove some borders and
render edge to edge, use `btn` to add hover icon

```
<div class='card'>
  <div class='list-group list-group-flush text-center'>
    <%= button_tag class: 'list-group-item list-group-item-action btn' do %>
      <i class="fa fa-pencil-alt" aria-hidden="true"></i>
      <%= t('edit') %>
    <% end %>
  </div>
```


## Login box

On responsive, instead of margin (margin is used with auto so it is
centered on bigger screens) you can use `max-width: 90vw` or better is to wrap
inside container with `padding: 0.5rem`.
Since I like to put a logo and text which is larger than box, I use two times
max with and mx-auto, one for wrapper (60rem) and one for white box (sm size).


```
# app/assets/stylesheets/common/sizes.sass
.login--wrapper-max-width
  max-width: 60rem

.login--max-width
  max-width: 30rem

# app/views/layouts/application.html.erb
  <% if login_layout? %>
    <body>
      <div class='text-center'>
        <%= link_to root_path do %>
          <%= image_tag 'cable_crm_logo.png', class: 'login-logo' %>
        <% end %>
        <h2><%= login_title %></h2>
      </div>
      <div class='container'>
        <div class='login--wrapper-max-width mx-auto'>
          <div class="<%= 'login--max-width mx-auto bg-white shadow p-4' unless controller_name == 'companies' %>">
            <%= yield %>
          </div>
        </div>
      </div>
    </body>
  <% else %>
```

## Sidebar

https://bootstrapious.com/p/bootstrap-sidebar#3-fixed-scrollable-sidebar-menu-with-a-content-overlay
```
# app/assets/stylesheets/components/sidebar.sass
$sidebar-width: 20rem
#sidebar
  &.active
    right: 0
  min-width: $sidebar-width
  max-width: $sidebar-width
  height: 100vh
  position: fixed
  top: 0
  right: -$sidebar-width
  background: #7386D5
  color: #fff
  transition: right 0.3s
  z-index: 9999

.sidebar__overlay
  display: none
  position: fixed
  width: 100vw
  height: 100vh
  background: rgba(0, 0, 0, 0.7)
  z-index: 998
  opacity: 0
  transition: all 0.5s ease-in-out
  &.active
    display: block
    opacity: 1

.sidebar__dismiss
  width: 35px
  height: 35px
  position: absolute
  top: 10px
  right: 10px

# app/views/layouts/_sidebar.html.erb
<nav id='sidebar'>
  <div class='sidebar__dismiss' data-toggle-active='#sidebar,.sidebar__overlay'>
    <i class="fa fa-times"></i>
  </div>

  <ul class='list-unstyled'>
    <p>My App</p>
    <li class='<%= 'active' if controller_name == 'tasks' %>'>
      <%= link_to 'Past tasks', '#' %>
    </li>
  </ul>
</nav>
<div class='sidebar__overlay' data-toggle-active='#sidebar,.sidebar__overlay'></div>
```

# Improvements

* create classes for text primary for list groups. There is only a color
  variants `list-group-item-primary` but I do not see variant with white
  background and blue text
  https://github.com/twbs/bootstrap/blob/v4-dev/scss/_list-group.scss#L148
  My fix is:
  ```
  // by defauls list-group-item-action.active background is white, but when we add
  // text-primary if overrides that, so we need to define again
  .list-group-item-action.active.text-primary
    color: #ffffff !important
    &:hover
      color: #ffffff !important
  ```
