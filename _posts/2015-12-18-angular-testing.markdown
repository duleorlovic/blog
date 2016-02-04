---
layout: post
title: Angular Testing
tags: angular testing jasmine protractor
---

[video](https://www.youtube.com/watch?v=iP0Vl-vU3XM) TDD helps to write proper tests (so tests don't have bugs). Tests helps to maintain and refactor code. Also is easier to understand code if test titles are properly constructed. It's very helpfull is to do test code review in person (pair programming). RTMF (readable, trustworthy, maintainable, fast).

One big mistake is to use mocking too much. Difference between mock object and stub is that mock is verified,
but stub is not. Unit of work can have 3 outcomes: return value/exception, noticeable state change or 
3th party call (only here should use mock, that code should be less than 5% of all test code). So only when
end result should be a call to 3 service, we create mock object. Stub is just data flow get/return.

Don't specify internal implementation, it's better to create method that will return result (different result when state change). Example `userMgr.createUser('asd','pas')` should not test `equal(userMgr._internalUsers[0].name,'asd')` but it's better to test `ok(login('asd','pas'))` because other methods can use different internal implement.

* don't mix unit and integration tests (integration are more brittle).
* keep variable declaration and it's usage close to each other
* don't comment out test, remove them
* small tests. if method has multiple end results, than create test for each end result and choose good name
* unit testing controllers: usually initial values (no business logic) and copuling UI actions to services
* unit testing services: correct business logic
* unit test directive: inject `$compile`, manually call `$digest()` for that scope.
* e2e protractor [ElementArrayFinder](http://angular.github.io/protractor/#/api?view=ElementArrayFinder) uses [locators](http://www.protractortest.org/#/locators) argument

# Protractor

* use [page objects](https://docs.google.com/presentation/d/1B6manhG0zEXkC-H-tPo2vwU06JhL8w9-XCF9oehXzAQ/edit#slide=id.gf9c971a8_00) to define elements, by css, repeater, [text](https://github.com/angular/protractor/issues/456)
  * `this.imgEl = element(by.css('img'));`
  * `this.rowItems = element.all(by.repeater('row in vm.rows'));`
  * `this.elWithText = element(by.xpath("//*[contains(text(),'Log in')]"));`
* action
  * `browser.get('/index.html');`
  * `page.emailInput.sendKeys('asd@asd.asd' + protractor.Key.TAB + 'asdasd');`need to be inside same sandKeys
  * `page.loginButton.click();`
* expectations
  * `expect(page.imgEl.getAttribute('src')).toMatch(/image.png$/);`
  * `expect(page.rowItems.count()).toBeGreatedThan(5);`
  * `expect(element(by.binding('person.name')).isPresent()).toBe(true);` that is `{{ person.name }}`
  * `expect(page.someEl.isDisplayed()).toBe(true, "Should be displayed");` is different from `isPresent()`

[debug](https://github.com/angular/protractor/blob/master/docs/debugging.md)

Debugging using `browser.pause()` to get WebDriver control flow,
than `c` to move to next webdriver command,
`d` to next debugger statement,
`repl` to get interactive mode where you can `element(by.css('red')).isEnabled()`.
Chrome Dev Tools does not work in this regime

The same debug/repl is with `browser.debugger()` and call with `protractor debug`

Run single test with `protractor protractor.conf.js --specs
e2e/login/oauth.spec.js`

https://docs.angularjs.org/tutorial/step_05#test
During test we use angular `inject` and `module`.
When we inject `_flash_` , Angular knows that service `flash` should be injected (_ are ignored_)

~~~
# src/app/compoments/mycomponent/myService_spec.coffee
describe("myService", function() {
  var httpBackend, scope, ctrl;
  beforeEach(module('myService'));
  beforeEach(inject(function($httpBackend, $scope, $controller){
    httpBackend = $httpBackend;
    scope = $scope;
    ctrl = $controller('myService', {$scope: scope});
  }));

  it("should load phones.json", function(){
    httpBackend.expectGET('phones.json').respond([{name: 'A1'}]);
    expect(scope.phones).toBeUndefined();`
    httpBackend.flush();
    expect(scope.phones).toEqual([{name: 'A1'}]);
  });
});
~~~

* `expectGet` is expecting request so it fails when there are no requests

Instead of `$httpBackend` we could create a mock `apiService` [video](https://www.youtube.com/watch?v=UDB-jm8MWro)

~~~
# src/app/compoments/mycomponent/myController_spec.coffee
describe('Controller: MainCtrl', function () {
  beforeEach(module('diyBrewControllerApp'));
  var MainCtrl;
  var scope;
  var apiService;
  var status;
  var q;

  beforeEach(inject(function ($controller, $rootScope, $q) {
    q = $q;
    scope = $rootScope.$new();
    MainCtrl = $controller('MainCtrl', {
        $scope: scope,
        ControllerApi: apiService
    });
  }));
  beforeEach(function () {
    status = { "temperatureCelsius": "22" }
    apiService = {
      getSettings: function () {
        deferred = q.defer();
        return deferred.promise;
      },
      updateSettings: function () {
        return;
      }
    };
  });
  it('should request current status during init', function () {
    spyOn(apiService, 'getStatus').andCallThrough();

    scope.init();

    deferred.resolve(status);
    scope.$root.$digest();

    expect(apiService.getStatus).toHaveBeenCalled();
    expect(scope.currentStatus.temperatureCelsius).toBe("22");
  })
});
~~~

# Jasmine

[Jasmine](http://jasmine.github.io/1.3/introduction.html) have similar syntax to rspec.

* `describe` `it` (prefix with x to disable suite or specifix spec)
* `spyOn(foo, 'setBar')` is a spy (test double is defined with `foo={setBar: function(v) {}};` that can track calls and arguments with this two matchers `expect(foo.setBar).toHaveBeenCalled();` and `expect(foo.setBar).toHaveBeenCalledWith(123)`
* Chain spy with `spyOn(foo, 'getBar').andCallThrough();` to actually delegate to implementation. Chain spy with `spyOn(foo, 'getBar').andReturn(745);` so all calls to `foo.getBar` will return 745 (similar to `.andCallFake(function(){});`)
* To create a mock with multiple spies with one command, use `tape = jasmine.createSpyObj('tape', ['play','pause']);`.




TODO
http://artofunittesting.com/

