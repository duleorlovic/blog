---
layout: post
title: Angular Material
tags: css angular
---

* [md-autocomplete](https://material.angularjs.org/latest/demo/autocomplete) to
  automatically open, you need `md-min-length="0"`
* to align button to right you can insert `<span flex></span>` before so it
  covers the space before element, and wrap inside `layout="row"`

  ~~~
  div(layout="row")
    span(flex)
    button
  ~~~

  Another approach is to use `layout-align="end"`

* [md-autofocus](https://material.angularjs.org/1.0.4/api/directive/mdAutofocus)
  is nice tool to save users clicks, just add to attribude like `<input
  md-autofocus="!vm.menuItem.name">`. It works only on `$mdDialog`,
  `$mdBottomSheet` and `$mdSidenav`

* [md-select](https://material.angularjs.org/latest/api/directive/mdSelect) and
  `md-option` should have values inside quotes like
  `md-option(ng-value="'delivery_only'")`
  
  * use `$scope.$watch 'vm.restaurantCart.pickUp'` instead of `ng-click` since
    value is not yet updated in `ng-click` hook

# Animations

* 
