---
layout: post
title: Typescript begginer examples
---

<https://www.typescriptlang.org/docs/handbook/basic-types.html>

You can install and start watching typescript compilation using

~~~
npm install -g typescript
tsc -w
~~~

Types are checked when you assign or use method on it.

# Basic types

* `boolean`, `number`, `string` for example `let name: string = 'Duke';`
* `_[]` or `Array<_>`: `let list: number[] = [1, 2, 3];`, generic array type:
`let list: Array<number> = [1, 2, 3];`
  * type is used when array has different types `let x: [string, number]` so
  first element is string and second is number, and all others are union of
  those (string | number).
* `enum Color {Red, Green. Blue}` usage is `let c: Color = Color.Red`. By
default value is 0, 1, ... You can also use lookup to get from number to color
ie color name `let colorName: string = Color[2]; // "Blue"`
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

# Interfaces

Interface describe objects

stao ovde https://www.typescriptlang.org/docs/handbook/interfaces.html

~~~
interface Person {
  firstName: string;
  lastName: string;
}
~~~

Classes

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

Type
Hero[] is array of Hero
? optional

https://www.youtube.com/watch?v=-PR_XqW9JJU
