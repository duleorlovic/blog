---
layout: post
---

# Bundler and Gemfile

When you install another version of existing gem, for example `gem install rails
-v 4.2.0`, than you can find it with `gem list rails` and use it with `rails
_4.2.0_ new mystore`

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

# RVM

* install specific version and patch level `rvm install 2.3.3-p451`

# Bower

Bower is deprecated https://github.com/bower/bower/issues/2298

[bower](http://bower.io)  Adding new package will be saved in `bower.json` if
you add option `--save` or `--save-dev`. To generate bower.json run

~~~
bower init
~~~


With [install](https://bower.io/docs/api/#install) you can use package name, git
url, local folder, with a version like `#v3.0.x`

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
# install json with: npm install json
~~~

Note that if you remove some files from package folder (for example
`vendor/assets/bower_components/iCheck/skins` than `bower install` will not see
that is was removed. You need to `rm -rf folder` so than `bower install` will
get fresh copy of the package.

# Gulp

* if you have error `'watch' errored after Error: watch
  /home/orlovic/rails/menucards-frontend/src/ ENOSPC
  ` solution is to run `echo fs.inotify.max_user_watches=524288 | sudo tee -a
  /etc/sysctl.conf && sudo sysctl -p`
  [link](http://stackoverflow.com/questions/16748737/grunt-watch-error-waiting-fatal-error-watch-enospc)

# Yarn

Yarn will install packages to `node_modules` folder.

~~~
yarn init # to generate package.json
yarn remove [package]
yarn # to install dependencies
yarn run build # to run "scripts" -> "build"
~~~

You can add `package@version` vesion could be `"^1.0.0"`
<https://yarnpkg.com/en/docs/dependency-versions> When you `yarn
upgrade` it will upgrade version to next, for example `"^2.0.0"`.

Example adding jquery 3, bootstrap 4, fontawesome 4

~~~
yarn add jquery bootstrap font-awesome popper.js
~~~

If you want to install to specific folder instead of `node_modules` than use
`yarn install --modules-folder ./public/assets` or add to
[.yarnrc](https://yarnpkg.com/lang/en/docs/yarnrc/#toc-cli-arguments)

~~~
# .yarnrc
--install.modules-folder "./public/assets"
--add.modules-folder "./public/assets"
~~~

# NPM

To create package.json in current folder run `npm init -y`

To show all versions and install specific version you can

~~~
npm info ionic
npm install ionic@1.4.0
npm uninstall ionic
~~~

You can install packages localy (recommended) so you can access them
`node_modules/.bin/` or using `npx webpack` or with `npm run build` if inside
package.json `"scripts":{ "build": "webpack"}`. If you need to pass additional
params to build command you can use two dashes `npm run build -- --colors`
If you need to access from command line than install globaly `-g`
Use `--save` if package is used on production env, and `--save-dev` if it is
only on development (like linters or testing tools).

# Webpack

<https://webpack.js.org/guides/getting-started/>

~~~
npm init -y
npm install webpack webpack-cli --save-dev
~~~

Add to `package.json` build command so you can run

~~~
// package.json
  "scripts"; {
    "build"; "webpack"
  }

// run build with
npm run build

// if webpack is installed globally than you can use directly
webpack # not recomended to use as global
~~~

Entry point is module for which webpack start building out dependency graph and
create bundles.
Output is where to emit the bundles (`path` is `./dist`).
Loader is used to process webpack modules (files that use `import`, `require()`,
`@import`, `url()`) and use coffescript, typescript, sass processors. It has
`test` property (which files should be transformed) and `use` which loader.
Plugins are use to perform wider range of tasks like uglify.
`./dist/` is default output path folder, and you can open `gnome-open
dist/index.html`


~~~
// webpack.config.js
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
~~~

Asset Management with loaders (`npm install --save-dev style-loader`)

*  `style-loader` will in javascript create and attach to head as inline `<style
  type="text/css">.my-style-from-file{}</style>`. It needs to used in js `import
  './my_style.css'` to define which file will be imported.

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

https://github.com/jantimon/html-webpack-plugin
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

Add `devtool: 'inline-source-map',` to provide source maps


https://webpack.js.org/guides/output-management/
/home/orlovic/rails_temp/webpack_guide

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

From sprockets to webpacker

https://medium.com/@coorasse/goodbye-sprockets-welcome-webpacker-3-0-ff877fb8fa79

# Parcel

Install with `yarn global add parcel-bundler'
