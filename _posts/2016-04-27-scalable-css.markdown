---
layout: post
title: Scalable css
tags: css
---

# Smacss

[book](https://smacss.com/book/) 5 categories:

* base: usually single element  (`a`,`h1`) or pseudo-class (`a:hover`)
  selectors (but not a class `my-class` selector). Its default styling for
  elements in all occurences on the page.
  * `p + p` is sibling selector, so only for paragraph that is after paragraph
  (so it wont affect first p, only ones that are immediately after p)
  * `p ~ p` is [general
    sibling](https://www.w3.org/TR/selectors/#general-sibling-combinators) so
    all p elements who has sibling p which if before (not neccessary
    immediatelly before).
* layout: divide page into sections. prefix with `l-` like `l-inline`
* module: reusable parts: `callouts`, `products`... When used in different part
  of page you should use subclassing class_name-module, for example: `<div
  class="pod pod-callout">` instead of specificity war `.callout .pod`
* state: how module or layout looks in particular state (active/inactive, home
  page/sidebar, small/big screens), prefixed with `is-`, like `is-hidden`. For
  specific states of module, you can use `is-callout-minimized` and place it
  inside module code so its loaded only when module is loaded (for just-in-time
  loading). State changes can happen by: class name (js add/remove class),
  pseudo class (:hover,:focus) and media query. Always keep module all css code
  inside same file.
* theme: how module or layout might look

# BEM: Block __Element - -Modifier

[bem.info](https://en.bem.info/) & [getbem](http://getbem.com/introduction/) is
a way to write classes with `--` and `__` so instead of `menu-item-visible`
write *block-name__elem-name--mod-name* `menu__item--visible`

* *blocks* are independent page component that contains other blocks or
  *elements*
* elements can't be used outside of a block and they start with
  `__`
* *modifiers* defines appearance and behavior on block or elements and
  start with `--` (or `_`). They always prefixed with parent block/element
  name, so instead of `button active` we write `button button-active` 3 reasons
  for that: single level specificity, `item button active` don't know if
  `item--active` or `button--active` and we clearly see that it is modifier and
  not some other block/element
* do not use semantic tags, its better to write `<button class="button
  button--mod">` beause you can change tag to `<a>` and you can mix `button`
  class on other places
* do not write nested rules, except when you write for block modifier element
  `.block--modifier .block__element`
* do not use global modifier that you want to apply to any class like `block
  hidden`. Better is to use `block--hidden`
* when you have nested blocks than instead
  `block__nested-block__double-nested-block` use `block__nested-block` and
  `block__double-nested-block` or `nested-block__double-nested-block` depending
  on how you can move *double-nested-block*

* do not override styles of another block


* <http://work.stevegrossi.com/2014/09/06/how-to-write-css-that-scales/>
* <https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Writing_efficient_CSS#Avoid_the_descendant_selector.21>

* *key selector* is the right most part of selector (could be `#id`, `.class`,
  tag `h1` or universal `[hidden="true"]` category). Avoid plain universal and
  avoid mixing other three categories.
* avoid descendant selector `table[hidden="true"] td tr`, betters is to use
  child selector `table[hidden="true"] > td > tr` but event that should be
  avoided with duplicating attributes `tr[hidden="true"]`
* rely on inheritance, instead of `#bookmarkMenuItem > .menu-left {
  list-style-image: url(blah) }` its enough to write `#bookmarkMenuItem {
  list-style-image: url(bla) }`

# CSS variables

* you can set to `:root { --my-var: red; }` and than use later with `p { color:
  var(--my-var); }`. Those properties are inherited from elements where are
  defined.


# Live reloading in less than a second

[Sharetribe](https://github.com/sharetribe/sharetribe/blob/master/docs/scss-coding-guidelines.md) uses [fast reloading](https://mattbrictson.com/lightning-fast-sass-reloading-in-rails)

# SCSS Sass

* you can set default value of variable `$my-var: 123 !default;`. This has no
  effect if variable is already defined.
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

<https://responsivedesign.is/develop/getting-started-with-sass>

* `@extend .email-container` will copy definitions from email container
