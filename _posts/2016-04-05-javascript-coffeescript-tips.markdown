---
layout: post
title: Javascript & coffeescript tips
---

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

# Always on page, floating elements

There is nice plugin [stickyjs](http://stickyjs.com/) but for simple scroll you
can use this
[answer](http://stackoverflow.com/questions/2177983/how-to-make-div-follow-scrolling-smoothly-with-jquery)
First version is enought if you do not need to scroll big elements. Note that
calculation of `originalTop` could be wrong if there are some not images and
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

* Symretic rules are:
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
When element is bigget than page, user needs to scrol down to the bottom to see
bottom of the element.

<iframe width="100%" height="300" src="//jsfiddle.net/duleorlovic/ffbnr62f/8/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

# Tips

* to see all keys of `myObj` you can use `Object.keys(myObj)`
* alert object with `alert(JSON.stringify(myObj))`
* if you want to break array `forEach` method when you find el, you can use
  [some](http://stackoverflow.com/questions/2641347/how-to-short-circuit-array-foreach-like-calling-break)

* to search file in google developer tools you can open console window (with
  ESC) than on three dots, open dropdown menu and find *Search*
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
  // [1]`.
* if variable point to the same array or object, than it will be the same
  `a=[1];b=a;a==b//true`. But if it different object (event with same data)
  variables will not have the same value. `a=[1];b=[1];a==b//false` or
  `{a:1}=={a:1}//false`. Comparing arrays or objects should be done one by one
  [link](http://stackoverflow.com/questions/7837456/how-to-compare-arrays-in-javascript)

# Coffeescript

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
  better than angular `forEach` since we can `break` from the loop. For example
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

# Jade

Very nice tool, like coffescript. You can convert all html files with:

~~~
npm install -g html2jade
html2jade *.html --bodyless --donotencode --noattrcomma --noemptypipe
# --bodyless          do not output enveloping html and body tags
# --donotencode       do not html encode characters (useful for templates)
# --noattrcomma       omit attribute separating commas
# --noemptypipe       omit lines with only pipe ('|') printable character
rm *.html

# version for all files in subdirectories
find . -name '*.html' -exec html2jade {} --bodyless --donotencode --noattrcomma --noemptypipe \;
find . -name '*.html' -exec  rm {} \;
~~~

Please check if format is good. At least this command will ignore <body> tag so
you need to build that (index.html) separately. Usually index.html needs to be
at specific location so I leave it in html format, and all other move to
`jade_build` folder.

* arguments could be in new line, but strings need `\`. comments are not allowed
  inside ()

  ~~~
  // comment should be outside of (arguments)
  md-button(ng-really-click='vm.delete()'
  ng-really-message='Are you sure to \
  remove this option?')
  ~~~

  I prefer not to indent attributes even they suggest to [indent
  them](http://jade-lang.com/reference/attributes/)

* if you are using `window.location.replace('http://a.b');` then browser back
  button is not working as espected. Its better to use
  `window.location.assign('new_page');` because the page is stored in history
* when user click back button previous page is reshown, and chrome reruns
  the javascript, but mozilla don't. if you want in mozilla to run again,
  write `window.onunload = function(){};` or use cache buster.
* disable turbolinks `document.body.setAttribute('data-no-turbolink','true')`
* turbolinks are good if some of page content is changed using ajax.
  Browser back button when turbolink is enabled shows last content
  (not first that was fatched). When turbolinks is disabled, you can
  force refreshing the page with set_cache_buster before filter.
  Note that any input field stays populated (also hidden input field
  which you eventually populated in javascript):

  ~~~
  def set_cache_buster
    response.headers["Cache-Control"] = "no-cache, no-store, max-age=0, must-revalidate"
    response.headers["Pragma"] = "no-cache"
    response.headers["Expires"] = "Fri, 01 Jan 1990 00:00:00 GMT"
  end
  ~~~

