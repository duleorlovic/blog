---
layout: post
title: Angular begginer examples
tags: angular
---

Some of angular attribute directives:

* `ng-app="store"` defines root element of module "store" which can have many
  controllers
* `ng-controller="StoreController as store"` defines controller, but it is
  usually defined in routes for specific template
* `ng-repeat="product in store.products"` iteration of this element (you can 
  use also `ng-if="$even"`, or `$index` variables)
* `ng-show="product.CanPurchase", ng-hide="expession"`, `ng-disabled="mySwitch"` to show/hide/disable element
* `<img ng-src="{{ "{{ product.image" }}}}">` to prevent browser to load empty src, this is only place where directive use {{ "{{" }} }}
* `ng-click=" tab = 2 "` to run a code on click
* `ng-class="{ active: tab === 1 }"` set class to key when value is true (good to have `ng-init="tab = 0"`)
* `ng-model="review.terms"` bind value of current input element to variable
* `ng-bind="review.terms"` bind innerHTML of this element to the variable (can be expression)
* `ng-submit="reviewCtrl.addReview(product)"` call function on submit
* `ng-include="'product-title.html'"` include template
* `ng-view` is placeholder where routes inject templates

Angular expressions can't have conditionals and loops, but have filters.

# Builtin $properties:

For form name *myForm* we can check `myForm.$valid` for all fields that we put
validation with `required` property on input elements (form can have `novalidate`
attribute to prevent browser default behavior, or we can use `ng-required="true"`). Also fields have internal
properties: `$dirty` (user has interacted), `$valid`, `$invalid` and`$pristine`
(has not interacted with field yet). 
You can use `ng-messages` instead of `ng-show`

~~~
<form name="myForm">
  <input type="email" name="email" ng-model="email" required>
  <div ng-messages="registration.email.$error">
    <div ng-messages-include src="default-messages"></div>
  </div>
  <div ng-messages="vm.serverErrors.registrationForm.email">
    <div>{{ vm.serverErrors.registrationForm.email.join(', ') }}</div>
  </div>
</form>
<script type="text/ng-template" id="default-messages">
  <div ng-message="required">This field is required</div>
</script>
~~~

`$scope.$watch('email', function() { $scope.test(); })` can be used to rerun validation check.
Expressions in angular directives `<button ng-click="foo = 5">` are parsed and evaluated. `$parse()` takes expression and returns function which takes two arguments: context and locals (overrides).

~~~
$scope.foo = 3;

var parseFn = $parse(‘foo = 5’);
parseFn($scope);
// or
$scope.$eval(‘foo = 5’);

$scope.foo; // returns 5
~~~

You can check if some property `bar` on `foo` is not null: `$parse('bar.baz.quux')(foo)` will return undefined instead of throwing an exception.

* you can manually trigger digest with `.$apply("foo = 5")` (it calls `.$digest()` on the root scope which propagates down to every child scope).

# Filters: {{ " {{ data | filter:options" }} }}

* strings `{{ "{{ 'Some string' | date:'MM/dd/yyy @ h:mma'"}} }}` can also be `currency, lowecase, uppercase`
* iterators `ng-repeat="product in store.products | filter: search | orderBy: '-price' | limitTo: -5 "` use only products that match value of search (any property of product that match search string) order by desc price and show last 5. If we want to filter only product.name than bind input to `search.name` so search becomes object, for example `ng-model="search.name"`.

View can use directives, filters. Controller and services can depend on some providers.

* all objects are joined using [dependency injection](https://docs.angularjs.org/guide/di)
  for [specialized objects](https://docs.angularjs.org/guide/providers) 
  (controllers, directives, filters, animations) or custom service. There are 5
  types of recipe for creating object with injector: Provider (is the main one,
  other are syntactic sugar on top of this provider recipe), Value, Factory,
  Service and  Constant.
  You can use DI when defining components (only controller can have `$scope` and
  `resolve` dependency) or in `run` methods (can not inject Providers) or 
  `config` method (can not inject Service or Value) on module. Only module can
  define other components: `controller` and factories: `directive`, `filter`,
  `value`, `factory`, `service`.
* all services in Angular are singletons. injector uses each recipe at most once
  to create the object. The injector then caches the reference for all future
  needs. Controllers are instantiated every time app needs it.
* use strict dependency injection `ng-app="myApp" ng-strict-di"` to raise error
  when we use implicit annotations (`function($scope)` instead of `['$scope',
  function($scope)]`)

## Special Purpose Objects: controlles, filters, directives, animations

* controller methods can be defined on $scope or on this (so we reference them
  `MyController.myMethod`)

~~~
angular.module('myApp',['controllers']);
angular.module('controllers', [])
  .controller('MyController', ['$http', '$scope', function($http, $scope) {
    this.my_value = 2;
    $scope.myMethod = function() { return 2 };
}]);
~~~

* new filter definition should return function

~~~
# filter
angular.module('myApp',['myFilters']);
angular.module('myFilters',[])
.filter('checkmark', function() {
  return function(input) {
    return input ? '\u2713' : '\u2718';
  };
});
~~~

It can be used like `{{ "{{ phone.available | checkmark" }} }}`

* two types of directives: element directive (UI widget) `<product-title></product-title>`
  or attribute directive (mixins like tooltip) `<div product-title></div>`.

~~~
angular.module('myApp')
  .directive('productTitle', function(){
    return {
      restict: 'E', // 'E' element, 'A' attribute (default), 'C' class
      templateUrl: 'product-title.html',

      controller: function() {},
      controllerAs: 'panel',
    };
  });
  .directive('enter', function(scope, element, attrs){}
    // default is 'A' so no need to write return {  link: function(){}
    return function(scope,element, attrs) {
      element.bind("mouseenter", function() {
        scope.$apply(attrs.enter); # this will call argument string, ie some controller method
      });
    };
  });
~~~

* directives has `transclusion()` to create new scope

## Services for creating objects: Value, Factory, Service, Provider, Constant

* Value recipe, can't have other dependencies

~~~
var myApp = angular.module('myApp', []);
myApp.value('clientId', 'a12345654321x');
~~~

* Factory recipe use other services, is a function and lazy initialized. It returns object.

~~~
myApp.factory('apiToken', ['clientId', function apiTokenFactory(clientId) {
  var encrypt = function(data1, data2) {
    // NSA-proof encryption algorithm:
    return (data1 + ':' + data2).toUpperCase();
  };

  var secret = window.localStorage.getItem('myApp.secret');
  var apiToken = encrypt(clientId, secret);

  return apiToken;
}]);
# factory
angular.module('myApp',['myServices']);
angular.module('myServices')
.factory('sport', function() { 
  return {
    title: "Kayak"
  }
});
# factory that use ngResource provider
angular.module('myServices',['ngResource'])
  .factory('Phone', ['$resource', function($resource) {
    return $resource('phones/:phoneId', { phoneId: "@id", format: 'json' }, {
      query: {method: 'GET', params: { phoneId: 'phones' }, isArray: true },
      save:  { method: 'PUT' },
      create: { method: 'POST' }
    });
  }]);
~~~

* new service can be added with `.factory` which returns `new SomeFun(arg)` or with `.service` function. Service methods are defined as properties of `this`. Service just use $injector to create new instance of service's contructor function.

~~~
function factory(name, factoryFn) { 
    return provider(name, { $get: factoryFn }); 
}

function service(name, constructor) {
    return factory(name, ['$injector', function($injector) {
      return $injector.instantiate(constructor);
    }]);
}
~~~

~~~
function UnicornLauncher(apiToken) {

  this.launchedCount = 0;
  this.launch = function() {
    this.launchedCount++;
  }
}
myApp.factory('unicornLauncher', ["apiToken", function(apiToken) {
  return new UnicornLauncher(apiToken);
}]);
// or
myApp.service('unicornLauncher', ["apiToken", UnicornLauncher]);
~~~

Factory has more flexibility since they can return functions which can be `new`ed.

* [$resource](https://docs.angularjs.org/api/ngResource/service/$resource) by default `save` is POST, so we need to change for rails PUT, and add `create`. Factories need to return object. That object is bindable in all places where we use `Phone` factory.
  It can be used like `$scope.phone = Phone.get({ phoneId: $routeParams.phoneId }); $scope.phones = Phone.query();`

* Provider is custom type with a `$get` method that is a factory function. Provider can be configured. For provider name `unicornLauncher` angular registers `unicornLauncher` and `unicornLauncherProvider` injectables.

~~~
myApp.provider('unicornLauncher', function UnicornLauncherProvider_name_not_important() {
  var useTinfoilShielding = false;

  this.useTinfoilShielding = function(value) {
    useTinfoilShielding = !!value;
  };

  this.$get = ["apiToken", function unicornLauncherFactory(apiToken) {
    return new UnicornLauncher(apiToken, useTinfoilShielding);
  }];
});

myApp.config(["unicornLauncherProvider", function(unicornLauncherProvider) {
  unicornLauncherProvider.useTinfoilShielding(true);
}]);
~~~

* Constant recipe can create value that is available in both config and run phase

~~~
myApp.constant('planetName', 'Greasy Giant');
myApp.config(['unicornLauncherProvider', 'planetName', function(unicornLauncherProvider, planetName) {
  unicornLauncherProvider.useTinfoilShielding(true);
}]);
~~~

~~~
.constant('USER_ROLES', {
  all: '*',
  admin: 'admin',
  editor: 'editor',
  guest: 'guest'
})
~~~

# Angular routes

`angular-route` is separated package and need to be added as module dependency and configured. `config` needs Providers (factory for service that will be configured) for example `$routeProvider` and `run` or `controller` needs service for example `$route`

~~~
angular.module('myApp',[ 'ngRoute', 'myAppControllers' ])
.config(function($routeProvider, $locationProvider){
  $routeProvider
  .when('/phones/:id', {
    templateUrl: 'details.html',
    controller: 'PhoneDetailCtrl',
  })
  .when('/phones', {
    templateUrl: 'list.html',
    controller: 'PhoneListCtrl',
  })
  .otherwise({redirectTo: '/phones'});
});
~~~

It is used with directive `<ng-view></ng-view>` which is replaced with given template.

# RootScope

* `$rootScope` has not parent and was created directly from `Scope()` class (not through `$.new` method). It is used for event handling `$.broadcast()` (down) and `$.emit()` (up in the scope hierarchy).


# Share error messages

~~~
<div ng-messages="myForm.myField.$error" ng-messages-include="my-messages">
  <div ng-message="required">Custom error required</div>
</div>
<script type="text/ng-template" id="my-messages">
  <div ng-message="required">This field is required</div>
  <div ng-message="email">This field must be an email</div>
</script>
~~~

# Tips:

* use `controller as vm` in view and `var vm = this` in controller so you don't need to inject `$scope` (we need `$scope` when we want to access something in promise `catch = -> $scope.vm.profileForm = "a"`)

* in directives you need to prefix `$root` to access rootScope, for example `$root.user`

* by [Angular conventions](https://github.com/mgechev/angularjs-style-guide), lowerCamelCase is used for factory names that won't be new'ed.

Check some [awesome links](https://github.com/gianarb/awesome-angularjs)

# Testing
