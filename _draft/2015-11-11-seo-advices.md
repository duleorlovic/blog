---
layout: post
title: SEO advices
---

Tips:

* Be sure that each page has a unique `<title>Nice page</title>`, 10-70 chars,
  with keywords
* also `<meta name="description" content="It's really nice page <%=
  @user.name if @user.present? %>">` should be 70-160 chars, uniq per page
* use one <h1> per page with keywords but do not duplicate title, use h2-h6 for
  other keywords.
* use alt atribute for images
* 25-70% treba da bude cist text a ne html tags
* add [XML sitemap](#sitemap) (with correct protocol http/s, subdomain,
  trailing slash)
* use 301 redirect from root to www domain (or vice versa) do not split value to
  two different domains (also from IP address)
* replace undrescore _ with dash (hyphens) - in urls
* flash and frames are not indexed
* nice mobile rendering (buttons at least 48px width/height, 32px padding around tap targets)
* add viewport meta tag `<meta name="viewport" content="initial-scale=1.0, width=device-width"> and use CSS media queries to apply different style depending of screen size
* enable gzip compresion, optimize image, eliminate render-blocking javascript and css, leverage browser caching, minify css, to have small page size (<320Kb) and load under one second
* add favicon
* custom 404 error page
* add language `<html lang="en">`
* SSL secure (http should redirect to https), add STS in header, update robots, xml sitemap, link to css files to use https
* social media

# Rails gzip

It should show in Response Headers:

Content-Encoding:gzip

~~~
# Gemfile
gem 'heroku-deflater', :group => :production

# config/environments/production.rb
  config.assets.compress = true
~~~


# Meta tags

* add `<meta name="robots" content="noodp,noydir">` to disable Open Directory 
  Project and Yahoo Directory to provide description for your site


# Sitemap

[sitemap_generator](https://github.com/kjvarga/sitemap_generator) gem is
excellent tool.

~~~
echo "gem 'sitemap_generator'" >> Gemfile
bundle
rake sitemap:install
vi config/sitemap.rb
~~~

For heroku you need to move it on
[S3](https://github.com/kjvarga/sitemap_generator/wiki/Generate-Sitemaps-on-read-only-filesystems-like-Heroku#configure-carrierwave).
It is straighforward configuration, AWS S3 host can be accesses in two ways

~~~
# with bucket name (some apps make problems when when they assume us regions)
SitemapGenerator::Sitemap.sitemaps_host = "http://"\
 "#{Rails.application.secrets.aws_bucket_name}"\
 ".s3.amazonaws.com/"
# with region (for example AWS_HOST_NAME=s3-ap-southeast-1.amazonaws.com)
SitemapGenerator::Sitemap.sitemaps_host = "http://"\
 "#{Rails.application.secrets.aws_host_name}/"\
 "#{Rails.application.secrets.aws_bucket_name}"
~~~

Than you need robots txt to point to that url.

Check your sitemap on [google
webmaster tools](https://www.google.com/webmasters/tools/home?hl=en)
Also validate your "#{Rails.application.secrets.aws_bucket_name}"\
 ".s3.amazonaws.com/"

If you have more domains on same server than you need to iterate for each
`default_host` and to call `SitemapGenerator::Sitemap.ping_search_engines`
(since by default is to ping when whole sitemap process finishes, so than it
will ping only last one).

~~~
# test locally
rake sitemap:refresh:no_ping
gunzip public/sitemap.xml.gz
# remote
heroku run sitemap:refresh
~~~

# Schema.org

You can follow <http://schema.org/docs/gs.html> to add microdata in your markup
like: `<div itemscope><h1 itemprop="name">Duke</h1></div>`

You can add head tags for facebook [open graph protocol](http://ogp.me/). In
Rails it will be like:

~~~
/ haml
- content_for :head do
  %meta{property: "og:type",        content: "article"}
  %meta{property: "og:title",       content: "MySite | #{@post.title}"}
  %meta{property: "og:image",       content: share_image_url(@post)}
  %meta{property: "og:description", content: @post.description}
  %meta{property: "og:url",         content: post_url(@post.id)}
  %meta{property: "og:site_name",   content: "MySite"}
  %meta{name: "twitter:card",       content: "photo"}
  %meta{name: "twitter:site",       content:"@mysite"}
  %meta{name: "twitter:title",      content: "MySite | #{@post.title}"}
  %meta{name: "twitter:image",      content: share_image_url(@post)}
  %meta{name: "twitter:url",        content: post_url(@post.id)}
~~~
