---
layout: post
---

# Minitest

Minitest real test examples: openstreetmap, redmine

Mini test for rails `TestCase`s (like `ActiveSupport::TestCase`) inherits from
`Minitest::Test` so you can use it `def test_password`, but Rails adds `test`
method so it can be used like `test "my password" do`.  In minitests you can not
have same test descriptions since it will be converted to same
`test_my_password`.

You can run specific test use `:line` or `-n test_name`.
This also works for system test (you do not need `system` in `rails
test:system`)

~~~
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

class MyClassTest < ActiveSupport::TestCase
  test "a completed" do
    task = Task.new
    refute(task.complete?)
    task.mark_completed
    assert(task.complete?)
  end

  def test_that_is_skipped
    skip "later"
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
* `assert_select 'h1', user.email`
* `assert_difference "User.count", 1 do`
* You have access to `@request`, `@controller` and `@response` object, but only
after you call `get`, `post`. You can see page body with `response.body`
[available instance
variables](http://guides.rubyonrails.org/testing.html#instance-variables-available):

`assigns` and `assert_template` are moved to separated template.
Those are similar to Rspec request spec, and works full stack with no use of
capybara.

~~~
# test/controllers/projects_controller_test.rb
require 'test_helper'

class ProjectsControllerTest < ActionDispatch::IntegrationTest
  setup do
    @project = create :project
  end

  test 'show' do
    get project_path(@project)
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

# Assert select

View can be tested with `assert_select` similar to capybara `have_selector` but
separate implementation `html selector`.
<http://www.rubydoc.info/github/rails/rails-dom-testing/Rails%2FDom%2FTesting%2FAssertions%2FSelectorAssertions%3Aassert_select>
assert_select can be used in functional or integration tests

* `assert select 'input', 2` second param is integer so assertion is true if
exactly that number of elements, if `false` than no element exists
* `assert_select 'input', /my_name/` second param string/regexp so it is true
if text value of at least one element matches
* `assert_select  'input', text: /my_name/, count: 0` is also possible and used
when you do not want to see some text
* you can nest and use 
* target with `element[attribute_name=attribute_value]` like
`assert_select "a[href='http://www.example.com/diary/new']"`
* substitute with `assert_select input[value=?], username`
* use custom pseudo class `:match(attribute_name, attribute_value)`
like: `assert_select "ol>li:match('id', ?)", /item-\d+/`



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

## Minitest helpers

Add a line in `test/test_helper.rb` to include support
`Dir[Rails.root.join('test/support/**/*.rb')].each { |f| require f }` files.

~~~
# test/support/pause_helper.rb
module PauseHelper
  # you can use byebug, but it will stop rails so you can not navigate to other
  # pages or make another requests in chrome while testing
  def pause
    $stderr.write('Press ENTER to continue') && $stdin.gets
  end
end

class ActionDispatch::SystemTestCase
  include PauseHelper
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

# Capybara

* `assert_selector`
