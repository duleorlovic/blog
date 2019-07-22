---
layout: post
---

# Theme customizations with ThemeKit

https://www.shopify.com/partners/blog/78118150-4-essential-tips-for-building-your-first-shopify-theme

Install with commands from <https://shopify.github.io/themekit/>. Create private
app for your store and add read/write access to Theme templates and theme
assets. Save password to `SHOPIFY_PRIVATE_APP_PASSWORD`. Find theme id with
```
theme get --list -p $SHOPIFY_PASSWORD -s $SHOPIFY_STORE_URL
```
and save to `SHOPIFY_THEME_ID`. Download theme with
```
theme get -p $SHOPIFY_PASSWORD -s $SHOPIFY_STORE_URL -t $SHOPIFY_THEME_ID
```
Or you can create new theme
```
theme new -p $SHOPIFY_PASSWORD -s $SHOPIFY_STORE_URL -n 'New Theme'
```

Other commands
~~~
theme download
theme deploy
theme open
theme watch
~~~

# Sections

Static sections can be included in a code, for example on cart page in
`templates/cart.liquid` we can insert `{ % section 'my-section' %}`.
Note that if you include same section on other pages, it will be the same
content.

~~~
# sections/my-section.liquid
<div id="my-section">
  <h1>{{ section.settings.header-id }}</h1>
  <h3>{{ section.settings.content-id }}</h3>
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
    ]
  }
{ % endschema %}

{ % stylesheet %}
{ % endstylesheet %}

{ % javascript %}
{ % endjavascript %}
~~~

Home page is different because it contains `content_for_index` which means that
it can use dynamic sections, ie sections that has additional property `presets`.
You can include section on home page (so you have both `content_for_index`
and `section 'my_section'` - this means it is static) but data for static is
single, and for dynamic it can be multiple. You can include on any page (not
needed to be home page) and it will show the same data.
Beside basic (with `settings`) and dynamic (`presets`), there is also a `block`
type of sections (example is
https://shopify.github.io/liquid-code-examples/example/homepage-quotes )
In schema you can define: name, class, tag, default, locales. Also:
* settings(array of id, type, label, default),
* blocks(array of name, type and settings), max_blocks,
* presets(array of category, name, settings and blocks).

~~~
{ % schema %}
  {
    "name" : "My Section",
    "settings": [
    ],
    "presets": [
      {
        "name": "Call to action",
        "category": "Call to action"
      }
    ]
  }
{ % endschema %}
~~~

We can have several types:
<https://help.shopify.com/themes/development/theme-editor/settings-schema#input-setting-types>
simple: text, richtext, image_picker, radio, select, checkbox, range
special: product, url, page, collection (it contains 'Edit collection' link) ...

# Adding custom properties

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

https://www.shopify.com/partners/blog/78118150-4-essential-tips-for-building-your-first-shopify-theme


# Liquid Tags

[Liquid for
designers](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers) and
<http://shopify.github.io/liquid/> are all docs you need.
<https://help.shopify.com/en/themes/liquid>
<http://cheat.markdunkley.com/>
<https://shopify.github.io/liquid-code-examples/>

## Tags

Tags are for programming logic (`{ % if ... assign ... %}`)
* `{ % assign my_var = [1, 2] %}` assigns some value to variable
* `{ % capture my_id %} name-{ { item.name | handleize }} { % endcapture %}`
  captures block text
* `{ % include some_file, my_variable: 'some_value' %}` include snippet, you can
  pass arguments to include and use it inside. Other way is to define variable
  in parent template `assign my_variable = 'some_value'`. Or you can use keyword
  `with` to assign local variable with the same name as section
  `{ % include 'color' with 'red' %}` so you can use `{{ color }}` inside
  section.
* `{ % comment %} my comment { % endcomment %}`

* `{ % if statement %} { % elsif false %} { % endif %}` where statement can be
  some of the operators
  * comparison `==`, `!=`, `<=` (`product.type == 'Shirt'`)
  * arrays and string `contains` (`product.tags contains 'outdoor'`)
  * boolean operator `and` `or` are evaluated from right to left. Note that
    there is no parenthesis so you need to use order (`true or false and false`
    is true) or nested `if` statements
  * There is no negative, nor `!` so you can use `unless` in this case.

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
* `{ % tablerow product in collection.products cols:2 limit:3 offset:2 %}`

## Objects

Objects are used to show some informations `{ { product.title }}`. You can
access object attributes using handles. Handle is created but not updated when
object is updated, so you can use `{ { pages.about-us.title }}` or `{ {
pages['about-us'].title }}`

global variables on shopify are:
* `all_products['short'].title`
* `articles['news/hello']`
* `blogs.myblog.articles`
* `cart`
* `collections.frontpage.products`
* `current_page` if available on paginated content
* `current_tags`
* `customer`
* `linklists` for example `for link in linklists.main-menu.links`, `link.url`,
  `link.active`, `link.title`, `link.links`
* `images`
* `pages`
* `page_description` description of product, page or collection that is rendered
* `page_title`
* `recommendations`
* `settings`
* `template` used like `<body class='{{ template }}'>`

There are three content objects that need to be included in layout files:
`content_for_header` and `content_for_layout` (in `theme.liquid` inside head and
body) and `content_for_index` (in `templates/index.liquid` which is landing page
and is used for dynamic sections which can be edited with theme editor).

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
  `strip_html`, `truncate`, `url_encode`
  * `{ { page.date| date: 'B %d, %Y' }}` or `{ { page.date | date_to_string }}`
  or `{ { page.date | date: '%d-%m-%Y]]`
* [arrays](https://help.shopify.com/themes/liquid/filters/array-filters)
`first`, `join`, `last`, `map`, `reverse`, `size`, `slice`, `uniq`
  * `map` uses string as argument (`" "` are required). Another example is
  [liquid github](https://shopify.github.io/liquid/filters/map/)
  * create array with `{ % assign my_array = "ants, bugs, bees, bugs, ants" |
  split: ", " %}`
* [url](https://help.shopify.com/en/themes/liquid/filters/url-filters#img_url)
  like `article | img_url: '400x300', scale: 2 | img_tag: article.title`

If you need to assign filter output to variable you can use `{ % capture
my_var %} { { var | my_filter }} { % endcapture %}` or use it inside 
assign tag `{ % assign all_categories = site.posts | map: "categories" %}`.

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

TODO
https://www.youtube.com/user/shopify/playlists
