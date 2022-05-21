---
layout: post
title:  Jekyll tips for github pages
tags: jekyll github rakefile
---

# Installation

Here are the steps you can follow to generate myblog

```
gem install jekyll
jekyll new myblog
cd myblog
git init .
git add .
git commit -m "Initial jekyll new myblog"
```

Add usefull gems
```
cat >> Gemfile << 'HERE_DOC'
gem 'jekyll-seo-tag'
gem 'jekyll-sitemap'
HERE_DOR
```

Live reload is now built in (no need for `guard-livereload`). You just need to
run with option `jekyll serve --livereload`

For livereload you need to install [Chrome
plugin](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei/related?hl=en)
enable it and activate by clicking on icon when you open a page.
There is also plugin for firefox https://addons.mozilla.org/en-US/firefox/addon/livereload-web-extension/
You can use javascript version.

# Custom domain

If your repo is the only or is primary, you can use <duleorlovic.github.io> url
you just need to rename repository to that name.

Do not need to rename if you want custom domain, you can  enable it from
settings <https://github.com/duleorlovic/trk.in.rs/settings> or you can just
create CNAME file manually since that is the same

~~~
git chechout -t origin/gh-pages
echo "www.trk.in.rs" > CNAME
~~~

You need to configure your domain name provider to point to your github
respository with [CNAME
record](https://help.github.com/articles/setting-up-a-www-subdomain/)
to point to `{username}.github.io`, for example `CNAME duleorlovic.github.io`.
Check when it is updated, usually in one hour

```
# note that second column is TTL time to live in secods
dig +nostats +nocomments +nocmd www.trk.in.rs
;www.trk.in.rs.			IN	A
www.trk.in.rs.		3600	IN	CNAME	kulakajak.github.io.
kulakajak.github.io.	3600	IN	A	185.199.111.153
kulakajak.github.io.	3600	IN	A	185.199.110.153
```

Use this only for subdomains (www.trk.in.rs, blog.trk.in.rs) and not for apex
domain. Adding CNAME record to the root of your domain will disable MX and TXT
records. If you need to apex domain (and not to destroy other records) you can
try with ANAME record (not supported in loopia), or use Cloudflare
[flattening](https://support.cloudflare.com/hc/en-us/articles/200169056-CNAME-Flattening-RFC-compliant-support-for-CNAME-at-the-root)

Default `duleorlovic.github.io` is serverd under HTTPS, but custom domain is
serverd under HTTP. To enable HTTPS for custom domains you can try
[cloudflare](https://www.cloudflare.com/) but recently, all github pages are
enypted with free <https://letsencrypt.org/> certificate.

# Cloudflare

Adding CNAME on root domain is possible, it is calling CNAME Flattening.
You can use cloudflare and heroku to catch all subdomains to same herokuapp
https://devcenter.heroku.com/articles/custom-domains#add-a-wildcard-domain
https://support.cloudflare.com/hc/en-us/articles/360017421192%20#CloudflareDNSFAQ-DoesCloudflaresupportwildcardDNSentries
Add `*.myapp.com` (same as it is normal subdomain) to heroku, and use that CNAME
value on for cloudflare.

Note that only paid plans will use Cloudflare CDN... this subdomains will go
directly to herokuapp. For example if you add root and www and wildcard
subdomain you will see that root and www goes to Cloudflare IP addresses.
```
dig +noall +answer move-index.org
move-index.org.		300	IN	A	104.28.14.249
move-index.org.		300	IN	A	104.28.15.249

dig +noall +answer www.move-index.org
www.move-index.org.	300	IN	A	104.28.14.249
www.move-index.org.	300	IN	A	104.28.15.249
dig +noall +answer asd.move-index.org

asd.move-index.org.	300	IN	CNAME	vertical-radish-wbe8hucc40fc2vqud0b15mzk.herokudns.com.
vertical-radish-wbe8hucc40fc2vqud0b15mzk.herokudns.com.	60 IN A	52.2.175.150
vertical-radish-wbe8hucc40fc2vqud0b15mzk.herokudns.com.	60 IN A	52.21.103.149
```

# Cloudfront

```
dig  +nostats +nocomments +nocmd projektor.trk.in.rs
;projektor.trk.in.rs.		IN	A
projektor.trk.in.rs.	3405	IN	CNAME	d1ub4fmsp3scvp.cloudfront.net.
d1ub4fmsp3scvp.cloudfront.net. 47 IN	A	13.32.22.82
```

# Loopia

If you want redirection from non www to www, than use Forwatd -> Url redirect
(301 or 302) but uncheck "Synchronize the domain and subdomain www"
![loopia dns]({{ site.baseurl }}/assets/posts/loopia_dns.png "Loopia DNS")

# Jekyll on github

Easiest way to start a blog on github it to fork
<https://github.com/barryclark/jekyll-now> and rename it to
`duleorlovic.github.io` and start editing using <http://prose.io/>. There is
nothing special there, just simple jekyll project, with
[_includes](https://github.com/duleorlovic/jekyll-now/tree/master/_includes) for
discuss, analytics, meta and uses `gems: jekyll-sitemap` which github supports.
It does not use theme. It is very similar to default jekyll theme
[minima](https://github.com/jekyll/minima)


Jekyll default template minima can be overriden with
```
bundle show minima
# ~/.rvm/gems/ruby-2.5.1/gems/minima-2.5.0
mkdir _includes
cp `bundle show minima`/_includes/@(footer|header|head).html _includes/
git add .
git commit -am 'cp `bundle show minima`/_includes/@(footer|header|head).html _includes/'
```


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
    {{ '{% if jekyll.environment == "development"' }} %}
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

# Moving avaw from gh pages so we can see the access logs

https://belief-driven-design.com/moving-from-github-pages-to-self-hosted-ab1231fb7fa/

# Adding Table of Content

For README you can run command that extract all headers https://github.com/ekalinin/github-markdown-toc that you can copy and paste to README. Alternativelly you can insert

```
## Table of Contents
<!--ts-->
<!--te-->
```

and run
```
gh-md-toc --insert README.md
```

If you need automatic Toc for a any jekyll page than use
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
  they are descending order. To access most recent you can use loop `{ % for
  post in site.posts limit: 3 %}` or assign `{% assign post = site.posts.first
  %}`
  Inside loops you can `break` or `continue`. To filter based on specific
  property you can use `where`
  ```
  {% assign posts=site.posts | where:"lang", page.lang %}
  {% for post in posts %}
      <li>
          <a href="{{ post.url }}">{{ post.title }}</a>
      </li>
  {% endfor %}
  ```
* `site.pages` array of all pages `Jekyll::Page`


`{ { page }}` is an object with
* `page.content` preproccessed content
* `page.name` like `contact.md`
* `page.path` like `example/contact.md`
* `page.url` like `/example/contact.html`

+ all front matters variables like `page.layout`, `page.title`

if current page is `post` we have additional properties

* `page.title`
* `page.excerpt` it is first paragraph. if you need to limit you can use `{{
  post.excerpt | strip_html | truncate: 100 }}`. You can use specific delimiter
  on a post in frontmatter `excerpt_separator: <!--more-->` or in config (but
  than default "paragraph" excerpt will not be used, and you need to use
  separator on every post)
* `page.date`
* `page.id`
* `page.categories`
* `page.tags`
* `page.next` `page.previous` can be used to link to previous post
  ```
  {% if page.previous.url %}
    <a href="{{page.previous.url}}">&laquo;&nbsp;{{page.previous.title}}</a>
  {% endif %}
  ```

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

Yet another theme
<https://tympanus.net/codrops/2018/04/20/freebie-oasis-jekyll-website-template>

# Markdown

* [table](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#tables)
  can be generated with `|header1|header2`
* url can be written like `<http://projektor.trk.in.rs>` (`http://` at the
  beggining is mandatory) instead of writting it twice
  `[http://projektor.trk.in.rs](http://projektor.trk.in.rs)`
* internal link to post is with `{{ '{% link' }} %}` or `post_url`
  ```
  [My page]({ % link _posts/2016-03-02-my-page.md %})
  [My page]({ % post_url 2016-03-02-my-page %})
  ```
* images `![alt text]({ { site.baseurl }}/assets/path_to_image "Title text")` If
  you want to use image in `README.md` than you need to store image and use
  ```
  ![trk-datatables](test/trk_datatables_with_daterangepicker.png "TRK Datatables")
  ```

  This will work on github but not if README is shown on other sites
* strikethrough (words crossed, deleted, removed text) can be used with
  `<s>Example</s>` or `<del>Example</del>` (with kramdown no shortcut works
  like `-- ~~`)
* for code you can use `{{ '{%' }} highlight javascript  %}`, `{{ '{%' }}
highlight ruby %}`, `{{ '{%' }} highlight bash %}`
* use inline html in markdown if you use span level tags `<span>`,`<bold>`...
  for block level tags `<div>` you need to add blank line.

# Kramdown

* inline and block html elements
  ```
  <div class='pull-right'>
    Text right
  </div>
  {::options parse_block_html="true" /}
  <div>
  Use *markdown* for span elements
  </div>
  ```

* definition list dl dt dd
  ```
  term inside dt
  : definition inside dd
  ```
* block attributes are set with folloing IAL inline attribute list
  ```
  > A blockquote
  {: title="My title" .class1 #my-id}
  ```
* share attributes using ALD atribute list definition
  ```
  {:refdef: .my-class}
  paragraph
  {: refdef}
  ```
* span level elements using IAL inline attribute list
  ```
  This is *red*{: style='color: red'}.
  ```
* extension using ::, like `::comment`
  ```
  My 
  {::comment}
  this is ignored completely
  {:/comment}
  paragraph
  ```

# Liquid in markdown

* to show jekyll mustache inline you can add space, like `{ % bla_tag %}`. Note
  that in blocks of code you can use mustache, so this is only for inside tick.
* to show `{{ '{{ ' }} }}` using *markdown* you need to escape them like
`{{ "{{ '{{ " }} ' }} }}` first closing brackets will close everything that
comes after first opening brackets. All other closing brackets are simply
rendered. This does not work if that line breaks in mutliple lines (probably
string need + or something). To simplify I just put space between first
`{{ '{ {' }} }}`
* if you need to show **\`** than use backslash or put a space ` so it
wont be applied`

# Internal jekyll tags

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


For string manipulation you can use `replace` filters. For interpolation you
need to use `capture`
```
{% capture extension %}{{ page.lang| append: '.html' }}{% endcapture %}
/blog{{ page.url | replace: extension, 'sr.html' }}
```

Note that in liquid version 4 you can use `concat` filter but it is not
supported in jekyll 3.4 (liquid 3.0.6) [concat
example](https://help.shopify.com/themes/liquid/filters/array-filters#concat).
In old liquid you can use `append` but that is only for strings.
You should point to latest jekyll from github which uses Liquid 4

~~~
# get latest jekyll which uses liquid 4
gem "jekyll", github: 'jekyll/jekyll'
~~~

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
`assets/main.scss` and it needs to have double three lines `---`. From there you
can use `@import "file_from_scss";` to load partials from `_sass` folder

~~~
---
# this is assets/main.sass which imports other sass files
# NOTE that you can not import css file, it needs to be sass/scss
# import "file.css" # this will not be imported at compile time, but in browser
# import "file" # this will search for file.sass and import at compile time
---

@import "overrides";
~~~

~~~
<!-- override default template file _includes/head.html -->
...
  <link rel="stylesheet" href="{{ "/assets/main.css" | relative_url }}">
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

# Rails

https://www.sitepoint.com/jekyll-rails/
Example on https://github.com/trkin/premesti.se/tree/master/blog

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

# Slides

Google io slides can be found https://github.com/willnorris/io-slides
After you clone the repository

# Tips

<https://jekyllrb.com/docs/plugins/>
<https://github.com/planetjekyll/awesome-jekyll>
<https://github.com/planetjekyll/awesome-jekyll-plugins>

https://mademistakes.com/articles/using-jekyll-2016/

* date `{{ 'now' | date: "%Y" }}`
* add bootstrap
  ```
  yarn init
  yarn add bootstrap jquery popper.js

  # _config.yml
  sass:
    load_path:
      - _sass
      - node_module

  # css/main.sass
  
  ```

* debug slow rendering with
  ```
  jekyll s --profile --verbose --trace
  ```
  To render only last post
  ```
  jekyll s --watch --limit_posts 1
  ```
  Rebuild only that is changed
  ```
  jekyll s --incremental
  ```
* permalinks only support date and categories. To use author in permalink you
  need to use plugin https://gist.github.com/peey/897e8ed33e412fdfe0fcacf002acc150
* seo tags and Open Graph meta name tags are added using https://github.com/jekyll/jekyll-seo-tag
  
* jekyll with webpack
  https://github.com/sandoche/Jekyll-webpack-boilerplate
  To add boootstrap, run `npm add bootstrap jquery popper.js` and add

  ```
  // _src/index.js
  import 'jquery';
  import 'popper.js';
  import 'bootstrap';

  // _src/index.scss
  @import '~bootstrap/scss/bootstrap';
  ```
  For links you need to use relative url `<a href="{{ post.url |
  relative_url}}">{{ post.title }}</a>`

  I disabled minify_html so it runs faster on development
  https://github.com/sandoche/Jekyll-webpack-boilerplate/pull/26

  To rebuild you need to stop and start again `npm start`
* data is yml and you can access
  ```
  # _data/prices.yml
  kajak_jednosed: 800
  kajak_dvosed: 1200
  ```
  ```
  {{ site.data.prices.kajak_jednosed }}
  {{ site.data.prices['kajak_jednosed'] }}
  ```

  similarly you can translate
  ```
  # _data/t.yml
  title:
    en: Hi
    sr: Cao

  {{ site.data.t['title'][page.lang] }}
  ```

* code samples https://github.com/bwillis/jekyll-github-sample
  https://bwillis.github.io/2014/05/28/include-github-repo-code-in-jekyll/
* for error for ruby 3
  ```
  /home/orlovic/.rvm/gems/ruby-3.0.2/gems/jekyll-4.2.1/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)
  ```
  you just need to add gem
  ```
  bundle add webrick
  ```
