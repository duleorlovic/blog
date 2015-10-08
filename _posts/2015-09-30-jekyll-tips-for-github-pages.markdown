---
layout: post
title:  Jekyll tips for github pages
categories: jekyll github rakefile
noToc: false
---

After initial jekyll init

## Serve under subfolder

Since github pages serve your project under subfolder `username.github.io/project-name`
you need to use special [config variable](http://jekyllrb.com/docs/configuration/) `baseurl`.
Add to your `_config.yml` a line `baseurl: /blog`

To properly serve your assets and link your pages you need to prepend all links with *site.baseurl*. For example in markdown `![My picture]({{ "{{site.baseurl" }}}}/assets/my_picture.png)` or `<link rel="stylesheet" href="{{ "{{site.baseurl" }}}}/assets/css/main.css">` or `<a class="post-link" href="{{ "{{ post.url | prepend: site.baseurl " }} }}">{{ "{{ post.title " }} }}</a>`

## Development environment

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

## Automatic deploy using rake

Rake tasks are perfect for deploying since you need just to type `rake` and it will be live (if you set up properly). This is my Rakefile that builds my blog (stored on bitbucket) and push to github

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


## Adding Table of Content


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
