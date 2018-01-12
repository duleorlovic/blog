---
layout: post
title:  Jekyll tips for github pages
tags: jekyll github rakefile
---

# Installation

Here are the steps you can follow to generate myblog

~~~
gem install jekyll
jekyll new myblog
cd myblog
git init .
git add .
git commit -m "Initial jekyll new myblog"

cat >> Gemfile << HERE_DOC
gem "guard"
gem "guard-livereload"
HERE_DOC
bundle

cat >> Guardfile << HERE_DOC
# A samle Guardfile
guard "livereload" do
  watch(%r{_site/.+})
end
HERE_DOC

git add . && git commit -m "Adding livereload"
jekyll serve
guard
~~~

For livereload you need to install [Chrome
plugin](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei/related?hl=en)
enable it and activate by clicking on icon when you open a page. Or you can use
javascript version.

# Custom domain

If your repo is the only or is primary, you can use <duleorlovic.github.io> url
you just need to rename repository to that name.

Do not need to rename if you want custom domain, you can  enable it from
settings <https://github.com/duleorlovic/trk.in.rs/settings> or you can just
create CNAME file manually

~~~
git chechout -t origin/gh-pages
echo "www.trk.in.rs" > CNAME
~~~

You need to configure your domain name provider to point to your github
respository with [CNAME
record](https://help.github.com/articles/setting-up-a-www-subdomain/)

~~~
# CNAME duleorlovic.github.io
# check when it is updated, usually in one hour
dig www.trk.in.rs +nostats +nocomments +nocmd
# note that second column is TTL time to live in secods
~~~

Use this only for subdomains (www.trk.in.rs, blog.trk.in.rs) and not for apex
domain. Adding CNAME record to the root of your domain will disable MX and TXT
records. If you need to apex domain (and not to destroy other records) you can
try with ANAME record (not supported in loopia), or use Cloudflare
[flattening](https://support.cloudflare.com/hc/en-us/articles/200169056-CNAME-Flattening-RFC-compliant-support-for-CNAME-at-the-root)

Default `duleorlovic.github.io` is serverd under HTTPS, but custom domain is
serverd under HTTP. To enable HTTPS for custom domains tiy beed to use
[cloudflare](https://www.cloudflare.com/)

On cloudflare you can use universal SSL service, or you can sign up for free 90
days <https://letsencrypt.org/> certificate.

# Travis

This [gist](https://gist.github.com/domenic/ec8b0fc8ab45f39403dd) explain what
to do. In additional you need to use `bundle install && bundle exec jekyll build
-d out` as build command, use proper ruby supported by travis `rvm: - 2.2`,
exclude `vendor` from jekyll, and note the label string that `travis` cli
generate when you encrypt.

You need to generate keys which Travis will use to deploy. You need to write to
`deploy_key` and to not label from travis cli.

~~~
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" deploy_key
travis encrypt-file deploy_key
git add deploy_key.enc
mv deploy_key deploy_key.pub ~/.ssh/
~~~

~~~
cat >> deploy.sh << HERE_DOC
#!/bin/bash
set -e # Exit with nonzero exit code if anything fails

SOURCE_BRANCH="master"
TARGET_BRANCH="gh-pages"

function doCompile {
  bundle install
  bundle exec jekyll build -d out
}

# Pull requests and commits to other branches shouldn't try to deploy, just build to verify
if [ "$TRAVIS_PULL_REQUEST" != "false" -o "$TRAVIS_BRANCH" != "$SOURCE_BRANCH" ]; then
    echo "Skipping deploy; just doing a build."
    doCompile
    exit 0
fi

# Save some useful information
REPO=`git config remote.origin.url`
SSH_REPO=${REPO/https:\/\/github.com\//git@github.com:}
SHA=`git rev-parse --verify HEAD`

# Clone the existing gh-pages for this repo into out/
# Create a new empty branch if gh-pages doesn't exist yet (should only happen on first deply)
git clone $REPO out
cd out
git checkout $TARGET_BRANCH || git checkout --orphan $TARGET_BRANCH
cd ..

# Clean out existing contents
rm -rf out/**/* || exit 0

# Run our compile script
doCompile

# Now let's go have some fun with the cloned repo
cd out
git config user.name "Travis CI"
git config user.email "$COMMIT_AUTHOR_EMAIL"

# If there are no changes to the compiled out (e.g. this is a README update) then just bail.
if [ -z `git diff --exit-code` ]; then
    echo "No changes to the output on this push; exiting."
    exit 0
fi

# Commit the "changes", i.e. the new version.
# The delta will show diffs between new and old versions.
git add .
git commit -m "Deploy to GitHub Pages: ${SHA}"

# Get the deploy key by using Travis's stored variables to decrypt deploy_key.enc
ENCRYPTED_KEY_VAR="encrypted_${ENCRYPTION_LABEL}_key"
ENCRYPTED_IV_VAR="encrypted_${ENCRYPTION_LABEL}_iv"
ENCRYPTED_KEY=${!ENCRYPTED_KEY_VAR}
ENCRYPTED_IV=${!ENCRYPTED_IV_VAR}
openssl aes-256-cbc -K $ENCRYPTED_KEY -iv $ENCRYPTED_IV -in deploy_key.enc -out deploy_key -d
chmod 600 deploy_key
eval `ssh-agent -s`
ssh-add deploy_key

# Now that we're all set up, we can push.
git push $SSH_REPO $TARGET_BRANCH
HERE_DOC
~~~

~~~
cat >> .travis.yml << HERE_DOC
language: ruby
script: bash ./deploy.sh
rvm:
  - 2.2
env:
  global:
  - ENCRYPTION_LABEL: "find-label-in-output"
  - COMMIT_AUTHOR_EMAIL: "your-email@gmail.com"
HERE_DOC
~~~

~~~
cat >> _config.yml << HERE_DOC
exclude:
  - Gemfile
  - Gemfile.lock
  - vendor
  - deploy.sh
HERE_DOC
~~~

~~~
git add deploy.sh .travis.yml _config.yml deploy_key.enc
git commit -m "Adding travis deploy scripts"
~~~

Very usefull one line command when debugging travis

~~~
git add . && git commit --amend --no-edit && git push -f
~~~

You can edit online with <http://prose.io>. Make sure you are editing `master`
branch. Make sure github setting
<https://github.com/duleorlovic/blog/settings/branches>
default branch is master branch.

You can add prose related
[config](https://github.com/prose/prose/wiki/Prose-Configuration).
You can exclude files with `ignore: `.
You can add custom fields for some front matter with `metadata: _posts: ...`.
First key is folder name not the ruby class so if you have page or post inside
folder, than use folder name instead of `_posts`. 
Title is updated in header so do not need to define title metadata. Use `""` for
all files. Nice [example of
prose](https://github.com/ccppbrasil/ccppbrasil.github.io/blob/master/_config.yml).


# Heroku

There is heroku buildpack than can build jekyll for you and serve static site so
you can use any plugins.
https://blog.heroku.com/jekyll-on-heroku

# Serve under subfolder

Since github pages serve your project under subfolder
`username.github.io/project-name` you need to use special [config
variable](http://jekyllrb.com/docs/configuration/) `baseurl`.  Add to your
`_config.yml` a line `baseurl: /blog`. This is not needed if you use custom
domain like [blog.trk.in.rs](blog.trk.in.rs)

To properly serve your assets and link your pages you need to prepend all links
with *site.baseurl*. For example in markdown `![My picture]({ { site.baseurl
}}/assets/my_picture.png)` or `<link rel="stylesheet" href="{ { site.baseurl
}}/assets/css/main.css">` or `<a class="post-link" href="{ { post.url | prepend:
site.baseurl }}">{ { post.title }}</a>`

# Development environment

If you need to separate production from development (for example analytics), you
can use liquid variable `jekyll.environment` which can be exported or used
inline with `JEKYLL_ENV=development jekyll serve --watch`

~~~
<!-- index.html -->
<html>
  <body>
    {{ '{% if jekyll.envirnoment == "development"' }} %}
      Hello admin
    {{ "{% endif " }}%}
  </body>
</html>
~~~

# Automatic deploy to gh-pages using rake

Rake tasks are perfect for deploying since you need just to type `rake` and it
will be live (if you set up properly). This is my Rakefile that builds my blog
(stored on bitbucket) and push to github.
This is nice since I do not want to share history (all commits from the
beggining). On github, you can find only one commit "Site updated at ..."

~~~
git remote add origin git@bitbucket.org:duleorlovic/trk.in.rs.git
git remote add github git@github.com:duleorlovic/trk.in.rs.git
~~~

~~~
# Rakefile
#
# You need to set up git remote to github, for example:
#
# git remote add github git@github.com:duleorlovic/blog.git
#
# Require jekyll to compile the site.
require "jekyll"
require 'tmpdir'

task :default => "blog:publish"
# Github pages publishing.
namespace :blog do
  #
  # Because we are using 3rd party plugins for jekyll to manage the asset pipeline
  # and suchlike we are unable to just branch the code, we have to process the site
  # localy before pushing it to the branch to publish.
  #
  # We built this little rake task to help make that a little bit eaiser.
  #

  # Usaage:
  # bundle exec rake blog:publish
  desc "Publish blog to gh-pages"
  task :publish do
    # Compile the Jekyll site using the config.
    Jekyll::Site.new(Jekyll.configuration({
      "source"      => ".",
      "destination" => "_site",
      "config" => "_config.yml"
    })).process

    # Get the origin to which we are going to push the site.
    origin = `git config --get remote.github.url`

    # Make a temporary directory for the build before production release.
    # This will be torn down once the task is complete.
    Dir.mktmpdir do |tmp|
      # Copy accross our compiled _site directory.
      cp_r "_site/.", tmp

      # Switch in to the tmp dir.
      Dir.chdir tmp

      # Prepare all the content in the repo for deployment.
      system "git init" # Init the repo.
      system "git add . && git commit -m 'Site updated at #{Time.now.utc}'" # Add and commit all the files.

      # Add the origin remote for the parent repo to the tmp folder.
      system "git remote add origin #{origin}"

      # Push the files to the gh-pages branch, forcing an overwrite.
      system "git push origin master:refs/heads/gh-pages --force"
    end

    # Done.
  end
end
~~~


# Adding Table of Content

If you need automatic Toc than use
[dafi jekyll-toc-generator](https://github.com/dafi/jekyll-toc-generator).

I modify to use original id-s (that is hypernated version of header content) in
[duleorlovic/jekyll-toc-generator](https://github.com/duleorlovic/jekyll-toc-generator/commit/9c83b67e183600bb51dcea533aaa8eeeae4eb18a)

You can put
[defaults](http://jekyllrb.com/docs/configuration/#front-matter-defaults) to not
have a Toc, and enable on post that you want Toc to show.

~~~
# _config.yml
defaults:
  -
    scope:
      path: ""
      type: "posts"
    values:
      noToc: true
~~~

# Adding sitemap

~~~
echo "gem 'jekyll-sitemap'" >> Gemfile
bundle
echo "url: https://trk.in.rs
gems:
  - jekyll-sitemap
" >> _config.yml
~~~

Sitemap will be automatically generated with `jekyll serve`. Add `sitemap:
false` for pages that you don't want to appear in sitemap (like google site
verification).

# Jekyll variables and categories

Jekyll contains [variables](https://jekyllrb.com/docs/variables/): `site`,
`layout`, `content`, `paginator` and `page`.

`{ { site }}` is `Jekyll::Drops::SiteDrop` class and contains
* `site.posts` array of all posts objects `Jekyll::Document collection=post`
* `site.pages` array of all pages `Jekyll::Page`


`{ { page }}` is an object with
* `page.content` preproccessed content
* `page.name` like `contact.md`
* `page.path` like `example/contact.md`
* `page.url` like `/example/contact.html`

+ all front matters variables like `page.layout`, `page.title`

if current page is post we have additional properties

* `page.title`
* `page.excerpt`
* `page.date`
* `page.id`
* `page.categories`
* `page.tags`
* `page.next` `page.previous`

You can put your posts inside folder, for examle `kayak/canoe/_posts`. This
means that posts will have `['kayak', 'canoe']` categories. You can add more
categories using front matter `categories: sprint`.

Since pages do not have category, you cat mark them manually or check path, like
this

~~~
{ % capture page_category = page.path | split: '/' | first %}
~~~


To list similar posts you can  use this snippet in `_layout/default.html`

~~~
<style type='text/css'>
  .similar-links {
    display: inline-block;
    padding: 10px;
  }
</style>
{{ '{% for cat in page.categories' }} %}
  {{ '{% if site.categories.[cat].size != 1 ' }} %}
    <ul class="similar-links">
      <li>Look similar <b>{{ ' {{ cat' }} }}</b> tag:</li>
      {{ '{% for p in site.categories.[cat]' }} %}
        {{ '{% unless p.url == page.url ' }} %}
          <li>
            <a href="{{ '{{ p.url' }} }}">{{ '{{ p.title' }} }}</a>
          </li>
        {{ '{% endunless' }} %}
      {{ '{% endfor' }} %}
    </ul>
  {{ '{% endif ' }} %}
{{ '{% endfor' }} %}
~~~

# Search

Just copy and paste `search.json` and other snippets from
[Simple-Jekyll-Search](https://github.com/christian-fei/Simple-Jekyll-Search)

# 404

For gh-pages, you just need `404.html` at root for server to use it when it does
not find the page.

# Config

You can serve using different port, just add `port: 4001` to your `_config.yml`
file.

# Jekyll now on github

Easiest way to start a blog on github it to fork
<https://github.com/barryclark/jekyll-now> and rename it to
`duleorlovic.github.io` and start editing using <http://prose.io/>. There is
nothing special there, just simple jekyll project, with
[_includes](https://github.com/duleorlovic/jekyll-now/tree/master/_includes) for
discuss, analytics, meta and uses `gems: jekyll-sitemap` which github supports.
It does not use theme. It is very similar to default jekyll theme
[minima](https://github.com/jekyll/minima)

# Theme

If you want to [create
theme](https://jekyllrb.com/docs/themes/#creating-a-gem-based-theme) you can
follow this steps:

~~~
jekyll new-theme jekyll-theme-booster
vi jekyll-theme-booster.gemspec
# edit description and url
gem build jekyll-theme-booster.gemspec
gem install jekyll-theme-booster-0.1.0.gem
~~~

## Running example site

You can put posts inside root folder but than you do not know which file belongs
to theme and which is just example. So better is to use separated folder and use
`rake example` to run that site.

~~~
# Rakefile
# based on
# https://github.com/mmistakes/minimal-mistakes/blob/master/Rakefile
require "bundler/gem_tasks"
require "jekyll"
require "listen"

def listen_ignore_paths(base, options)
  [
    /_config\.ya?ml/,
    /_site/,
    /\.jekyll-metadata/
  ]
end

def listen_handler(base, options)
  site = Jekyll::Site.new(options)
  Jekyll::Command.process_site(site)
  proc do |modified, added, removed|
    t = Time.now
    c = modified + added + removed
    n = c.length
    relative_paths = c.map{ |p| Pathname.new(p).relative_path_from(base).to_s }
    print Jekyll.logger.message("Regenerating:", "#{relative_paths.join(", ")} changed... ")
    begin
      Jekyll::Command.process_site(site)
      puts "regenerated in #{Time.now - t} seconds."
    rescue => e
      puts "error:"
      Jekyll.logger.warn "Error:", e.message
      Jekyll.logger.warn "Error:", "Run jekyll build --trace for more information."
    end
  end
end

task :example do
  base = Pathname.new('.').expand_path
  options = {
    "source"        => base.join('example').to_s,
    "destination"   => base.join('example/_site').to_s,
    "force_polling" => false,
    "serving"       => true,
    "theme"         => "jekyll-theme-booster"
  }

  options = Jekyll.configuration(options)

  ENV["LISTEN_GEM_DEBUGGING"] = "1"
  listener = Listen.to(
    base.join("_includes"),
    base.join("_layouts"),
    base.join("_sass"),
    base.join("assets"),
    options["source"],
    :ignore => listen_ignore_paths(base, options),
    :force_polling => options['force_polling'],
    &(listen_handler(base, options))
  )

  begin
    listener.start
    Jekyll.logger.info "Auto-regeneration:", "enabled for '#{options["source"]}'"

    unless options['serving']
      trap("INT") do
        listener.stop
        puts "     Halting auto-regeneration."
        exit 0
      end

      loop { sleep 1000 }
    end
  rescue ThreadError
    # You pressed Ctrl-C, oh my!
  end

  Jekyll::Commands::Serve.process(options)
end
~~~

## Editing theme

Only files that are in repository are used for gem. gemspec uses `git ls-files`
and default gemspec is to inlude `assets`, `_includes`, `_layouts` and `_sass`.
You can see what is in gem with `gem open jekyll-theme-booster`.

Some files will not be used: `_config.yml`.

## Using theme

When you edit template, you need to build, install and restart your jekyll serve
proccess to load updated gem.

It is enough to specify
path as `<link rel="stylesheet" href="assets/css/animate.css">` but for relative
folder `<link rel="stylesheet" href="{ { "assets/css/animate.css" | relative_url }}">` (that is needed for all navigational links).

It is advised to be in same ruby version `rvm list` and `rvm gemset list`.


Another theme example is
[minimal-mistakes-jekyll](https://github.com/mmistakes/minimal-mistakes)

# Markdown

* [table](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#tables)
  can be generated with `|header1|header2`
* url can be written like `<http://projektor.trk.in.rs>` (`http://` at the
  beggining is mandatory) instead of writting it twice
  `[http://projektor.trk.in.rs](http://projektor.trk.in.rs)`
* internal link to post is with `{{ '{%' }} %}` like
  `[My page]({{ '{% link 2016-03-02-my-page.markdown' }} %})`
* images `![alt text]({ { site.base_url }}/assets/path_to_image "Title text")`
* strikethrough (words crossed, deleted, removed text) can be used with
  `<s>Example</s>` or `<del>Example</del>` (with kramdown no shortcut works
  like `-- ~~`)
* for code you can use `{{ '{%' }} highlight javascript  %}`, `{{ '{%' }}
highlight ruby %}`, `{{ '{%' }} highlight bash %}`


# Liquid in markdown

* to show `{{ '{{ ' }} }}` using *markdown* you need to escape them like
`{{ "{{ '{{ " }} ' }} }}` first closing brackets will close everything that
comes after first opening brackets. All other closing brackets are simply
rendered. This does not work if that line breaks in mutliple lines (probably
string need + or something). To simplify I just put space between first
`{{ '{ {' }} }}`
* similar to show `{ % bla_tag %}`
* if you need to show **\`** than use backslash or put a space ` so it
wont be applied`

# Liquid Tags

[Liquid for
designers](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers) and
<http://shopify.github.io/liquid/> are all docs you need.
<http://cheat.markdunkley.com/>

* `{ % assign my_var = [1, 2] %}` assigns some value to variable
* `{ % capture my_id %} name-{ { item.name | handleize }} { % endcapture %}`
  captures block text
* `{ % include some_file id="some_value" %}` include snippet, you can pass
arguments to include and use it inside like `{ % if include.id %} { % assign
target_page = page.[include.id] %} { % endif %}`
* `{ % comment %} my comment { % endcomment %}`
* `{ % if statement %} { % elsif false %} { % endif %}` where statement can be
  * comparison `==`, `!=`, `<=`
  * arrays `contains`
  * boolean operator `and` `or`
  * There is no negative, not `!` and there is no parentheses (use nested if)
* `{ % for item in array %} { % break %} { % continue %} { % endfor %}`
  * when iterating a hash `item[0]` is key and `item[1]` is value
  * iterating over ranges `{ % for i in (1..item.quantity) %}` or `{ % for
  member in ste.data.members %}`
  * helper variables inside loop `forloop.length`, `forloop.index0`,
  `forloop.last`
  * 3 optional arguments `{ % for item in array limit:2 offset:3 reversed %}`
  * you can use `{ % else %}` to show when array is empty
* `{ % cycle 'blue', 'white', 'red' %}` will repeat those colors, Usefull when
iterating for bootstrap row col

  ~~~
    {% for product in site.data.products %}
      {% cycle '<div class="row">', '', '' %}
        <div class="col-sm-4">
        ...
        </div>
      {% cycle '', '', '</div>' %}
    {% endfor %}
  ~~~

* `{ % case my_var %} { % when 'dule' or 'mile' %} { % else } { % endcase %}`

Internal jekyll tags

[link](https://jekyllrb.com/docs/templates/#links)
Note that you do not have to use quotes: `'` or `"`.

* `{ % post_url %}`. Note that you can not use string filters like `prepend: `
 inside tags (works only with double curly brackets `{ { }}`, so for link to
another post (by it's file name) when `site.baseurl` is set, [you
need](https://github.com/jekyll/jekyll/issues/3708) `[my-post]({ {
site.baseurl }}{ % post_url 2015-12-20-my-post %})`
* when you want to link assets you can use `{ { "/assets/style.css" |
relative_url }}`
* you can link to (link_to) pages also (not just posts) with `{ { site.baseurl
}}{ % link news/index.html %}`. Parameter for `link` should contain extension
(for example `.md`).

# Types:

* boolean, nil, string, integer, array and hash
  * integer can be incremented `{ % increment my_int %}` or `{ % decrement
  my_int %}`
  * range is defined similar to ruby `{ % for i in (1..my_int) %}`
  * to use sum operation, for example add two number you can use

  ~~~
  {% assign number_of_columns = 3 | minus: site.data.footer_links.size %}
  { % for i in (1..number_of_columns) %}
  ~~~

* array elements can be accessed only like `my_array[2]`
* hash elements can be accessed with `my_hash['name']` or `my_hash.name`
* you can call `my_array.size` or `my_hash.size`
* all yml files from `_data` folder will be available under
`site.data.file_name.item`

# Filters

Filters or pipes is used to process data inside `{ { }}`. Filter nam could be
followed with colon `:` to pass additinal params, for example `{ { page.path |
split: '/' | first | alert }}`

* [strings](https://help.shopify.com/themes/liquid/filters/string-filters)
  `append`, `prepend`, `capitalize`, `date`, `escape`, `lstrip`, `replace`,
  `strip_html`, `truncate`, `url_encode`
  * `{ { page.date| date: 'B %d, %Y' }}` or `{ { page.date | date_to_string }}`
  or `{ { page.date | date: '%d-%m-%Y]]`
* [arrays](https://help.shopify.com/themes/liquid/filters/array-filters)
`first`, `join`, `last`, `map`, `reverse`, `size`, `slice`, `uniq`
  * `map` uses string as argument (`" "` are required). Another example is
  [liquid github](https://shopify.github.io/liquid/filters/map/)
  * create with `{ % assign my_array = "ants, bugs, bees, bugs, ants" | split:
  ", " %}`

If you need to assign filter output to variable you can use `{ % capture
my_var %} { { var | my_filter }} { % endcapture %}` or use it inside 
assign tag `{ % assign all_categories = site.posts | map: "categories" %}`.

Note that in liquid version 4 you can use `concat` filter but it is not
supported in jekyll 3.4 (liquid 3.0.6) [concat
example](https://help.shopify.com/themes/liquid/filters/array-filters#concat).
In old liquid you can use `append` but that is only for strings.
You should point to latest jekyll from github which uses Liquid 4

~~~
# get latest jekyll which uses liquid 4
gem "jekyll", github: 'jekyll/jekyll'
~~~

# Debug

Debug liquid is simply output `{ { my_var | inspect }}`

Somehow if I use `contains_` variable name

~~~
{ % assign contains_sidebar = true %}
{ { contains_sidebar }}
~~~

than I got error like:

~~~
[:comparison, "contains"] is not a valid expression in "contains_sidebar ==
false" in /_layouts/page.html`
~~~

Show I rename variable `{% assign show_sidebar = true %}`

# Adding tag, filter

All [plugins](https://jekyllrb.com/docs/plugins/) will be loaded from `_plugins`
folder. Githib ignore this folder.

~~~
# _plugins/alert_tag.rb
module Jekyll
  module AlertFilter
    def alert(text)
      unescaped = if text.class == String
                  CGI.unescapeHTML text
               else
                 text.to_s
               end
      %(<script>
        alert('#{unescaped}');
        </script>)
    end
  end
end
Liquid::Template.register_filter(Jekyll::AlertFilter)
~~~

This is used to alert text `{ % alert this is text %}`

~~~
# _plugins/alert_filter.rb
module Jekyll
  module AlertFilter
    def alert(text)
      unescaped = if text.class == String
                  CGI.unescapeHTML text
               else
                 text.to_s
               end
      %(<script>
        alert('#{unescaped}');
        </script>)
    end
  end
end
Liquid::Template.register_filter(Jekyll::AlertFilter)
~~~

Filter is used like `{ { page | alert }}`

# Scss

For adding scss support, you need to put your main scss file inside for example
`css/main.scss` and it needs to have double three lines `---`. From there you
can use `@import "file_from_scss";` to load partials from `_sass` folder

~~~
---
# this is css/main.scss which imports other scss files
---

@import "overrides";
~~~

~~~
<!-- override default template file _includes/head.html -->
...
<link rel="stylesheet" href="css/main.css">
~~~

# Coffee script

To enable coffee script you need to add to config.yml

~~~
cat >> _config.yml << HERE_DOC
gems:
  - jekyll-coffeescript
HERE_DOC
~~~

And you coffee files need to have three lines `---` for example

~~~
---
# assets/js/cart.coffee
---
class Cart
  constructor: (@buttons, @cart) ->
    @buttons.click (event) =>
      @textContent = 'Added'
      do event.preventDefault
      itemContainer = event.target.parentNode.parentNode.parentNode
      itemNumber = do $ itemContainer
      .find 'h3.panel-title'
      .text

      listItem = do $ @cart
      .find 'li:last'
      .clone

      listItem.text itemNumber
      @cart.append listItem
      .hide()
      .fadeIn 'slow'


      console.log itemContainer

$ ->
  cart = new Cart $('form button'), $('ul#cart')
~~~

and they will be compiled, so you can include them `<script
src="assets/js/cart.js"></script>`

# Tips

* if you want to move page it is advised to leave old page some time so you do
not confuse crawlers. You can add redirection in html

~~~
<!-- narucivanje.html -->
<html>
  <head>
    <meta http-equiv="refresh" content="0; url=http://tasterkljuc.rs/naruci"/>
  </head>
  <body>
    You are redirected to http://tasterkljuc.rs/naruci
  </body>
</html>
~~~

<https://jekyllrb.com/docs/plugins/>
<https://github.com/planetjekyll/awesome-jekyll>
<https://github.com/planetjekyll/awesome-jekyll-plugins>

https://mademistakes.com/articles/using-jekyll-2016/

