---
layout: post
title: Scraping using mechanize or selenium
---

Bot, spider, scraper.

# Kimurai

https://github.com/vifreefly/kimuraframework

```
require 'kimurai'

class SimpleSpider < Kimurai::Base
  @name = "simple_spider"
  @engine = :selenium_chrome
  @start_urls = ["https://example.com/"]

  def parse(response, url:, data: {})
  end
end

SimpleSpider.crawl!
```

Use console

```
kimurai console --url https://www.tripadvisor.rs/Hotel_Review-g295380-d1383552-Reviews-Prezident_Hotel-Novi_Sad_Vojvodina.html
response.css('script[type="application/ld+json"]').first
# in plains ruby
url = 'https://www.tripadvisor.rs/Hotel_Review-g295380-d1383552-Reviews-Prezident_Hotel-Novi_Sad_Vojvodina.html'
response = Nokogiri::HTML(open(url))

# use non headless, actually fire up a browser
HEADLESS=false kimurai console --engine selenium_chrome --url http://google.com
```

To install chromedriver you can use `gem install webdrivers` and symlink
```
sudo ln -s /home/orlovic/.webdrivers/chromedriver /usr/local/bin/

# or on production with rvm, check that is it in Gemfile (not in TEST)
# /home/deploy/.rbenv/versions/2.6.3/lib/ruby/gems/2.6.0/gems/webdrivers-4.1.3/lib/webdrivers/chromedriver.rb is just ruby file
bundle exec rake webdrivers:chromedriver:update
sudo ln -s /home/deploy/.webdrivers/chromedriver /usr/local/bin/
```

Use data to pass data between pages

```
def parse(response, url:, data: {})
  response.xpath().each do |product_url|
    request_to :parse_product, url: product_url[:href], data:
    data.merge(product_name: product_url[:title])
  end
end

def parse_product(response, url:, data: {})
  puts data[:product_name]
end
```

Save screenshot using `browser.save_screenshot`
Save cookies using `browser.driver.save_cookies`
Refresh page `browser.refresh` but to update `response` you need to `response =
browser.current_response`

# CSV, input & output

Your script probably needs some output, CSV is good enough (it will wrap inside
quotes if comma `,` is detected)

~~~
require 'csv'
CSV.open("candidates.csv","w") do |csv|
  csv << [id, name]
end

# or without indent

output = CSV.open('data/craiglist.csv', 'wb') # folder data must exists
output << [id, name]
output.close
~~~

For strings, you can use [mustache](https://github.com/mustache/mustache)

~~~
require 'mustache'
MESSAGE_TEXT = "Hi there,

I noticed your great page. Please see my profile here {{profile_url}}

Thanks!"
element.send_keys Mustache.render( MESSAGE_TEXT, profile_url: profile_url)
~~~

Params can be passed as arguments (first param is ARGV[0]) or hard coded

~~~
OUTPUT_FILE = ARGV[0] || 'output.csv'
TEST_MODE = true
SIMULATE_REAL_USER_DELAY = true

sleep rand(8..15) if SIMULATE_REAL_USER_DELAY
~~~

# Debug

~~~
# run with: ruby myscript.rb
# debug with byebug
# or in irb:
# driver=wait=id=nil # selenium
# agent=page=nil # mechanize
# eval File.open('myscript.rb').read
# put rubocop ready break somewhere in your loops:
# loop do
#   break if false != true # rubocop ready
# end
~~~

# Mechanize

If you have simple site without ajax than you can use
[Mechanize](http://docs.seattlerb.org/mechanize/GUIDE_rdoc.html). It uses
nokogiri and `automatically stores and sends cookies, follows redirects, and can
follow links and submit forms`. It provides
[forms_with](http://docs.seattlerb.org/mechanize/Mechanize/Page.html#method-i-forms_with)
so you can find forms (or links), fill input and submit.

~~~
require 'rubygems'
require 'mechanize'

agent = Mechanize.new
page = agent.get('trk-inovacije.com')
page.link_with text: 'Next' # exact match
page.search('#updates div a:first-child') # css match
~~~

For using plain selenium (not capybara) you need to implement waiting for ajax
results. I used three steps

~~~
element = nil
wait.until { element = driver.find_element(:name, 'UserName') }
element.send_keys "asdasd"
~~~

In this way, it is waiting for element to appear. In case it is not showing
exception `TimeOutError` so if you expect that, you need to wrap inside `begin
rescue end` block. Last line in `until` block (ie return value) is important,
since if it is false `TimeOutError` is raised.

~~~
require "selenium-webdriver"
# http://selenium.googlecode.com/git/docs/api/rb/Selenium/WebDriver.html
# http://docs.seleniumhq.org/docs/index.jsp

USER_EMAIL = "asd@asd.asd"
USER_PASSWORD = "asdasd"
TEST_MODE = false
SIMULATE_REAL_USER_DELAY = true

if driver.nil? # driver is defined if we use irb and eval File.open('f').read
  driver = Selenium::WebDriver.for :firefox
  # driver.manage.timeouts.implicit_wait = 10
  # do not use implicit wait since it can hang out
  wait = Selenium::WebDriver::Wait.new(timeout: 30) # seconds

  driver.navigate.to "https://trk.in.rs"

  puts "Signing in..."
  element = nil
  wait.until { element = driver.find_element(:name, 'UserName') }
  element.send_keys USER_EMAIL

  element = driver.find_element(:name, 'Password')
  element.send_keys USER_PASSWORD

  # puts "Please fill in reCAPTCHA... and click on login"
  # gets
  element.submit
end

puts "Finding profile_search..."
begin
  wait.until { element = driver.find_element(:xpath, '//*[text()[contains(.,"Profile")]]') }
rescue Selenium::WebDriver::Error::TimeOutError
  puts "Missing Profile link..."
  retry or break or next
end
unless TEST_MODE
  element.click
end
sleep rand(8..15) if SIMULATE_REAL_USER_DELAY
begin
  phone_element = driver.find_element :xpath, '//li[contains(text(),"☎")]'
  phone = phone_element.text[1..-1].strip
  puts phone
rescue Selenium::WebDriver::Error::NoSuchElementError
  # this error is raised when find_element is called outside of wait.until
  phone = nil
end
wait.until do
  begin
    driver.find_element(:xpath, "//*[@data-cid]").attribute('data-cid') != id
  rescue Selenium::WebDriver::Error::StaleElementReferenceError
    puts "old elemenet is no longer attached to the DOM"
    false
  end
end
begin
  driver.navigate.to link[:href]
rescue Net::ReadTimeout
  puts "timeout for #{link[:href]}"
  next
rescue Selenium::WebDriver::Error::UnhandledAlertError
  puts "UnhandledAlertError probably some model dialog on page"
  next
end
begin
  element.click
rescue Selenium::WebDriver::Error::ElementNotVisibleError
  puts "apply button hidden"
  next
end
driver.switch_to.frame driver.find_elements( :tag_name, 'iframe').last
~~~

# XPath

Xpath is nice since it can traverse back up the dom tree with `..` and that can
select element based on existence of a child `//p[a]` (all `p` with `a` child)

https://www.w3schools.com/xml/xpath_intro.asp
7 type of nodes: element, attribute, text, namespace, processing-instructions,
comment and document node. *Atomic values* are nodes with no children or parent.
Items are Atomic values or nodes.
Each element and attribute has one parent. Element nodes may have zero, one or
more children. Sibling nodes are nodes with same parent. Ancestors are node's
parent and parent's parent etc... Descendant are node's children, children's
children etc...
Selecting node by:
* `nodename` selects all nodes with the name nodename
* `/` selects from the root (if path starts with `/` than it is absolute path).
  `bookstore/book` selects all book elements that are children of bookstore
* `//` select nodes from the current node that match the selection no matter
  where they are. `bookstore//book` selects all book elements that are
  descentant from bookstore
* `.` selects the current node
* `..` selects parent of the current node
* `@` selects attributes `//@lang` selects all attributes that are named lang

Predicates are used to find specific node `[1]` or that contains specific value.
Predicates are always in square brackets
* `/bookstore/book[1]` first book that is child of bookstore
* `/bookstore/book[last()]` last book that is child of bookstore
* `/bookstore/book[position()<3]` first two book elements that are child of
  bookstore
* `//title[@lang]` selects all title elements that have attribute named lang
* `//title[@lang='en']` selects all title elements that have attribute lang with
  value en
* `/bookstore/book[price>35]/title` selects title elements of book element
  which have price element with value (inner text) greater than 35

If we use `/bookstore/book/price[text()]` than it will select all text from
price nodes.


Wildcards can be used to select unknown nodes
* `*` matches any element node `/bookstore/*` select all child elements of
  bookstore
* `@*` matches any attribute node. `//title[@*]` select title elements which
  have at least one attribute
* `node()` matches any kind of node

Several path can be selected using pipe `|` (or operator) `//title | //price`
select all title and price elements. You can use also `=` equal, `+` addition,
`div` division operators...

Axes can be used to traverse (in addition to simply child `/`)
* `child::book`, `descentant::`, `descentant-or-self::`
* `ancestor::`, `ancestor-or-self::`, `parent::`
* `attribute::`
* `namespace::`
* `following::`, `following-sibling::` after current node. To select text after
  certain element you can `//foo/following-sibling::text()[1]`
* `preceding::`, `preceding-sibling::` before current node except ancestors,
  attribute and namespace nodes. for example select li before li with text my
  `//li[text()='my']/preceding-sibling::li`
* `self::` current node

You can check in [developer
tools](http://stackoverflow.com/questions/22571267/how-to-verify-an-xpath-expression-in-chrome-developers-tool-or-firefoxs-firebug)
with `$x('//*[po-my-button]')` in console, or with *CTRL+f* search in elements
panel.
In ruby you can parse some text with nokogiri
http://cheat.errtheblog.com/s/nokogiri
Nokogiri cheat sheet https://github.com/sparklemotion/nokogiri/wiki/Cheat-sheet

```
doc = Nokogiri::HTML(html_page)
node = doc.at('some_xpath')
node = doc.at_css('h1')
node
```

* <http://www.w3.org/TR/xpath](http://www.w3.org/TR/xpath)>
* [usefull selectors](http://ejohn.org/blog/xpath-css-selectors)

* find by id  `//*[@id='my_id']` (note that it needs quotes inside
  squarebrackets)
* by class `//*a[contains(@class,'my_class')]`
* text `//*[contains(text(),'ABC')]` or `//*[text()='exact_match']`
* parrent `../`
* some child of this `.//`
* to get text without child nodes, call `text()` in xpath 
  `page.search('//h1/text()').text`
* get [input by
  label](http://stackoverflow.com/questions/34712495/protractor-select-a-form-element-using-label)
  `var input = element(by.xpath("//label[. = '" + labelName + "']/following-sibling::input"));`
* find all text with @, check if they look like an email and join them [link](http://stackoverflow.com/questions/3655549/xpath-containstext-some-string-doesnt-work-when-used-with-node-with-more)

~~~
# selenium example
email_text = driver.find_element(:xpath, '//*[@id="msg_container"]').find_elements(:xpath, ".//*[text()[contains(.,'@')]]").map { |e| e.text }.join(",")
# mechanize
email_text = page.search("//text()").map(&:text).join ','
# http://stackoverflow.com/questions/535644/find-email-addresses-in-large-data-stream
r = Regexp.new(/\b[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,4}\b/)
emails = email_text.scan(r).uniq
if emails.length > 1
  puts "Found several emails in body " +  emails.join(',')
end
user_email = emails.first.strip if emails.first
~~~

# Examples

## Search images and show target page in case of errors

~~~
class ImageService

  attr_accessor :agent

  def initialize
    @agent = Mechanize.new
  end

  def get_links(name)
    page = agent.get 'https://www.trk.in.rs/imghp'
    form = page.form('f')
    form.q = name
    page = agent.submit(form)

    # results
    table = page.search(".//table[@class='images_table']").first
    results = []
    table.search('a').each do |a|
      results.append( {
        image_url: a.search('img').first.attributes["src"].to_s,
        site_url: a.attributes["href"].to_s[7..-1], # /url?q=
      })
    end
    results
  rescue
    page.body.to_s
  end
end
~~~

~~~
<% if @results.class == Array %>
  <% results.each do |res| %>
    <%= res[:image_url] %>
  <% end %>
<% else %>
  <iframe id="FileFrame" src="about:blank"></iframe>
  <script type="text/javascript">
    var doc = document.getElementById('FileFrame').contentWindow.document;
    doc.open();
    doc.write('<%=raw @results %>');
    doc.close();
  </script>
<% end %>
~~~

* some sites provide nice rss feed, for example elance https://www.elance.com/php/search/main/resultsproject.php?matchType=project&rss=1&matchKeywords=rails+-php&statusFilter=10037&sortBy=timelistedSort&sortOrder=1

# Headless chrome on heroku

To run chrome on heroku you need chrome and chromedriver
<https://github.com/heroku/heroku-buildpack-google-chrome>
<https://github.com/heroku/heroku-buildpack-chromedriver>
After adding buildpacks you need to initialize Selenium with correct path to
google chrome

~~~
options = Selenium::WebDriver::Chrome::Options.new
options.add_argument('--headless')
if chrome_bin = ENV.fetch('GOOGLE_CHROME_SHIM', nil)
  options.binary = chrome_bin
end
driver = Selenium::WebDriver.for :chrome, options: options
~~~

One alternative solution, which does not rely on selenium
<https://github.com/yujiosaka/headless-chrome-crawler>

# Scraper

Find selectors using javascript https://github.com/cantino/selectorgadget

# Trk bot


