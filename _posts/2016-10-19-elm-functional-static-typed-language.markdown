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
  * strings `"asd"` (concatenation `"asdf" ++ "asdf"`)
  * numbers `1` (integer division `5//2`)
* functions

  ~~~
  -- function definitions
  half n = n / 2
  -- using unonymous function
  half = \n -> n / 2
  -- when it has two arguments than we can use partial application
  divide x y = x / y
  <function> : Float -> Float -> Float
  -- it is the same as divide = \x -> (\y -> x / y)
  divide 3
  <function> : Float -> Float
  divide 3 2
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

  * update values with pipe (non destructive, it will create new attribute and
  share existing attributes) `{ point | x = 1 }`
  * difference with js objects is that you can't ask for nonexisting field, all
    fields are defined
* types
  * type annotations are written statement for types of arguments and output

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


* use backslash `\ ` to split in multiple lines

# User input

~~~
import Html exposing (

-- MODEL
type alias Model = {

-- UPDATE
type Msg = Reset |
update : Msg -> Model -> Model
update msg model =
  case msg of
    Reset -> ...

-- VIEW

view : Model -> Html Msg
view model =
~~~

# Examples

* [pairs.one](https://github.com/mxgrn/pairs.one) game in elixir and phoenix


tutorial todo <https://guide.elm-lang.org/get_started.html>

http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html
<https://www.toptal.com/front-end/make-web-front-end-reliable-with-elm>
https://www.youtube.com/watch?v=28jCDXfZCgE#t=0.417205
* [monads for rubyist](https://wuest.me/blog/e/20170528-io-monad-for-rubyists/)

# Haskell Books

* how to think in haskell [Maybe Haskell](https://gumroad.com/l/maybe-haskell)
* understanding syntax [Learn You a Haskell](http://learnyouahaskell.com/)

# Monads

https://stackoverflow.com/questions/3870088/a-monad-is-just-a-monoid-in-the-category-of-endofunctors-whats-the-problem
