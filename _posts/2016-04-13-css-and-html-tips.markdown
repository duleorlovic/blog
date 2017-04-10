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

# Pizza cut

Interesting design <http://hmfaysal.me/hmfaysal-omega-theme/> since it uses svg
to cut the image and add animations.

~~~
<div class="decor-wrapper">
  <svg id="header-decor" class="decor bottom" xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="0 0 100 100" preserveAspectRatio="none">
    <path class="large left" d="M0 0 L50 50 L0 100" fill="rgba(255,255,255, .1)"></path>
  </svg>
</div>
~~~

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

  In rails, you can use something like

  ~~~
    <%= select_tag "job[job_type_id]",("<option #{ "selected='selected'".html_safe unless fjob.object.job_type_id } disabled='disabled'>Job Type</option>".html_safe+ options_from_collection_for_select(JobType.active.all, :id, :name,{selected: fjob.object.job_type_id})), { class: "e1" } %>
  ~~~
* `<a href="javascript:void(0)">` is required when you use `onclick=` and do not
  want page to reload
* `input type="checkbox" onchange="perform(this.checked)` you can get value with
  input.checked. with jquery `$(input).is(':checked')`
* in css file you can include comment for source maps, for example `/*#
sourceMappingURL=bootstrap-datepicker3.css.map */`
