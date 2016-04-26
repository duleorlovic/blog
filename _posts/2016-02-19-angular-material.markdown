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

* [md-chips](https://github.com/angular/material/issues/2829) does not work with
  `ng-repeat`. You need to use md-chips api like

  ~~~
  <md-chips ng-model="ctrl.items" readonly="true">
    <md-chip-template>
        {{$chip.name}}
    </md-chip-template>
  </md-chips>
  ~~~

* if `ng-messages` are shown before you interact with input, solution is to
  update angular-material. Check the version with `cat
  bower_components/angular-material/.bower.json | node_modules/json/lib/json.js
  version` (install json with `npm install json`). Update by changing
  `bower.json` file, replace the number with start `"angular-material": "*",`
  and run `bower update`

* [md-scroll-shrink](https://material.angularjs.org/latest/api/directive/mdToolbar)
  on md-toolbar does not work always, so use this
  [fix](http://codepen.io/mikkokam/pen/PqpZoN) to add outher wrapper

* issue when validation message is not shown can be fixed with:
  [1](https://github.com/angular/material/issues/5837)

  ~~~
  md-input-container.md-input-invalid ng-message {
    opacity: 1 !important;
  }
  ~~~
  
  or in js by adding two attributes
  [2](https://github.com/angular/material/issues/6767) (animation stop working)

  ~~~
  md-messages md-auto-hide="false" ng-if="myForm.myField.$touched"
  ~~~

  Problematic validations are also shown if your use `ng-if` instead of
  `ng-show`

# Animations
