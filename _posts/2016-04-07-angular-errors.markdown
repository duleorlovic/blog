---
layout: post
title: Angular errors
---

* `TypeError: Cannot read property 'childNodes' of undefined` is problem when
  you are using DOM elements in jQuery in controller
  [issue](https://github.com/angular/angular.js/issues/5069). Solution is to
  wrap inside timeout `setTimeout( function() {someJquery()}, 0)`
* if you are rendering google map on element which is shown after some 
  activation (like when '$diest' finishes and add some `ng-class="{active:
  vm.showMap}"`) it is advisable to use `$timeout -> ` so we have DOM ready
