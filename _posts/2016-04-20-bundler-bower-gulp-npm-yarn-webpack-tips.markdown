---
layout: post
---

# Bundler and Gemfile

When you install another version of existing gem, for example `gem install rails
-v 4.2.0`, than you can find it with `gem list rails` and use it with `rails
_4.2.0_ new mystore`.
To list by exact name (not regexp) use `-e`, to show remote gems `-r` so to find
all available versions including pre release run `gem list rails -re
--prerelease`

To find help some `gem` command you can run `gem help list`

To rebuild Gemfile.lock you can run `bundle update`. You can upgrade specific
gem `bundle update rails` and only patch version of rails and its dependencies
will be updated in Gemfile.lock. You should also update gemfile to point to new
patch version. If you want to update minor version than you
need to change in Gemfile, like `gem 'rails', '~>5.2.0'`
To install specific groups in Gemfile run `bundle install --with postgresql`.
To specify github source of gem use

~~~
gem 'jekyll', github: 'jekyll/jekyll' # branch: 'master'
~~~

Issue with rmagic on ubuntu 16

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

To remove all gems from current gemset

~~~
rvm gemset empty
~~~

# Creating gems


```
# create initial gem files
bundle gem datatables_rails_helpers
# bundle the gem
gem build rails-datatables-helpers.gemspec
# release
gem release
```

Semantic versioning Major(breaking change).Minor(add stuff).Patch(bugfix) level.

If you gem depend on some gem version `4.3.2` it can not be used for any other
version. `~> 4.3.2` (pesimistic) it better since it can be used with `4.3.3` but
not for previous versions `4.3.1`, and not for `4.4.0`. If gem follow semantic
version it can be ugraded to `4.4`, so `~> 4.3` means that it can be `4.x` but
at least `4.3`.  You should support broad range of gems `>= 3.2, < 5.0`,

# RVM

Install specific version and patch level `rvm install 2.3.3-p451`
To see current ruby `rvm list`


# Bower

Bower is deprecated https://github.com/bower/bower/issues/2298

# Gulp

* if you have error `'watch' errored after Error: watch
  /home/orlovic/rails/menucards-frontend/src/ ENOSPC
  ` solution is to run `echo fs.inotify.max_user_watches=524288 | sudo tee -a
  /etc/sysctl.conf && sudo sysctl -p`
  [link](http://stackoverflow.com/questions/16748737/grunt-watch-error-waiting-fatal-error-watch-enospc)

# NPM

To create package.json in current folder run `npm init -y`

To show all versions and install specific version you can

~~~
npm info ionic
npm install ionic@1.4.0
npm uninstall ionic
npm uninstall -g ionic
~~~

You can install packages localy (recommended) so you can access them
`node_modules/.bin/` or using `npx webpack` or with `npm run build` if inside
package.json `"scripts":{ "build": "webpack"}`. If you need to pass additional
params to build command you can use two dashes `npm run build -- --colors`
If you need to access from command line than install globaly `-g`
Use `--save` if package is used on production env, and `--save-dev` if it is
only on development (like linters or testing tools).

You can use scoped packages https://docs.npmjs.com/misc/scope that will be used
from subfolder, for example

~~~
npm install @myorg/mypackage
~~~

Scope can be associated with a registry at login `npm login
--registry=http://reg.example.com --scope=@myco` or at config `npm config set
@myco:registry http://reg.example.com`.

Package lock <https://docs.npmjs.com/files/package-locks> is used to prevend
using latest dependencies if they are not desirable `package-lock.json` or
`npm-shrinkwrap.json`

Note that if you remove some files from package folder (for example
`node_modules/iCheck/skins` than `npm install` will not see
that is was removed. You need to `rm -rf folder` so than `npm install` will
get fresh copy of the package.

Example adding jquery 3, bootstrap 4, fontawesome 4 on rails you can find on {{ site.baseurl }} {% post_url 2014-07-01-ruby-on-rails-layouts-and-rendering %})


# Npmjs publish package

https://docs.npmjs.com/getting-started/publishing-npm-packages

~~~
npm whoami
npm adduser
npm publish
~~~

# Yarn

Yarn will install packages to `node_modules` folder.

~~~
yarn init # to generate package.json
yarn remove [package]
yarn # to install dependencies
yarn run build # to run "scripts" -> "build"

yarn add [package]
~~~

You can add `package@version` vesion could be `"^1.0.0"`
<https://yarnpkg.com/en/docs/dependency-versions> When you `yarn
upgrade` it will upgrade version to next, for example `"^2.0.0"`.

If you want to install to specific folder instead of `node_modules` than use
`yarn install --modules-folder ./public/assets` or add to
[.yarnrc](https://yarnpkg.com/lang/en/docs/yarnrc/#toc-cli-arguments)

~~~
# .yarnrc
--install.modules-folder "./public/assets"
--add.modules-folder "./public/assets"
~~~

When you are developing, you can use link
```
# in package folder
yarn link

# in usage folder
# eventual remove from package and node_modules
yarn remove trk_datatables
yarn link trk_datatables
```

Note that there are problems with missing `.bin` folder
https://github.com/yarnpkg/yarn/issues/3724

# Commonjs

All bundle tools explained in video https://www.youtube.com/watch?v=U4ja6HeBm6s
CommonJS is used by node.js. `module.exports` is defined in environment, and
it's value is returned when you `require('my_module')` (no need to include
`.js`). You can export object with multiple functions. Build with `browserify`
or `webpack`.
Each module is run inside closure so there is no global variables. When you
include `<script>` tag code is run and that is.

~~~
# index.html
<html>
  <body>
    <div id="s"></div>
    <script src="./main-bundle.js"></script>
  </body>
</html>
~~~

~~~
# add.js
function add(a, b) {
  return a + b;
}
module.exports = add;

# main.js
var add = require('./add');
function sum(list) {
  var m = 0;
  list.forEach(function(item) {
    m = add(m, item);
  });
  return m;
}
document.getElementById('s').innerHTML = sum([1,2,3])


npm install browserify -g
browserify main.js -o main-bundle.js

npm install -D webpack-cli
webpack main.js -o main-bundle.js
~~~

# AMD Asynchronous module definition

When you `require('jQuery')` it should be there at that moment, but if we want
to run the code without jQuery loaded yet use async `define(dependencies, f)`
Build with `webpack` since it will recognize that it is AMD and define `define`
correctly.

~~~
# add.js
define([], function() {
  return function add(a, b) {
    return a + b;
  }
});

# main.js
define(['./add'], function(add) {
  function sum(list) {
    var m = 0;
    list.forEach(function(item) {
      m = add(m, item);
    });
    return m;
  }
  document.getElementById('s').innerHTML = sum([1,2,3])
});

webpack main.js -o main-bundle.js
~~~

For development you can use [RequireJS](http://requirejs.org/docs/whyamd.html)
(works with AMD modules) that can see if `define` is called and fetch other
dependencies (with another `<script>` tags) and include them so `main.js` can
execute it's code.

[RequireJS Optimizer](http://requirejs.org/docs/optimization.html) or
`amd-optimize` and almond (minimized AMD for optimized builds which define
`define`),
Or you can transform AMD modules to CommonJS js modules with `deamdify` (so no
need for almond to define `define`) and than run `browserify`.

# ES6 Ecmascript modules Import Export

~~~
# add.js
function add(a, b) {
  return a + b;
}
export { add }

# main.js
import { add } from './add';
function sum(list) {
  var m = 0;
  list.forEach(function(item) {
    m = add(m, item);
  });
  return m;
}

# build with webpack
webpack main.js -o main-bundle.js

# or build with browserify with babelify transform
npm install -g babelify
browserify -t babelify main.js -o main-bundle.js

# or older google tool Traceur with es6ify transform
~~~

If you want to dynamic load on development and bundle on production, you can use
systemjs, similar to RequireJS, but loads amd, es6 or commonjs modules and
compiles es6 in the browser.

[Rollup](https://github.com/rollup/rollup) bundle only functions that you use,
not the whole library (deprecate `esperanto`).

[jspm](https://jspm.io/)  can consume all formats in same time.

# UMD

Universal module definition so you can import in commonjs or amd or with script
tag.
Manualy is

```
(function( factory ) {
  "use strict";

  if ( typeof define === 'function' && define.amd ) {
    // AMD
    define( ['jquery'], function ( $ ) {
      return factory( $, window, document );
    } );
  }
  else if ( typeof exports === 'object' ) {
    // CommonJS
    module.exports = function (root, $) {
      if ( ! root ) {
        // CommonJS environments without a window global must pass a
        // root. This will give an error otherwise
        root = window;
      }

      if ( ! $ ) {
        $ = typeof window !== 'undefined' ? // jQuery's factory checks for a global window
          require('jquery') :
          require('jquery')( root );
      }

      return factory( $, root, root.document );
    };
  }
  else {
    // Browser
    factory( jQuery, window, document );
  }
}
(function( $, window, document, undefined ) {
  "use strict";

  function initialise() {
    alert('hi')
  }

  return { initialise }
}})
```

`use strict` is nice so you get error in javascript console when assigning
variable that has not been declared.

You can use rollbar or webpack to produce umd that will generate a global
`myGlobalModule` or plain js https://stackoverflow.com/questions/38638210/how-to-use-umd-in-browser-without-any-additional-dependencies

~~~
# main.js
import { add } from './add';
function sum(list) {
  var m = 0;
  list.forEach(function(item) {
    m = add(m, item);
  });
  return m;
}

// to use like `myGlobalModule([1,2,3])`.
export default sum
export default function sum(list) {

// to use like `myGlobalModule.sum([1,2,3])`.
export default {
  sum: sum
}
export function sum(list) {


// with webpack call with myGlobalModule.sum([1,2,3])
export function sum(list) {
// with webpack with myGlobalModule.default([1,2,3])
export default function sum(list) {
export default sum
// with webpack with myGlobalModule.default.sum([1,2,3])
export default {
  sum: sum
}
~~~

rollup
~~~
rollup main.js --format umd --name 'myGlobalModule' --file main-bundle-rollup-umd.js
~~~

webpack
~~~
# webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'main-bundle-webpack-umd.js',
    path: __dirname,
    library: "myGlobalModule",
    libraryTarget: "umd"
  }
}
~~~

~~~
webpack
~~~

# Webpack

<https://webpack.academy>
<https://www.what-problem-does-it-solve.com/webpack>
<https://survivejs.com/webpack/>

<https://webpack.js.org/guides/getting-started/>
/home/orlovic/rails_temp/webpack_guide

~~~
npm init -y
npm install webpack webpack-cli --save-dev
~~~

Add to `package.json` build command so you can run

~~~
// package.json
  "scripts": {
    "build": "webpack"
  }

// run build with
npm run build

// if webpack is installed globally than you can use directly
webpack # not recomended to use as global
~~~

Entry point is module for which webpack start building out dependency graph
(search for imports, requires, defines) and create bundles.
Output is where to emit the bundles (default `path` is `./dist`).
Loader is used to process webpack modules (files that use `import`, `require()`,
`@import`, `url()`) and use coffescript, typescript, sass processors. It has
`test` property (which files should be transformed) and `use` which loader.
Plugins are use to perform wider range of tasks like uglify.
`./dist/` is default output path folder, and you can open `gnome-open
dist/index.html`.

Asset Management with loaders (`npm install --save-dev style-loader`)

*  `style-loader` will in javascript create and attach to head as inline `<style
  type="text/css">.my-style-from-file{}</style>`. It needs to be called in js
  `import './my_style.css'` to define which file will be imported.

* `css-loader` will interprets `@import` and `url()` and copy file using random
  file name
  ~~~
  // src/style.css
  .background {
    background: url('./img.png');
  }
  ~~~

* `file-loader` handles files
  ~~~
  import myImage from './img.png'
  myImage // random-name-of-image.png
  ~~~

* `html-loader` handles `<img src="./my-image.png">`.

  ~~~
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: [
          'file-loader'
        ]
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: [
          'file-loader'
        ]
      }
    ]
  }
  ~~~

Input and Output
For several input files you can generate `[name].bundle.js`.

~~~
// webpack.config.js
const path = require('path')

module.exports = {
  entry: {
    app: './src/index.js',
    print: './src/print.js'
  },
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
~~~

We need to manually include those generated files in `index.html`,  it is easier
to automatically include https://github.com/jantimon/html-webpack-plugin
Output management is needed to automatically include compiled assets (which are
fingerprinted) using `HtmlWebpackPlugin` installed with `npm install --save-dev
html-webpack-plugin` and it will generate `dist/index.html` which includes all
generated stuff.

~~~
// webpack.config.js
  plugins: [
    new HtmlWebpackPlugin({
      title: 'Output management'
    })
  ],
~~~

You can pass env params on command line and extract common config
```
// build-utils/webpack.common.js
const commonPaths = require('./common-paths.js');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: commonPaths.output,
  },
};
```

```
// build-utils/common-paths.js
const path = require('path');

module.exports = {
  outputPath: path.resolve(__dirname, '../', 'dist'),
};
```

```
// webpack.config.js
const commonConfig = require('./build-utils/webpack.common');

module.exports = (env) => {
  console.log(env);
  return commonConfig;
};
```
https://webpack.academy/courses/web-fundamentals/lectures/2987268

https://www.npmjs.com/package/clean-webpack-plugin
You can clean dist folder before each build
~~~
npm install clean-webpack-plugin --save-dev

# webpack.config.js
const CleanWebpackPlugin = require('clean-webpack-plugin');

  plugins: [
    new CleanWebpackPlugin(['dist']),
~~~

Use
[WebpackManifestPlugin](https://github.com/danethurber/webpack-manifest-plugin)
for manifest about what is included in build.

Add `devtool: 'inline-source-map',` to provide source maps.

webpack-dev-server for live reloading when code is changed

~~~
npm install --save-dev webpack-dev-server

# webpack.config.js
  devServer: {
    contentBase: './dist'
  },

# package.json
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack-dev-server --open",

npm start
~~~

HMR Hot module replacement
~~~
# webpack.config.js
const webpack = require('webpack');
module.exports = {
  devServer: {
    contentBase: './dist',
    hot: true
  },
  plugins: [
    new webpack.NamedModulesPlugin(),
    new webpack.HotModuleReplacementPlugin()


# src/index.js
if (module.hot) {
  module.hot.accept('./print.js', function() {
    console.log('Accepting the updated printMe module!');
    printMe();
    document.body.removeChild(element);
    element = component(); // Re-render the "component" to update the click handler
    document.body.appendChild(element);
  })
}
~~~

Babel-loader
https://github.com/babel/babel-loader

~~~
npm install "babel-loader@^8.0.0-beta" @babel/core @babel/preset-env webpack
~~~



<https://medium.com/statuscode/introducing-webpacker-7136d66cddfb>

Webpack uses `webpack.config.js` and compile for your using Loaders and Plugins.
Install loaders with `yarn add --dev webpack babel-core babel-loader
babel-preset-react babel-preset-es2015 node-sass css-loader sass-loader
style-loader` and use with:

~~~
// webpack.config.js
// This library allows us to combine paths easily
const path = require('path');
module.exports = {
   entry: path.resolve(__dirname, 'src', 'index.jsx'),
   output: {
      path: path.resolve(__dirname, 'output'),
      filename: 'bundle.js'
   },
   resolve: {
      extensions: ['.js', '.jsx']
   },
   module: {
      rules: [
         {
             test: /\.jsx/,
             use: {
                loader: 'babel-loader',
                options: { presets: ['react', 'es2015'] }
             }
         },
         {
            test: /\.scss/,
            use: ['style-loader', 'css-loader', 'sass-loader']
         }
      ]
   }
};
~~~

For `umd` you need to disable `amd` parser
https://webpack.js.org/configuration/module/#ruleparser
```
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        parser: { amd: false },
      },
    ]
  }
}
```

# Webpack plugins

~~~
yarn add webpack-bundle-analyzer --dev
// config/webpack/development.js
const BundleAnalyzerPlugin =
require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
module.exports = environment.toWebpackConfig().merge({
  plugins: [
    new BundleAnalyzerPlugin({analyzerMode: 'static'})
  ]
});

# bin/webpack-dev-server
# /home/orlovic/rails/myApp/public/packs/report.html
~~~

# Webpacker

From sprockets to webpacker
<https://medium.com/@coorasse/goodbye-sprockets-welcome-webpacker-3-0-ff877fb8fa79>

~~~
# Gemfile
# instead of sprockets
gem 'webpacker', '~> 3.4'

bundle
bundle exec rails webpacker:install
~~~

Put your 'entry' module files in `app/javascript/packs` and your pure js files
in `app/javascript` or `app/javascript/src` which you can load (just run)
`import '../my_file'`.
Global functions will not pollute global scope, you need to attach to window or
global object.
Reload js files automatically with webpacker dev server

~~~
bin/webpack-dev-server
~~~

Refence each file in packs folder with (restart dev seerver)

~~~
<%# app/views/layouts/application.html.erb %>
  <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
~~~

Add npm packages: jquery turbolinks bootstrap

~~~
yarn add bootstrap jquery popper.js
~~~

Configure jquery as plugin

~~~
// config/webpack/environment.js
const { environment } = require('@rails/webpacker')
const webpack = require('webpack');
environment.plugins.append('Provide', new webpack.ProvidePlugin({
  $: 'jquery',
  jQuery: 'jquery',
  Popper: ['popper.js', 'default']
}));

// or if you load jQuery in script tag
environment.config.external = {
  jquery: 'jQuery'
}

module.exports = environment
~~~

In application pack
```
// app/javascript/packs/application.js
require("@rails/ujs").start()
require("turbolinks").start()
require("@rails/activestorage").start()
require("channels")

import 'bootstrap'
import '../stylesheets/application'
import './turbolinks.load'
```
and create separate pack for turbolinks:load events

```
// app/javascripts/packs/turbolinks.load.js
document.addEventListener('turbolinks:load', () => {
  $('[data-toggle="tooltip"]').tooltip()
})
```
small note, we could use ES6 syntax
~~~
// app/javascript/packs/application.js
import Rails from 'rails-ujs';
import Turbolinks from 'turbolinks';

Rails.start();
Turbolinks.start();

$(function () {
  console.log('Hello World from Webpacker');
});

# if you need jquery globally attach to window or global object
global.$ = require('jquery')
window.$ = window.jQuery = require("jquery")
~~~

https://getbootstrap.com/docs/4.0/getting-started/webpack/
For stylesheets create new module and move files and import in scss. Use tilda
when referencing node modules.

~~~
// app/javascript/stylesheets/application.scss
@import '~bootstrap/scss/bootstrap';
~~~

For css from gems create npm package
https://github.com/rails/webpacker/issues/57#issuecomment-327533578

If package.json contains `style` attribute than with `postcss-import` plugin,
which is by default enabled in `postcss.config.js`, you can import css with
`@import 'package-name'`
```
// app/javascripts/stylesheet/application.css
// this file should CSS not scss since @import command is different
@import 'package-name'
@import 'package-name/file-path/file.css'
```
also import that css file
```
// app/javascript/packs/application.js
import '../stylesheet/application.css'
```

To import sass file you need sass loader https://github.com/webpack-contrib/sass-loader
Rails supports it but on clean webpack project you need to:

```
```

Alternatively, you can import css file from node_modules in javascript.
```
// app/javascript/packs/application.js
import 'package-name/file-path/file.css'
```

## IIFE Immediatelly invoked function expression

It is used so any variable declared inside is not visible to the outside word.
Any unary operator `!` or `+` will make expression instead functional definition
(it is a statement, which is not executed) so we can call that funtion
```
+function(i) {
  alert(i)
}(1);
```
When we need return value, instead of unary operator, we can use parenthesis in
two variations (both are OK).
```
// Variation 1
(function() {
    alert("I am an IIFE!");
}());

// Variation 2
(function() {
    alert("I am an IIFE, too!");
})();
```

todo
https://youtu.be/fKOq5_2qj54?t=602

# Parcel

Install with `yarn global add parcel-bundler`

# Makefile


```
make help
```
