---
layout: post
---

# Theme customizations with ThemeKit

https://www.shopify.com/partners/blog/78118150-4-essential-tips-for-building-your-first-shopify-theme
https://www.youtube.com/watch?v=1xWFsYmBoX0

Install with using instructions on <https://shopify.github.io/themekit/>.

On your store admin create private app (name could be `themekit`, email could be
your email) and in "Admin API" add `read/write access` to *Theme templates and
theme assets*. Save password to `SHOPIFY_PASSWORD` or in keys like
```
# https://business-casual-theme.myshopify.com/admin/apps/private/203168317509
export SHOPIFY_STORE_URL=business-casual-theme.myshopify.com
export SHOPIFY_API_KEY=123123
export SHOPIFY_PASSWORD=123123
# minimal theme
export SHOPIFY_THEME_ID=82209210448
```

Find theme id with
```
theme get --list -p $SHOPIFY_PASSWORD -s $SHOPIFY_STORE_URL
Available theme versions:
  [177706181][live] debut
```
and save 177706181 to `SHOPIFY_THEME_ID`. Download theme with
```
theme get -p $SHOPIFY_PASSWORD -s $SHOPIFY_STORE_URL -t $SHOPIFY_THEME_ID
# this will also create config.yml with password, theme_id and store url
# so you do not need to use params (and env variables)
```
or you can create new theme based on Timber.
```
theme new -p $SHOPIFY_PASSWORD -s $SHOPIFY_STORE_URL -n 'New Theme'
```
Timber template is outdated since it does not use sections or snippets so do not
use it. Better is to look ata debut (default) theme create file one by one.

Publish current theme_id from config.yml
```
theme publish
```
You can save some variables to `config.yml` (flags like -p will override config
values)
~~~
theme download # fetch all files (very slow)
theme deploy # remove destination, copy all files to destination
theme open # open in browser and show url
theme watch # deploy change when it occurs, enable hooks like LiveReload
~~~

Watching will show eventual errors in liquid tags, wrong schema in sections...
Enable live reload https://shopify.github.io/themekit/faq/ with browser sync (I
did not try prepros)
https://medium.com/@jamesauble/auto-refresh-for-shopify-development-using-browser-sync-470b84e7e9cb
Also this issue https://github.com/Shopify/slate/issues/1055#issuecomment-633019928
```
browser-sync start --proxy https://bootstrap-business-casual-theme.myshopify.com/ --files "*/*.*" --reload-delay 1000 --config bs-config.js
```

# Folder structure

Root index home page is in `templates/index.liquid`, assets goes to `assets`.
Example routes to template
* `/blogs/blog-name/article-id-handle` article.liquid (you can create another
  new template for example `template/article.video.liquid` so on admin you can
  choose one of thow two templates `article` and `article.video`)
* `/blogs/blog-name` blog.liquid
* `/cart` cart.liguid
* `/collections` list-collections.liquid
* `/collections/collection-hadle` `/collections/collection-handle/tag`
  collection.liquid
* `/` index.liquid
* `/pages/page-handle` page.liquid
* `/products` list-collections.liquid
* `/products/product-handle` product.liquid
* `/search?q=search-term` search.liquid
* `unknown` 404.liquid

Inside template we can include small snipipet `snippets/social-sharing.liquid`
and provide params `{ % include 'social-sharing', share_title: product.title %}`
If you need user to be able to customize snippet using browser than you need to
create section with `schema`. For example they can upload logo, update copy.
For new type of section files, instead of `include` we need to use `render` like
```
{ % render "shopify://apps/product-reviews/snippets/star-reviews/5fb1e11a-9ef0-4898-892f-3feba729af78" %}
```

`config/settings_data.json` and `config/settings_schema.json` are used for
global theme settings (not local section settings). For example, when you change
global background color or add dynamic section, or change some section value, it
will be saved to `config/settings_data.json`.


# Sections

Include section with `{ % section 'my-section' %}`.
https://www.youtube.com/watch?v=jzhsYMxUp8s
https://shopify.dev/tutorials/add-an-app-section-to-your-app note that sections
could be defined in apps, so merchant can use them without changing theme (theme
has to be sections-compatible).

~~~
# sections/my-section.liquid
<div id="my-section">
  <h1>{{ section.settings.header-id }}</h1>
  <h3>{{ section.settings.content-id }}</h3>
  {% for block in section.blocks %}
    {{ block.settings.image | img_url: '100x100' | img_tag: block.settings.image-name }}
  {% endfor %}
</div>
{ % schema %}
  {
    "name" : "My Section",
    "settings": [
      {
        "id": "header-id",
        "label": "Header text",
        "type": "text",
        "default": "Header text here"
      },
      {
        "id": "content-id",
        "label": "Content text",
        "type": "richtext",
        "default": "<p>Add here</p>"
      }
    ],
    "blocks": [
      {
        "name": "Images",
        "type": "image",
        "settings": [
          {
          "type": "image_picker",
          "id": "image",
          "label": "Your image"
          },
          {
          "type": "text",
          "id": "image-name"
          "label": "Name of image",
          }
        ]
      }
    ]
  }
{ % endschema %}

{ % stylesheet %}
{ % endstylesheet %}

{ % javascript %}
{ % endjavascript %}
~~~

Home page is different because it contains `{ { content_for_index }}` which
means that it can use presets sections (sections can be added, removed,
reordered dynamically by online editor). When you define `presets` than you can
include section on home page. So for example on home page you can have both ways
(one dynamically from `content_for_index` and one statically - hardcoded
`section 'my_section'`). Note that statically included section has global data,
for example on page1 and page2 section will look the same (if you update on
page1 it will be updated on page2 since it use the same global data).
Data for settings is single (can not contain nested settings), but for blocks
user can create more than one instance of settings properties.
Example of blocks and presets:
Glossary https://shopify.dev/docs/themes/sections#glossary-of-terms
Validate json shema https://jsonlint.com/

https://shopify.dev/tutorials/develop-theme-use-sections
In schema you can define properties of section:
* name: required, it is shown in editor sidebar
* class: additional css classes to `shopify-section` class
* tag: default is `<div>` it includes also the id and class, like
  `<div id="shopify-section-[id]" class="shopify-section">`
* locales: define translated words
* max_blocks: number of block items that user can create, same effect is done
  with `limit` attribute inside `"blocks": [ { "name": "1", "limit": 2 } ]`
  (limit is only usefull when you have several blocks with different max length)
* default: for sections that are statically included

* settings is array of: https://shopify.dev/docs/themes/settings
  * id
  * type
  * label
  * default (optional)
  * info (optional, use like a placeholer)
* blocks is array of: name, type(not sure what is this), settings (array of
  elements so when adding new item, it will contain all those elements)
* presets is array of: name (this is shown on editor sidebar when selecting new
  section), category (Look as sidebar for example categories case sensitive),
  settings (object, not array) it will be assigned to the section when user adds
  to a home page, it overrides default value `"settings": { "my-id": "Hi" }`,
  blocks (array of objects with `type` and eventual `settings` as object) this
  will assign default initial values for blocks, note that settings is object,
  not array
  https://shopify.github.io/liquid-code-examples/example/homepage-quotes
  ```
  <h1>{{ section.settings.quote-header }}</h1>
  <ul>
    {% for block in section.blocks %}
      <li>
        {{ block.settings.quote-text }}
      </li>
    {% endfor %}
  </ul>
  { % schema %}
    {
      "name": "Quotes",
      "settings": [
        {
          "id": "quote-header",
          "type": "text",
          "label": "Quote header label",
          "default": "Defalt value"
        }
      ],
      "blocks": [
        {
          "name": "My Quote",
          "type": "text",
          "settings": [
            {
              "type": "text",
              "id": "quote-text",
              "label": "My quote text"
            }
          ]
        }
      ],
      "presets": [
        {
          "name": "Quote preset name",
          "category": "Text",
          "settings": {
            "quote-header": "My example header"
          },
          "blocks": [
            { "type": "text", "settings": { "quote-text": "My first text" }},
            { "type": "text", "settings": { "quote-text": "My second text" }}
          ]
        }
      ]
    }
  { % endschema %}
  ```

We can have several types:
https://shopify.dev/docs/themes/settings#input-setting-types
simple:
* text, richtext
* image_picker (use it with `img_url` filter as `section.settings.my-image |
  img_url: "450x450"`).
* radio for example `"type": "radio", "id": "display_type", "label": "Select collections to show", "default": "all", "options": [ { "value": "all", "label": "All" }, { "value": "selected", "label": "Selected" } ]`
* select, checkbox, range
* header (this is just placeholder text on sidebar, it should have "content"
  attribute)
* link_list (picker for menus)

special:
* product it is just handler, so to access actual product you need to use `{%
  assign product = all_products[block.settings.product-id] %}`
* collection (it contains 'Edit collection' link) it is just handler, so to
  access actual collection you need to use
  `collections[block.settings.collection]`. Note that it could be empty so to
  skip those not selected collections you need to use
  ```
  {% for block in section.blocks %}
    {% if block.settings.collection == empty %}
      {% continue %}
    {% endif %}
    # do something with collections[block.settings.collection]
  ```
* url
* page

https://shopify.dev/tutorials/develop-theme-use-sections#javascript-and-stylesheet-tags
To pass custom properties to javascript, you can use
`data-slide-speed="{{ section.settings.speed }}"`

# Tips

Design tutorials youtube list https://www.youtube.com/playlist?list=PLlMkWQ65HlcEJMRRdnqxpbGImqBkIOctd
<https://shopify.github.io/liquid-code-examples/>

## Accessible pagination



## Adding cart custom properties

You need to pass `properties[name-of-property]` to the `/card/add`. If the
`name-of-property` starts with underscore `_` than it wont be shown on checkout.
You can use
<https://ui-elements-generator.myshopify.com/pages/line-item-property>
to generate something like

~~~
<p class="line-item-property__field">
  <label>Layout</label><br>
  <input required class="required" type="radio" name="properties[Layout]" value="left"> <span>left</span><br>
  <input required class="required" type="radio" name="properties[Layout]" value="right"> <span>right</span><br>
</p>
~~~


## Cart permalinks

To create checkout link (direct to give me money page). First you need to find
variant id using xml extension `admin/products/123.xml` than add to `/cart` path
comma separated list of variant-id:amount `/cart/123:1.456:2 ` (note that this
will override existing cart items)

# Liquid Tags

[Liquid for
designers](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers) and
<http://shopify.github.io/liquid/> are good for start.
<https://help.shopify.com/en/themes/liquid>
<http://cheat.markdunkley.com/> image size, product variables...

For new liquid features on shopify look for example at
https://shopify.dev/docs/themes/liquid/reference/filters/media-filters

## Tags

Tags are for programming logic
* `{ % assign my_var = [1, 2] %}` assigns some value to variable. instead of
  checking if something is not empty you can use default, for example
  `{%- assign featured_media = product.selected_or_first_available_variant.featured_media | default: product.featured_media -%}`
* `{ % capture my_id %} name-{ { item.name | handleize }} { % endcapture %}`
  captures block text
* `{ % include some_file, my_variable: 'some_value' %}` include snippet, you can
  pass arguments to include and use it inside. Other way is to define variable
  in parent template `assign my_variable = 'some_value'`. Or you can use keyword
  `with` to assign local variable with the same name as section
  `{ % include 'color' with 'red' %}` so you can use `{{ color }}` inside
  section.
* `{ % comment %} my comment { % endcomment %}`

* `{ % if statement %} { % elsif false %} { % endif %}` statement can be
  some of the operators
  * comparison `==`, `!=`, `<=` (`product.type == 'Shirt'`)
  * arrays or string `contains` (`product.tags contains 'outdoor'` or `unless
    image contains featured_image`)
  * boolean operator `and` `or` are evaluated from right to left. Note that
    there is no parenthesis so you need to use order (`true or false and false`
    is true) or nested `if` statements
  * There is no negative, nor `!` so you can use `unless` in this case.

* `{ % for item in array %} { % break %} { % continue %} { % endfor %}` for loop
  * when iterating a hash `item[0]` is key and `item[1]` is value
  * iterating over ranges `{ % for i in (1..item.quantity) %}` or `{ % for
  member in ste.data.members %}`
  * helper variables inside loop `forloop.length`, `forloop.index0`
    (`foorloop.first`) `forloop.last`
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
* `{ % tablerow product in collection.products cols:2 limit:3 offset:2 %}`

## Objects

Objects are used to show some informations `{ { product.title }}`. You can
access page using it's handle. Handle is automatically created based on title,
(space is replaced with dash, number is added if already exists). This two are
equivalent
```
{{ pages.about-us.title }}
{{ pages['about-us'].title }}
```
Product handle can be edited using 'Edit website SEO' on product page. Product
handle determines url ie path after `/products/product-handle`.

Global variables on shopify are:
* `all_products['short'].title`
* `articles['news/hello']`
* `blogs.myblog.articles`
* `cart`
* `collections.frontpage.products`
* `collection` inside collection template you can use `collection`
* `current_page` if available on paginated content
* `current_tags`
* `customer`
* `linklists` for example `for link in linklists.main-menu.links`, `link.url`,
  `link.active`, `link.title`, `link.links`
* `images`
* `pages`
* `page_description` description of product, page or collection that is rendered
* `page_title` usually the name of product, page or collection, can be
  overridden with page title in *Search enging listing preview* section
* `product` (in `template/product.liquid`)
  https://shopify.dev/docs/themes/liquid/reference/objects/product
  `product.selected_or_first_available_variant` first or based on `?variant`
  parameter
  `product.options_with_values` returns some json with "name" and "values"
  `[{"name":"Size","position":1,"values":["single","double","triple"]}]`
  `product.variants` returns array of variant objects (number_of_values ** per
  each option).
  `product.url` is path (domain is in `shop.url`)
* `recommendations`
* `settings` global settings object from `config/settings_schema.json`
* `template` name without .liquid, used like `<body class='{{ template }}'>`
* `handle` returns handle for blogs, articles, collections, pages and products
* `canonical_url` return url without parametars (like collection or variant
  selected)
* `shop`, for example `shop.url`

There are three content objects that need to be included in layout files:
`content_for_header` (inside `<head>` used for plugins) and `content_for_layout`
(similar to yield, in `theme.liquid` inside `<body>`) and `content_for_index`
(in `templates/index.liquid` which is landing page and is used for dynamic
sections which can be edited with theme editor).

You can create another `layout/second_layout.liquid` file and instruct template
to use it (for `templates/404.liquid` you can use another layout)
```
# templates/product.liquid
{ % layout "second_layout.liquid" %}
```

To include assets you can use filters. Note that you can not have subdirectories
subfolders under assets folder so all files should be inside same directory,
```
# layout/theme.liquid
  {{ 'application.scss.css' | asset_url | stylesheet_tag }}
  {{ 'application.js' | asset_url | script_tag }}

  # or if you need custom attribute like defer, you can use
  <script src="{{ 'theme.js' | asset_url }}" defer="defer"></script>
```

## Types:

* boolean, nil, string, number, array, hash and EmptyDrop
* number can be incremented `{ % increment my_int %}` or `{ % decrement
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
* emptydropt exists when you access non existing object
  `pages.this-handle-does-not-exists`
* to check if object exists you can use
  ```
  {% unless pages == empty %}
  {% endunless %}
  ```
  or for strings you can use `blank`
  ```
  {% unless settings.heading_text == blank %}
  {% endunless %}
  ```
* to remove empty line around command tags you can use minus `{ %- -%}`

# Filters

Filters or pipes is used to process data inside `{ { }}`. Filter nam could be
followed with colon `:` to pass additinal params, for example `{ { page.path |
split: '/' | first | alert }}`

* [strings](https://help.shopify.com/themes/liquid/filters/string-filters)
  `append`, `prepend`, `capitalize`, `date`, `escape`, `lstrip`, `replace`,
  `strip_html`, `truncate`, `url_encode`, `remove`, `remove_last`
  To create id or class from title (ie replace space with dash), you can use
  filter handleize or handle `id={ { page_title | handle }}`.
* `{ { page.date| date: 'B %d, %Y' }}` or `{ { page.date | date_to_string }}`
  or `{ { page.date | date: '%d-%m-%Y]]`
* [arrays](https://help.shopify.com/themes/liquid/filters/array-filters)
`first`, `join`, `last`, `map`, `reverse`, `size`, `slice`, `uniq`
  * `map` uses string as argument (`" "` are required). Another example is
  [liquid github](https://shopify.github.io/liquid/filters/map/)
  * create array with `{ % assign my_array = "ants, bugs, bees, bugs, ants" |
  split: ", " %}`
  * append to array in for loop you can use two approaches `capture` or `append`
    with a `split`.  Note that you can only create array of string. There is
    `concat` filter that joins two arrays, but you can not create array of
    object, you can create only array of strings.
  ```
  # first approach is using append (preferred)
  {% assign collections_string = "" %}
  {% for block in section.blocks %}
    {% assign collections_string = collections_string | append:
    collections[block.settings.collection].title | append: ";" %}
  {% endfor %}
  {% assign collections_arrays = collections_string | remove_last: ";" | split: ";" %}

  # another approach is using capture
  {% capture collections_string %}
    {% for block in section.blocks %}
      {{collections[block.settings.collection].title}};
    {% endfor %}
  {% endcapture %}
  {% assign collections_arrays = collections_string | remove_last: ";" | split: ";" %}
  ```
* [url](https://help.shopify.com/en/themes/liquid/filters/url-filters#img_url)
  like `article | img_url: '400x300' | img_tag: article.title` (if
  request "400x300" is smaller than original image's dimension, shopify will
  scale the image for you and serve that scalled image). For product img_url
  will use featured image. If no dimension is specified, it will be small
  100x100 so better is to use specific size (original image will preserve aspect
  ration when it is scalled down, shopify will never scale up image). You can
  also use crop and scale filter.
* money (convert cents to number and currency)

If you need to assign filter output to variable you can use `{ % capture
my_var %} { { var | my_filter }} { % endcapture %}` or use it inside 
assign tag `{ % assign all_categories = site.posts | map: "categories" %}`.

# Debug

Debug liquid is simply output `{ { my_var | inspect }}` or with objects you can
use `{ { product.variants | json }}`

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

Sho I rename variable `{ % assign show_sidebar = true %}`

# Localization

https://shopify.dev/tutorials/develop-theme-localization-use-translation-keys
i18n

Translated content is escaped by default (any html like `<` is converted to
equivalent) so if you need raw html than use capture
```
# theme.liquid
{% capture link %}
  <a href="https://support.mystore.com">{{ 'layout.header.support_link' | t }}</a>
{% endcapture %}
<h1>{{ 'layout.header.welcome' | t: link: link }}</h1>
```
```
# locales/en.default.json
{
  "layout": {
    "header": {
    "support_link": "support",
    "welcome": "Welcome to my store. Please contact {{ link }} should you
      need any assistance."
    }
  }
}
```

# Environment

You can switch theme id in `config.yml` so you can `theme download` or `theme
deploy`.

# Shopify app cli

Install based on instructions on https://shopify.github.io/shopify-app-cli/

```
eval "$(curl -sS https://raw.githubusercontent.com/Shopify/shopify-app-cli/master/install.sh)"
```

```
shopify create
shopify serve
```

but I receive an error https://github.com/Shopify/shopify-app-cli/issues/463
```
.shopify-app-cli/lib/shopify-cli/tasks/update_dashboard_urls.rb:12:in `call': undefined method `[]' for nil:NilClass (NoMethodError)
```
When I `shopify logout` than I receive
```
/home/orlovic/.shopify-app-cli/lib/shopify-cli/tasks/update_dashboard_urls.rb:27:in `check_application_url': undefined method `match' for nil:NilClass (NoMethodError)
```

Note that this is not the same as gem `shopify` (which does not include cli).

# Tutorials

Videos:
Shopify Partners https://www.youtube.com/channel/UCcYsEEKJtpxoO9T-keJZrEw
for example Getting Started with Shopfy Themes https://www.youtube.com/watch?v=-IGvYvL86_M

Partner academy:
https://partner-training.shopify.com/catalog
https://partner-training.shopify.com/outline/r5uxatsx/cover
https://partner-training.shopify.com/outline/uhxkymdg/cover

TODO:
https://www.shopify.com/partners/blog/the-essential-list-of-resources-for-shopify-app-development

