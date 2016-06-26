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
  does not have efect unless the element has the width (because it can not
  calculate margins). For text you can use `text-center`
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
  can set `rows="30"` or use [autosize](http://www.jacklmoore.com/autosize/)

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

# SCSS

* you can set default value of variable `$my-var: 123 !default;`. This has no
  effect if variable is already defined.
* <https://responsivedesign.is/develop/getting-started-with-sass>

* chrome 39 for android use different color for toolbar, just add `<meta
  name="theme-color" content="#db5945">`


# Head meta tags

* here is a list of used tags [HEAD](https://github.com/joshbuchea/HEAD)
