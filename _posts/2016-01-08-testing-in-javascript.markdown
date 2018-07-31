---
layout: post
title: Testing in javascript
---

# Mocha

~~~
npm install --save-dev mocha
./node_modules/mocha/bin/mocha

# or if you put in package.json
# "scripts": {
#   "test": "mocha"
# }

npm test
~~~

~~~
// test/test.js
var assert = require('assert');
describe('alertTextShowHide', function() {
  describe('#elementText', function() {
    it('should show message', function() {
      assert.equal(1, 1);
    });
  });
});
~~~

# Sinon

~~~
npm install --save-dev sinon
# var sinon = require('sinon');
~~~

## Spy

Spies can be used to test how many times, and how functions or methods are
called

~~~
it('should call_method'), () => {
  const spy = sinon.spy(object, 'method')
  object.method(3)
  assert(spy.withArgs(3).calledOnce)
}
~~~


# TODO

gem 'teaspoon-jasmine'
gem 'phantomjs'

Teaspoon and jasmine

Do not use `it` outside of inner describe block

Todo

https://developers.google.com/web/updates/2017/06/headless-karma-mocha-chai

https://medium.com/welldone-software/an-overview-of-javascript-testing-in-2018-f68950900bc3
