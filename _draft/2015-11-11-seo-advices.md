https://www.woorank.com/

* Be sure that each page has a unique title, 10-70 chars, with keywords
* also for meta description 70-160 chars, uniq per page
* use one <h1> per page with keywords but do not duplicate title, use h2-h6 for other keywords
* use alt atribute for images
* 25-70% treba da bude cist text a ne html tags
* add XML sitemap (with correct protocol http/s, subdomain, trailing slash)
* use 301 redirect from root to www domain (or vice versa) do not split value to two different domains (also from IP address)
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

Rails gzip

It should show in Response Headers:

Content-Encoding:gzip

~~~
# Gemfile
gem 'heroku-deflater', :group => :production

# config/environments/production.rb
  config.assets.compress = true
~~~



