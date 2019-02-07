---
layout: post
---

Test your site on
<https://developers.google.com/speed/pagespeed/insights/>
Free trial seo report rank <https://www.woorank.com/>
Also on <https://www.webpagetest.org/>

# WEBPAGETEST

Videos https://www.youtube.com/watch?v=6UeRMMI_IzI&index=7&list=PLWa0Ky8nXQTaFXpT_YNvLElTEpHUyaZi4
Github https://github.com/WPO-Foundation/webpagetest
Docs https://sites.google.com/a/webpagetest.org/docs/advanced-features/webpagetest-restful-apis


# General Tips

* Be sure that each page has a unique title on each page `<title>Nice
page</title>`, 10-70 chars, with keywords
* also `<meta name="description" content="It's really nice page <%=
  @user.name if @user.present? %>">` should be 70-160 chars, uniq per page
* use one `<h1>` per page with keywords but do not duplicate title, use h2-h6
for other keywords.
* use alt atribute for description of images, but also image source link should
we unique and self explanatory
* Links within your content tend to carry more weight than links within a
sidebar or footer.
* try to have more and more inbound links from other sites. Increase link count
using: social media, directories, some sites links to competition, try to write
them explaining why your site is better. To find inbound links use google
webmaster tools, <https://moz.com/researchtools/ose/>
* if your content expires (like job posts, wheater data) than link old pages to
category pages (group of similar results)
* outbound links can be no-follow, especially outgoing links that are not
relevant (do not have quality content). For example, links to a Feedburner page.
* anchor text plays the most important role in link building. If you want to
rank for 'blue widget' then you want the anchor text of the link to be `<a>blue
widget</a>`
* 25-70% should be only text (not html markup tags)
* add [XML sitemap](#sitemap) (with correct protocol http/s, subdomain,
  trailing slash) if more than 100 than only most popular
* use 301 redirect from root to www domain (or vice versa) do not split value to
  two different domains (also from IP address)
* replace undrescore _ with dash (hyphens) - in urls
* flash and frames are not indexed
* nice mobile rendering (buttons at least 48px width/height, 32px padding around
  tap targets)
* add viewport meta tag `<meta name="viewport" content="initial-scale=1.0,
  width=device-width">` and use CSS media queries to apply different style
  depending of screen size
* enable gzip compresion, optimize image, eliminate render-blocking javascript
  and css, leverage browser caching, minify css, to have small page size
  (less than 320Kb) and load under one second
  * [google web
  fonts](https://fonts.googleapis.com/css?family=Montserrat|Open+Sans) are
  cached only for one day (each day, it is downloaded again). For
  different browsers it uses diffent files: woff2, svg ... Download fonts using
  [this heroku app](https://google-webfonts-helper.herokuapp.com/fonts). Change
  folder prefix to `poppins/`, and extract downloaded fonts (without folder) to
  `app/assets/fonts/poppins` so you can see the file
  `app/assets/fonts/poppins/poppins-v1-latin-regular.svg`. Copy/paste the
  content to some `app/assets/stylesheets/popins.scss` file and include in
  `applications.scss` with `@import 'popins';`

* add favicon. It is fine just to add to the root of web site
  (for rails just copy your png to `public/favicon.ico`) Size should be 32x32.
  Some of the best favicons is
  https://cdn.shopify.com/shopify-marketing_assets/static/shopify-favicon.png
  More info on
  https://en.wikipedia.org/wiki/Favicon#How_to_use
* custom 404 error page
* add language `<html lang="en">`
* SSL secure (http should redirect to https), add STS in header,
  xml sitemap, link to css files to use https
* social media
* optimize css https://csswizardry.com/2018/11/css-and-network-performance/

# Rails gzip

It should show in Response Headers:

Content-Encoding:gzip

~~~
# Gemfile
gem 'heroku-deflater', :group => :production

# config/environments/production.rb
  config.assets.compress = true
~~~


# Robots

https://en.wikipedia.org/wiki/Robots_exclusion_standard

`/robots.txt` is used to tell some good crawlers wheter they should read or not.
Bad crawlers will read the site anyway.

~~~
# public/robots.txt
# See http://www.robotstxt.org/robotstxt.html for documentation on how to use the robots.txt file
#
# To ban all spiders from the entire site uncomment the next two lines:
# User-agent: *
# Disallow: /
# Sitemap: http://www.example.com/sitemap.xml.gz
~~~

You can use meta tags to disable some well known robots. Add `<meta
name="robots" content="noodp,noydir">` to disable Open Directory Project and
Yahoo Directory to provide description for your site


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
Sitemap: http://s3.amazonaws.com/my-site/my-folder/sitemap_index.xml.gz
~~~

Test with `heroku run sitemap:refresh`.

Also interesting tool to check validity of sitemap is
https://www.websiteplanet.com/webtools/sitemap-validator/

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
* link: find links for specific domain or subdirectory or pages (this is no
  longer supported as of Jan 2017)
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

# Other tools

Chrome plugins

* <https://chrome.google.com/webstore/detail/browserstack-local/mfiddfehmfdojjfdpfngagldgaaafcfo?utm_source=chrome-app-launcher-info-dialog>
* <https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk?utm_source=chrome-app-launcher-info-dialog>

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

# Addwords

You can enable `Destination URL Auto-tagging. Automatically tag my ad
destination URLs` in Account Settings -> Preferences

# User mailing list

If someone is linking your site or your competitor site, you can email the site
owner (need a script that will search for email address on that site) saying
that you have some new content, video or other material that will be interesting
for his audience.

# Customer Analitics

You can use some of the services: Google Analytics, Firebase anal, Amazon anal,
Fabric Answers, Mixpanel, Keen, Segment, Amplitude, Localytics, Ionic Analytics

# SEO search enginge optimization

hangouts tutorials <https://plus.google.com/collection/oSHR9>

* subdomain is not bad for rank, since google can treat as two different sites
<https://www.youtube.com/watch?v=onNwqFa-e_s#t=2904>
* url is most important and should point to single state (not multiple states).
After hash `#` usually everythings is ignored so that should not be a router for
your app. Do not use secret stuff in url like `/session=1234/...?user=john`
* canonical url is the main key for this content, and can be defined using link
rel canonical, redirects, sitemaps
* imporant content that needs to be indexed should be on visible part of the
page, not on ajax call since crawl will be able to index that

TODO
https://www.youtube.com/watch?v=Afy7H04X9Us
