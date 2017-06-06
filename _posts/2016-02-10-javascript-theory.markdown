---
layout: post
title: Javascript theory
tags: javascript
---

# Closures

[source](http://www.jibbering.com/faq/notes/closures/)

>  A closure is formed when one of those inner functions is made accessible
>  outside of the function in which it was contained, so that it may be executed
>  after the outer function has returned. At which point it still has access to
>  the local variables, parameters and inner function declarations of its outer
>  function.

Nice example on
[wiki](https://en.wikipedia.org/wiki/Closure_(computer_programming))
Example from
[stackoverflow](http://stackoverflow.com/questions/643542/doesnt-javascript-support-closures-with-local-variables/643664#643664)

~~~
var closures = [];
function create() {
  for (var i = 0; i < 5; i++) {
    closures[i] = function() {
      alert("i = " + i);
    };
  }
}

function run() {
  for (var i = 0; i < 5; i++) {
    closures[i]();
  }
}

create();
run();
~~~

This will print 5,5,5,5,5 since at the time function is created it reference to
variable `i`, but when function is called `for` loop already set `i` to 5. More
correct is to call inner function when you define it

~~~
function create() {
  for (var i = 0; i < 5; i++) {
    closures[i] = (function(tmp) {
        return function() {
          alert("i = " + tmp);
        };
    })(i);
  }
~~~

Thats why `do` exist in coffee script

> When using a JavaScript loop to generate functions, it’s common to insert a
> closure wrapper in order to ensure that loop variables are closed over, and
> all the generated functions don’t just share the final values. CoffeeScript
> provides the do keyword, which immediately invokes a passed function,
> forwarding any arguments.

~~~
closures = []
create ->
  for i in [0..5]
    # instead of closuers[i] = ->
    closures[i] = do (i) ->
      -> alert i
~~~

Here is example in ruby

~~~
def create
  r = []
  # 1.upto 5 do |i| # this will print 1,2,3,4,5 since i is not shared between
  for i in 1..5
    r.push Proc.new { i }
  end
  r
end

a = create
a.each {|f| puts f.call } # [5,5,5,5,5] since i is shared for all closures
~~~

## Resolution of property names on objects

~~~
// assignment of values
var objectRef = new Object();
objectRef.testNumber = 5;
objectRef["testNumber"] = 5;
// reading of values
alert(objectRef.testNumber);
~~~

[w3 prototypes](http://www.w3schools.com/js/js_object_prototypes.asp)

> All JavaScript objects inherit their properties and methods from their
> prototype. The Object.prototype is on the top of the prototype chain.

`prototype` property is on functions only. Prototype is an object.
Function by default have Object.prototype prototype object.
We can add properties to prototype object even after we create object.
Function is constructor function when we use it to create new objects and they
are UpperCased. Object, Array, Date are function constructors.
The default prototype for the Object constructor has a null prototype.

~~~
function People(name) {
  this.name = name;
}
function Man(car) {
  this.car = car;
}
Man.prototype = new People('dule');
var p = new Man('renault');
People.prototype.age = 30;
Man.prototype.say = function() { return this.name + " (" + this.age +
  ") drive " + this.car;
};
alert(p.say()); // dule (30) drive renault
~~~

## Identifier resolution, execution contexts and scope chains


### Intro

Scope of variable is from a moment of declaration to the end of function where
it is declared (global context acts like one big function encompassing the code
on the page)

Scope of a function is **entire** function where it is declared (ie function
hoising).
Block nesting like in `if` statement, does not affect scope. Scope of inline
function `var b=function a(){}` is only inside function (this is function
expression and hoising is not applied here).

Function context (*this* variable) is:

* *window* for invocation as function `a()`
* current object for invocation as method `objB={};objB.a=a;objB.a()`

Invocation as contructor `var n=new A();` creates new object and pass to
function as *this*.

Invocation with `a.apply(objB,[1,2])` and `a.call(objB,1,2)` set manually the
context `objB` of a function (apply and call are function methods).

You can see the source code of a method with `objB['a']`

When function is called it enters an execution context:

1. first is "Activation" object is created with `arguments` property
1. *scope chain* (list of objects) from last to first:
  * global object
  * Activation object for inner functions
  * `y` if `y` is object for execution of `x()` in `var x;with(y){
    x=function(){}; }`
1. variable instantiation to "Variable" object
  * function's formal parameters with name and value if value is provided
  * inner function definitions are creating **function objects** with inner
    function name as key
  * named properties for all local variables, just key, value is not yet defined
1. value is assigned to *this* keyword

## Identifier Resolution

1. it starts from first object in the scope chain (it is Activation/Variable)
1. it also check it's prototype chain
1. go to the next object from scope chain, until global object

Functions are objects and are created:

* during "Variable" instantiation from function declaration
* during evaluation of function expressions
* or invoking *Function* contructor

Upon exiting an execution context all scope chain, Activation/Variable object
and any object created (including function objects) are no longer accessible.

## Forming closure

A closure is formed by returning a function object that was created withing an
execution of a function call and assigning that reference to a property of
another object. Or assigning (not returning) reference of such function object
to global variable, or a property of global object or object passed by reference
as an argument to the outer function call.

Tip: in chrome console when you stop with `debugger` and try to reference
variable than has not been referenced than is could say `undefined reference`
although you can use it

~~~
function() 
~~~

Examples

* closure for function without arguments can be passed to

  ~~~
  function callLater(arg1) {
    return (function() {
      return "I'm " + arg1;
    });
  }
  var fRef=callLater('arg1');
  setTimeout(fRef,3000);
  ~~~

* associating functions with a lot of object instances

  ~~~
  function associateObjWithEvent(obj, methodName) {
    return (function(e) {
      e = e || window.event;
      return obj[methodName](e, this); // this will be element object
    });
  }
  function DhtmlObject(elementId) {
    var el = getElementWithId(elementId);
    if (el) {
      el.onclick = associateObjWithEvent(this, "doOnClick");
      // el.onclick = this.doOnClick; //
      el.onmouseover = associateObjWithEvent(this, "doMouseOver");
    }
  }
  DhtmlObject.prototype.doOnClick = function(event, element) {
    // doOnClick body
  }
  DhtmlObject.prototype.doMouseOver = function(event, element) {
    // doMouseOver body
  }
  ~~~

* Encapsulating related functionality. Crete execution context by executing a
  function expression in-line and return function (or object)

  ~~~
  var getImgInPositionedDivHtml = (function(){
    var innerArray = [
      '<div id="',
      '', // index 1 DIV ID
      '"></div>',
    ];
    return (function(id){
      innerArray[1] = id;
      return innerArray.join;
    }
  })(); /* the inline execution of the outer function expression */
  ~~~

* Accidental closures. Do not use inner functions as event handlers that will be
  used in a lot of places since every time new function object is created

  ~~~
  var quantaty = 5;
  function addGlobalQueryOnClick(linkRef) {
    if (linkRef) {
      linkRef.onlick = function() {
        this.href += ('?quantaty='+escape(quantaty));
        return true;
      }
    }
  }
  // better is to define on property
  var quantaty = 5;
  function addGlobalQueryOnClick(linkRef) {
    if (linkRef) {
      linkRef.onlick = forAddQuaryOnClick;
    }
  }
  addGlobalQueryOnClick.prototype.forAddQuaryOnClick = function() {
    this.href += ('?quantaty='+escape(quantaty));
    return true;
  };
  
  // similar is for object constructor functions for a lot of objects
  // instead
  function ExampleConstuctor(p) {
    this.publicProp = p;
    this.method1 = function() {
      // method body
    }
  }
  // we shuold write
  function ExampleConstuctor(p) {
    this.publicProp = p;
  }
  ExampleConstuctor.prototype.method1 = function() {
    // method body
  };
  ~~~

# Difference ES6 (ie ES2015) and ES5

This [post](http://kamranahmed.info/blog/2016/04/04/es6-in-depth/) explains what
is new in ES6 (Ecma Script 2015). Almost all those features already exists in
coffee script [nice reply](https://gist.github.com/benjie/d0e39fbe8a61bc30ed93).
It's good that no need to write `function` keyword. Also nice table of
[es6-features](http://es6-features.org/) and [supported
browsers](http://kangax.github.io/compat-table/es6/)

* `use strict` mode by default (by this
  [info](http://arcturo.github.io/library/coffeescript/07_the_bad_parts.html)
  you should use in development but not on production
* `let myVar;` has block scope like inside `if () { let myVar=1; }` (`var` has
function scope). You can notice difference in

  ~~~
  for (var i = 1; i< 5; i++) {
    console.log(i)
    setTimeout(function() {
      console.log(i) # here we have i = 5,5,5,5 since same variable is used
      # if we used let i=1;i<5;i++ than it will be different variable: 1,2,3,4
    }, 1000)
  }
  ~~~

* also `const myConst = document.querySelect(".my-class");` have block scope,
and it should be used for all non volatile variables
* template literals with backticks and curly braces (coffe version on "Hello
#{name}")

  ~~~
  var name = "Duke";
  console.log(`Hello ${name}`);

  const multiLine = `this
  is
  multi
  line`
  ~~~

* array functions are abbreviated syntax for anonymouse functions (coffee fat
  arrow instead of slim arrow `->`, binds to the outer `this`). If there is only
  one parameter than you can omit parenthesis, if there are no params than you
  can replace parenthesis with `_`. One liner syntax (without block) use
  implicit return, for other you need explicit return if you need.

  ~~~
  var sayHello = (name) => `Hello ${name}!`;
  var sayCiao = (name) => {
    return `Ciao ${name}!`;
  }
  ~~~

  Note that `self` inside arrow functions does not bind to current object for
  object methods. It binds to outer `this`. So it is usefull only if you have
  function inside method that is defined in clasical way, like

  ~~~
  let o = {
    myMethod: function() {
      setTimeout( () => {
        console.log(this) //o
      }, 3000)
    }
  }
  ~~~

* destructuring with default values and renaming (also in coffescript
  `[_, month, date = 1] = ...`)

  ~~~
  # destructuring arrays
  let [, month, date = 1] = '2010-10-11'.split('-')
  # month = '10' and date = '11'
  let [f, s, ...rest] = [1, 2, 3, 4]
  # f=1, s=2, rest = [3,4[

  # destructuring objects
  let { name, age, gender:sex = 'male' } = { name: 'Duke', age: '33', gender: 'male' }
  # is the same as
  let name = o.name
  let age = o.age
  let sex = o.gender # here we also renaming
  ~~~

  You can also use descructuring when defining methods

  ~~~
  function call({
    name: 'My Name',
    phone: '123123',
  } = {}) {
    console.log(name + ' ' + phone)
  }
  ~~~

  There exists rest `...` parameter which convers to array

  ~~~
  const sum = (...a) => a.raduce((sum, current) => sum + current, 0)
  ~~~

  and same operator `...` inside method call is called *spread* and it expands
  array to params (coffee calls this splats `f(items...)`)

  ~~~
  let a = [1, 2, 3]
  f(...a)
  # is the same as
  f(1, 2, 3)
  ~~~

* default parameter values `function g(a=2){}` (the same for coffeescript)

* `for(i=0;i<cars.loeght;i++) {}` is not concise `cars.forEach(myFunction)` is
  concise but can not break out of the loop. `for(let ... of ...)` is concise
  and can break `for(let car of cars) { }` (coffee script has `.each`)
* extend classes `class Employee extends Person {}` so we don't need to write
  `Employee.prototype = new Person`
* new data structure called `Map` and `WeakMap` (keys are objects, not plain
  values like number, string or symbol)

  ~~~
  var map = new Map();
  map.set('year', '123');
  map.get('year'); // '123'
  ~~~

* new data structure called `Set` and `WeakSet`
* new string functions
  `'asd'.startsWith('a');'asd'.endsWith('a');'asd'.includes('as');'a'.repeat(3);`
* new array functions `Array.from([1,2,3], x => x + x)`
 and `Array.find( user => user.age > 15)` same as `Array.findIndex` but returns
 object instead of index
* modules like `import User from 'user';`
[link](https://www.sitepoint.com/understanding-es6-modules/)

  ~~~
  // lim/math.js
  export function sum(x, y) {
    return x + y;
  }
  export let pi = 3.141593;
  ~~~

  ~~~
  import * as math from "lib/math";
  console.log("2pi = " + math.sum(math.pi, math.pi));

  // or we could
  import { pi, sum } from "lib/math";
  console.log("2pi = " + sum(pi, pi));
  ~~~
* property value shorthands, when you define object and you already have
variable with same same as key, you can use

  ~~~
  const name = 'dule'
  const anPerson = {
    name
  }
  // is the same as
  const aPerson = {
    name: name
  }
  ~~~

  it also exists for methods

  ~~~
  const aPerson = {
    speak (word) {},
    // same as
    speak: function (word) {},
    // do not use arroy because you can not acsess this
    // speak: () => {},
  }
  ~~~

* computed object property names

  ~~~
  const name = 'dule'

  const aPerson = {
    'mile': 'value',
    [name]: 'dule is key and value',
    [name+'car']: 'value',
  }
  ~~~
