---
layout: post
---

* when you want to have smaller input field type number (smaller in terms of
  width) you need to provide both `min` and `max`, like `<input type="number"
  min="0" max="99">`
* to hide long text on div/dt `width: 100px` than you can:
  * long string  `overflow: hidden; text-overflow: ellipsis;` To show on hover,
  focus or active (when selecting text) use this helper
  ~~~
  // <div class='long-string-three-dots'>long long</div>
  .long-string-three-dots {
    text-overflow: ellipsis;
    overflow: hidden;
    // white-space: nowrap;
    &:active, &:hover, &:focus {
      text-overflow: initial;
      overflow: auto;
    }
  }
  ~~~

  * long text can be in one line `white-space: nowrap;` no wrap means it will no
  go to the next line even for white space (this is used in `navbar-brand`). To
  make it go to next line define something like `.white-space-normal`.
  * long strings can be shown on multiple lines with `word-wrap: break-word`
  this will break long string to multiple lines (for text `white-space: normal`)

* if you have `<small>` position relative, and apply left -100px, it will still
  occupy the space where it was. It is better to use `position: absolute`
* to move element to the right but keep inside parent element you need to mark
  parent as `position: relative` and than use `position: absolute; right: 10px`
* align element at bottom use position relative/absolute pair, for parrent
  `position: relative` (does not have any effects) and for child: `position:
  absolute;bottom: 10px;`. This
  [question](http://stackoverflow.com/questions/17656623/position-absolute-scrolling)
  explains why outer need to be relative.
* use `mouseenter` instead `mouseover` (which will retrigger on inner elements).
* use relative units instead of absolute
  <http://www.w3schools.com/cssref/css_units.asp>, for example header `h1 {
  font-size: 20vh;}`. vh is 1% of vertical heigh of view port (window size, not
  page size). For width you can use horizontal width for example `width: 8vw;`.
  You should use relative size `rem` (to the html el) or `em` (to the current
  font) since you might want to change font size on whole page, and you do not
  need to go to all nested components to update that. 1rem is default font size
  usually 16px, bootstrap was 14px now is also 16px.

* you can not set the width of inline elements, so to set width of `span` you
  need to make it `display:inline-block; width: 100px`. Also if you have text
  than `inline` or `inline-block` element with `block` inside, it will behave
  differently
  ~~~
            inline-block
  some_text nested_block
  ~~~

  ~~~
  some_text inline
  nested_block
  ~~~
* also top and bottom padding has no efect for label since it is an inline
  element
  [link](http://stackoverflow.com/questions/7168658/why-is-the-padding-not-working-on-my-label-elements).
  You need to make the label a block level element for it to work.
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

* prefer mixins to `@extend`. You can pass parameter to mixin and set default
  value

  ~~~
  @mixin grid($my_var: true) {
    // code here
  }
  ~~~

  When defining mixin you can use `@content` for example

  ```
  @mixin hover-focus {
    &:hovoer,
    &:focus {
      @content;
    }
  }
  # usage:
  # .class {
  #   @include hover-focus {
  #     text-decoration: none;
  #   }
  # }
  ```

## SASS

Sass is shorter since it uses indent, and does not require semicolon.
Atomatic convert scss to sass
~~~
sass-convert -F scss -T sass application_styles.css.scss application_styles.css.sass
~~~

Instead of scss
~~~
# scss
@mixin border-radius($radius) {
  border-radius: $radius;
}

.box {
  @include border-radius(10px);
}

# sass
=border-radius($radius)
  border-radius: $radius

.box
  +border-radius(10px)
~~~


# Head meta tags

* here is a list of used tags [HEAD](https://github.com/joshbuchea/HEAD)
* chrome 39 for android use different color for toolbar, just add `<meta
  name="theme-color" content="#db5945">`

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

// add class show to stay visible when not hovering, bootstrap dropdown does dat
.show-on-hover-target.show {
  visibility: visible;
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

To expand block on click use input checkbox (non js toggle)
https://www.sitepoint.com/pure-css-off-screen-navigation-menu/

Note that input is before label, which is before target

~~~
# app/assets/stylesheets/common/display_visibility.sass
// toggle active without javascript
// <label for="some-id"><%= t('add') %></label>
// <input type="checkbox" id="some-id" class="toggle-active" />
// <div class="toggle-active-adjacent-sibling hide-not-important">
// </div>
// https://www.sitepoint.com/pure-css-off-screen-navigation-menu/
.toggle-active {
  position: absolute;
  clip: rect(0, 0, 0, 0);

  &:checked ~ .toggle-active-adjacent-sibling {
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

To set image as background and put some text and form on it you need to make the
image transparent (or white) on that part.
For navbar, easiest way is to use `fixed-top` and `z-index: 1031` on image so it
is above navbar.
Inside page links with anchor does not work with turbolinks, so you need to
disable it.

# Flex

https://css-tricks.com/snippets/css/a-guide-to-flexbox/#flexbox-background
https://medium.com/@js_tut/flexbox-the-animated-tutorial-8075cbe4c1b2

Parent element is flex container and children are flex items. `flex-direction`
determines main axis on which items will be laid out from `main-start`
(`flex-start`) to `main-end` (`flex-end`), that is total `main-size`.
Perpendicular to the main axis is `cross` axis with its `cross-start`
`cross-end` and `cross-size`.

On container `display: flex` (synonim is `inline-flex`) we can have:
* `flex-direction: row | row-reverse | column | column-reverse` row means that
flexible items are aligned horizontally as row
* `flex-wrap: nowrap | wrap | wrap-reverse` default is no wrap so in single
  line, wrap means that it is allowed to go onto multiple lines. This two
  properites can be shorhanded in one `flex-flow: row nowrap` which is default
  (child with `width: 100px` will be `width: 12px` if there is no space, unless
  `flex-shrink` is used).

* `justify-content: flex-start | flex-end | center | space-between |
  space-around | space-evenly` alignment when items are inflexible. space around
  means that all items have same space around (before first there is no items so
  that's why it is single space) if you need exactly same space from start to
  first and first to second, use space-evenly.
* `align-items: flex-start | flex-end | center | stretch | baseline` behavior
  along CROSS AXIS on current line similar to `justify-content` but for cross
  axis. `stretch` is from `cross-start` to `cross-end` still respect
  min-width/max-width. `baseline` when text are on the same line.
* `align-content: flex-start | flex-end | center | stretch | space-between |
  space-around` behavior of LINES (wrap is enabled) when there is extra space in
  cross-axis (no effect is single line) similar to `justify-content` for main
  axis, but for lines.

on items https://www.w3.org/TR/css-flexbox-1/#flexibility
* `flex-grow: 0` ability to grow as proportion to other items (all items need to
  defined `flex-grow`). If one has 2 and all others have 1, first will be twice
  bigger (`width` is changed, but do not use it since it will have some impact,
  for example if `width: 150px` for sidebar and content `flex-grow: 1` than
  if content is smaller -> sidebar 150px, but when content has a table than
  sidebar will be less than 150px, so better is to use `min-width: 150px`).
* `flex-shrink: 1` ability to shrik (compress), if `flex-shrink: 0` than it will
  keep it's `width: 500px` property
* `flex-basis: 20% | auto` default size before remaining space is distributed
  (use this if you want max-size: 20%)
  This three items have shorthand `flex: flex-grow flex-shrink flex-basis
  (default 0 1 auto)` which you can attach to flex item. For example:
  * `flex: auto` auto means that if item can use space it will use space. it is
  synonim to `flex: 1 1 auto` bigger item will take bigger space (on container
  remove `justify-content`)
  * `flex: 1` is eq to `flex: 1 1 0` all items same width (flex-basis 0 so their
  content is not taken into consideration)
  * `flex: 1 100%` (eq to `flex: 1 1 100%`) inside `wrap` all items takes row
  * `flex: 3 0px` this item is 3 times bigger than others which are `1` (`wrap>*
   { flex: 1 auto}`) and do not take it's initial size into account

* `align-self: flex-start | flex-end | center | stretch | baseline` override
  `align-items` for specific item, on cross axis
* `order: 0` when you want to show in different order `order: 1` `order: 2`

* [justify-items](https://developer.mozilla.org/en-US/docs/Web/CSS/justify-items)
 property defines the default justify-self for all items for a `display: grid`.

Examples:
https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Typical_Use_Cases_of_Flexbox
* perfect centering horizontaly and vertically of one width/height card
  ```
  .card-container
    display: flex
    height: 20px
    justify-content: center
    align-items: center
  .card
    width: 10px
    height: 10px
  ```

* bootstrap layout for example `.col-md-6 { flex: 0 0 50%; max-witdh; 50% }`
You can use `margin-left: auto` to separate group of items since auto margin
will took as much space as it can. So you can align first three on left and last
two on right by adding `.push` on firt right item.

~~~
.box {
  display: flex;
}
.push {
    margin-left: auto;
}

<div class="box">
  <div>One</div>
  <div>Two</div>
  <div>Three</div>
  <div class="push">Four</div>
  <div>Five</div>
</div>
~~~

Om mobile you probably want all buttons to take all space so use `flex: auto` or
`flex: 1` if you want same size buttons.

https://hacks.mozilla.org/2018/01/new-flexbox-guides-on-mdn/
https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox


Flexbox

Used in layout in one dimension at a time (either row or column).

# Grid

https://mozilladevelopers.github.io/playground/css-grid

`fr` is unit which represents a fraction of the available space in the grid
container
If you want to repeat something multipletimes you can use `repeat()` css
notation
```
.container
  display: grid
  width: 800px
  // grid-template-columns: 1fr 1fr 1fr
  grid-template-columns: minmax(100px, auto) repeat(2, 1fr)
  grid-template-rows: repeat(2, 150px)
  grid-gap: 1rem
```

`minmax` can accept `auto` as first argument (min, so it represent `min-content`
so it does not overflow) or second argument (max, it reperesents `max-content`
which is smallest possible size the cell can be while still fitting around it's
contents unobtrusively). minmax is usefull only with `fr`, `%` or `auto`.

Position in the grid between lines, starting from 1, 2, ... number of columns +
1, so you can move any cell to the area
```
.item1
  // grid-row-start: 2
  // grid-row-end: 3
  grid-row: 2/3
```

Another way is to define `grid-template-areas` and assign each element with a
`grid-area: my-area`
```
.container
  grid-template-areas: "header header header" "sidebar content-1 content-1" "footer footer footer"

.header
  // grid-column: 1/4
  grid-area: header
```

# CSS box alignment

https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Alignment
older alignment methoods were:
* align text using `text-align` In bootstra for text you can use `text-center`
  `text-right` helper classes for to align text center and right. Note that if
  you have some `floating` element before than text will not be centered like in
  `<div class='float-right'>log out</div><div class='text-center'>Text</div>`
* center blocks using auto `margin` for block elemenent with `margin: auto`
  (bootstrap [center-block](http://getbootstrap.com/css/#helper-classes-center)
  helper) does not have effect unless the element has the width (because it
  can not calculate margins)
* in inline-block elements use `vertical-align: baseline` for image to be on
  same bootom as other elements
* to make vertical align you need parent to be `position: relative`, and target
  to be `position:absolute; top: 50%; margin-top: -1rem;` (if you show text
  1rem).


There are two axis: inline (main) and block (cros) axis.
When aligning items on inline axis we use properties that starts with `justify-`
and when aligning items on block axis we use `align-` (items, self, content).
With `flex-direction: row` than main axis is block, and cross is inline.
Three types of alignment:
* **positional alignment** for `-self` and `-content` specify position of
  subject with relation to container: `center`, `start`, `end`, `self-start`,
  `self-end`, `flex-start`, `flex-end`, `left`, `right`.
* **baseline alignment** for `-self` and `-content` specify relationship amount
  baselines of multiple subjects: `baseline`, `first baseline`, `last baseline`
* **distributed alignment** for `-content` specify what happens to space after
  subject have been displayed: `strech`, `space-between`, `space-around`,
  `space-evenly`


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

# CSS Selectors

https://www.w3.org/TR/2011/REC-css3-selectors-20110929/

* type selector
* universal
* attribute
* class
* id
* pseudo-classes
* pseudo-elements
* combinators

To find you can use https://www.w3.org/TR/selectors-api/

* `querySelector('#bar, #foo')` for example return first in document order
  `#foo`
* `querySelector('.class_name')` to get by css class
* `var lis = querySelectorAll('ul>li')` we can iterate using array notation
  `for(var i = 0; i < lis.lenght; i++) { lis[i] }`. NodeList are static (not
  live) so changes in DOM do not affect content of the lis

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

  I made helper data function which will also click. put on document ready or
  turbolinks load.

  ~~~
  # app/assets/javascripts/turbolinks_loads.coffee
  # f.text_field :email, 'data-prevent-submit-on-enter': '#email-continue'
  $(document).on 'turbolinks:load', ->
    $('[data-prevent-submit-on-enter]').on 'keyup keypress', (e) ->
      keyCode = e.keyCode || e.which
      if keyCode == 13
        e.preventDefault()
        $($(this).data().preventSubmit).click() if $(this).data().preventSubmitOnEnter != true
  ~~~

* `<input readonly="readonly">` is better to use than `<input
disabled="disabled">` since it can receive focus, included in tabbing navigation
and values are successfully posted (disabled input is not send to server).
* if you need two css files, than write two `<link rel="stylesheet"
  type="text/css" href="1.css" />` . Do not use `@import url("2.css")` in
  `1.css` since it will prevent downloading files in parallel (2.css is
  downloaded only after 1.css). Only [use of
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
* for modals you need overlay that cover content beside modal, also if you want
to have background image that is blured, you can use overlay with opacity
position absolute that cover whole area.
https://css-tricks.com/snippets/css/transparent-background-images/
This use case is when you have my-stuff with forms or something else so you need
to interact with it. Another use case when you want to add overlay like for
subscribed users only, so my-stuff can be opacited and with wrapping my-stuff.
First is solved with sibling div `position: fixed` and `z-index: -1` and adding
background image to the body (not to fixed element, which is only for overlay
and opacity) so it scrolls down with content.

Second is solved with position absolute relative pair.
Note that you should NOT use `height: 100%` on cover because it will not cover
the area which is not currently in view port (for example you have div height >
view port size). On `html, body { height: 100% }` is OK (and it will show window
width and height) but when cover has greater height than body on smaller screen
than it should have `min-height: 100%` (not `height: 100%`).
Note that `my-stuff` should be inside `gradient` so gradient wraps it
completelly (otherwise when they are sibling than gradient can not calculate
if there is margin on my-stuff).

~~~
<html>
  <head>
    <style>
      html, body {
        min-height: 100%;
        height: 100%;
        padding: 0px;
        margin: 0px;
      }

      .position-relative {
        position: relative;
      }
      .position-absolute {
        position: absolute;
        width: 100%;
        /* we need this when my-stuff is less than view port */
        min-height: 100%;
        /* do not use top bottom, or height: 100% since that is relative to view port */
        /*
        top: 0px;
        bottom: 0px;
        */
        /* height: 100%; */
      }
      .my-stuff {
        width: 200px;
        height: 200px;
        margin: 10px auto;
        background: blue;
        height: 600px;
      }
      .my-stuff:hover {
        height: 600px;
      }
      .blue-gradient {
        opacity: .5;
        -webkit-backface-visibility: hidden;
        background-color: #52d3aa;
        background-image: -webkit-gradient(linear, 0% 0%, 100% 100%, color-stop(0, #3f95ea), color-stop(1, #52d3aa));
        /* Android 2.3 */
        background-image: -webkit-repeating-linear-gradient(top left, #3f95ea 0%, #52d3aa 100%);
        /* IE10+ */
        background-image: repeating-linear-gradient(to bottom right, #3f95ea 0%, #52d3aa 100%);
        background-image: -ms-repeating-linear-gradient(top left, #3f95ea 0%, #52d3aa 100%);
      }

      .position-fixed-cover {
        position: fixed;
        top: 0px;
        bottom: 0px;
        width: 100%;
        z-index: -1;
      }
      .background-image {
        background-image: url('cat.jpg');
      }
    </style>
  </head>
  <body class='position-relative background-image'>
    <div class='blue-gradient position-absolute'>
      blue-gradient position-absolute
      <div class='my-stuff'>
        my-stuff
        <button>OK</button>
      </div>
    </div>
    <div class='blue-gradient position-fixed-cover'>blue-gradient position-fixed-cover</div>
  </body>
</html>
~~~

* another problem is when you add `margin: 10px` to some innver div, than
  scrollbar shows up (we use `box-sizing: border-box;` so that is not the case).
  https://stackoverflow.com/questions/22539053/nested-div-with-margin-top-set-causes-scrollbar-to-appear
  The problem is in collapsing margins and workaround is: use padding instead of
  margin, add `overflow: auto` to the element with margin, or introduce new
  [block formating
  context](https://www.w3.org/TR/CSS2/visuren.html#block-formatting) by changing
  to `display: inline-block`
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
* accordion is native in html (you do not need bootstrap accordion collapse in
  javascript)

  ~~~
  <details>
    <summary>Hi</summary>
    Bye
  </details>

  <details>
    <summary>How do I get to New Orleans?</summary>
    Use Google Maps.
  </details>
  ~~~

  ~~~
  details {
    margin: 1rem;
  }
  summary {
    font-weight: bold;
  }
  ~~~

  You can style different states of defails (`details[open]` and
  `details:not[open]`) https://codepen.io/jh3y/pen/mLaXRe
  https://codepen.io/stoumann/pen/ExydEYL

* prevent auto completing text inputs can be prevented with `<%= text_field_tag
* :other_reason, nil, autocomplete: 'off' %>` (note is should be 'off' not
'false'

* arrows (triangles) can be done with border and before pseudo element
  http://krasimirtsonev.com/blog/article/CSS-before-and-after-pseudo-elements-in-practice just paste this snippet

```
<style>
h2 {
    float: left;
    width: 170px;
}
.popup {
    float: left;
    width: 340px;
    background: #727272;
    padding: 10px;
    border-radius: 6px;
    color: #FFF;
    position: relative;
    font-size: 12px;
    line-height: 20px;
}
.popup:before {
    content: "";
    display: block;
    width: 0;
    height: 0;
    border-top: 12px solid transparent;
    border-bottom: 12px solid transparent;
    border-right: 12px solid #727272;
    position: absolute;
    top: 16px;
    left: -12px;
}
</style>

<h2>What is CSS?</h2>
<div class="popup">
    Cascading Style Sheets is a style sheet language used for describing
    the presentation semantics of a document written in a markup language.
</div>
```

* usually space is not important in html but if you have some `float` and
  `width: 40%` than number of spaces is important
* you can not have `<a>` tag inside other `<a>` tag, use this validator
  https://validator.w3.org/nu/#textarea

components and terminology (terms names) on a web page:

* accordion is list of blocks that can collapse
https://getbootstrap.com/docs/3.3/javascript/#collapse-example-accordion
* carousel is big image slider
https://getbootstrap.com/docs/3.3/javascript/#carousel
* navbar is top header with menu links, in bootstrap_4 it is fluid (spans all
  width)
* flush on some elements
  https://getbootstrap.com/docs/4.1/components/list-group/#flush means that
  rounded corners are removed
* B4 cards replace our old panels, wells, and thumbnails
* masonry type columns is when element are places optimally basedon vertical
  space (width is fixed, and height is variable)
* cloaking is when heading seo keywoards says nice words but content sell
   drugs

* if you have two data attributes on click, to disable one from another, you can
  disable button, for example
  ```
  $('[data-enable-if-valid]').on 'click', (e) ->
    if need_to_disable_both_clicks
      $button.prop('disabled', 'disabled')
      setTimeout(
        ->
          $button.prop('disabled', false)
        100
      )

    # no need to add e.preventDefault() since button is disabled
  ```

* page scroll when image is loaded can be prevented with padding bottom https://www.smashingmagazine.com/2016/08/ways-to-reduce-content-shifting-on-page-load/
  padding can be defined in % of the page width so for full width page images
  you can set padding bottom in %.
  For not full width images you can set the height in css so it will occupy the
  space before loading (you can set background also if it is not transparent).
  Do not set only the width since than height is 0 untill the image is loaded.
* elements with captions are:
  `<figure><img><figcaption>I'm caption</figcaption></figure>` and 
  `<fieldset><legend>I'm caption of this set</legend><input></fieldset>`
* add text node
  ```
  let el = document.getElementById('el');
  el.appendChild(document.createTextNode(' Some more text.'));
  console.log(el.childNodes.length); // 2
  ```
  and you can merge them with `el.normalize()`. Or grab all child nodes text
  with `console.log(el.childNodes[0].wholeText)`
* insert at specific positionin a dom with `insertAdjacentHTML()`,
  `insertAdjacentElement()`, and `insertAdjacentText()`. Position could be
  `beforebegin, afterbegin, beforeend, afterend`.
* to get height of the element with scrollable area when `overflow: auto`, or
  total space when `overflow: hidden`, you can use `scrollHeight`. To get actual
  height use `offsetHeight` (this includes any vertical padding and borders).
* customize ordered list type https://www.w3.org/TR/CSS2/generate.html#counters
  add roman number with brackets https://stackoverflow.com/a/1636635
  ```
  ol {
    counter-reset: list;
  }
  ol > li {
    list-style: none;
    position: relative;
  }
  ol > li:before {
    counter-increment: list;
    content: counter(list, lower-alpha) ") ";
    position: absolute;
    left: -1.4em;
  }
  ```
* instead of right click Save as you can add download attribute in anchor tag
  `<a download href="cat.jpg">Download cat</a>` so instead of opening picture in
  a new tab, it will download it
* white line can be added as absolute position with rotate (you can move left
  or right with `margin-left` positive or negative).
  ```
  .slant {
    position: absolute;
    transform: rotate(-9deg);
    background-color: #fff;
    display: inline-block;
    width: 676px;
    height: 170px;
    top: 300px;
    margin-left: -10px;
  }
  ```
* convert nodelist HTMLOptionsCollection to array
  ```
  Array.from(select.options)
  ```
* embed video using webm and subtitles using vtt
  ```
  <video controls="">
    <source src="https://swaywm.org/intro.webm" type="video/webm">
    <track src="subtitles/intro.vtt" kind="captions" srclang="en" label="English">
    <track src="subtitles/intro-it_IT.vtt" kind="captions" srclang="it" label="Italiano">
    <p>Your browser does not support HTML5 video.</p>
  </video>
  ```
