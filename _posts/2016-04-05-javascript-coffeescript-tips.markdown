---
layout: post
title: Javascript & coffeescript tips
---

* to see all keys of `myObj` you can use `Object.keys(myObj)`
* alert object with `alert(JSON.stringify(myObj))`
* if you want to break array `forEach` method when you find el, you can use
  [some](http://stackoverflow.com/questions/2641347/how-to-short-circuit-array-foreach-like-calling-break)

* to search file in google developer tools you can open console window (with
  ESC) than on three dots, open dropdown menu and find *Search*
* to remote item `myObject` from array `myArray.splice
  myArray.indexOf(myObject),1 `

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


# Coffeescript

* [try coffeescript](http://coffeescript.org/) and convertor
  [js2coffe](http://js2.coffee/)
* no need `;` at the end of line
* block {...} is replaced with `->` and proper indend (two spaces) or could be
  inline
* parantheses (arg) in one line can be ommited (when there are some arguments),
  in multiple lines they are required.  Note that () are required when you want to call
  function without arguments (otherwise it will just return reference to the
  function)
* object definition {...} braces can be ommited, name/value pairs could be on
  new lines (if object is only argument than parentheses can be ommited)
* No need to write return command. Since last line is returning, put empty
  `return` in angular constructor functions
* conditional ternary operator in `? : ` can be used with `if then else`
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
~~~

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
