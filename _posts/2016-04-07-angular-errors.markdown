---
layout: post
title: Angular errors
---

* `TypeError: Cannot read property 'childNodes' of undefined` is problem when
  you are using DOM elements in jQuery in controller
  [issue](https://github.com/angular/angular.js/issues/5069). Solution is to
  wrap inside timeout `setTimeout( function() {someJquery()}, 0)`
* when rendering map which is shown only after when we activate some element (ie
  when $digest finishes, it add some ng-class="{active: vm.showMap}"`) so we
  neet to wait for digest to finish and than show the map `$timeout -> ....`
