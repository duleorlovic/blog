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

For Gemfile you need to install `gem install bundler` (use `gem update --system`
if you are using system ruby). You can install old
version `gem install bundler -v 1.9.2` but `bundle install` will use latest,
unless you use specific `bundle _1.9.2_ install`. Bundle will call gem install.
To see location you can use `bundle show my_gem` or `gem which my_gem`. Also you
can use `gem unpack my_gem` to copy gem source code to current folder
`./my_gem`.
If you use `bundle package` command https://bundler.io/v2.0/bundle_package.html
than it will download all gems to `vendor/cache` (and create `./bundle/config`
file so next time you run `bundle` it will install to vendor/cache).

If you see error The git source https://github.com/activeadmin/activeadmin.git is not yet checked out. Please run `bundle install` before trying to start your application
You can install gems to local path `bundle _1.17.3_ install --path
vendor/cache/`

Gems that are using git github source y
To rebuild Gemfile.lock you can run `bundle update`. You can upgrade specific
gem `bundle update rails` and only patch version of rails and its dependencies
will be updated in Gemfile.lock. You should also update gemfile to point to new
patch version. If you want to update minor version than you
need to change in Gemfile, like `gem 'rails', '~>5.2.0'`
To install specific groups in Gemfile run `bundle install --with postgresql`.
To specify github source of gem use

~~~
gem 'jekyll', github: 'jekyll/jekyll' # branch: 'master'

gem 'actionview', path: '~/rails/rails' # local gem
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
sudo apt-get install libmagick++-dev
# this does not help much
# sudo apt-get install graphicsmagick-imagemagick-compat
# PATH="/usr/lib/x86_64-linux-gnu/ImageMagick-6.8.9/bin-Q16:$PATH" gem install rmagick -v '2.13.2'
PKG_CONFIG_PATH="/usr/lib/x86_64-linux-gnu/pkgconfig" gem install rmagick
~~~

To use gems in ruby scripts you can use `gem 'gemname'` for example
```
# my_script.rb
require 'bundler/inline'
gemfile do
  gem 'google-cloud-translate', '~> 1.4.0'
  gem 'byebug'
end
require 'google/cloud/translate'
require 'byebug'
```


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
at least `4.3`. You can actually use notation `4.x`.
You should support broad range of gems `>= 3.2, < 5.0`,

When you run `bundle update` and get several error messages like: `Bundler could
not find compatible versions for gem "actionmailer":` than look at last message
since other could be solved when we solve last one.
Check that `bundler` is latest, use `~>` instead of `=` version.



# RVM

Install specific version and patch level `rvm install 2.3.3-p451`
You can specify ruby in Gemfile `ruby '2.6.4'` or in `.ruby-version` and
`.ruby-gemset` files https://rvm.io/workflow/projects
To see current ruby `rvm list`
To remove all gems from current gemset

~~~
rvm gemset empty
~~~

If you want to install ruby 2.3 on Ubuntu 18, there is problem with libddbm3 so
you need to download https://ubuntu.pkgs.org/16.04/ubuntu-main-amd64/libgdbm3_1.8.3-13.1_amd64.deb.html
Error is /home/orlovic/.rvm/gems/ruby-2.4.5/gems/activesupport-4.2.7.1/lib/active_support/core_ext/numeric/conversions.rb:124:in block (2 levels) in <class:Numeric>': stack level too deep (SystemStackError)
```
sudo dpkg -i ~/Downloads/libgdbm3_1.8.3-13.1_amd64.deb 
```

# Nodenv

It is similar to rbenv since it uses shims
Install using https://github.com/nodenv/nodenv-installer#nodenv-installer

```
curl -fsSL https://raw.githubusercontent.com/nodenv/nodenv-installer/master/bin/nodenv-installer | bash

nodenv init
# add to ~/.bash_profile
# eval "$(nodenv init -)"

```

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

To run script from node_modules/some-package/package.json you can run
https://stackoverflow.com/a/48803884/287166
```
npm explore some-package -- npm run subscript

yarn --cwd node_modules/some-package/ run subscript

```

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
# yarn add install specific version 2.3.4
yarn add package@2.3.4
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

In this case, all package developments will be pulled from
package_folder/node_modules

Note that there are problems with missing `.bin` folder
https://github.com/yarnpkg/yarn/issues/3724

You can use https://classic.yarnpkg.com/en/docs/cli/create/ to install package
and run bin field

```
yarn create react-app my-app

# is equivalent to:

yarn global add create-react-app
create-react-app my-app
```

If you have dependencies in your main package.json and also in some of dependend
packages, than you might end up with two different versions of it. Use
resolution to install only one version
```
  "resolutions": {
    "jquery": "2.2.3"
  },
```

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
You can find templates https://github.com/umdjs/umd/tree/master/templates
My example for jQuery plugins is to use `module.exports = function(root, $) {
... factory($, root)}`
Manualy is:

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
<https://www.what-problem-does-it-solve.com/webpack/index.html>
<https://survivejs.com/webpack/>

<https://webpack.js.org/guides/getting-started/>
/home/orlovic/rails_temp/webpack_guide

Example using with webpack-dev-server for vue
https://github.com/duleorlovic/slidey-nav or `~/html/slidey-nav`

```
mkdir myproj
cd myproj
git init .
cat > .gitignore << 'HERE_DOC'
node_modules
dist
.yarn-error.log
HERE_DOC
```

Install using yarn
```
yarn init -y
yarn add -D webpack@4.44.0 webpack-cli@3.3.0 webpack-dev-server@3.11.0
```

You can run cli
```
$(yarn bin)/webpack  --mode=none --entry ./src/index.js  --output-filename=bundle.js
```

or add to `package.json` build command

~~~
// package.json
  "scripts": {
    "build": "webpack --config webpack.config.js"
  },
~~~

so you can run with
```
yarn run build
```

if webpack is installed globally than you can use directly `webpack` but it is
not recomended to use as global
Also version of webpack-cli and webpack-dev-server should be compatible. For
example webpack-cli 4 and webpack-dev-server 3 has an issue
```
webpack-dev-server --open
internal/modules/cjs/loader.js:638
    throw err;
    ^
```
so we need to fix than in package.json
```
{
  "devDependencies": {
    // "webpack": "^4.44.0"
    "webpack-cli": "3.3.0",
    "webpack-dev-server": "3.11.0"
  }
}
```

Create `webpack.config.js` with
```
module.exports = {
  entry: './src/index.js',
  mode: 'none'
}
```

Entry point is module for which webpack start building out dependency graph
(search for imports, requires, defines) and create bundles.
Output is where to emit the bundles (default `path` is `./dist`).
Loader is used to process webpack modules (files that use `import`, `require()`,
`@import`, `url()`) and use coffescript, typescript, sass processors. It has
`test` property (which files should be transformed) and `use` which loader.
Plugins are used to perform wider range of tasks like uglify.
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

For example for using sass you need `yarn add -D sass sass-loader` and
```
// webpack.config.js
  module: {
    rules: [
      {
        // https://github.com/webpack-contrib/sass-loader
        test: /\.(sass|css)$/i,
        use: [
          // Creates `style` nodes from JS strings
          'style-loader',
          // Translates CSS into CommonJS
          'css-loader',
          // Compiles Sass to CSS
          // https://github.com/webpack-contrib/sass-loader
          'sass-loader',
        ]
      }
    ]
  },
```

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
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  // https://github.com/jantimon/html-webpack-plugin
  plugins: [
    new HtmlWebpackPlugin({
      template: 'src/index.html'
    })
  ],
}
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

You can disable `amd` parser
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
or in webpacker

```
// config/webpack/environment.js
environment.config.set('amd', false)
```

Or specific module with imports-loader
https://webpack.js.org/guides/shimming/#granular-shimming
https://webpack.js.org/loaders/imports-loader/
```
// config/webpack/environment.js
const nonAmdLoader = {
  test: require.resolve('trk_datatables'),
  use: "imports-loader?define=>false",
}
environment.loaders.append('nonAmd', nonAmdLoader)
```
or in js
```
  require('imports-loader?define=>false,this=>window!datatables.net')(window, $)
```
Do not know how to disable amd for dependency of dependency

# Webpack plugins

To check if multiple versions of jquery package is included you can use

~~~
yarn add webpack-bundle-analyzer --dev

// config/webpack/development.js
process.env.NODE_ENV = process.env.NODE_ENV || 'development'

const environment = require('./environment')

// https://www.npmjs.com/package/webpack-bundle-analyzer
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
module.exports = environment.toWebpackConfig().merge({
  plugins: [
    new BundleAnalyzerPlugin({analyzerMode: 'static'})
  ]
});

bin/webpack-dev-server
gnome-open public/packs/report.html
~~~

To copy files to build directory https://github.com/webpack-contrib/copy-webpack-plugin
Also you can use https://www.npmjs.com/package/webpack-shell-plugin


# Webpacker

25 things you can do with webpack
https://rossta.net/blog/reasons-to-switch-to-webpacker.html

From sprockets to webpacker
<https://medium.com/@coorasse/goodbye-sprockets-welcome-webpacker-3-0-ff877fb8fa79>
https://gist.github.com/maxivak/2612fa987b9f9ed7cb53a88fcba247b3

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
Also you can include view files to trigger reload
```
# config/webpack/development.js
const path = require('path')

environment.config.devServer.watchContentBase = true
environment.config.devServer.contentBase = [
  path.join(__dirname, '../../app/views'),
]
```
In vim there is an issue that vim does not always trigger webpack reload (touch
trigger always) https://github.com/webpack/webpack/issues/781#issuecomment-95523711
so add `set backupcopy=yes` in `.vimrc`

~~~
bin/webpack-dev-server
# issue with webpack
# ActionController::RoutingError (No route matches [GET] "/packs/js/application-8e9284513a1fd7d0456f.js"):
# is needed when you do not have proper node version installed when you are
# running rails s command
~~~

Refence each file in packs folder with (restart dev server)

~~~
<%# app/views/layouts/application.html.erb %>
  <%= javascript_pack_tag 'application', 'data-turbo-track': 'reload' %>
~~~

Add npm packages: jquery turbolinks bootstrap

~~~
yarn add bootstrap jquery popper.js
# for bootstrap 5 you need @popperjs/core
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

// node_modules
import 'bootstrap'
import 'select2' // globally assign select2 fn to $ element
// it is not the same as: require('select2')

// our stimulus stuff
// install with bundle exec rails webpacker:install:stimulus
import 'controllers'

// our js stuff
import 'turbolinks.load'
import 'window_functions'

// our stylesheet, please add <%= stylesheet_pack_tag 'application' %> to layout
import 'stylesheet/application'
```

and create separate pack for turbolinks:load events

```
// app/javascripts/packs/turbolinks.load.js
document.addEventListener('turbolinks:load', () => {
  $('[data-toggle="tooltip"]').tooltip()
})
```
small note, we could use `import` syntax
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

For css from gems create npm package
https://github.com/rails/webpacker/issues/57#issuecomment-327533578

If package.json contains `style` attribute than with `postcss-import` plugin,
which is by default enabled in `postcss.config.js`, you can import css with
`@import 'package-name'`
```
/* this file should have extension CSS not scss since @import command is different */
/* here we include other packages so postcss-import plugin will load css file
 * from style attribute from package.json */
/* @import 'package-name' is the same as @import 'package-name/file-path/file.css' */
@import 'select2';
```
also import that css file
```
// app/javascript/packs/application.js
// our stylesheet, one for postcss and one for scss
import 'stylesheet/post_css'
import 'stylesheet/application'
```

To import sass file you need sass loader https://github.com/webpack-contrib/sass-loader
which is included in webpack
supports it but on clean webpack project you need to:

```
// webpack.config.js
const path = require('path')

module.exports = {
  module: {
    rules: [
      {
        test: /\.(scss|sass|css)$/i,
        use: [
          // Creates `style` nodes from JS strings
          'style-loader',
          // Translates CSS into CommonJS
          'css-loader',
          // Compiles Sass to CSS
          'sass-loader',
        ]
      },
    ],
  },
}
```

Webpacker supports babel preset out of box. In clear webpack you can enable
babel with
```
// webpack.config.js
const path = require('path')

module.exports = {
  module: {
    rules: [
      {
        test: /\.js?$/,
        exclude: /(node_modules)/,
        use: 'babel-loader',
      },
    ],
  },
}
```

To find jquery version you can type `$.fn.jquery`

Install zurb foundation https://www.reddit.com/r/rails/comments/bhk72q/how_to_require_foundationsites_after_yarn_add/

Use image in views by adding `media` to the path, for
`app/javascript/images/logo.jpg` image
```
<%= image_pack_path('media/images/avatar_male.png') %>

# or if you need just a path
<img src="<%= asset_pack_tag 'media/images/logo.jpg' %>" />
ActionController::Base.helpers.asset_pack_path('media/images/avatar_male.png')
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

```
# Makefile, first is default target
blah: blah.o
	cc blah.o -o blah

blah.o: blah.c
	cc -c blah.c -o blah.o

blah.c:
	echo "int main() { return 0; }" > blah.c

clean:
	rm -f blah.o blah.c blah
```

# TIPS

* node uses two core modules: `module` and `require`.
* `require('my')` will look for files in `./node_modules/my.js`,
  `~/node_modules/my.js`, `/node_modules/my.js`... It can be also a folder and
  index.js file `./node_modules/my/index.js` or other file or folder defined in
  `main` property of `./node_modules/my/package.json`
* `require.resolve('my')` will return full path of file
* `module` is an object with `id`, `exports`, ...
* `module.exports = function() {}`, or `module.exports = { id: 'my' }`. That
  will be return value when we `const MY = require('my')`
* in node we have `fs.readFile` (file system), `setImmediate(function(){})`
* `const { host, post } = require('./config.json')` can read json. You can see
  all parsers `require.extensions` and source code with
  `console.log(require.extensions['.js'].toString())`

* `require` is not the same as `import` in webpack
* instead of `require('select2')(window, $)` you can use
  ```
  import select2 from 'select2'
  select2(window, $)
  ```
