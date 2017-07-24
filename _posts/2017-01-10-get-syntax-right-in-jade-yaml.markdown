---
layout: post
title: Get syntax right in jade, yaml
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

# Yaml

key value pairs are delimited with colon space.
Value could be intented to mark the bounds of it.
String does not need `" "` only if you use some special symbol `|` `:`
Multiline strings starts with pipe `|` on the line with attribute name, and
indented.
Write dates in format `yyyy-mm-dd` so rails `Date.parse()` recognize it.

YML file can use anchor (`&`) and reference (`*`) so you do not repeat the code.
When you use reference `*` (as for testing) you can not add or update keys, but
with `<<` you can (as for [production secrets]( {{ site.baseurl }} {% post_url 2015-04-05-common-rails-bootstrap-snippets %}))

~~~
name: Dusan Orlovic
sport:
  kayak: |
    It is nice sport
    to play around

~~~

