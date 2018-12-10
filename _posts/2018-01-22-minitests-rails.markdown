---
layout: post
---

# Minitest

Minitest real test examples: openstreetmap, redmine

Mini test for rails `TestCase`s (like `ActiveSupport::TestCase`) inherits from
`Minitest::Test` so you can use it `def test_method`, but Rails adds `test`
method so it can be used like `test "my method" do`.  In minitests you can not
have same test descriptions since it will be converted to same
`test_my_method`.

You can run specific test use `:line` or `-n test_name`.
This also works for system test (so you do not need `system` in `rails
test:system somefile`)

~~~
rails test
rails test:system
# invoke test by line
rails test test/models/article_test.rb:6
# or name
rails test -n test_example
~~~

You could also use ENV variables `rails test
TEST=test/machine_with_callbacks_test.rb
TESTOPTS="--name=test_should_run_validations_for_specific_state -v"`

Each test run will load all fixture data, run setup blocks, run test method, run
teardown block, rollback fixtures.

# Fixtures

http://api.rubyonrails.org/v5.2.0/classes/ActiveRecord/FixtureSet.html

Rails performs loading fixtures in tree steps: removing old fixture data, load
fixture data and dump data into a method inside test case `users(:david)`

Defining fixtures:
* use labels for associated belongs_to and has_many items (separated by comma)
  ~~~
  # test/fixtures/users.yml
  my_user:
    name: My User
    company: my_company

  # test/fixtures/companies.yml
  my_company:
    name: My Company
  ~~~
* if you have polymorphic than put class name in brackets
  ~~~
  apple:
    eater: george (Monkey)
  ~~~

* `created_at`, `updated_at`, `created_on` and `updated_on` is automatically
  Time.now
* use ERB for dynamic creation. Note that you should use fixed dates instead of
  `published_at: <%= Date.today.strftime('%Y-%m-%d')` so they are stable.
  ~~~
  <% 15.times do |i| %>
  medium_<%= i %>:
    media_name: My <%= i %> Medium
    published_at: 2018-10-11
  <% end %>
  ~~~
* fixture label interpolation (`$LABEL` is `me`)

  ~~~
  me:
    name: 'duke'
    subdomain: me
    # or use label
    subdomain: $LABEL
    email: $LABEL@email.com
  ~~~

* defaults
  ~~~
  DEFAULTS: &DEFAULTS
    created_on: <%= 3.weeks.ago.to_s(:db) %>

  first:
    name: Smurf
    <<: *DEFAULTS

  second:
    name: Fraggle
    <<: *DEFAULTS
  ~~~

* if you really want hardcoded ids you can use
  ```
  my_language:
    id: 5
    name: English

  course:
    language_id: 5
    # note that this is just yml so you can not use
    language_id: my_language.id
    # but you can use erb
    language_id: <%= ActiveRecord::FixtureSet.identify :my_language %>
  ```
* `pre_loaded_fixtures`
* `use_transactional_tests`. In Rails 4 it was
  `use_transactional_fixtures` but since it could be factories not fixtures,
  that is renamed to `use_transactional_tests`. So to disable it you can use

  ~~~
  class FooTest < ActiveSupport::TestCase
    self.use_transactional_tests = false
  end
  ~~~
* when defining new fixtures, add them to the end, so you do not break current
  test for first page (when you use pagination)
* load fixtures in development `rake db:fixtures:load` (put in your seed
  ```
  # db/seed.rb
  Rake::Task['db:fixtures:load'].invoke
  ```
* to set devise password, you can add exncypted password on specific items
  ```
  admin_user:
    encrypted_password: <%= User.new.send(:password_digest, 'password') %>
  ```
* Usage of fixtures

  ~~~
  # note that we are calling method users()
  users(:my_user)
  # we can get two
  users(:duke, mike)
  ~~~

# Minitest classes

To see all 6 base test clases go
<http://guides.rubyonrails.org/testing.html#a-brief-note-about-test-cases>
ActiveSupport::TestCase
ActionMailer::TestCase
ActionView::TestCase
ActionDispatch::IntegrationTest
ActiveJob::TestCase
ActionDispatch::SystemTestCase

# Minitest model

~~~
require 'test_helper'

class TaskTest < ActiveSupport::TestCase
  test 'a completed' do
    task = Task.new
    refute(task.complete?)
    task.mark_completed
    assert(task.complete?)
  end

  def test_that_is_skipped
    skip "later"
  end

  test 'valid fixture' do
    # database constrains do not need to be tested because fixture will fail if
    # can not be inserted in db. We need to test only rails validations
    assert tasks.map(&:valid?).all?

    # you can add specific error message
    assert tasks.map(&:valid?).all?, tasks.select {|c| !c.valid?}.map {|c| c.errors.full_messages.join}
  end
end
~~~

Assertions
[all](http://docs.seattlerb.org/minitest/Minitest/Assertions.html)

* `assert true` or `assert_block do end`
* `assert_nil`
* `assert_empty`
* `assert_raises(NameError) do end`
* `assert_match regex, actual_string`
* `assert_includes(collection, object)`
* `assert_equal(expected, actual)`

They all have oposite `refute_` methods, or `assert_not`
They all accept additional string param that will be error message.

<http://guides.rubyonrails.org/testing.html#rails-specific-assertions>
Rails also defines `assert_difference`, `assert_blank`, `assert_presence`,
`assert_response`, `assert_redirected_to`, `assert_select`

~~~
    assert_difference "User.count", 1 do
    end
~~~

You can create your own assertiongs (assert sorted)
Reopen `module MiniTest::Assertions` when helper is used in all tests. If you
need only for system tests (assert_selector) or integration test (assert select)
than you need to reopen that particular class
https://github.com/duleorlovic/premesti.se/blob/master/test/support/assert_equal_when_sorted_by_id.rb

~~~
# test/application_system_test_case.rb
require 'test_helper'

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  def assert_alert_message(text)
    assert_selector 'div#alert-debug', text: text, visible: false
  end

  def assert_notice_message(text)
    assert_selector 'div#notice-debug', text: text, visible: false
  end
end
~~~

for integration tests

~~~
# test/support/assert_flash_message.rb
class ActionDispatch::IntegrationTest
  # assert_flash_message
  def assert_alert_message(text)
    assert_select 'div#alert-debug', text
  end

  def assert_notice_message(text)
    assert_select 'div#notice-debug', text
  end

  def assert_select_field_error(text)
    assert_select 'div.invalid-feedback', text
  end
end
~~~

For any method? you can assert for example

~~~
assert_kind_of Array, exp
~~~

# Minitest services

Similar to model test you can create tests for services since there are PORO

~~~
# test/services/credit_card_service_test.rb

require 'test_helper'

class CreditCardServiceTest < ActiveSupport::TestCase
  test 'it creates charges' do
  end
end
~~~

# Minitest controllers

Deprecated, use minitest integration tests

# Minitest integration

For integration we use `ActionDispatch::IntegrationTest` which gives methods:
* `get '/'`, `post path, params: {}` (params is required in rails_post_5),
* also `put`, `patch`, `head`, `delete`
* for ajax use `post path, xhr: true`
* add `as: :json` as third param for json requests so you can use something like
* `assert_equal({ id: Article.last.id,
title: "Ahoy!" }, response.parsed_body)`.

* `follow_redirect!`
* `assert_redirected_to post_url(Post.last)`
* `assert_response :success` (for http codes 200-299), `:redirect` (300-399), `:missing` (404), `:error` (500-599).
* `assert_select 'h1', user.email` . Note that white spaces and new lines are
  ignored
* `assert_difference "User.count", 1 do`
* You have access to `@request`, `@controller` and `@response` object, but only
  after you call `get`, `post`. You can set `request.remote_ip` by passing
  headers `get '/', headers: { 'REMOTE_ADDR': '123.123.123.123' }`
* `assert_select 'h1', 'Welcome'` is used to test view. View can be tested with
  `assert_match /Welcome/', response.body`.
  There are two forms of *assert_select selector, [equality], [message]* or
  using Nokogiri::XML::Node as element *assert_select element, selector,
  [equality]*.
  selector can be CSS selector as one string, or array with substitutions
  ~~~
  assert_select 'div#123' # exists <div id='123'>
  assert_select "a[href='http://www.example.com/diary/new']"`
  assert_select 'input[value=?]', username   # substitute username
  assert_select 'input[value*=?]', username   # match when containing username

  # use custom pseudo class `:match(attribute_name, attribute_value)`
  assert_select "ol>li:match('id', ?)", /item-\d+/       # exists li id='item-1'
  assert_select "div:match('id',?)", /\d+/        # exists div with non empty id

  assert_select 'div', 'Hello' # exists <div>Hello</div>
  assert_select 'div', /hello/ # if div text matches
  assert_select 'div', false # no divs exists
  assert_select 'div', 4 # exactly 4 divs
  assert_select 'div', 4..5 # number of dives is in range
  ~~~

  To perform more than one quality tests in one assert you can use hash
  (assert_selector also use this hash param)

  ~~~
  # instead of refute_select, for no div with my_name you can use
  assert_select 'a[href=?]', link, text: /my_name/, count: 0
  assert_select 'div', html: 'p', minimum: 2
  ~~~

  To check html in emails use assert_select_email

  ~~~
  assert_select_email do
    items = assert_select "ol>li"
    items.each do
      Work with items here...
    end
  end
  ~~~

Those integration tests are similar to Rspec request spec, and works full stack
with no use of capybara. More info
  https://github.com/rails/rails-dom-testing/blob/master/lib/rails/dom/testing/assertions/selector_assertions.rb
similar to capybara `have_selector` but separate implementation `html selector`.
<http://www.rubydoc.info/github/rails/rails-dom-testing/Rails%2FDom%2FTesting%2FAssertions%2FSelectorAssertions%3Aassert_select>
  After request is made you can also access `@controller`, `@request` and
  `@response` (same as `response`).

~~~
# test/controllers/projects_controller_test.rb
require 'test_helper'

class ProjectsControllerTest < ActionDispatch::IntegrationTest
  setup do
    @project = create :project
  end

  # called after every single test
  teardown do
    # when controller is using cache it may be a good idea to reset it afterwards
    Rails.cache.clear
  end

  test 'show' do
    get project_path(@project)
    assert_select "#tutorial-#{tutorials(:priced_course_first_free_tutorial).id}", /Watch this lesson now/
    assert_select "#tutorial-#{tutorials(:priced_course_third_tutorial).id}", text: /Watch this lesson now/, count: 0

    # or you can parse `response.body` do it manually

    doc = Nokogiri::HTML response.body
    # you can get all text with doc.text.split.join(' ')
    el = doc.search "#tutorial-#{tutorials(:priced_course_first_free_tutorial).id}"
    assert_match 'Watch this lesson now', el.text
    el = doc.search "#tutorial-#{tutorials(:priced_course_third_tutorial).id}"
    assert_no_match 'Watch this lesson now', el.text
  end

  test "the create method creates project" do
    post projects_url, params: { project: { name: 'Duke' } }
    assert_redirected_to projects_path
    # also available: assert_response :redirect
    follow_redirect!
    assert_select '#my-id', /dule/
  end
end
~~~

## Minitest routing

It uses `assert_routing`.

## Minitest helper

Uses `assert_dom_equal`

~~~
# test/helpers/projects_helper_test.rb
require 'test_helper'
class ProjectsHelperTest < ActionView::TestCase
  test "project on schedule" do
    project = Project.new name: 'Duke'
    project.stubs(:on_schedule?).returns(true)
    actual= name_with_status(project)
    expected = "<span class='on-schedule'>Duke</span>"
    assert_dom_equal expected, actual
  end
end
~~~

## Minitest system

Feature tests are run in different process than rails so we need database
cleaner gem. With system tests, we can use internal mechanism to roll back db
changes.
Before we used feature specs for full application integration testing, but now
we should use system specs (which also uses capybara and webdriver with chrome).

System tests are not included in default test suite if you run `rails test` (you
should run `rake test:system`), but if you run specific test than it will be run
`rails test test/system/my_test.rb`.

Note that if you use puma with multiple workers than
`ActionMailer::Base.deliveries.size` could be different in rails process and in
test process, so use `workers 0` in `config/puma/test.rb`.

In system test you can't mock OmniAuth.mock_auth so use integration test for
that.

Capybara assertions
<http://www.rubydoc.info/gems/capybara/Capybara/Minitest/Assertions>

* `assert_text /dule/`
* `assert_selector 'a', text: 'duke'`
* `assert_equal admin_path, page.current_url`
* `page.accept_confirm` to confirm alert dialog box

## Webmock

```
# Gemfile
gem 'webmock'

# config/initializers/webmock.rb
if Rails.env.test?
  require 'webmock'
  WebMock.disable_net_connect!(allow_localhost: true)
end

# test/test_helper.rb
require 'webmock/minitest'
```

## Minitest helpers

Add a line in `test/test_helper.rb` to include files from `test/a`
```
# test/test_helper.rb
ENV['RAILS_ENV'] ||= 'test'
require File.expand_path('../../config/environment', __FILE__)
Dir[Rails.root.join('test/a/**/*.rb')].each { |f| require f }
require 'rails/test_help'
require 'minitest/autorun'
require 'webmock/minitest'

class ActiveSupport::TestCase
  # create(:user) instead FactoryBot.create :user
  include FactoryBot::Syntax::Methods
  # stub_something
  include WebmockHelper
  # devise method: sign_in user
  include Devise::Test::IntegrationHelpers
  # Setup all fixtures in test/fixtures/*.yml for all tests in alphabetical order.
  fixtures :all
end
```

I prefer to use factories on specific integration test so when I need to remove
specific fixtures I use rails `Courses.delete_all`. I do not use database
cleaner because it complicates thinks because I need to load some specific
fixtures (like users) and remove other. Not sure if that will remove fixtures
for other tests
https://stackoverflow.com/a/28539464/287166

For pause

~~~
# test/a/pause_helper.rb
module PauseHelper
  # you can use byebug, but it will stop rails so you can not navigate to other
  # pages or make another requests in chrome while testing
  def pause
    $stderr.write('Press CTRL+j or ENTER to continue') && $stdin.gets
  end
end

class ActionDispatch::SystemTestCase
  include PauseHelper
end
~~~

## Minitest Mock

You need to require autorun

~~~
# test/test_helper.rb
require 'minitest/autorun'
~~~

http://ruby-doc.org/stdlib-2.0.0/libdoc/minitest/rdoc/MiniTest/Mock.html

You can mock return values:
~~~
@mock = Minitest::Mock.new
@mock.expect(:meaning_of_life, 42)
@mock.meaning_of_life # => 42
~~~

and also for arguments and verify their type or value

~~~
@mock.expect(:do_something_with, true, [some_obj, true])
@mock.do_something_with(some_obj, true) # => true

@mock.expect(:uses_any_string, true, [String])
@mock.uses_any_string("foo") # => true
@mock.verify  # => true

@mock.expect(:uses_one_string, true, ["foo"]
@mock.uses_one_string("bar") # => true
@mock.verify  # => raises MockExpectationError
~~~

You can use `Object.stub` https://github.com/seattlerb/minitest#stubs 
https://stackoverflow.com/questions/10990622/how-do-i-stub-a-method-to-raise-an-error-using-ruby-minitest
~~~
require "minitest/autorun"

class MyModel
  def my_method; end
end

class TestRaiseException < MiniTest::Unit::TestCase
  def test_raise_exception
    model = MyModel.new
    raises_exception = Proc.new { raise ArgumentError.new }
    model.stub :my_method, raises_exception do
      assert_raises(ArgumentError) { model.my_method }
    end
  end
end
~~~


Stub works also with some custom doubles mock

~~~
# test/models/user_test.rb
require 'test_helper'
class UserTest < ActiveSupport::TestCase
  test '#apply_subscription' do
    mock = Minitest::Mock.new
    def mock.apply; true; end

    SubscriptionService.stub :new, mock do
      user = users(:one)
      assert user.apply_subscription
    end
  end
end
~~~

## Minitest mocha feature tests

Rails calls "integration tests" as "request tests" and it covers both controller
and view code. If logic is more complex, it should be written in model anyway,
and use lower level tests.

With capybara:

~~~
# Gemfile
  gem "minitest-rails-capybara"
  gem "mocha", require: false

# test/test_helper.rb
require "minitest/rails/capybara"
require "mocha/mini_test"
~~~


~~~
  test "full double" do
    stubby = stub(name: "Dule", wight: 100)
    assert_equal("Dule", stubby.name)
  end

  test "verifying double" do
    stubby = stub(name: "Dule", weight: 100)
    stubby.responds_like(Project.new)
    # or
    stubby.responds_like_instance_of(Project)
    # this is find since project.name exists
    stubby.name
    # this will raise error
    skip
    stubby.weight
  end

  test "partial double" do
    project = Project.new name: 'Dule'
    # we can stub method on any ruby object
    project.stubs(:name)
    # returns nothing unless we use returns
    assert_nil(project.name)
    # we can stub non existing methods
    project.stubs(:asd)
    assert_nil(project.asd)
    # we can define return
    project.stubs(:due_date).returns(Date.today)
    assert_equal(project.due_date, Date.today)
    # we can stub classes
    Project.stubs(:find).returns(Project.new(name: 'dule'))
    project = Projet.find(1)
    assert_equal('dule', project.name)
    # we can stub any instance
    Project.any_instance.stubs(:save).returns(false)
  end

  test "mock expectation" do
    mock_project = Project.new name: 'Not important since expect returns'
    mock_project.expects(:name).returns('Duke')
    # previous expectation will fail if we have not called it
    assert_equal('Duke', mock_project.name)
    # we can set number of times it is called
    mock_project.expects(:name).twice
    mock_project.name
    mock_project.name
    # we can set arguments
    mock_task = Task.new
    mock_task.expects(:mark_as_completed).with(instance_of(Date)).returns(instance_of(Date))
    mock_task.mark_as_completed Date.today
    mock_task.expects(:mark_as_completed).with(instance_of(String)).returns(nil)
    mock_task.mark_as_completed "bla"
  end
~~~


# Guard

For guard minitest use `guard-minitest`

~~~
sed -i Gemfile -e '/group :development do/a  \
  gem 'guard'
  gem 'guard-minitest'
bundle
guard init minitest
vi Guardfile
# uncomment rails section
# change some lines in Guardfile
guard :minitest, spring: 'bin/rails test', failed_mode: :focus do
~~~

* to show full backtrace use `BACKTRACE=blegga rails test`
* to take screenshot you can call `take_screenshot` in tests.
  Automatically taking screenshot when tast fails is included in teardown by
  default, and you can override it
  https://api.rubyonrails.org/v5.2/classes/ActionDispatch/SystemTesting/TestHelpers/ScreenshotHelper.html#method-i-take_screenshot
  By default `image_name` is
  [method_name](https://github.com/rails/rails/blob/fc5dd0b85189811062c85520fd70de8389b55aeb/actionpack/lib/action_dispatch/system_testing/test_helpers/screenshot_helper.rb#L42)
  By default `display_image` will print `puts image_path` but we can open image


  ~~~
  # test/application_system_test_case.rb
  require 'test_helper'

  class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
    driven_by :selenium, using: :chrome, screen_size: [1400, 1400]

    def display_image
      system "gnome-open #{image_path} &"
      "Opening screenshot: gnome-open #{image_path}"
    end
  end
  ~~~

* test og meta tags for jekyll
* https://gist.github.com/thbar/10be2ea924b81f78d24ab800461bfee3
