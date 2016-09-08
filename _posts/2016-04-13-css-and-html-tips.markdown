---
layout: post
title: CSS and HTML tips
---

* when you want to have smaller input field type number (smaller in terms of
  width) you need to provide both `min` and `max`, like `<input type="number"
  min="0" max="99">`
* when you have some div/dt `float: left; width: 100px` than text can be on one
  line `white-space: nowrap;overflow: hidden;text-overflow: ellipsis;` or in
  multiple lines `white-space: normal`
* if you have `<small>` position relative, and apply left -100px, it will still
  occupy the space where it was. It is better to use `position: absolute`
* to move element to the right but keep inside parent element you need to mark
  parent as `position: relative` and than use `position: absolute; right: 10px`
* align element at bottom use position relative/absolute pair, for parrent
  `position: relative` (does not have any effects) and for child: `position:
  absolute;bottom: 10px;`
* use `vertical-align: baseline` for image to be on same bootom as other
  elements
* use `mouseenter` instead `mouseover` (which will retrigger on inner elements).
* use relative units instead of absolute
  <http://www.w3schools.com/cssref/css_units.asp>, for example header `h1 { font-size: 20vh;}`
* you can not set the width of inline elements, so to set width of `span` you
  need to make it `display:inline-block; width: 100px`
* also top and bottom padding has no efect for label since it is an inline
  element
  [link](http://stackoverflow.com/questions/7168658/why-is-the-padding-not-working-on-my-label-elements).
  You need to make the label a block level element for it to work.
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

# SCSS Sass

* you can set default value of variable `$my-var: 123 !default;`. This has no
  effect if variable is already defined.
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
~~~

~~~
// Here is example to show hide buttons that are not allowed for first or last
//
// <ol class="hide-first-child hide-last-child">
//   <!-- iterate over li -->
//   <li class="hide-first-target"><i class="fa fa-arrow-up"></li>
//   <li class="hide-last-target"><i class="fa fa-arrow-down"></li>
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
.show-on-hover {
  visibility: visible
}
~~~

# Bootstrap Media queries

With Bootstrap you can use their predefined media queries with `min-width`
(mobile first), for example

~~~
@media(min-width:$screen-sm){
  .alert {
    position: absolute;
    bottom: 0px;
    right: 20px;
  }
}
~~~

Bootstrap `container` has fixed width for big devices.
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


* submit button outside of a form is
  [possible](http://stackoverflow.com/questions/7020659/submit-form-using-a-button-outside-the-form-tag)

  ~~~
  <form id="myform" method="get" action="something.php">
  <input type="text" name="name" />
  </form>

  <input type="submit" form="myform" />
  ~~~

* if you need two css files, than write two `<link rel="stylesheet"
  type="text/css" href="1.css" />` . Do not use `@import url("2.css")` in
  `1.css` since it will prevent downloading files in parallel (2.css is
  downloaded only after 1.css). Only [user of
  @import](https://developer.mozilla.org/en-US/docs/Web/CSS/@import) is when you
  provide media queries as parameter and load only for specific screen. Note
  that sass preprocessor scss `@import` will include inline the code.
