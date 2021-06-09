---
layout: post
---

Some tools useful in creating bot, spider, scraper.
https://github.com/lorien/awesome-web-scraping/blob/master/ruby.md

# Capybara

Some of the most used capybara methods
[link](https://gist.github.com/duleorlovic/042178b92f1badc09490) or [cheat
sheet](https://thoughtbot.com/upcase/test-driven-rails-resources/capybara.pdf)

**Session methods** [link](http://www.rubydoc.info/github/teamcapybara/capybara/master/Capybara/Session#visit-instance_method)
you can set expectation for `current_path` or `current_url`. In feature test
`page` is actually `Capybara::Session` class so better is to use `session` name.
* `visit "/"`, `visit new_project_path`. Remember that if you want to go to
  different domain than `Capybara.app_host` than you need to use full url (with
  protocol) so instead `visit 'www.google.com` use rather `visit
  'http://google.con'`. Any relative url will use `Capybara.app_host` just note
  that it also needs protocol `Capybara.app_host = 'http://...'` otherwise error
  `undefined method  +  for nil:NilClass`
* `within "#login-form" do`
* generate capybara POST request using `submit` (this does not work for `js:
  true`, so please use `js: false`)

  ~~~
    session = Capybara.current_session.driver
    session.submit :post, customer_sign_up_path, mac: mac
  ~~~

* scroll to the bottom of the page since since elements needs to be visible when
`js: true` you can use `page.execute_script "window.scrollBy(0,10000)"` or make
anchor and use hash url `url/#my-form` since execute script is not available
when not `js: true`. For pagination I prefer to use helper
```
  def scroll_down
    page.execute_script(<<-SCRIPT)
    var next = $('a.next_page[rel="next"]');
    var maxScrolls = 15;
    var myInterval = setInterval(function(){
      window.scrollBy(0,10000)
      next = $('a.next_page[rel="next"]');
      maxScrolls -= 1;
      if (next.length && maxScrolls > 0) {
        console.log(maxScrolls);
      } else {
        clearInterval(myInterval);
      }
    }, 200);
    SCRIPT
  end
```
* back button `page.driver.go_back`

**Node actions** are on session object. Argument is target element by their: id
(without `#`), name, label text, alt text, inner text
[more](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Actions)
Note that locator is case sensitive. You can NOT use css or xpath (this is
only for finders). You can use substring or you can define `exact: true`
* `click_on "Submit"` (both buttons and links) `click_button "Sign in"`,
`click_link "Menu"`. `click_on` is alias of `click_link_or_button`
https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Node/Actions#click_link_or_button-instance_method
which is similar to `click_link` but for union of `:link` and `:button`
selectors.
* `fill_in "email", with: 'asd@asd.asd'` locator is input name, id, test_id,
  placeholder, label text. Note that it is case sensitive.
  alternative is find set `find("input[name='cc']").set 'asd@asd.asd'`, or using
  javascript `page.execute_script "$('#my-id').val('asd@asd.asd')"`
* `check 'my checkbox'` (or by id but without # `check 'my_check_id'`), `choose
 'my radio button'`, `select 'My Option or Value', from: 'My Select Box'`, also
 uncheck `uncheck 'my checkbox'`, `unselect`
* `page.attach_file 'doc[file]', "#{Rails.root}/test/fixtures/files/computer_text.png", make_visible: true`
* If you need to fill_in iframe than you can access it by id or number
~~~
  within_frame 0 do
  end
~~~

* If you need to switch window ie jump into new tab opened by target `_blank`
  than you can
~~~
old_window = page.driver.browser.window_handles.last
new_window = window_opened_by { click_link 'Something' }

page.within_window new_window do
  # code
end

# or
page.switch_to_window new_window
# page.current_window.close
page.driver.browser.close # this will close tab, not whole window
page.switch_to_window old_window
~~~

* Confirm alert dialog box in
  * selenium `page.driver.browser.switch_to.alert.accept  # can also be .dismiss`
  * webkit `page.accept_confirm { click_link "x" } }` so actions is wrapped with
this `page.accept_confirm`
* When you find some element you get `Capybara::Node::Element`, but you can
  create new node without going to selenium, using html text
~~~
node = Capybara.string <<-HTML
  <ul>
    <li class='logo'><a href='/' title='go_to_home'>Home</a></li>
    <li id="projects">Projects</li>
    <li data-test='another'><a href='#' title='Home'>Another link to home</a></li>
  </ul>
HTML
node.class # => Capybara::Node::Simple
node.find('#projects').text # => 'Projects'
# is actually
node.find(:css, '#projects').text # => 'Projects'
node.find(:link_or_button, '.logo') # Capybara::ElementNotFound (Unable to find link or button ".logo")
~~~


**Node finders**
[find](http://www.rubydoc.info/github/jnicklas/capybara/Capybara/Node/Finders#find-instance_method)
use selector with params `:kind` (optional defaults to `:css`), `locator` and
filters (some elements has more filters for example `input` has `type`).
https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Selector
Note that `find(:link, 'Home')` will match two elements in above example, while
`find('a', text: 'Home')` will match only one, since first string parameter
`locator` beside text link, matches id, name, value, title, alt, label.
`text_id` attribute). So first symbol parameter is type (by default is `:css`,
and locator is CSS selector). If not `:css` or `:xpath` it matches name, label..
I prefer to enable aria labels and use that also
`Capybara.enable_aria_label = true` and `click 'my-aria-label'`

* `find 'th', text: 'Total Customers'`
* `find('ng-model="newExpense.amount"').set('123')`
* `find_all('input').first.set(123)` but I think it is better to use
  `find('input', match: :first)` since it do not need to find all, similarly you
  if there are many buttons, you can use `click_on 'Edit', match: :first`
* `find('[data-test="id"]', visible: false)` to find invisible element
* `find('#selector').find(:xpath, '..')` find parent node of selector
  `.find(:xpath, '../..')` is parent of parent (grandparent).
* label of child of next adjacent
  `<h3>Name2</h3><div><label>enabled</label><label>disabled></div>` is
  ```
  find(:xpath, "//h3[contains(text(),'Name2')]/following-sibling::div/label[contains(text(),'enabled')]")
  ```
* to click on select2 I use `find('#original-select-id+span').click` so we find
  first next sibling of original select which was disabled and replaced by
  select2 spans. Also works `find('li', text: select.label).click` but I can not
  find that `li` in dom.
* Node element [more](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Element) `find('input').trigger('focus')` (does not work in selenium)


**Node matchers** and rspec matchers [more](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Matchers) [rspecmatchers](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/RSpecMatchers)

* `expect(page.has_css?('.asd')).to be true`
* `expect(page).to have_css(".title", text: "my title")`, `have_text /hi|bye/`,
`have_content`, `have_link`, `have_button`, `have_selector("#project_#{project.id} .name",
text: 'duke')` .
`have_no_selector` for opposite. It is not same `expect(page).not_to have_text`
and `expect(page).to have_no_text` since in later case it will wait until it
tries to fulfill expectation.
With all you can use `text: '...'` and `count: 2` which is number of occurences.
Instead of `page.body.include? text` (this will compare html tags) use
`page.has_text? text` (this will compare only text).
For testing if something is on the page, for example 3 rows, you can use
`page.has_selector? '[name="customer_ids[]"]', count: 3`
* If element is not visible, you can provide `visible: false` (does not work
with `have_content "d", visible: false` but works with `have_css 'div', text:
'd', visible: false`. Note that this is triggered only if `js: true` (and pass
if `js: false`) so it is better to allways use visible: false for some elements
in popups.
* test sort order is with regex `expect(page).to have_text
/first.*second.*third/`  create three objects with first in the middle. If you
have new lines, than you can match multiline `/first.*second.*third/m` or
replace `page.body.gsub "\n", ''`

* test if input has value:
  * `expect(page).to have_xpath("//input[@value='John']")`
  * `expect(page).to have_selector("input[value='John']")`. To match disabled
    option you can use `expect(page).to have_selector(:option, 'Name of o',
    disabled: true)`
  * `expect(page).to have_field('Your name', with: 'John')` this does not match
  disabled input field. If you want to match disabled use `have_field('Your
  name', disabled: true, with: 'John')`


## Debug

Debug capybara
* `save_and_open_page` to visually inspect the page. It works when `js: false`
and uses `lunchy` gem. It does not load images with relative path (images on
your server).
* <https://github.com/mattheworiordan/capybara-screenshot> screen shots and
html are saved in `tmp/capybara`. You you use `chrome` or another driver that is
not `selenium` than register with
<https://github.com/mattheworiordan/capybara-screenshot/issues/84>
<https://github.com/mattheworiordan/capybara-screenshot/blob/master/lib/capybara-screenshot.rb#L159>

Also if you use system test you will see screenshot in terminal. to disable you
can `export RAILS_SYSTEM_TESTING_SCREENSHOT=simple` or set ENV
https://github.com/rails/rails/blob/5-1-stable/actionpack/lib/action_dispatch/system_testing/test_helpers/screenshot_helper.rb#L58

~~~
# spec/support/capybara_screenshot.rb
ENV["RAILS_SYSTEM_TESTING_SCREENSHOT"] = 'simple'
Capybara::Screenshot.register_driver(:chrome) do |driver, path|
  driver.browser.save_screenshot(path)
end
Capybara::Screenshot.register_driver(:headless_chrome) do |driver, path|
  driver.browser.save_screenshot(path)
end
# after Saver#save_html
Capybara::Screenshot.after_save_html do |path|
  $stderr.write('Press ENTER to continue') && $stdin.gets
end
# after Saver#save_screenshot
Capybara::Screenshot.after_save_screenshot do |path|
  Launchy.open path
end
~~~

You can create gifs from test in two steps. First capture all pages that you are
interested with manual screenshot `screenshot_and_save_page`, review them and
rename last one to `final.png` and than create animated gif
with a http://www.imagemagick.org/Usage/anim_basics

~~~
convert -delay 50 -loop 0 tmp/capybara/m/screenshot_2018-*.png -delay 400 tmp/capybara/m/final.png animated.gif
~~~

## Waiting ajax

<https://github.com/teamcapybara/capybara#asynchronous-javascript-ajax-and-friends>
capybara is smart enough to wait if some ajax is called and text is not found.
So it will retry (default_max_wait_time=2) untill failure is not raised. Note
that `!page.has_xpath?('a')` is not the same as `page.has_xpath?('a')` in
example where you are removing `a` in ajax. First will fail since it find `a`
negate (it does not wait when capybara is success). Second will wait until it
is removed. So use expectations which are going to be met until after ajax.

Wait for ajax to finish is not needed in latest capybara, but here is reference:

~~~
# spec/support/features/wait_helper.rb
# https://robots.thoughtbot.com/automatically-wait-for-ajax-with-capybara

module WaitHelper
  # You can use this flash and force driver to wait more time, expecially on
  # destroy action when there is slow deleting data
  # app/views/users/destroy.js.erb
  # window.location.assign('<%= customer_path @customer %>');
  # jQuery.active = 1;
  #
  def wait_for_ajax
    printf "jQuery.active"
    start_time = Time.current
    Timeout.timeout(Capybara.default_max_wait_time) do
      loop until _finished_all_ajax_requests?
    end
    printf '%.2f', Time.current - start_time
  rescue Timeout::Error
    printf "timeout#{Capybara.default_max_wait_time}"
  end

  def _finished_all_ajax_requests?
    output = page.evaluate_script('jQuery.active')
    printf "." unless output.zero?
    output.zero?
  end

  def wait_for_visible(target)
    Timeout.timeout(Capybara.default_max_wait_time) do
      loop until page.find(target).visible?
    end
  rescue Timeout::Error
    flunk "Expected #{target} to be visible."
  end
end
RSpec.configure do |config|
  config.include WaitHelper, type: :feature
end
# for minitest use
class ActionDispatch::SystemTestCase
  include WaitHelper
end
~~~

This is not needed any more, also
[wait_until](https://www.varvet.com/blog/why-wait_until-was-removed-from-capybara/)
is removed from capybara.

I see only two problem: first is ajax loaded form in modal and you click on
button to submit it immediatelly, but modal uses `fade` and is not visible yet.
Solution is to remove `fade` class.
<https://github.com/teamcapybara/capybara/issues/1890>

Second is when respone is redirection `window.location.assign('/users/1')`
(usually just to reload a page). Capybara does not wait for this
`window.location` change when it is run in `headless_chrome` driver. I tried
with `window.location.replace` and `$(location).attr('href',)`. Only solution is
to use expectation that find element which is not yet on a page. Maybe `visit
customer_path customer` again, before expectation, could help.

I noticed that when using `js: true` session is preserved between examples even
from multiple files. This is on both chrome and headless_chrome. Since there is
randomization, that could be a tricky problem to reproduce. I tried to add

~~~
  config.before(:example) do
    Capybara.reset_sessions!
  end
  before do
    Capybara.reset_session!
    browser = Capybara.current_session.driver.browser
    if browser.respond_to?(:clear_cookies)
      # Rack::MockSession
      browser.clear_cookies
    elsif browser.respond_to?(:manage) and browser.manage.respond_to?(:delete_all_cookies)
      # Selenium::WebDriver
      browser.manage.delete_all_cookies
    else
      raise "Don't know how to clear cookies. Weird driver?"
    end
  end
~~~

but still problem, sometimes fails/sometimes pass.

## Download helpers

Inspect file that is downloaded like `respond_to do |format| format.csv { render
text: CSV.generate { |csv| csv << [1,2] } } end`

~~~
# spec/support/download_feature_helpers.rb
# https://collectiveidea.com/blog/archives/2012/01/27/testing-file-downloads-with-capybara-and-chromedriver
#
#    csv_content = DownloadHelpers.download_content
#    expect(csv_content.count("\n")).to eq 3
#    expect(csv_content).to include first_customer.name
#
module DownloadFeatureHelpers
  TIMEOUT = 10
  PATH    = Rails.root.join("tmp/downloads")

  extend self

  def downloads
    Dir[PATH.join("*")]
  end

  def download
    downloads.first
  end

  def download_content
    wait_for_download
    File.read(download)
  end

  def wait_for_download
    Timeout.timeout(TIMEOUT) do
      sleep 0.1 until downloaded?
    end
  end

  def downloaded?
    !downloading? && downloads.any?
  end

  def downloading?
    downloads.grep(/\.crdownload$/).any?
  end

  def clear_downloads
    FileUtils.rm_f(downloads)
  end
end

RSpec.configure do |config|
  config.include DownloadFeatureHelpers, type: :feature
end
~~~

Set headless browser https://gist.github.com/bbonamin/4b01be9ed5dd1bdaf909462ff4fdca95

Old way is using profile
~~~
require "selenium/webdriver"
Capybara.register_driver :chrome do |app|
  profile = Selenium::WebDriver::Chrome::Profile.new
  profile["download.default_directory"] = DownloadFeatureHelpers::PATH.to_s
  Capybara::Selenium::Driver.new(app, browser: :chrome, profile: profile)
end

# another way to set profile is with prefs in desired_capabilities
# Currently it does not work if you use headless chrome since it is not
# supported <https://github.com/SeleniumHQ/selenium/issues/4437>
# Probably works in headless firefox.

Capybara.register_driver :headless_chrome do |app|
  desired_capabilities = Selenium::WebDriver::Remote::Capabilities.chrome(
    chromeOptions: { args: %w(headless disable-gpu window-size=1024,768) },
    prefs: {
      "download.default_directory": DownloadFeatureHelpers::PATH.to_s,
    }
  )

  Capybara::Selenium::Driver.new(
    app,
    browser: :chrome,
    desired_capabilities: desired_capabilities,
  )
end
~~~

Usage in test is to clear downloads first since failing expectation will break
and not remove files.

~~~
RSpec.describe 'Location Reports', js: true do
  it 'downloads' do
    DownloadFeatureHelpers.clear_downloads
    click_on 'Generate Report'
    csv_content = DownloadFeatureHelpers.download_content
    expect(csv_content.count("\n")).to eq 3
    expect(csv_content).to include first_customer.name
  end
end
~~~

# Selenium

<https://github.com/SeleniumHQ/selenium/wiki/Ruby-Bindings>
Selenium for ruby use gem `selenium-webdriver`.
You also need executables for firefox `geckodriver` (just download from
<https://github.com/mozilla/geckodriver/releases> to `/usr/local/bin`)
and for chrome `chromedriver` (download from
<https://sites.google.com/a/chromium.org/chromedriver/downloads> to
`/usr/local/bin`) or you can use `gem 'chromedriver-helper'` that will install
chromedriver to `.rvm/gems/ruby-2.3.3/bin/chromedriver`.

Make sure you have version of firefox and chrome that matches drivers.

Same DSL to drive browser (selenium-webdriver, chrome-driver or capybara-webkit)
or headless drivers (`:rack_test` or phantomjs). `Capybara.current_driver` could
be `:rack_test` (when no `js: true`) or `:headless_chrome` or `':chrome`.

## Errors

If you see error `unable to obtain stable firefox connection in 60 seconds
(127.0.0.1:7055) (Selenium::WebDriver::Error::WebDriverError)` you need to `gem
update selenium-webdriver` or to install matched version.

Minitest is included in ruby and also in rails. If your system tests shows error
`Selenium::WebDriver::Error::WebDriverError: unable to connect to chromedriver
127.0.0.1:9515` than add gem `gem 'chromedriver-helper'` to your development and
test group.

If you see `KeyError: key not found: 102` than upgrade chromedriver to 2.33 by
downloading from <https://sites.google.com/a/chromium.org/chromedriver/> and `mv
chromedriver bin` or if you are using `chromedriver-helper` run

~~~
rm -rf ~/.chromedriver-helper/
chromedriver-update
~~~

For `Selenium::WebDriver::Error::UnknownError: unknown error: call function
result missing 'value' (Session info: headless chrome=67.0.3396.48)` you need to
update chromedriver to 2.38

If you see error `SocketError: getaddrinfo: Name or service not known` than make
sure you have defined localhost `127.0.0.1 localhost` in `/etc/hosts`

If you see error `Selenium::WebDriver::Error::StaleElementReferenceError: stale
element reference: element is not attached to the page document` it could be
that element was removed, page reloaded in javascript, or when you use `within
#id` and than make expectation inside `within` block. Try to move expectation
outsite of `within` block or to reload

~~~
page.driver.browser.navigate.refresh
page.evaluate_script 'window.location.reload()'
~~~

Chrome driver usually starts with `data:,` url and than redirects to for example
<http://127.0.0.1:34623/posts>

In rails console you should see starting browser with

~~~
options = Selenium::WebDriver::Chrome::Options.new
options.add_argument('--headless')
driver = Selenium::WebDriver.for :chrome #, options: options
~~~

To run in browser javascript use `js: true`. Note that in this mode, drop down
links are not visible, you need to click on dropdown. Also `data-confirm` will
be ignored.
Note that in this mode you can't use `rails-rspec` default
`config.use_transactional_fixtures` since selenium can't know that refresh or
navigation to another page should be in the same transaction, so you need to use
`database_cleaner` as we do configuration below.
<http://elementalselenium.com/tips/29-chrome-driver>
<https://robots.thoughtbot.com/headless-feature-specs-with-chrome>

~~~
# spec/support/capybara.rb
require "selenium/webdriver"

Capybara.register_driver :chrome do |app|
  # set download directory using Profile (can be set using :prefs in options)
  profile = Selenium::WebDriver::Chrome::Profile.new
  profile["download.default_directory"] = DownloadFeatureHelpers::PATH.to_s
  Capybara::Selenium::Driver.new(app, browser: :chrome, profile: profile)
end

Capybara.register_driver :headless_chrome do |app|
# I prefer to use Options instead Capabilities
#   capabilities = Selenium::WebDriver::Remote::Capabilities.chrome(
#     chromeOptions: { args: %w(headless disable-gpu window-size=1024,768) }
#   )
#   Capybara::Selenium::Driver.new app, browser: :chrome, desired_capabilities: capabilities
  options = Selenium::WebDriver::Chrome::Options.new(
    args: %w[headless disable-gpu window-size=1024,768],
    # can not use prefs for headless driver since it is not supported
    # prefs: {
    #   "download.default_directory": DownloadFeatureHelpers::PATH.to_s,
    # }
  )

  Capybara::Selenium::Driver.new(app, browser: :chrome, options: options)
end

RSpec.configure do |config|
  files = config.instance_variable_get :@files_or_directories_to_run
  if files == ["spec"]
    # when run all spec use headless
    Capybara.javascript_driver = :headless_chrome
  else
   Capybara.javascript_driver = :chrome
  end
end
Capybara.enable_aria_label = true


# if you need to use custom domain , you can set host, but also set server port
Capybara.app_host = "http://my-domain.loc:3333"
Capybara.server_port = 3333
# you can read host and port
Capybara.current_session.server.host
Capybara.current_session.server.port
# normally chrome starts with url: data; and than redirects to app_host
# app_host should ends with .loc or .lvh.me so it point to localhost
~~~

For newer Firefox I needed to download
[geckodriver](https://github.com/mozilla/geckodriver/releases) and put it
somewhere like `/user/local/bin/geckodriver`. Also [Firefox
47.0.1](https://ftp.mozilla.org/pub/firefox/releases/47.0.1/) is suggested, but
my ver 50 also works.

# Remote Selenium

You can control remote selenium server. Download
[selenium-server-standalone.jar](https://www.seleniumhq.org/download/) and run
selenium server.  For error `Unsupported major.minor
version 52.0` you need to update java: 51 -> java7, 52 -> java8, 53 -> java9.

~~~
java -jar selenium-server-standalone.jar
~~~

To test in `rails c` try

~~~
# chrome
driver = Selenium::WebDriver.for :remote, desired_capabilities: :chrome, url: "http://192.168.5.56:4444/wd/hub"
# same as
# driver = Selenium::WebDriver.for :chrome, url: "http://192.168.5.56:4444/wd/hub"
driver.navigate.to 'http://google.com' #=> nil

# firefox
# driver = Selenium::WebDriver.for :firefox, url: "http://192.168.5.56:4444/wd/hub"

# headless chrome
options = Selenium::WebDriver::Chrome::Options.new(
  args: %w[headless disable-gpu window-size=1024,768],
)
driver = Selenium::WebDriver.for :chrome, url: "http://192.168.5.56:4444/wd/hub", options: options
~~~

If there is a screen you can run both headless and not. If you run server from
ssh (there is no screen) than you can run headless or you can run server using X
virtual frame buffer

~~~
xvfb-run java -Dwebdriver.chrome.driver=/usr/local/bin/chromedriver -jar /usr/local/bin/selenium-server-standalone.jar
~~~

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

# Alternatives

https://github.com/gokhandemirhan/KimonoAlternatives

* Find selectors using javascript https://github.com/cantino/selectorgadget and
chrome extension plugin addon selector gadget (click first on element to become
yellow, and than click on all yellows to mark them as red to remove)
* https://webscraper.io/ browser extension https://www.webscraper.io/tutorials
  nice documentation with images
  https://www.webscraper.io/documentation/selectors/link-selector
  https://github.com/webscraperio
  https://www.webscraper.io/test-sites
* browser extension and api https://www.agenty.com/docs/video-tutorials.aspx
  with example integrations https://www.agenty.com/integrations/
* https://www.octoparse.com/ without coding, graphical algorithm
  https://www.youtube.com/channel/UCweDWm1QY2G67SDAKX7nreg

https://import.io
https://www.parsehub.com/
https://scrape.it/
https://morph.io/
http://scrapinghub.com/
https://www.scrapehero.com/
http://datahut.co/
http://scrapy.org/
