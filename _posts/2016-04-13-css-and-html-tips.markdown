---
layout: post
title: CSS and HTML tips
---

* when you want to have smaller input field type number (smaller in terms of
  width) you need to provide both `min` and `max`, like `<input type="number"
  min="0" max="99">`
* to hide long text on div/dt `float: left; width: 100px` than:
  * text can be on one line `white-space: nowrap;overflow: hidden;text-overflow:
  ellipsis;` no wrap means it will no go to the next line
  * multiple lines `white-space: normal`
  * `word-wrap: break-word`

* if you have `<small>` position relative, and apply left -100px, it will still
  occupy the space where it was. It is better to use `position: absolute`
* to move element to the right but keep inside parent element you need to mark
  parent as `position: relative` and than use `position: absolute; right: 10px`
* align element at bottom use position relative/absolute pair, for parrent
  `position: relative` (does not have any effects) and for child: `position:
  absolute;bottom: 10px;`. This
  [question](http://stackoverflow.com/questions/17656623/position-absolute-scrolling)
  explains why outer need to be relative.
* use `vertical-align: baseline` for image to be on same bootom as other
  elements
* use `mouseenter` instead `mouseover` (which will retrigger on inner elements).
* use relative units instead of absolute
  <http://www.w3schools.com/cssref/css_units.asp>, for example header `h1 {
  font-size: 20vh;}`. vh is 1% of vertical heigh of view port (window size, not
  page size). For width you can use horizontal width for example `width: 80vw;`
  another relative unit is `em` (1em is default font size usually 16px,
  bootstrap was 14px now is 16px)

* you can not set the width of inline elements, so to set width of `span` you
  need to make it `display:inline-block; width: 100px`
* also top and bottom padding has no efect for label since it is an inline
  element
  [link](http://stackoverflow.com/questions/7168658/why-is-the-padding-not-working-on-my-label-elements).
  You need to make the label a block level element for it to work.
* to make vertical align you need parent to be `position: relative`, and target
  to be `position:absolute; top: 50%; margin-top: -1rem;` (if you show text
  1rem).
* for block elemenent with `margin: auto` (bootstrap
  [center-block](http://getbootstrap.com/css/#helper-classes-center) helper)
  does not have effect unless the element has the width (because it can not
  calculate margins). For text you can use `text-center` `text-right` helper
  classes for to align text right.
* space inside a element is important. for example `<label>1</label><label> 2
  </label><label> 3 </label>` with `label { background: blue;}` this background
  will occupy only character and it will be disconnected from 1-2, but connected
  2-3. This margin-like separation will also be presented if `label { display:
  inline-block; }`. It seems that space is important only with inline elements.
* if you need to render divs horizontaly (`float: left`) you need parent to have
  `width = number_of_items * (item_width+2*item_border_width)` I calculate that
  in javascript

  ~~~
  <script>
    $(function() {
      $('.slots-preview').width($('.slots-preview-item').first().outerWidth() * <%= scheduler.number_of_free_slots.count %>);
    });
  </script>
  ~~~

  You can add scrollbar by wrapping it `.outer { overflow-x: scroll }`
  [link](http://stackoverflow.com/questions/9672176/prevent-floated-divs-from-wrapping-to-new-line)
* textarea (rails `f.text_area`) has constant size, so if you want bigger you
  can set `rows="30"` (and `cols="150"`) or use
  [autosize](http://www.jacklmoore.com/autosize/)

* when you use padding than child with `width: 100%` it is calculated based on
  parent content (parent border and padding is added). Solution is to use
  `box-sizing: border-box` in which calculated size will include border and
  padding <https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing>. This is
  usually included in bootstrap. Problem with padding arise when you use
  `position: relative` on parent and `position: absolute` on child
  <https://stackoverflow.com/questions/17115344/absolute-positioning-ignoring-padding-of-parent>
  Solution is to add relatively position div with no padding around absolutely
  position div. So when you are using padding and absolute position child,
  always add child relative wrapper with no padding.
  Another solution is to use `left: 0;` on absolutively child.

# Examples

* nice radio and checkbox <http://codepen.io/BrianSassaman/pen/iLrpC?editors=1100>

* to see cursor position just run this code in console (it works with bootstrap
  title)

  ~~~
  document.onmousemove = function(e){
  var x = e.pageX;
  var y = e.pageY;
  e.target.title = "X is "+x+" and Y is "+y;
  };
  }
  ~~~

* hide something but still occupy space you can use `visibility: hidden` and
  `visibility: visible`
* glitch efect <https://tympanus.net/Development/GlitchSlideshow/index2.html>

# SCSS Sass

* you can set default value of variable `$my-var: 123 !default;`. This has no
  effect if variable is already defined.
  In CSS you can define var with `--main-color: black;` and use with `color:
  var(--main-color);`. Var are usually decraler on pseudo class `:root {
  --main-color: black; }` so it is accessible everywhere, instead of local scope
  (since this is custom property, it is accessible only on matching selector and
  its descendants) which is used only for specific elements like BEM
* <https://responsivedesign.is/develop/getting-started-with-sass>
* you can select `this` using `&`

  ~~~
  .a:hover {
    background: blue;
  }
  // is the same as
  .a {
    &:hover {
      background: blue;
    }
  }
  ~~~

* prefer mixins to `@extend`

* chrome 39 for android use different color for toolbar, just add `<meta
  name="theme-color" content="#db5945">`


# Head meta tags

* here is a list of used tags [HEAD](https://github.com/joshbuchea/HEAD)

# Css helper classes

~~~
// add some space below
.m-b-10 {
  margin-bottom: 10px;
}

// similar to pull-right just without float
.text-align-right {
  text-align: right;
}

// hide  submit buttons, since android has problems when element is not visible
.hide-to-up {
  position:absolute;
  top: -1000px;
}
~~~

~~~
// Here is example to show hide buttons that are not allowed for first or last
//
// <ol>
//   <!-- iterate over li -->
//   <li  class="hide-first-child hide-last-child">
//     <a class="hide-first-target"><i class="fa fa-arrow-up"></li></a>
//     <a class="hide-last-target"><i class="fa fa-arrow-down"></li></a>
//   </li>
// </ol>

.hide-first-child:first-child .hide-first-target {
  display: none;
}

.hide-last-child:last-child .hide-last-target {
  display: none;
}

// if you need to nest two show hide (you have another ul inside li)
.hide-first-child:first-child {
  .hide-first-target {
    display: none;
  }
  .hide-first-child {
    .hide-first-target {
      display: initial;
    }
    &:first-child {
      .hide-first-target {
        display: none;
      }
    }
  }
}

.hide-last-child:last-child {
  .hide-last-target {
    display: none;
  }
  .hide-last-child {
    .hide-last-target {
      display: initial;
    }
    &:last-child {
      .hide-last-target {
        display: none;
      }
    }
  }
}
~~~

~~~
// show on hover
//
//  <li class="show-on-hover">
//    Text<a href="" class="show-on-hover-target">X</a>
//  </li>
.show-on-hover-target {
  visibility: hidden;
}
.show-on-hover:hover {
  .show-on-hover-target {
    visibility: visible;
  }
}
~~~

~~~
// expand on hover
// used on index tables when some data is too long to be always shown
//
// <td class="expand-on-hover"><%= account.uuid %></td>

.expand-on-hover {
  width: 20px;
  height: 10px;
  overflow: hidden;
  float: left;
}
.expand-on-hover:hover {
  width: inherit;
}
~~~

To expand block on click use input checkbox
https://www.sitepoint.com/pure-css-off-screen-navigation-menu/

Note that input is before label, which is before target

~~~
// toggle active without javascript
// <input type="checkbox" id="toggle-active" class="toggle-active" />
// <label for="toggle-active"><%= t('add') %></label>
// <div class="toggle-active-target hide-not-important">
// </div>
// https://www.sitepoint.com/pure-css-off-screen-navigation-menu/
.toggle-active {
  position: absolute;
  clip: rect(0, 0, 0, 0);

  &:checked ~ .toggle-active-target {
    display: initial;
  }
}
label[for="toggle-active"] {
  cursor: pointer;
}
~~~

To show active on click in pure css use `li:hover { color: blue }`

# Design

* when you are asking user to select items from long lists, you should have
  `Next` before and after the list so user does not need to scroll
* be proactive with messages, when user needs to type password again (for
  example signin with fb and confirm) than write `Excellent, we need just...`
* when hover on footer link, other link in same block should be opacity
[demo](https://www.parse.com/home/index)

  ~~~
  ul:hover li:hover {
    opacity: 1;
  }
  ul:hover li {
    opacity: 0.4;
  }
  ~~~

# Image

Swap images and zoom on hower https://www.filamentgroup.com/lab/sizes-swap/

# Layout

https://hacks.mozilla.org/2018/01/new-flexbox-guides-on-mdn/

Flexbox

Used in layout in one dimension at a time (either row or column).

CSS Grid https://mozilladevelopers.github.io/playground/css-grid

# Table

Table has [two layouts](http://www.w3schools.com/cssref/pr_tab_table-layout.asp)
* `table-layout: auto` dependends on widest unbreakable content in the cells
* `fixed` faster since browser does not need full content

Rotate header [link](https://css-tricks.com/rotated-table-column-headers/)

~~~
<th class="rotate"><div><span>Column header 1</span></div></th>
~~~

~~~
th.rotate {
  /* Something you can count on */
  height: 140px;
  white-space: nowrap;
}

th.rotate > div {
  transform:
    /* Magic Numbers */
    translate(25px, 51px)
    /* 45 is really 360 - 45 */
    rotate(315deg);
  width: 30px;
}
th.rotate > div > span {
  border-bottom: 1px solid #ccc;
  padding: 5px 10px;
}
~~~

# jQuery

[some
tips](https://code.tutsplus.com/tutorials/14-helpful-jquery-tricks-notes-and-best-practices--net-14405)

* jquery selectors
  * `$('thead th[data-searchable!="false"]')` select all th that do not have
  `data-searchable` or have it but different than `false`
* if jQuery finder returns `n,fn.init` that means it did not find DOM elemenent
[link](http://stackoverflow.com/questions/34494873/why-is-my-jquery-selector-returning-a-n-fn-init0-and-what-is-it)

> when found an element based on the selector criteria it returns the matched
> elements; when the criteria does not match anything it returns the prototype
> object of the function.

With jquery you can create elements `$('<div>')`. If you want additional
properties you can pass as additional params `var $div = $('<div>', { id: 'foo',
class: 'my_class'})`.
You can join elements with: append, prepend, after and before. Here you can use
jquery objects or plain html: `$('#box').append($div).append('<div
id="foo"></div>')`.


# Tips

* submit button outside of a form is
  [possible](http://stackoverflow.com/questions/7020659/submit-form-using-a-button-outside-the-form-tag)

  ~~~
  <form id="myform" method="get" action="something.php">
  <input type="text" name="name" />
  </form>

  <input type="submit" form="myform" />
  ~~~

* note that if you have multiple submit inputs and one text input field, when
you press Enter, only first submit button will be used (so commit value will be
it's value). you can disable submit on enter with

  ~~~
  $('#notification-form').on('keyup keypress', function(e) {
    var keyCode = e.keyCode || e.which;
    if (keyCode === 13) {
      e.preventDefault();
      return false;
    }
  });
  ~~~

* `<input readonly="readonly">` is better to use than `<input
disabled="disabled">` since it can receive focus, included in tabbing navigation
and values are successfully posted (disabled input is not send to server).
* if you need two css files, than write two `<link rel="stylesheet"
  type="text/css" href="1.css" />` . Do not use `@import url("2.css")` in
  `1.css` since it will prevent downloading files in parallel (2.css is
  downloaded only after 1.css). Only [user of
  @import](https://developer.mozilla.org/en-US/docs/Web/CSS/@import) is when you
  provide media queries as parameter and load only for specific screen. Note
  that sass preprocessor scss `@import` will include inline the code.
* if option select is required, we usually prompt user with one more option
  "Please select", but it is bad to allow user to pick that option "Please
  select". You can disable option, and you should force selected on it if other
  option is not selected [demo](http://jsfiddle.net/u8PWX/1/)

  ~~~
    <select onchange="this.form.submit()">
      <option selected="selected" disabled="true">--Please Select --</option>
      <option value="volvo">Volvo</option>
      <option value="saab">Saab</option>
    </select>
  ~~~

  In rails, you can use with `select_tag`:

  ~~~
    <%= select_tag "job[job_type_id]",("<option #{ "selected='selected'".html_safe unless fjob.object.job_type_id } disabled='disabled'>Job Type</option>".html_safe+ options_from_collection_for_select(JobType.active.all, :id, :name,{selected: fjob.object.job_type_id})), { class: "e1" } %>
  ~~~

  or with `f.collection_select`

  ~~~
    <%= f.collection_select :user_id, User.all.unshift(User.new id: 0, name:
    "Please Select User"), :id, :name), {}  %>
  ~~~

  or `f.select`

  ~~~
    <%= f.select :shopify_custom_collection_id, [['Please select collection',0]] + @shopify_custom_collections.map { |shopify_custom_collection| [shopify_custom_collection.title, shopify_custom_collection.id] }, disabled: 0, selected: 0 %>
  ~~~

* if you want to filet group select based on first option than use something
like
<https://stackoverflow.com/questions/32405077/show-second-dropdown-options-based-on-first-dropdown-selection-jquery>

  ~~~
  ~~~

* `<a href="javascript:void(0)">` is required when you use `onclick=` and do not
  want page to reload
* `input type="checkbox" onchange="perform(this.checked)` you can get value with
  input.checked. with jquery `$(input).is(':checked')`
* in css file you can include comment for source maps, for example `/*#
sourceMappingURL=bootstrap-datepicker3.css.map */`
* plain `<input>` has default with (probably 157px), but if you want to set
size, use `style="width: 100%"`
* if you want background image to be blured, you can use overlay with opacity
position absolute that cover whole area. Your element should be position
relative and defined after cover to stand out of that cover.

~~~
<body>
  <div class="gradient"></div>
  <div class="my-stuff"></div>
</div>
~~~

~~~
.my-stuff {
  position: relative;
}
.gradient {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  opacity: .7;
  -webkit-backface-visibility: hidden;
  background-color: #52d3aa;
  background-image: -webkit-gradient(linear, 0% 0%, 100% 100%, color-stop(0, #3f95ea), color-stop(1, #52d3aa));
  /* Android 2.3 */
  background-image: -webkit-repeating-linear-gradient(top left, #3f95ea 0%, #52d3aa 100%);
  /* IE10+ */
  background-image: repeating-linear-gradient(to bottom right, #3f95ea 0%, #52d3aa 100%);
  background-image: -ms-repeating-linear-gradient(top left, #3f95ea 0%, #52d3aa 100%);
}
~~~

* for phone you need to use `<a href="tel:123123">+123-123</a>` (rails `<%=
link_to "+123-123", "tel:123123" %>`)
* for mail `<a href="mailto:asd@asdasd">asd@asd.asd</a>` (rails `<%= mail_to
"asd@asd.asd", "asd@asd.asd" %>`)
* for event listeners always use `e.currentTarget` since it is element on which
  listener was bound. Do not use `e.target` since it could be child element.
  Example `$('[data-user-preferences]').click(function(e) {
  e.currentTarget.dataset.userPreferences
  })`
* if you need to click on the underlying elements under other, you can use
  <https://stackoverflow.com/questions/3680429/click-through-a-div-to-underlying-elements>
  ~~~
  pointer-events: none
  ~~~
