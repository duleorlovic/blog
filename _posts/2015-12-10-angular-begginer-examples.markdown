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
  use also `ng-if="$even"`, or `$index` variables). Iterating over object is
  `ng-repeat="(key, value) in data"`
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
Note that angular expressions don't have access to `window` only `scope` so no
help from `parseInt()`, `String()` and similar.


`$scope.$watch('email', function() { $scope.test(); })` can be used to rerun
validation check.
[watch](https://docs.angularjs.org/api/ng/type/$rootScope.Scope) first argument
(string or function) is evalued on each digest and second argument (callback
listener) is called when a returning value of first argument is changed (for
array, use length).
To use with `controllerAs` syntax you need to use bind

~~~
app.controller('Ctrl', function ($scope) {
  this.name = 'name';

  $scope.$watch(angular.bind(function () {
    return this.title
  }), function (oldVal, newVal) {

  });
});
~~~

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

* strings `{{ "{{ 'Some string' | date:'MM/dd/yyy @ h:mma'"}} }}` can also be
  `currency, lowecase, uppercase`
* iterators `ng-repeat="product in store.products | filter: search | orderBy:
  '-price' | limitTo: -5 "` use only products that match value of search (any
  property of product that match search string) order by desc price and show
  last 5. If we want to filter only product.name than bind input to
  `search.name` so search becomes object, for example `ng-model="search.name"`.

View can use directives, filters. Controller and services can depend on some
providers.

* all objects are joined using [dependency
  injection](https://docs.angularjs.org/guide/di)
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

* two types of directives: element directive (UI widget)
  `<product-title></product-title>` or attribute directive (mixins like tooltip)
  `<div product-title></div>`.

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
  * `controllerAs: 'vm'` have some problems (but works as `vm1`)

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

Factory has more flexibility since they can return functions which can be
`new`ed.


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

* [$resource](https://docs.angularjs.org/api/ngResource/service/$resource) by
  default `save` is POST, so we need to change for rails PUT, and add `create`.
  Factories need to return object. That object is bindable in all places where
  we use `Phone` factory.
  It can be used like `$scope.phone = Phone.get({ phoneId: $routeParams.phoneId
  }); $scope.phones = Phone.query();`

* Constant recipe can create value that is available in both config and run
  phase. Don't use for example DEFAULT image_url since is better to do that in
  backend on one place

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

* `$rootScope` has not parent and was created directly from `Scope()` class (not
  through `$.new` method). It is used for event handling `$.broadcast()` (down
  to child scopes) and `$.emit()` (up in the scope hierarchy).
* you can also use for something that does not change often, like
  `$rootScope.user = user`. In templates you can just write `{{ user.email }}`
  and it will find `user` at root scope (can't use `{{ vm.user }}`).


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
<form name="editMenuItemForm" ng-submit="vm.update(vm.email,editMenuItemForm)">
  <input type="email" name="email" ng-model="vm.menuItem.email"
  ng-required="true" ng-pattern="/^.+@.+\..+$/"
  ng-change="editMenuItemForm.email.$setValidity('server', true)">
  <div ng-messages="editMenuItemForm.email.$error">
    <div ng-message="required">This field is required</div>
    <div ng-message="email">This field must be an email</div>
    <div ng-message="minlength">Your field is too short</div>
    <div ng-message="pattern">Must look like an email</div>
    <div ng-message="server">{{ vm.serverErrors.editMenuItemForm.email.toString() }}</div>
  </div>

  <md-button type="submit" class="md-raised md-primary"
  ng-disabled="!editMenuItemForm.$valid">Update</md-button>
</form>
~~~

This works also for nested attributes, just need to reference them with
`editMenuItemForm["menu_item_options.price"]` (snake case is rails style) and
create one in controller `vm.menuItem.menu_item_options_attributes = [{}] unless
vm.menuItem.id` and reference with
`input(ng-model='vm.menuItem.menu_item_options_attributes[0].price'`

This has problem with destroy button and serverside validation since ng-message
is visible only when you focus that input (probably digest is perfomed only on
submit)

On server we should respond with `render json: @menu_item.errors, status:
:bad_request` so we catch model errors, for example `resp.data.email`. If that
field is not on form, for example unauthorized response `resp.data.errors =
['Authorized only']` than we show toast. If you need to show toast but don't
know if `resp.data` is object or array than you can use `toastr.error
JSON.stringify resp.data`. It should be object and you need to know the key.

For arrays you can use two methods: `s.toString()` and `String(s)`.
`[].toString()` works fine but will raise if instead of `[]` is nil. So I prefer
to use `String(...)` in controllers `String(null)=="null"`. Please not that
`String` is `window` function so it is not available in angular expressions ie
for `ng-message` but there we are sure that value is not nill.

Fields that are not valid (for example we put space in name
`menuItem.name == undefined`) are not sent so we need to disable button when
form is not valid or check if `editMenuItemForm.$valid` before we call
`menuItem.update()`.  If we disable button than we need to put
`ng-change="form.field.$setValidity('server', true)"` to remove server error on
change, so  submit button is shown again.


~~~
# in controller
vm.update = (menuItem, editMenuItemForm) ->
  menuItem.update(menuItem).then(
    (menu_item_from_server) ->
      $mdDialog.hide()
    (resp) ->
      $log.debug adminEditMenuItemController: 'update error', resp_data: resp.data
      for field of resp.data
        if editMenuItemForm[field]
          editMenuItemForm[field].$setValidity('server', false)
        else
          toastr.error String resp.data[field]
      vm.serverErrors =
        editMenuItemForm: resp.data
  )
~~~

# $q service

In [$q service](https://docs.angularjs.org/api/ng/service/$q) for promise object
we can call `then(successCallback, errorCallback, notifyCallback)` but also
`finally(callback, notifyCallback)` for clean up.

Here is example of Customer service

~~~
# www/js/services/customerAuth.service.coffee
angular.module 'starter'
  .service 'CustomerAuth', ($http, CONSTANT, $q) ->
    service = this
    service.sessionId = null

    service.login = (customer_login) ->
      $q(
        (resolve, reject) ->
          $http.post(
            CONSTANT.SERVER_URL + '/customer/sessions.json'
            customer_session: customer_login
          ).then(
            (response) ->
              resolve response
            (response) ->
              reject response
          )
      )
~~~


# Tips:

* use `controller as vm` in view and `var vm = this` in controller so you don't
  need to inject `$scope` (we need `$scope` when we want to access something in
  promise `catch = -> $scope.vm.profileForm = "a"`)
* in directives you need to prefix `$root` to access rootScope, for example
  `$root.user`

* by [Angular conventions](https://github.com/mgechev/angularjs-style-guide),
  lowerCamelCase is used for factory names that won't be new'ed. Controllers and
  services should be UpperCased.

  Filenames are lowerCamerCase, but folders are underscored or dashed
  `src/app/menu_items/menuItems.jade`. Template always match controller name

* [johnpapa style guide](https://github.com/johnpapa/angular-styleguide#controlleras-controller-syntax)
  * use `var vm = this;` (or more descriptive `loginVm`). Use `$scope` only for publish/subscribe events `$emit`, `$broadcast` and `$on`.
  * use `factory` that returns  instead of `service`
* for enabling html5 links, you need to put `<base href="/">` before `app.css`
  [link](http://stackoverflow.com/questions/28807251/relative-href-path-goes-wrong-in-angularjs-with-html5mode)
* sometimes you need to `rm -rf .tmp` since gulp server cache miss
* [angular.copy](https://docs.angularjs.org/api/ng/function/angular.copy) is
  usefull for forms and cancel button. Note that object is disconnected, so if
  you store a list of all items, you need to angular.copy back to original.

  ~~~
  ~~~

  Also when you receive an array from server `MyService.locations =
  locationsFromServer` will assign new pointer, but old `vm.locations =
  MyService.locations` will still point to old array.

* [$log](https://docs.angularjs.org/api/ng/service/$log) can show object with
  source location, for example `$log.debug adminController: 'update error',
  resp_data: resp.data`. It is angular service which you can disable in
  production 

  ~~~
  angular.module 'my-module'
    .config($logProvider) ->
      $logProvider.debugEnabled = false
  ~~~

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

# Data disable with Processing...

Similar to rails `data-disable-with="Processing..."` here is directive.
Angular ignores `data-` from `data-disable-with`.
Similar to
[angular-autodisable](https://github.com/kirstein/angular-autodisable)
There was some problem with JQlite and find with `[type=submit]` so use [my
fork](https://github.com/duleorlovic/angular-autodisable)

# Errors

If you get circular dependency error like `Uncaught Error: [$injector:cdep]
Circular dependency found: $state <- unauthorizedInterceptor <- $http <-
$templateFactory <- $view <- $state`, than use `$injector` and `$state =
$injector.get '$state' like in the following example.

Redirect to login state on every `unauthorized` response

~~~
# www/js/interceptors/unauthorized.interceptor.coffee
angular.module 'starter'
  .factory 'unauthorizedInterceptor', ($q, $injector) ->
    responseError: (rejection) ->
      if rejection.status == 401
        $state = $injector.get('$state')
        $state.go 'login'
      $q.reject rejection

# www/js/app.config.coffee
angular.module 'starter'
  .config ($httpProvider) ->
    $httpProvider.interceptors.unshift 'unauthorizedInterceptor'
~~~

Interceptor that will set `data.error` in case there is network error

~~~
angular.module 'starter'
  .factory 'connectionInterceptor', ($q, NotifyService) ->
    responseError: (rejection) ->
      if rejection.status <= 0 # server offline
        unless rejection.data && rejection.data.error
          rejection.data =
            error: 'There is a problem with connection'
      $q.reject rejection
~~~

# UI View, Angular route

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

# UI Router

[ui-router](https://github.com/angular-ui/ui-router) is nice but documentation 
is not so simple, so here are examples:

* [stateParams](https://github.com/angular-ui/ui-router/wiki/URL-Routing#stateparams-service)
  can be defined with `:contactId` or `{contactId}`
* query params are defined like `?contactId`
* navigating with
  [ui-sref](https://github.com/angular-ui/ui-router/wiki/Quick-Reference#ui-sref)
  `ui-sref="admin.menu_items({staurantId: restaurant.id})"` or with `$state.go
  'tab.location_ticket_details', locationTicketId: locationTicket.id`
* [ui-sref-active](http://angular-ui.github.io/ui-router/site/#/api/ui.router.state.directive:ui-sref-active) to add class on navigational links that are active
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

*  When you need double click to go to nested state, you are probably coverting
  that state with some other state
  [stackoverflow](http://stackoverflow.com/questions/25548798/angular-ui-router-having-to-click-twice-for-view-to-update-as-expected-with-d)
  route resolution problem.

* resolve initialData before new state is activated
  [link](http://odetocode.com/blogs/scott/archive/2014/05/20/using-resolve-in-angularjs-routes.aspx)

  ~~~
  # src/app/my_account/myAccountInitialData.coffee
  # http://odetocode.com/blogs/scott/archive/2014/05/20/using-resolve-in-angularjs-routes.aspx
  # https://promisesaplus.com/#point-41 the promise resolution procedue, ie
  # onFulfilled returns promise, when it is fulfilled than fullfill parent
angular.module 'menucardsAngular'
  .factory "myAccountInitialData", (Restaurant, $auth, $q) ->
    ->
      user =  $auth.validateUser()
      $q.all([user]).then (results) ->
        restaurant = Restaurant.query({}, { id: results[0].restaurant_id })
        $q.all([restaurant]).then ( results ) ->
          restaurant: results[0]

  # src/app/my_account/myAccount.controller.coffee
  angular.module 'menucardsAngular'
    .controller 'MyAccountController', (myAccountInitialData, $log) ->
      'ngInject'
      $log.debug "MyAccountController"
      vm = this
      vm.restaurant = myAccountInitialData.restaurant
      return

  # src/app/my_account/myAccount.router.coffee
  angular.module 'menucardsAngular'
    .config ($stateProvider) ->
      'ngInject'
      $stateProvider
        .state 'myAccount',
          url: '/my-account'
          templateUrl: 'app/my_account/myAccount.html'
          controller: 'MyAccountController'
          controllerAs: 'vm'
          resolve:
            myAccountInitialData: (myAccountInitialData) ->
              myAccountInitialData()
  ~~~

* if you need to do some jQuery on state change (not only when controller loads,
  but when we change states inside current controller) you can listen to events
  on `$scope` (this scope is destroyed when controller is destroyed)

  ~~~
  $scope.$on '$stateChangeStart', (e, toState, toParams, fromState, fromParams) ->
    $scope.sectionName = toParams.sectionName
    if toState.name == 'menu'
      $timeout ->
        new scrollItem 'cart'
    return
  # this is when user lands on menu page
  $timeout ->
    if $('#cart').first()
      new scrollItem 'cart'
  ~~~

* [multiple
  views](https://github.com/angular-ui/ui-router/wiki/Multiple-Named-Views) you
  can not have two different active state, so it will toggle

# Batarang

[egghead batarang](https://egghead.io/lessons/angularjs-angularjs-batarang).
Just select element and in console type `$scope.vm.name = 'dule';$scope.apply()`

Usually I define module `angular.module 'name', []` only in one place
*src/app/index.module.coffee*

Since some old deleted templates could remove stuff, I run with `gulp clean &&
gulp serve --api localhost:3001 -l`

# Interval for automatic updates

~~~
Order.query({status: 'active'}, {restaurantId: user.restaurant_id}).then (orders) ->
  vm.orders = orders

vm.youngerThan = new Date
# could be that order happens after previous query on server and before current time, not likely
# more likely is that order happened before query on server but after current time
# so it will show up twice, that's why we check if alredy there
$interval(
  ->
    Order.query({status: 'active', youngerThan: vm.youngerThan}, {restaurantId: user.restaurant_id}).then (orders) ->
      for order in orders.reverse
        vm.orders.unshift(order) if vm.orders[0] && vm.orders[0].id != order.id
    vm.youngerThan = new Date
  15000)
~~~

# Pagination

You can use
[angular-paginate-anything](https://github.com/begriffs/angular-paginate-anything)
to show pagination links on your page. On server you can use [kaminari with
some before
filter](http://www.tejusparikh.com/2014/pagination-in-angular-js.html) or
[clean_pagination](https://github.com/begriffs/clean_pagination) which I prefer.
It is very easy to start. For angular-rails-resource I need to use [$timeout hack](https://github.com/begriffs/angular-paginate-anything/issues/95)

~~~
$scope.$on 'pagination:loadPage', (event, status, config) ->
  $timeout ->
    vm.users = vm.usersObject.map (userObject) -> new User userObject
~~~

# Tips

* you can have another ng-click inside ng-click (ie button inside button) and
  you can stop propagation with `ng-click="vm.edit();
  $event.stopPropagation();"`
* don't know why someone would use `ng-repeat="o in vm.objects tack by o.id`
  since DOM is updated always, even `o.title` is changed.
  [link](http://www.codelord.net/2014/04/15/improving-ng-repeat-performance-with-track-by/)
  only usage is that directive link function is not called until we add new
  items or updateId [look console log](http://jsfiddle.net/6k834fzx/4). If we
  update title Link function will not be called although DOM will be updated.
