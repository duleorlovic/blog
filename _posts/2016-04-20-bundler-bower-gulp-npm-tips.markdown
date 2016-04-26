---
layout: post
title: Bundler bower gulp npm tips
---

# Bundler

When you install another version of existing gem, for example `gem install rails
-v 4.2.0`, than you can find it with `gem list rails` and use it with `rails
_4.2.0_ new mystore`

To rebuild Gemfile.lock you can run `bundle update`


# Bower

[bower](http://bower.io) `bower list` `bower install packageName` `bower
uninstall packageName`. Adding new package will be saved in `bower.json` if you
add option `--save` or `--save-dev`

~~~
bower install ng-token-auth --save
~~~

You can update

~~~
# use latest angular material
sed -i bower.json -e '/angular-material/c\
    "angular-material": "*",'
bower update angular-material --save
~~~

