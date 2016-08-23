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

echo 'source "https://rubygems.org"
gem "jekyll"
gem "guard"
gem "guard-livereload"' > Gemfile
bundle

echo '# A samle Guardfile
guard "livereload" do
  watch(%r{_site/.+})
end' > Guardfile

git add . && git commit -m "Adding livereload"
jekyll serve
guard
~~~

For livereload you need to install [Chrome plugin](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei/related?hl=en), enable it and activate when you open a page. Or you can use javascript version.

# Liquid tips

* to show `{{ '{{ ' }} }}` using *markdown* you need to escape them like `{{ "{{ '{{ " }} ' }} }}` (first closing brackets will close everything than comes after first opening brackets - if there are no errors,
all closing brackets are simply rendered)
* similar to show `{{ '{% bla ' }} %}` you need `{{ "{{ '{% bla '" }} }} %}`
* liquid `prepend: ` works only with double curly brackets, so for link to
  another post (by it's file name) when `site.baseurl` is set, [you need](https://github.com/jekyll/jekyll/issues/3708) `[my-post]{{'('}}{{ '{{  site.baseurl ' }} }}{{ '{% post_url 2015-12-20-my-post ' }} %})`

# Serve under subfolder

Since github pages serve your project under subfolder `username.github.io/project-name`
you need to use special [config variable](http://jekyllrb.com/docs/configuration/) `baseurl`.
Add to your `_config.yml` a line `baseurl: /blog`. This is not needed if you use
custom domain like [blog.trk.in.rs](blog.trk.in.rs)

To properly serve your assets and link your pages you need to prepend all links with *site.baseurl*. For example in markdown `![My picture]({{ "{{site.baseurl" }}}}/assets/my_picture.png)` or `<link rel="stylesheet" href="{{ "{{site.baseurl" }}}}/assets/css/main.css">` or `<a class="post-link" href="{{ "{{ post.url | prepend: site.baseurl " }} }}">{{ "{{ post.title " }} }}</a>`

# Development environment

If you need to separate production from development (for example analytics), you can use liquid variable `jekyll.environment` which can be exported or used inline with `JEKYLL_ENV=development jekyll serve --watch`

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
echo "www.trk.in.rs" > CNAME
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


If you need automatic Toc than use [jekyll-toc-generator](https://github.com/dafi/jekyll-toc-generator). I put [defaults](http://jekyllrb.com/docs/configuration/#front-matter-defaults) to not have a Toc, and enable on post that I want Toc to show.

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

# Categories

You can put your posts inside folder, for examle `sports/_posts`. This was all
posts will be assigned with `sports` category. You can add more categories using
`categories: sprint water` front matter.

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

# Markdown

* [table](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#tables)
  can be generated with `|header1|header2`
* url can be written like `<http://projektor.trk.in.rs>` (`http://` at the
  beggining is mandatory) instead of writting it twice
  `[http://projektor.trk.in.rs](http://projektor.trk.in.rs)`
* internal link to post is with `{{ '{%' }} %}` like
  `[My page]({{ '{% 2016-03-02-my-page.markdown' }} %})`
* images `![alt text]({{ site.base_url }}/assets/path_to_image "Title text")`
* strikethrough (words crossed, deleted, removed text) can be used with
  `<s>Example</s>` or `<del>Example</del>` (with kramdown no shortcut works
  like `-- ~~`)
