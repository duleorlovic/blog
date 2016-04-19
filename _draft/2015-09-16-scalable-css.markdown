---
layout: post
title: Scalable css
tags: css
---

5 categories:

* base: usually single element selectors or pseudo-class (but not a class
  selector). Its default styling for elements in all occurences on the page.
* layout: divide page into sections. prefix with `l-` like `l-inline`
* module: reusable parts: callouts, product lists... When used in different part
  of page you should use subclassing class_name-module, for example: `<div
  class="pod pod-callout">` instead of specificity war `.callout .pod`
* state: how module or layout looks in particular state (active/inactive, home
  page/sidebar, small/big screens), prefixed with `is-`, like `is-hidden`. For
  specific states of module, you can use `is-callout-minimized` and place it
  inside module code so its loaded only when module is loaded (for just-in-time
  loading).  State changes can happen by: class name (js add/remove class),
  pseudo class (:hover,:focus) and media query. Always keep module all css code
  inside same file.
* theme: how module or layout might look


http://work.stevegrossi.com/2014/09/06/how-to-write-css-that-scales/

https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Writing_efficient_CSS#Avoid_the_descendant_selector.21

* *key selector* is the right most part of selector (could be `#id`, `.class`,
  tag `h1` or universal `[hidden="true"]` category). Avoid plain univarsal and
  avoid mixing other three categories.
* avoid descendant selector `table[hidden="true"] td tr`, betters is to use
  child selector `table[hidden="true"] > td > tr` but event that should be
  avoided with duplicating attributes `tr[hidden="true"]`
* rely on inheritance, instead of `#bookmarkMenuItem > .menu-left {
  list-style-image: url(blah) }` its enough to write `#bookmarkMenuItem {
  list-style-image: url(bla) }`


https://en.bem.info/

* *blocks* are independent page component that contains other blocks or
  *elements* (can't be used outside of a block). Modifiers defines appearance
  and behavior
* instead of `menu-item-visible` (block-element-modifier) write
  *block-name__elem-name--mod-name* `menu__item--visible`


# Live reloading in less than a second

[Sharetribe](https://github.com/sharetribe/sharetribe/blob/master/docs/scss-coding-guidelines.md) uses [fast reloading](https://mattbrictson.com/lightning-fast-sass-reloading-in-rails)


# Tips

* if you have `<small>` position relative, and apply left -100px, it will still
occupy the space where it was
* to move element to the right but keep inside parent element you need to mark
  parent as `position: relative` and than use `position: absolute; right: 10px`
* use `mouseenter` instead `mouseover` (which will retrigger on inner elements).
* use relative units instead of absolute
  <http://www.w3schools.com/cssref/css_units.asp>, for example header `h1 { font-size: 20vh;}`

 https://responsivedesign.is/develop/getting-started-with-sass
