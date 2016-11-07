---
layout: post
title: Elm functional static typed language
---

# Installation

~~~
npm install -g elm
~~~

Plugin [elm.vim](https://github.com/lambdatoast/elm.vim)

Command line:

* `elm-repl` like console
* `elm-reactor` web server
* `elm-make Main.elm --output=main.html` to compile
* `elm-package` for downloading libs

# Core language

* values
  * stings `"asd"` `"asdf" ++ "asdf"`
  * numbers `1` `5//2`
* functions

  ~~~
  half n = n / 2
  -- using unonymous function
  half = \n -> n / 2
  -- when it has two arguments than we can use partial application
  divide x y = x / y
  <function> : Float -> Float -> Float
  -- it is the same as divide = \x -> (\y -> x / y)
  divide 3
  <function> : Float -> Float
  divise 3 2
  1.5 : Float
  ~~~

* lists: all values must have same type

  ~~~
  name = [ "Alice", "Bob" ]
  ~~~

* tuples: can hold different types

  ~~~
  import String
  goodName name = \
    if String.length name <= 20 then \
      (True, "ok") \
    else
      (False, "nope")
  ~~~

* records: key value pairs `point = { x = 3, y = 4 }`
  * access with dot `point.x`
  * access functions is when variable starts with dot `.name person1`

    ~~~
    List.map .name [ {name='Dule'}, ]
    ~~~

  * pattern matching (extrapolation) inside function arguments with `{arg}`

    ~~~
    under70 {age} = age < 70
    ~~~

  * update values (non destructive, it will create new attribute and share
    existing attributes) `{ point | x = 1 }`
  * difference with js objects is that you can't ask for nonexisting field, all
    fields are defined
* types
  * type annotations is written statement for types of arguments and output

    ~~~
    name : String
    name = "Dusan"

    add : Int -> Int
    ~~~

  * type alias is used short type annotations

    ~~~
    type alias User =
      { name: String
      , bio: String
      }
    ~~~

  * union types is used to create new type with new values

    ~~~
    type Visibility = All | Active | Completed
    -- All is type of Visibility ie All : Visibility
    f : Visibility -> String
    f visibility =
      case visibility of
        All ->
          "all"
    -- all case posibilities are checked by compiler

    type Msg
      = Change String
    -- in this declaration new constructors are created Change: String -> Msg

    type IntList = Empty | Node Int IntList
    -- linked list is recursive: Node 64(Node 42 Empty)

    type List a = Empty | Node a (List a)
    -- generic linked list
    ~~~


* use backslash `\` to split in multiple lines

# Examples

* [pairs.one](https://github.com/mxgrn/pairs.one) game in elixir and phoenix


tutorial todo <https://guide.elm-lang.org/get_started.html>

