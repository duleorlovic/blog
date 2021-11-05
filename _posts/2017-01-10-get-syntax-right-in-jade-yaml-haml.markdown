---
layout: post
---

# Jade

Very nice tool, like coffescript. You can convert all html files with:

~~~
npm install -g html2jade
html2jade *.html --bodyless --donotencode --noattrcomma --noemptypipe
# --bodyless          do not output enveloping html and body tags
# --donotencode       do not html encode characters (useful for templates)
# --noattrcomma       omit attribute separating commas
# --noemptypipe       omit lines with only pipe ('|') printable character
rm *.html

# version for all files in subdirectories
find . -name '*.html' -exec html2jade {} --bodyless --donotencode --noattrcomma --noemptypipe \;
find . -name '*.html' -exec  rm {} \;
~~~

Please check if format is good. At least this command will ignore <body> tag so
you need to build that (index.html) separately. Usually index.html needs to be
at specific location so I leave it in html format, and all other move to
`jade_build` folder.

* arguments could be in new line, but strings need **\**. comments are not
allowed inside ()

  ~~~
  // comment should be outside of (arguments)
  md-button(ng-really-click='vm.delete()'
  ng-really-message='Are you sure to \
  remove this option?')
  ~~~

  I prefer not to indent attributes even they suggest to [indent
  them](http://jade-lang.com/reference/attributes/)

# Yaml yml

* comments are single line and starts with `#` number sign
* list (arrays) are denoted by a leading hyphen `-` for each member per line, or
  in single line with `[a, b]` square brackets separated with comma space (this
  short notation is called flow collections)
  ```
  # flow style
  array: [ foo bar, baz ]
  # block style
  array:
    - foo bar
    - baz
  ```
* associative arrays (hash) are written with colon space `key: value`
  (multiline) or in single line with `{a: 1, b: 2}`, so this two are the sam
  ```
  # flow style
  hash: { a: 1 }
  # block style
  hash:
    a: 1
  ```

  Array of object (combination array and hash)
  ```
  combination:
    - a: 1
      b: 2  # this is { a: 1. b: 2 }
    - [ a: 1, b: 3]
  ```
* String does not need `" "` except you start with special symbol `[] {} > | *
  & ! % # @ ,` (or you use flow collections with those symboles). `: ` colon
  followed by a space is indicator for mapping. ` #` a space followed by pound
  sign starts a comment, so you need to quote whole string if it contains that.
  You can use both signle and double quotes, but double quotes enables to use
  escapes like `"an escaped \' single quote. \n"`
* Multiline strings starts with pipe `|` (Literal block scalar) on the line with
  attribute. `>` Folded block scalar will generate single line string, except
  there are empty lines (ie double new line characters) which are converted to
  "\n". Following lines should be indented.
  ```
  kayak: |
    It is nice sport
    to play around

  fold_some_newlines: >
      a
      b

      c
      d
        e
      f
  same_as: "a b\nc d\n  e\nf\n"
  ```
* Write dates in format `yyyy-mm-dd` so rails `Date.parse()` recognize it.
* YML file can use anchor ie alias (`&`) and reference (`*`) so you do not
  repeat the code. When you use reference `*` (as for testing) you can not add
  or update keys, but with `<<` you can (as for [production secrets]
  ({{ site.baseurl }} {% post_url 2015-04-05-common-rails-bootstrap-snippets %}))
  ~~~
  default: &default
    key: 123

  development:
    <<: *default
  ~~~
  Merge key can be used to merge multiple maps, and you can override them also
  ```
  - # Override
    << : [ *BIG, *LEFT, *SMALL ]
    x: 1
    label: center/big
  ```


In rails you can use $LABEL and $DEFAULTS and custom_name will be ignored
https://github.com/rails/rails/blob/master/activerecord/lib/active_record/fixtures.rb#L422
```
# _fixture is in Rails 6.1
_fixture:
  ignore: custom
custom: &custom
  description: %LABEL description
DEFAULTS: &DEFAULTS
  name: $LABEL name
  <<: *custom
my:
  <<: *DEFAULTS
```

# Haml


* html tags are written with `%`, and attributes is hash `{src: post.image}` but
for class you can use css notation `.class` and `#id` (and `%div` is not
needed), and for content from ruby code use `=` immediatelly or idented in new
line, like: `%div.col-md-6= @user.name`
* if you need more nested content, use tabs
* if ruby line spans multiple lines than use

~~~
= link_to "Add",
  url: posts_url
~~~

* plain text is without `%` or `=`
* excaping html `&= "Me & you"` will sanitize HTML sensitive characters `Me
&amp; you`, unescaping HTML `!= <br>` will output `<br>` instead of `&lt;br&gt;`
* if conditional classes is false, than it is ignored `.item{class: @item.empty?
&& "empty"}` can be `<div class="item">` or `<div class="item empty">`
* whitespace can be removed immediatelly within the tag `<` (or surrounding the
tag `>`), for example

  ~~~
  %blockquoute<
    %div
      Foo
  ~~~

  is

  ~~~
  <blockqoute><div>
    Foo
  </blockquote></div>
  ~~~
* HTML comments `<!-- -->` is with `/`
* ruby code which is not inserted `-` like `- title = 'home'`. Iteration and
other blocks use indent

  ~~~
  - case 2
  - when 1
    = "1"
  - when 2
    = "2?"
  ~~~

* ruby comments `-# this is a comment`
* filters like `:javascript`, `:coffeescript`, `:scss`

  ~~~
  :javascript
    $(document).ready(function() {
      alert("hi");
    });
  ~~~

# Json

Can not add comments, only you can is to add anoter fieeld
```
{
  "_comment": "This is a comment",
  "name": "My name"
}
```
