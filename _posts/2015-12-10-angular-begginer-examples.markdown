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
* `ng-show="product.CanPurchase"`,`ng-hide="expession"`, `ng-disabled="mySwitch"`
  to show/hide/disable element. In contrast, `ng-if` adds new scope or totally
  removes DOM element.
* `<img ng-src="{{ "{{ product.image" }}}}">` to prevent browser to load empty
  src, this is only place where directive use {{ "{{" }} }}
* `ng-click=" tab = 2 "` to run a code on click
* `ng-class="{ active: tab === 1 }"` set class to key when value is true (good
  to have `ng-init="tab = 0"`)
* `ng-model="review.terms"` bind value of current input element to variable
* `ng-bind="review.terms"` bind innerHTML of this element to the variable (can
  be expression)
* `ng-submit="reviewCtrl.addReview(product)"` call function on submit
* `ng-include="'product-title.html'"` include template
* `ng-view` is placeholder where routes inject templates

Angular expressions can't have conditionals and loops, but have filters.


`$scope.$watch('email', function() { $scope.test(); })` can be used to rerun
validation check.
[watch](https://docs.angularjs.org/api/ng/type/$rootScope.Scope) first argument
(string or function) is evalued on each digest and second argument (callback
listener) is called when a returning value of first argument is changed (for
array, use length).

Expressions in angular directives `<button ng-click="foo = 5">` are parsed and
evaluated. `$parse()` takes expression and returns function which takes two
arguments: context and locals (overrides).

~~~
$scope.foo = 3;

var parseFn = $parse(‘foo = 5’);
parseFn($scope);
// or
$scope.$eval(‘foo = 5’);

$scope.foo; // returns 5
~~~

You can check if some property `bar` on `foo` is not null:
`$parse('bar.baz.quux')(foo)` will return undefined instead of throwing an
exception.

You can manually trigger digest with `$scope.$apply("foo = 5")` it calls
`$rootScope.$digest()` which propagates down to every child scope
[link](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$apply).
With ControllerAs syntax, you need to inject `$scope` and can not use
expression, but just empty `$scope.$apply()`. You can not call `$apply` in
callbacks for `ng-mouseover` `ng-click` since digest is in progress. It is
usable in non angular callbacks `document.getElementById('b').addEventListener
'mouseover'`.

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

* [egghead learn when to use a service, factory, or provider](https://egghead.io/playlists/learn-when-to-use-a-service-factory-or-provider-e2507e4b)
* Value recipe, can't have other dependencies

  ~~~
  var myApp = angular.module('myApp', []);
  myApp.value('clientId', 'a12345654321x');
  ~~~

* Factory recipe use other services, is a function and lazy initialized. It
  returns object

  ~~~
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

* service can be added with `.factory` which returns `new SomeFun(arg)`, but
  it's shorter with `.service` function. Service methods are defined as
  properties of `this`.  Service just use $injector to create new instance of
  service's contructor function.

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


* Provider is broader factory, it returns object with a `$get` method that is a
  factory function.  Provider can be configured. For provider name
  `unicornLauncher` angular registers `unicornLauncher` and
  `unicornLauncherProvider` injectables.

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

* [$resource](https://docs.angularjs.org/api/ngResource/service/$resource) by default `save` is POST, so we need to change for rails PUT, and add `create`. Factories need to return object. That object is bindable in all places where we use `Phone` factory.
  It can be used like `$scope.phone = Phone.get({ phoneId: $routeParams.phoneId }); $scope.phones = Phone.query();`

* Constant recipe can create value that is available in both config and run
  phase. Best example is to hard code values.

  ~~~
  angular.module('starter')
    .constant( 'CONFIG', {
      SERVER_URL: 'http://192.168.3.2:3001',
      S3UPLOAD: {
        BUCKET: 'duleorlovic-test1',
        API_URL: '/api/v1/s3_access_token',
      },
      USER_ROLES: {
        ADMIN: 'admin',
        EDITOR: 'editor',
        GUEST: 'guest',
      },
    });
    .run(['CONFIG','$rootScope',function(CONFIG, $rootScope) { 
      // in html use $root.CONFIG.USER_ROLES.ADMIN
      $rootScope.CONFIG = CONFIG
    }])
  ~~~

# RootScope

* `$rootScope` has not parent and was created directly from `Scope()` class (not through `$.new` method). It is used for event handling `$.broadcast()` (down) and `$.emit()` (up in the scope hierarchy).


# Share error messages

For form name *myForm* we can check `myForm.$valid` for all fields that we put
validation with `required` property on input elements (form can have
`novalidate` attribute to prevent browser default behavior, or we can use
`ng-required="true"`). Also fields have internal properties: `$dirty` (user has
interacted), `$valid`, `$invalid` and`$pristine` (has not interacted with field
yet).  To show client side errors, you can use
[ng-messages](https://docs.angularjs.org/api/ngMessages) instead of dealing with
`ng-show`.  Some example validations are: `ng-minlength=5` `ng-maxlength=20`
`ng-required="true"`.  We can share messages with `ng-messages-include`, but
that somehow does not work well.  There are also errors from server
(`resp.errors` or `resp.data.errors`).  Button is disabled until form is valid.

~~~
<form name="editMenuItemForm" ng-submit="update(vm.email,editMenuItemForm)">
  <input type="email" name="email" ng-model="vm.menu_item.email"
  ng-required="true" ng-pattern="/^.+@.+\..+$/"
  ng-change="editMenuItemForm.email.$setValidity('server', true)">
  <div ng-messages="editMenuItemForm.email.$errors">
    <div ng-message="required">This field is required</div>
    <div ng-message="email">This field must be an email</div>
    <div ng-message="minlength">Your field is too short</div>
    <div ng-message="pattern">Must look like an email</div>
  </div>
  <div ng-messages="vm.serverErrors.editMenuItemForm.email">
    <div>{{ vm.serverErrors.editMenuItemForm.email.toString() }}</div>
  </div>
  <md-button type="submit" class="md-raised md-primary"
  ng-disabled="!editMenuItemForm.$valid">Update</md-button>
</form>
~~~

On server we should respond with `render json: @menu_item.errors, status: :bad_request`
 so we catch model errors, for example `resp.data.email`. If that field is not
 on form, for example unauthorized response `resp.data.errors = ['Authorized
 only` than we show toast. `[].toString()` (raise if [] is nil) or `String([])`
Fields that are not valid (for example we put space in name `menu_item.name ==
undefined`) are not send so we need to disable button when form is not valid or
check if `editMenuItemForm.$valid` before we call `menu_item.update()`.
If we disable button than we need to put 
`ng-change="form.field.$setValidity('server', true)"` to remove server error on
change, so  submit button is shown again.


~~~
# in controller
vm.update = (menu_item, editMenuItemForm) ->
  menu_item.update(menu_item).then(
    (menu_item_from_server) ->
      $mdDialog.hide()
    (resp) ->
      $log.debug adminEditMenuItemController: 'update error', resp_data: resp.data
      for field of resp.data
        if editMenuItemForm[field]
          editMenuItemForm[field].$setValidity('server', false)
        else
          toastr.error resp.data[field].toString()
      vm.serverErrors =
        editMenuItemForm: resp.data
  )
~~~

# Tips:

* use `controller as vm` in view and `var vm = this` in controller so you don't need to inject `$scope` (we need `$scope` when we want to access something in promise `catch = -> $scope.vm.profileForm = "a"`)

* in directives you need to prefix `$root` to access rootScope, for example `$root.user`

* by [Angular conventions](https://github.com/mgechev/angularjs-style-guide), lowerCamelCase is used for factory names that won't be new'ed. Controllers and services should be UpperCased.

* [johnpapa style guide](https://github.com/johnpapa/angular-styleguide#controlleras-controller-syntax)
  * use `var vm = this;` (or more descriptive `loginVm`). Use `$scope` only for publish/subscribe events `$emit`, `$broadcast` and `$on`.
  * use `factory` that returns  instead of `service`
* for enabling html5 links, you need to put `<base href="/">` before `app.css`
  [link](http://stackoverflow.com/questions/28807251/relative-href-path-goes-wrong-in-angularjs-with-html5mode)
* sometimes you need to `rm -rf .tmp` since gulp server cache miss
* [angular.copy](https://docs.angularjs.org/api/ng/function/angular.copy) is
  usefull for forms and cancel button

~~~
vm = this
vm.old_menu_item = angular.copy menu_item
vm.cancel = ->
  angular.copy(vm.old_menu_item, vm.menu_item)
  $mdDialog.hide()
vm.update = (menu_item) ->
  menu_item.update(menu_item).then ->
    $mdDialog.hide()
~~~
* `$log.debug` should show object with location and data, for example
 `$log.debug adminController: 'update error', resp_data: resp.data`
* inside ui router resolve, js errors are completelly hidden


Check some [awesome links](https://github.com/gianarb/awesome-angularjs)

# Testing

# Confirm


Usage `<md-button ng-really-click="vm.delete(item)"
ng-really-reject="vm.cancel(item)" ng-really-message="Are you
sure?">Remove</md-button>`

~~~
# app/components/ng-really-click.directive.coffee
angular.module 'myApp'
  .directive 'ngReallyClick', ->
    restrict: 'A'
    link: (scope, element, attrs) ->
      element.bind 'click', ->
        message = attrs.ngReallyMessage
        if message && confirm(message)
          scope.$apply attrs.ngReallyClick
        else if attrs.ngReallyReject
          scope.$apply attrs.ngReallyReject
~~~

# UI View, Angular route, ui-router

`angular-route` is separated package and need to be added as module dependency
and configured. `config` needs Providers (factory for service that will be
configured) for example `$routeProvider` and `run` or `controller` needs service
for example `$route`

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

It is used with directive `<ng-view></ng-view>` which is replaced with given
template.


[ui-router](https://github.com/angular-ui/ui-router) is nice but documentation 
is not so simple, so here are examples:

* [stateParams](https://github.com/angular-ui/ui-router/wiki/URL-Routing#stateparams-service)
  can be defined with `:contactId` or `{contactId}`
* query params are defined like `?contactId`
* navigating with
  [ui-sref](https://github.com/angular-ui/ui-router/wiki/Quick-Reference#ui-sref)
  `ui-sref="admin.menu_items({staurantId: restaurant.id})"`
* if you want to completely change the template with edit template, than wrap it

~~~
# show.html
<ui-view>
  My name is {{ name }}
</ui-view>

# edit.html
<form>
  <input name="name">
</form>

# my.router.coffee
angular.module 'my'
.config ($stateProvider) ->
  $stateProvider
    .state 'myAccount',
      url: '/my-account'
      templateUrl: 'show.html'
      controller: 'MyController'
      controllerAs: 'vm'
      resolve:
        myAccountInitialData: (myAccountInitialData) ->
          myAccountInitialData()

    .state 'myAccount.edit',
      url: '/edit'
      templateUrl: 'edit.html'
      # no need to resolve here since this is inside MyController
~~~

# Batarang

[egghead batarang](https://egghead.io/lessons/angularjs-angularjs-batarang).
Just select element and in console type `$scope.vm.name = 'dule';$scope.apply()`

