---
layout: post
title: Bundler bower gulp npm tips
---

# Bundler

When you install another version of existing gem, for example `gem install rails
-v 4.2.0`, than you can find it with `gem list rails` and use it with `rails
_4.2.0_ new mystore`

To rebuild Gemfile.lock you can run `bundle update`

Issues:

* rmagic on ubuntu 16

  ~~~
  checking for Magick-config... no
  Can't install RMagick 2.12.0. Can't find Magick-config in
  ...
  An error occurred while installing rmagick (2.12.0), and Bundler cannot continue.
  Make sure that `gem install rmagick -v '2.12.0'` succeeds before bundling.
  ~~~

  I upgraded to rmagic 2.13.2 and follow this
  [issue](https://github.com/ttscoff/Slogger/issues/344)

  ~~~
  sudo apt-get install libmagickwand-dev
  sudo apt-get install graphicsmagick-imagemagick-compat
  PATH="/usr/lib/x86_64-linux-gnu/ImageMagick-6.8.9/bin-Q16:$PATH" gem install rmagick -v '2.13.2'
  ~~~


# Bower

[bower](http://bower.io)  Adding new package will be saved in `bower.json` if
you
add option `--save` or `--save-dev`

You can add github repos, just replace version with github url with `#v3.0.x`

~~~
{
  "_filename": "bower.json",
  "dependencies": {
    "angular-autodisable": "https://github.com/duleorlovic/angular-autodisable"
  }
}
~~~

~~~
bower list
bower install packageName
bower uninstall packageName
bower install ng-token-auth --save
~~~

You can update to the latest version by changing bower.json

~~~
# use latest angular material
sed -i bower.json -e '/angular-material/c\
    "angular-material": "*",'
bower update angular-material --save
~~~

Check installed version of some package, for example `angular-material`

~~~
cat  bower_components/angular-material/.bower.json | node_modules/json/lib/json.js version
# install json with npm install json
~~~

# Gulp

* if you have error `'watch' errored after Error: watch
  /home/orlovic/rails/menucards-frontend/src/ ENOSPC
  ` solution is to run `echo fs.inotify.max_user_watches=524288 | sudo tee -a
  /etc/sysctl.conf && sudo sysctl -p`
  [link](http://stackoverflow.com/questions/16748737/grunt-watch-error-waiting-fatal-error-watch-enospc)


