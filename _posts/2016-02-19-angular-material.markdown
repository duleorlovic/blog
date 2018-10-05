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
  md-autofocus="!vm.menuItem.name">`. It works ONLY on `$mdDialog`,
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
        { {$chip.name}}
    </md-chip-template>
  </md-chips>
  ~~~

* if `ng-messages` are shown before you interact with input, solution is to
  update angular-material by changing
  `bower.json` file, replace the number with start `"angular-material": "*",`
  and run `bower update`

* [md-scroll-shrink](https://material.angularjs.org/latest/api/directive/mdToolbar)
  on md-toolbar does not work always, so use this
  [fix](http://codepen.io/mikkokam/pen/PqpZoN) to add outher wrapper

* when you perfom server request on `ng-click` (not on `form(ng-submit)` than
  there is an issue that validation message is not shown before user interacts
  with that field.
  Idea to add `button(type="submit")` does not work for me since I don't
  know how to stop form from submit, since `vm.save = (item, myForm) -> return
  if myForm.$errors` don't see errors.
  Solution is to use
  `md-input-container(md-is-error="myForm.price.$error.server")` so it will put
  this container in error mode
  [md-is-error](https://material.angularjs.org/latest/api/directive/mdInputContainer)

  Another thing that could help is to add css:
  [#5837](https://github.com/angular/material/issues/5837)

  ~~~
  md-input-container.md-input-invalid ng-message {
    opacity: 1 !important;
  }
  ~~~

  or in js by adding two attributes
  [#6767](https://github.com/angular/material/issues/6767) (animation stop working)

  ~~~
  md-messages md-auto-hide="false" ng-if="myForm.myField.$touched"
  ~~~

  Problematic validations are also shown if your use `ng-if` instead of
  `ng-show`.

* [mdDialog](https://material.angularjs.org/latest/api/directive/mdDialog) works
  fine (you can put `form` element inside `md-dialog`). You can open and wait
  for results to resolve

  ~~~
  $mdDialog.show
    controller: 'VerifyOrderController'
    controllerAs: 'vm'
    templateUrl: 'app/menu/verifyOrder.html'
    locals:
      restaurant: vm.restaurant
  .then(
    (resp) ->
    (cancel) ->
  )
  ~~~

* show hide attributes
  [link](https://material.angularjs.org/latest/layout/options) `<div
  hide-gt-sm>` or `<div hide show-gt-sm>`

# Theme

Material pick 4 colors (hues/shades) from palette: 500, 300, 800, A100 for
primary` and `warn` intentions, and A200, A100, A400, A700 for `accent`
intention.
[link](https://material.angularjs.org/latest/Theming/03_configuring_a_theme)
[video](https://design.google.com/videos/palette-perfect/).
Button can have different intention with `md-primary md-accent md-warn` classes
and you can pick slightly different variation with `md-hue-1 md-hue-2 md-hue-3`
