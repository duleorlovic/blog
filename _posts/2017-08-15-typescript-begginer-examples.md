---
layout: post
title: Typescript begginer examples
---

<https://www.typescriptlang.org/docs/handbook/basic-types.html>
<https://scrimba.com/g/gintrototypescript>

You can install and start watching typescript compilation using

~~~
npm install -g typescript
tsc -w
~~~

For vim use https://github.com/leafgarland/typescript-vim
Types are checked when you assign or use method on it.

# Basic types

* `boolean`, `number`, `string` for example `let name: string = 'Duke';`. You
  can use union/or for example `let name: string | number = 12`
* Array of numbers `let list: number[] = [1, 2, 3];`, another way to write
  instead of `number[]` you can use `let list: Array<number> = [1, 2, 3];`
  * type inside brackets can be used when array has different types `let x:
  [string, number]` so first element is string and second is number (there can
  not be 3th element)
  * to have union of string or number use parenthesis `let e: (number
  | string){} = [1, '2', 3]`
* `enum Color {Red, Green, Blue}` usage is `let c: Color = Color.Red`. By
default value is 0, 1, ... but you can assign `enum Color { Red = 3, Green = 5,
Blue = 2 }`.  You can also use lookup to get from number to color ie color name
`let colorName: string = Color[2]; // "Blue"`
* `any` is type that can be used for existing code, it can contain any value
and can call any method on it
* `void` is only usefull to mark return type of a function (that returns
nothing)
* `null` and `undefined` are types for which you can assignly only `null` and
`undefined`. `null` and `undefined` are subtypes of all other types so you can
assign `null` to some `:number`. But if `--strictNullChecks` is enabled
(default) `null` can be assignable only to `:void` or `:null`. Use union if
you don't know if it will exists `number | null`.
* `never` type represents value that never occur. For example if function
always throw exception, it has return type `never`. it is subtype of all
types. And no type is subtype of `never` (except `never`)

Type assertions `(<_>)` are similar to type cast, but has no runtime impact,
it's only used for compiler `let length: number = (<string>name).length;` or
`as` syntax `let length: number = (name as string).length;`

To check a type you can use
```
if (bear instanceof Bear) {
```

Return types of function, and optional `?` arguments

```
function add(a?: number[]): number {
  return a[0] + 1
}
```

# Custom types

Creating custom types using `type` is deprecated (use Interface and Class)
```
type person = {firstName: string}
let e: person = {firstName: 'D' }
```

Intersection types is when it inherited all properties (combine types)

```
let manBearPig: Bear & Man & Pig
# or
type ManBearPig = Bear & Man & Pig
let manBearPig: ManBearPig
```

Interface describe objects. All properties needs to be initialised


stao ovde https://www.typescriptlang.org/docs/handbook/interfaces.html
~~~
interface Person {
  firstName: string;
  lastName: string;
}
~~~

Classes are used to create objects (it provides `constructor`)

~~~
class Student {
  fullName: string;

  constructor(public firstName, public middleInitial, public lastName) {
    this.fullName = firstName + " " + middleInitial + " " + lastName;
  }
}
~~~

We can declare and initialize properties, or use variable assigment

~~~
class A {
  title: string;
  constructor(public title: string) {
    this.title = title;
  }
  // or simply variable assigment with type anotation
  title: string = title;
}
~~~

`implements`


Generic functions is using `<T>` instead of using `any`
```
function e<T>(arg: T): T {
  return arg
}
```

Access modifier keywords:
* public
* readonly (like constant, cannot change)
* protected (accessed with extended classes)
* private (is not accessible outside of class methods)

https://www.youtube.com/watch?v=-PR_XqW9JJU
