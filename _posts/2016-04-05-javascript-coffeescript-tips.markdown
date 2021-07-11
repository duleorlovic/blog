---
layout: post
title: Javascript & coffeescript tips
---

# How is javascript loaded

First are loaded scripts from `<head>` tag, than from `<body>` tag. It could
happend that one `<script src=...` failed to load remote scipt, so probably
the following script `<script>` tag will not work.
You can add script dynamically in javascript using `appendChild` and it will be
executed after all inline scripts are done (after all defered so setting `defer`
option in javascript has no efects).
You can run your script as callback when script is loaded
~~~
# if you are using script src tag
<script onload="loadedContent();" src ="/myapp/myCode.js"  ></script>

# or using script src tag and event listenter
<script id="myscript" src ="/myapp/myCode.js"></script>
<script>
  var script = document.querySelector('#myscript');
  script.addEventListener('load', function() {
    myScipt Initialisation code
  });
</script>

# or if loaded using appendChild

var razorpay_script = document.createElement('script');
razorpay_script.setAttribute('src','https://checkout.razorpay.com/v1/checkout.js');
razorpay_script.onload=function(){
  #{razorpay_options_js}
  var rzp1 = new Razorpay(options);
  rzp1.open();
};
document.head.appendChild(razorpay_script);

# third solution could be to use setTimeout(callback(){}, 200)
~~~

Threre are two phases: downloading and executing. All modern browsers download
in parallel so we do not need to consider downloading. Execution could be
immediatelly or after parsing.

There are two additional attributes [html living standard 4.12
Scripting](https://html.spec.whatwg.org/multipage/scripting.html)

* `<script src="" async>` execute as soon as it is downloaded (download is not
blocking). It could be in
any order regarding other scripts and it could be before parser finishes.
* when `async` is not present than`<script src="" defer>` (download is not
blocking) execute in same order when page has finished parsing (just before
DOMContentLoaded)
`defer` is ignored on scripts without `src` (so defer works only for external
scripts)


# Scroll into view of element

Sometimes you need to scroll the page to show some notification on top, or to
display element from the bottom of the page. We can use
[scrollIntoView](https://developer.mozilla.org/en/docs/Web/API/Element/scrollIntoView)

~~~
function checkIfInView($element, options){
  // Not in view so use https://developer.mozilla.org/en-US/docs/Web/API/Element.scrollIntoView
  // use options parameter to force scrolling even element is visible on current view
  // if options == true, the top of the element will be aligned to the top of the visible area of the scrollable ancestor.
  if (typeof document.getElementsByTagName('body')[0].scrollIntoView == 'undefined')
  {
    LOG && console.log("scrollIntoView not defined");
    return true;
  }
  else if (typeof options != 'undefined')
  {
    LOG && console.log("ckeckIfInView "+options);
    $element[0].scrollIntoView(options);
  }
  else
  {
    // check if visible
    var offset = $element.offset().top - $(window).scrollTop();
    if(offset < 0){
        LOG && console.log("alignWithTop=true");
        $element[0].scrollIntoView();
        return false;
    }
    else if (offset > window.innerHeight){
        LOG && console.log("alignWithTop=false");
        $element[0].scrollIntoView(false);
        return false;
    }
    else
    {
      LOG && console.log("no scroll");
      return true;
    }
  }
}
~~~

# Scroll to bottom

http://jsfiddle.net/5ucD3/13/
```
$('#textdiv').append('<p>asd</p>,,,,')
$('#textdiv').animate({scrollTop: $('#textdiv').prop("scrollHeight")}, 500);
```

# Always on page, floating elements

There is nice plugin [stickyjs](http://stickyjs.com/) but for simple scroll you
can use this
[answer](http://stackoverflow.com/questions/2177983/how-to-make-div-follow-scrolling-smoothly-with-jquery)
First version is enought if you do not need to scroll big elements. Note that
calculation of `originalTop` could be wrong if there are some images and
their parent does not have fixed height.

~~~
// app/assets/javascripts/follow-scroll-smoothly.js
// call with `follow_scroll_smoothly('.navbar');
function follow_scroll_smoothly(elementSelector) {
  var element = $(elementSelector);
  var originalTop = element.offset().top;
  console.log("follow_scroll_smoothly originalTop=" + originalTop);

  // Space between element and top of screen (when scrolling)
  var topMargin = 5;

  // Should probably be set in CSS; but here just for emphasis
  element.css('position', 'relative');

  $(window).on('scroll', function(event) {
    // get the vertical position of scrollbar
    var scrollTop = $(window).scrollTop();

    var newTop = scrollTop < originalTop ? 0 : scrollTop - originalTop + topMargin;

    element.stop(false, false).animate({
        top: newTop,
    }, 300);
    console.log("follow_scroll_smoothly scrollTop=" + scrollTop + " newTop=" + newTop);
  });
}
~~~

If target element is big enought to go beyond the bottom line of the page, you
need to stop scroll untill user see the bottom.
Note that we use
[outerHeight](http://api.jquery.com/outerheight/) to include element padding and
margin.
There are two approaches. First is symetric which do not scroll if current view
is inside element. Scroll only when user see's top or bottom or does not see
element.

* Symetric rules are:
  * if I scroll down and I see bottom and top is about to hide
    * if I see whole element than follow with top of the element
    * if I don't see whole element than follow with bottom
  * if I scroll up and I see top and bottom is about to hide
    * if I see whole element than follow with bottom of the element
    * if I don't see whole element than follow with top
  * if I don't see both top and bottom and I can not see element (it happens
    when user press `Home` and `End` keys) probably element is smaller than
    window height
    * if it scroll up than follow bottom
    * if it scroll down than follow top

<iframe width="100%" height="300" src="//jsfiddle.net/duleorlovic/ffbnr62f/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

~~~
function follow_scroll_smoothly(elementSelector, footerSelector) {
  var element = $(elementSelector);
  var elementHeight = element.outerHeight();
  var windowHeight = $(window).height();
  var documentHeight = $(document).height();
  var headerMinimumTop = element.offset().top;
  var footerMaximumTop = documentHeight;
  if (footerSelector && $(footerSelector).length) {
    footerMaximumTop = documentHeight - $(footerSelector).outerHeight();
  }
  console.log("follow_scroll_smoothly headerMinimumTop=" + headerMinimumTop);

  // Space between element and top/bottom of screen (when scrolling)
  var topMargin = 5;
  var bottomMargin = 15;

  // Should probably be set in CSS; but here just for emphasis
  element.css('position', 'relative');

  function checkRangeRelativeTop(relativeTop) {
    var result = relativeTop;
    if (result < 0) {
      // can not go above initial position
      result = 0;
    } else if (result > footerMaximumTop - elementHeight - headerMinimumTop) {
      // can not go bewond footer
      result = footerMaximumTop - elementHeight - headerMinimumTop;
    }
    return result;
  }

  function stickTop(scrollTop) {
    var newRelativeTop = checkRangeRelativeTop(scrollTop - headerMinimumTop + topMargin);
    element.stop(false, false).animate({
        top: newRelativeTop,
    }, 300);
    $('#debug').prepend('<br>stickTop ' + newRelativeTop);
  }
  function stickBottom(scrollTop) {
    // we don't use animate bottom since it is relative to element, we use top
    var newRelativeTop = checkRangeRelativeTop(scrollTop - headerMinimumTop + windowHeight - elementHeight - bottomMargin);

    element.stop(false, false).animate({
        top: newRelativeTop,
    }, 300);
    $('#debug').prepend('<br>stickBottom ' + newRelativeTop);
  }
  var lastScrollTop = 0;
  $(window).on('scroll', function(event) {
    // get the vertical position of scrollbar
    var scrollTop = $(window).scrollTop();
    var scrollDownDirection = scrollTop > lastScrollTop;
    lastScrollTop = scrollTop;
    var elementTop = element.offset().top
    var elementBottom = elementTop + elementHeight;
    var weSeeBottom = windowHeight + scrollTop > elementBottom && scrollTop < elementBottom;
    var weSeeTop = windowHeight + scrollTop > elementTop && scrollTop < elementTop;
    $('#info').html(JSON.stringify(
      {
        windowHeight: windowHeight,
        headerMinimumTop: headerMinimumTop,
        elementHeight: elementHeight,
        elementTop: elementTop,
        footerMaximumTop: footerMaximumTop,
        // scrollTop: scrollTop,
        // weSeeBottom: weSeeBottom,
        // weSeeTop: weSeeTop,
        // scrollDownDirection: scrollDownDirection,
      }
    ));

    if (scrollDownDirection && weSeeBottom && !weSeeTop)
    {
      if (windowHeight > elementHeight) {
        // we see whole element
        stickTop(scrollTop);
      } else {
        // we don't whole element because top is out of page
        stickBottom(scrollTop);
      }
    }
    else if (!scrollDownDirection && !weSeeBottom && weSeeTop)
    {
      if (windowHeight > elementHeight) {
        stickBottom(scrollTop);
      } else {
        stickTop(scrollTop);
      }
    }
    else if (!weSeeBottom && !weSeeTop && (elementBottom < scrollTop || elementTop > scrollTop + windowHeight))
    {
      if (scrollDownDirection) {
        stickTop(scrollTop);
      } else {
        stickBottom(scrollTop);
      }
    }
  });
}
~~~

Another approach is to keep top as much as he can (until bottom reaches bottom).
When element is bigger than page, user needs to scroll down to the bottom to see
bottom of the element.

<iframe width="100%" height="300" src="//jsfiddle.net/duleorlovic/ffbnr62f/8/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

# Coffeescript tips

* [try coffeescript](http://coffeescript.org/) and convertor
  [js2coffe](http://js2.coffee/)
* no need `;` at the end of line
* block {...} is replaced with `->` and proper indend (two spaces) or could be
  inline
* parantheses (arg) in one line can be ommited (when there are some arguments),
  in multiple lines they are required. Don't mix inline and multiline, since it
  will resolve to two arguments (with type object). Don't know how to break long
  inline arguments to two line. At least first arg should be in new line, others
  can be on the same line or in separate line.
* `()` are required when you want to call function without arguments
  (otherwise it will just return reference to the function)
* object definition {...} braces can be ommited, name/value pairs could be on
  new lines (if object is only argument than parentheses can be ommited)
* No need to write return command. Since last line is returning, put empty
  `return` in angular constructor functions
* conditional ternary (triple) operator in `? : ` can be used with `if then else`
* [loops](http://coffeescript.org/#loops) are simiral to ruby, and it's
  better than ES5 `forEach` since we can `break` from the loop. For example
  find by objectId in array carts:

  ~~~
  service.findOrCreateCartForRestaurant = (restaurant) ->
    resultCart = null
    for cart in service.carts
      if cart.restaurantId == restaurant.id
        resultCart = cart
        break
    if !resultCart
      resultCart =
        restaurantId: restaurant.id
        restaurantName: restaurant.name
        total: 0
        cartItems: []
      service.carts.push resultCart
    resultCart
  ~~~

  Another solution is to use `resultCart = $filter('filter')(service.carts,
  (cart) -> { cart.id == restaurant.id })` or from
  [cookbook](https://coffeescript-cookbook.github.io/chapters/arrays/filtering-arrays)
  ` resultCart = service.carts.filter (cart) -> cart.id == restaurant.id`


* print empty lines in log

  ~~~
  console.log ("\n" for x in [0..10]).join('')
  ~~~

* [fat arrow =>](http://coffeescript.org/#fat-arrow) allow to `this` in callback
  functions

  ~~~
  Account = (customer, cart) ->
    @customer = customer
    @cart = cart

    $('.shopping_cart').on 'click', (event) =>
      @customer.purchase @cart
  ~~~

* multiline text without or with new lines

  ~~~
  s = "This is
    very long line"
  p = "this-long-url\
      -joined-without-space"
  q =
    """
    This is
    very long line
    """
  ~~~

* excellent reference for
  [coffeescript-cookbook](https://coffeescript-cookbook.github.io/chapters/arrays/filtering-arrays)
* global variables can be attached to `window` object like `window.my_var = 42`
or you can use `@` syntax `@my_var = 42`
* [iterate loops](http://coffeescript.org/#loops) over hash object `for k, v of
my_object`. If you need index of array use also two params `for value, index of
array`. In ES6 you can `myArray.forEach(function(val) { console.log(val) }`. To
convert HtmlCollection to array you can use `Array.from(htmlCollection)`
* you can call functions without parantesis but if you have params. But if you
do not have params, that would be just a reference to a function, to call it you
can use `do`
[link](https://coffeescript-cookbook.github.io/chapters/functions/parentheses)

  ~~~
  event.preventDefault()
  # or
  do event.preventDefault
  ~~~

  another usage of do is for [closures]({{ site.baseurl }} {% post_url 2016-02-10-javascript-theory %}#closures)
* you can use `class`, `constructor`, `@instance_valiable =`, `@class_variable:
` for
[classes](http://coffeescript-cookbook.github.io/chapters/classes_and_objects/)
for example you can define constants
https://coderwall.com/p/1dckba/constants-in-coffeescript
```
# app/assets/javascripts/const.coffee.erb
class window.Const
  @DATATABLE_SR_LANGUAGE_URL: '<%= asset_path 'plugins/datatables/i18n.sr.json.erb' %>'
  @SOME_OBJECT: {
    a: 1
  }
```

so you can access in other js files

```
url = Const.DATATABLE_SR_LANGUAGE_URL
my_obj = Const.SOME_OBJECT
```

# Difference ES6 (ie ES2015) and ES5

This [post](http://kamranahmed.info/blog/2016/04/04/es6-in-depth/) explains what
is new in ES6 (Ecma Script 2015). Almost all those features already exists in
coffee script [nice reply](https://gist.github.com/benjie/d0e39fbe8a61bc30ed93).
It's good that no need to write `function` keyword. Also nice table of
[es6-features](http://es6-features.org/) and [supported
browsers](http://kangax.github.io/compat-table/es6/)

* `use strict` mode by default (by this
  [info](http://arcturo.github.io/library/coffeescript/07_the_bad_parts.html)
  you should use in development but not on production

# Books

* https://performancejs.com/post/hde6d32/The-Best-Frontend-JavaScript-Interview-Questions-%28written-by-a-Frontend-Engineer%29
* https://github.com/getify/You-Dont-Know-JS
* best practices https://github.com/wearehive/project-guidelines

# Colors

You can generate random color palete in javascript

~~~
  function getRandomColor() {
    var letters = '0123456789ABCDEF'.split('');
    var color = '#';
    for (var i = 0; i < 6; i++ ) {
        color += letters[Math.floor(Math.random() * 16)];
    }
    return color;
  }

  var color = getRandomColor()
  // add transparency for background colors
  var backgroundColor = color + 'A0'
~~~

If you need same distinct colors based on integer numbers you can create a
deterministic function.
Here is example in ruby
~~~
  def add_colors(datasets)
    # http://martin.ankerl.com/2009/12/09/how-to-create-random-colors-programmatically/
    h = 0.5
    golden_ratio_conjugate = 0.618033988749895
    colors = (1..datasets.length).to_a.map do
      h += golden_ratio_conjugate
      h %= 1
      hsv_to_rgb(h, 0.5, 0.95)
    end.map do |r,g,b|
      {
        full: "rgba(#{r},#{g},#{b},1)",
        transparent: "rgba(#{r},#{g},#{b},0.2)"
      }
    end.map { |c| OpenStruct.new c }

    result = datasets.each_with_index.map do |dataset,i|
      {
        fillColor: colors[i].transparent,
        strokeColor: colors[i].full,
        pointColor: colors[i].full,
        pointStrokeColor: "#fff",
        pointHighlightFill: "#fff",
        pointHighlightStroke: colors[i].full,
      }.merge dataset
    end
    result
  end

  # HSV values in [0..1]
  # returns [r, g, b] values from 0 to 255
  def hsv_to_rgb(h, s, v)
    h_i = (h*6).to_i
    f = h*6 - h_i
    p = v * (1 - s)
    q = v * (1 - f*s)
    t = v * (1 - (1 - f) * s)
    r, g, b = v, t, p if h_i==0
    r, g, b = q, v, p if h_i==1
    r, g, b = p, v, t if h_i==2
    r, g, b = p, q, v if h_i==3
    r, g, b = t, p, v if h_i==4
    r, g, b = v, p, q if h_i==5
    [(r*256).to_i, (g*256).to_i, (b*256).to_i]
  end
~~~

# Javascript tips

* [modern js cheatsheet](https://github.com/mbeaudru/modern-js-cheatsheet)
* to see all keys of `myObj` you can use `Object.keys(myObj)`
* alert object with `alert(JSON.stringify(myObj))`
* if you want to break array `forEach` method when you find el, you can use
  [some](http://stackoverflow.com/questions/2641347/how-to-short-circuit-array-foreach-like-calling-break)

* to remove item `myObject` from array `myArray.splice
  myArray.indexOf(myObject),1 `
* `!!variable` will return false only for `0, null, "", undefined, NaN` NaN=Not
  a number
* to check if some property is defined on a object (method or variable), you can
  use `in` for example `a = {b:1}; 'b' in a // returns true`
* get last element of array `array.slice(-1);`
* replaceAll is the same as `/g` example `"asdasd".replace(/s/g,"***")`
* merging arrays is better to append `array1.push.apply(array1, array2)` than to
  create new `array1.concat(array2)`
* instead of `parseInt('10')` you can use `+'10'` to type cast string to number
* `jQuery.data()` is different than plain javascript `this.dataset` because
  javascript always returns string `this.dataset.a // "[1]"` but jQuery use
  JSON.parse and returns string in case it is not valid json `$(this).data('a')
  // [1]`. Note that integers will be integers not strings.
  Also IE10 does not support dataset, so better is to use `$(this).data().a`.
* if variable point to the same array or object, than it will be the same
  `a=[1];b=a;a==b//true`. But if it different object (event with same data)
  variables will not have the same value. `a=[1];b=[1];a==b//false` or
  `{a:1}=={a:1}//false`. Comparing arrays or objects should be done one by one
  [link](http://stackoverflow.com/questions/7837456/how-to-compare-arrays-in-javascript)


* if you are using `window.location.replace('http://a.b');` then browser back
  button is not working as espected. Its better to use
  `window.location.assign('new_page');` because the page is stored in history
  or `window.location.href = 'http://www.google.com'`
* when user click back button previous page is reshown, and chrome reruns
  the javascript, but mozilla doesn't. if you want in mozilla to run again,
  use cache buster or try with:

  * `window.onunload = function(){};`
  * `$(window).unload(function () {});`
  * `$(window).bind("pageshow", function() {});`

  In both mozilla and chrome, form fields stays populated, so you can use
  `$('form').each(function() { this.reset(); });` which is not user friendly, it
  is better to target only one form

  ~~~
  <form id="reset-form"></form>
  <script>
    $(window).bind("pageshow", function() {
      document.getElementById('reset-form').reset();
    });
  </script>
  ~~~


* submit button should be disabled when text input or textarea are blank. You
can enable button on change, but it should listen [other
events](http://www.w3schools.com/tags/ref_eventattributes.asp) as well so it is
enable for example, on onpaste.

~~~
<form>
  <input onchange="d(this)" onkeyup="this.onchange()" onpaste="this.onchange()" oninput="this.onchange()" \>
  <button id="send_button" disabled="disabled">Send</button>
</form>
<script>
function d(e){
  document.getElementById("send_button").disabled = e.value.length == 0;
}
</script>
~~~

this is unobtrusive version, works for input and select elements

~~~
# app/assets/javascripts/common.coffee
  $('[data-disable-button-if-empty]').on 'change paste keyup input', ->
    disabled = this.value.length == 0
    $(this).parents('form').first().find('[type=submit]').prop('disabled', disabled)

  # initial status
  $.each $('[data-disable-button-if-empty]'), (index, el) ->
    $(el).trigger('change')

# app/views/posts/form.html.erb
    <%= f.text_field :name, class: "input-large", 'data-disable-button-if-empty' => true %>
    <%= f.collection_select :post_id, some_posts, :id, :name, { include_blank: 'Select Package' }, class: "field span5", 'data-disable-button-if-empty' => true %>
~~~

# regex
[match](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)

* when you are sending object with GET than you need to make it encoded (or you
can use jQuery.get(url, data):

  ~~~
    var xhr = new XMLHttpRequest();
    // http://stackoverflow.com/questions/5505085/flatten-a-javascript-object-to-pass-as-querystring
    function toQueryString(obj) {
      var parts = [];
      for (var i in obj) {
          if (obj.hasOwnProperty(i)) {
              parts.push(encodeURIComponent(i) + "=" + encodeURIComponent(obj[i]));
          }
      }
      return parts.join("&");
    }
    xhr.open('GET', '/notify-javascript-error?' + toQueryString(data));
    xhr.send();
  ~~~

* to create jquery elements you can select parent element and append to it:
`$('p').append('<span>hi</span>')` or you can create and use appendTo
`$('<span>hi</span>').appendTo('body');`. Instead of elements as html blocks, if
you have a lot of javascript variables, you can pass additinal parameters and
use quick self closed tags like

~~~
$('<div/>', {
  id: 'foo',
  css: {
    fontWeight: 600
  },
  click: function() {
    alert('Hi');
  }
});
~~~

you can add `$('p').after("some text")` or `$('p').before('some text')`

* click event buble up, so when you attach click

  ~~~
  <button type="button" class="btn btn-box-tool" data-widget="collapse" data-user-preferences="expand_location_reports_customers"><i class="fa fa-minus"></i>
  ~~~

  target will will be inner `<i>` and currentTarget will be `<button>` so
  targetElement (e.target) should be e.currentTarget

  ~~~
  @initializeUserPreferences = ->
    $('[data-user-preferences]').click (e) ->
      preference = e.currentTarget.dataset.dataUserPreferences
  ~~~

* disable right click on a page with `document.addEventListener('contextmenu',
  event => event.preventDefault());`. If you want to reenable on some page,
  use global search (Ctrl+Shift+F in chrome) for `contextmenu` and put
  breakpoint and comment that line, save and continue F8.

* `fetch` makes js requests. If you need to pass header for cookie session than
  use param https://github.com/github/fetch#sending-cookies

  ~~~
  fetch('/users', {
    credentials: 'same-origin'
  })
  ~~~

* jquery.each should not return `false` since it will break the loop, for in
  coffeescript you should add `true`
  ```
    $('[data-enable-buttons]:checked').each ->
      this.checked = false
      true
  ```
