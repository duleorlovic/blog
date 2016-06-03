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
This gem follows [protocol](http://www.sitemaps.org/protocol.html)

~~~
echo "gem 'sitemap_generator'" >> Gemfile
bundle
rake sitemap:install # this will generate config/sitemap.rb
vi config/sitemap.rb # put some `add path, options`
rake sitemap:refresh:no_ping # to generate sitemap to public folder without ping
less public/sitemap.xml.gz
gunzip public/sitemap.xml.gz && chromium-browser public/sitemap.xml
~~~

Use `Post.find_each` instead of `Post.all.each` since we do not want to load all
posts in memory.

To run every day, you can use [whenever](https://github.com/javan/whenever) to
call `rake sitemap:refresh -s`

## `SitemapGenerator::Sitemap` options

* `Sitemap.default_host = 'http://example.com'` is your default url
* `Sitemap.create_index = true` always create index file
  `<sitemapindex><sitemap><loc>http://www.example.com/sitemap1.xml.gz</loc></sitemap></sitemapindex>`
  that will link to other sitemap files
  [definition](http://www.sitemaps.org/protocol.html#index)
* `Sitemap.sitemaps_host = 'http://s3.amazonaws.com/my-site/'` where we will
  deploy sitemaps. User with `Sitemap.adapter = `
  * Here `include_index` (include
  `<url><loc>http://www.example.com/sitemap.xml.gz</loc></url>`) be off since
  `sitemaps_host` is different than `default_host` and we can not have different
  domains in same sitemap
* `Sitemap.public_path` directory used to write localy before eventual upload
   (default is 'public/').
* `Sitemap.sitemaps_path = 'my-folder/'` is relative folder where to store
  sitemaps, it will be used to determine url in index file.

Options for
[add](https://github.com/kjvarga/sitemap_generator#supported-options-to-add)
base on [Sitemaps.org
protocol](http://www.sitemaps.org/protocol.html#xmlTagDefinitions) are:

  * `changefreq` default is `weekly`
  * `lastmod` default is `Time.now`
  * `priority` default is `0.5`
  * `expires` request removal `expires: Time.now + 1.day`

You can add some existing sitemaps with `add_to_index '/my-old-sitemap.xml.gz'`

## Heroku

For heroku you need to move it on
[S3](https://github.com/kjvarga/sitemap_generator/wiki/Generate-Sitemaps-on-read-only-filesystems-like-Heroku#configure-carrierwave).
It is straighforward configuration, AWS S3 does not need to be public.

~~~
SitemapGenerator::Sitemap.sitemaps_host = "http://"\
 "#{Rails.application.secrets.aws_bucket_name}"\
 ".s3.amazonaws.com/"
SitemapGenerator::Sitemap.sitemaps_path = 'my-sitemaps/'
SitemapGenerator::Sitemap.adapter = SitemapGenerator::S3Adapter.new(
  fog_provider: 'AWS',
  aws_access_key_id: Rails.application.secrets.aws_access_key_id,
  aws_secret_access_key: Rails.application.secrets.aws_secret_access_key,
  fog_directory: Rails.application.secrets.aws_bucket_name,
)
~~~

If your bucket is not on `us-east-1` region you need to add fog_region option to
adapter

~~~
  fog_region: Rails.application.secrets.aws_region
~~~

Than you need robots txt to point to that sitemap url (
`bucket-name`.s3.amazonaws.com/`filename`)

~~~
# public/robots.txt
# See http://www.robotstxt.org/robotstxt.html for documentation on how to use the robots.txt file
#
# To ban all spiders from the entire site uncomment the next two lines:
# User-agent: *
# Disallow: /
# Sitemap: http://www.example.com/sitemap.xml.gz
Sitemap: http://s3.amazonaws.com/my-site/my-folder/sitemap_index.xml.gz
~~~

Test with `heroku run sitemap:refresh`.

## Webmaster tools

Check your sitemap on [google
webmaster tools](https://www.google.com/webmasters/tools/home?hl=en).
Just click *Add property* and and verify your domain.
(For AWS S3 you need to enable website hosting).
Refresh robots.txt to point on new sitemap location.

From [sitemaps.org](http://www.sitemaps.org/protocol.html#index) you can see
that one sitemap should not have more than 50.000 links and uncompressed size
10MB.
Url should be less than 2048 chars.

You can use search to see current [index
status](https://support.google.com/webmasters/answer/35256)
Just type `info:www.my-site.com` and you can find links there.
Or you can add to https://www.google.rs/search?q=site:`www.my-site.com`

* site: search only for this domain or even subdirectory. You can use this to
  see if some page is indexed, just add uniq part of page url
* link: find links for specific domain or subdirectory or pages
* cache: see current archived copy of domain or page. It is better to use
  <https://www.google.com/webmasters/tools/googlebot-fetch> since google crawler
  can execute javascript. You can submit to index on the page, but even google
  renders it properly (with ajax/angular data) it will not add to index for some
  other reasons.
  
Some reasons could be
[post](https://moz.com/ugc/8-reasons-why-your-site-might-not-get-indexed)

  * server can not respond fast enough to all crawler requests, maybe it will
  help to reoder sitemap

All links in sitemap should return 200 (not redirection 3xx). Also you need to
check [this answer](https://support.google.com/webmasters/answer/2642366?hl=en)
for duplicates (this is common for angular since data is fetched after page
load), canonical url (this is common when you have search queries that return
same results).

If you have more domains on same server than you need to iterate for each
`default_host` and to call `SitemapGenerator::Sitemap.ping_search_engines`
(since by default is to ping when whole sitemap process finishes, so than it
will ping only last one).


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
