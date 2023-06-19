---
layout: post
---

# Minitest

Minitest real test examples: openstreetmap, redmine

Mini test for rails `TestCase`s (for example `ActiveSupport::TestCase`) inherits
from `Minitest::Test` so you can use it `def test_method`, but Rails adds `test`
method so it can be used like `test "my method" do`.  In minitests you can not
have same test descriptions since it will be converted to same `test_my_method`.

You can run specific test with `:10` or `-n test_name`. If test is defined like
`test "my test"` than you can use regexp `ruby test/file_test.rb -n /my.test/`
This also works for system test (so you do not need `system` in `rails
test:system somefile`)

~~~
rails test
rails test:system
# sometimes assets are not compiled properly or node is different version so to
# trigger precompile you should run `rm -rf tmp public && git checkout tmp public`
# so with `rails test:system` new folder will be created `public/packs-test`
# invoke test by line
rails test test/models/article_test.rb:6
# or name
# works only when you use `def test_example` syntax
# for `test "test_example" do` you should use -n test_test_example
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
Note that when using `smoke: Yes` or `smoke: No` it might be converted to true
false, so better is to use `smoke: 'Yes'` or `smoke: 'No'`

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
  # test/fixtures/notifications.yml
  notification_for_task:
    notifiable: my_task (Task)
  notification_for_comment:
    notifiable: my_comment (Comment)
  ~~~

  No need to use brackets if you have `belongs_to :associated, class_name:
  'Activity'`
* for activestorage attachments you do not need to create fake models. just
  create real fixtures
  https://stackoverflow.com/questions/50453596/activestorage-fixtures-attachments
  When I skip `key` not null column in fixture than I receive strange errors

  ```
  /home/orlovic/.rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/drb/drb.rb:565:in `dump': no _dump_data is defined for class PG::Connection (TypeError)
/home/orlovic/.rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/drb/drb.rb:565:in `dump': can't dump hash with default proc (TypeError)
  # those are just backtrace errors for runners, not important, messages before
  # this are more important
/home/orlovic/.rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/drb/drb.rb:1738:in `current_server': DRb::DRbServerNotFound (DRb::DRbServerNotFound)
/home/orlovic/.rvm/gems/ruby-2.6.3/gems/activesupport-6.0.2.1/lib/active_support/testing/parallelization.rb:98:in `block (3 levels) in start': undefined method `exception=' for #<Minitest::UnexpectedError: Unexpected exception> (NoMethodError)
  ```
  Here are fixtures so you have some blobs in db (another way to test is using
  fixture_file_upload)
  ```
  # test/fixtures/active_storage/blobs.yml
  my_blob:
    key: or9sbwfely5gby30qdvtoa1cu09a
    filename: computer_text.png
    content_type: image/png
    # jsonb
    metadata: '{"identified":true}'
    metadata: <%= { "identified" => true }.to_json %>
    byte_size: 56024
    checksum: wHaMfHXpSThHCX/zvm5fFg==

  # test/fixtures/active_storage/attachments.yml
  DEFAULTS: &DEFAULTS
    # you can use same fake blob for all attachments
    blob_id: <%= ActiveRecord::FixtureSet.identify(:my_blob) %>
    created_at: 2019-12-06

  my_attachment:
    <<: *DEFAULTS
    <%# this has to be the same name as in has_one_attached %>
    name: file
    record: my_doc (Doc)
    <%# record_type: Doc %>
    <%# record_id: <%= ActiveRecord::FixtureSet.identify(:my_doc) %1> %>
  ```
  If you want to have real file for this blobs, you can use before(:all) setup
  before whole suite, to copy file to appropriate location so seed from fixtures
  can be used on developing
  ```
  # test/test_helper.rb
    def self.initialize_fixture_blob
      return if File.exist? "#{Rails.root}/tmp/storage/or/9s/or9sbwfely5gby30qdvtoa1cu09a"

      FileUtils.mkdir_p "#{Rails.root}/tmp/storage/or/9s"
      FileUtils.cp(
        "#{Rails.root}/test/fixtures/files/computer_text.png",
        "#{Rails.root}/tmp/storage/or/9s/or9sbwfely5gby30qdvtoa1cu09a"
      )
    end
    initialize_fixture_blob
  ```

  To use in request use `blob.signed_id`
  ```
  test 'upload one picture' do
    user = users(:user)
    attachment = user.member_profile.member_picture.active_storage_attachment
    sign_in user

    blob = attachment.blob.signed_id
    assert_difference 'ActiveStorage::Blob.count', 0 do
      assert_difference 'MemberPicture.count', 1 do
        patch profile_update_path, params: { member_profile: { id: user.member_profile, member_pictures_attributes: { '0' => { active_storage_attachment: blob } } } }
        assert_response :redirect
      end
    end
  end
  ```

  File upload from `test/fixtures/files/logo.png`
  https://apidock.com/rails/ActionDispatch/TestProcess/FixtureFile/fixture_file_upload
  ```
  # include if not fixture_file_upload is not available

  include ActionDispatch::TestProcess # for fixture_file_upload

  post :create, logo: fixture_file_upload('files/logo.png', 'image/png')
  ```

* `enum parent_relations: %i[child]` can be set using erb
  ```
  child:
    parent_relation: <%= User.parent_relations[:child] %>
  ```
* `created_at`, `updated_at`, `created_on` and `updated_on` is automatically
  Time.now
* use ERB for dynamic creation. Note that you should use fixed dates instead of
  `published_at: <%= Date.today.strftime('%Y-%m-%d')` so they are stable.
  Note that erb is evaluated before yml (for example `$LABEL`)
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

  $Label is not interpolated within object like `name: { en: $LABEL }` (so I18n
  mobility does not work)

* defaults
  ~~~
  DEFAULTS: &DEFAULTS
    name: $LABEL Name
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
* load fixtures in development `rake db:fixtures:load` so put in your seed
* to set devise password, you can add encrypted password on specific items
  ```
  admin_user:
    encrypted_password: <%= User.new.send(:password_digest, 'password') %>

  # without device it is with gem bcrypt
    password_digest: <%= BCrypt::Password.create('password') %>
  ```
* if you use `serialize :recurrence` than in fixture you have to wrap with
  double quotes around and use .to_yaml
```
  recurrence: "<%= { a: 1 }.toyaml %>"
```

* if there are no colomns you can use `user: {}` curly brackets
* Usage of fixtures

  ~~~
  # note that we are calling method users()
  users(:my_user)
  # we can get two
  users(:duke, mike)
  ~~~
* do not need to put all data in fixtures, you can create simple factories
  ```
  # test/models/user_test.rb
  require 'test_helper'

  class UserTest < ActiveSupport::TestCase
    def create_user!(attr = {})
      user = users(:user).dup
      user.email = "#{rand}@email.com"
      user.password = "password"
      user.assign_attributes attr
      user.save!
      user
    end

    test "all users" do
    def test_example
      users = []
      users.append create_user!
      users.append create_user! unsubscribe_email_groups: ["remainders"]
      users.append create_user! unsubscribe_email_groups: ["messages"]

      assert_emails 2 do
        result = WeeklyRemainderWorker.new.perform(users)
        assert result.success?
      end
    end
  end
  ```

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
    # similar to xtest xit
  end

  test 'valid fixture' do
    assert_valid_fixture activities
  end

  # test/test_helper.rb
  def assert_valid_fixture(items)
    assert items.map(&:valid?).all?, (items.reject(&:valid?).map { |c| (c.respond_to?(:name) ? "#{c.name} " : '') + c.errors.full_messages.to_sentence })
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
Rails also defines `assert_difference`, `assert_empty`, `assert_presence`,
`assert_response`, `assert_redirected_to`, `assert_select` (assert select for
custom html text is done with additional argument)
https://stackoverflow.com/questions/4739261/is-there-a-way-to-use-assert-select-on-some-custom-html-text
```
    doc = HTML::Document.new(CGI::unescapeHTML(extracted_text)).root
    assert_select(doc, tag, content)
```

You can create your own assertiongs (assert sorted)
Reopen `module MiniTest::Assertions` when helper is used in all tests. If you
need only for system tests (assert_selector) or integration test (assert_select)
than you need to reopen that particular class
https://github.com/duleorlovic/premesti.se/blob/master/test/support/assert_equal_when_sorted_by_id.rb

~~~
# test/a/assert_flash_message.rb
# https://github.com/duleorlovic/minitest_rails/blob/main/test/a/assert_flash_message.rb
class ActionDispatch::IntegrationTest
  # assert_flash_message
  def assert_alert_message(text)
    assert_select '[data-test=alert]', text
  end

  def assert_notice_message(text)
    assert_select '[data-test=notice]', text
  end
end

class ActionDispatch::SystemTestCase
  def assert_alert_message(text)
    assert_selector '[data-test=alert]', text: text
  end

  def assert_notice_message(text)
    assert_selector '[data-test=notice]', text: text
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

## VCR

To install use
```
# Gemfile
group :test do
  gem 'vcr'
  gem 'webmock'
end

# test/a/vcr.rb
VCR.configure do |config|
  config.cassette_library_dir = 'test/vcr_cassettes'
  config.hook_into :webmock
  config.ignore_localhost = true
end

# test/services/my_service_test.rb
require 'test_helper'

class MyServiceTest < ActiveSupport::TestCase
  test 'success' do
    VCR.use_cassette 'my_service' do
      result = MyService.new(users(:my)).perform
      assert result.success?
    end
  end

  # you could also use in setup and teardown
  # https://github.com/vcr/vcr/wiki/Usage-with-MiniTest
   before do
      VCR.insert_cassette name
    end

    after do
      VCR.eject_cassette
    end
  end
```

Also works for system test, just I need to retry with delay for long processes
(you can simulate by inserting `sleep 5` in your service).
```
# test/system/docs_test.rb
require 'application_system_test_case'

class DocsTest < ApplicationSystemTestCase
  test 'creating a Doc' do
    VCR.use_cassette 'detect_text_and_phi_computer_text' do
      visit docs_url
      click_on t_crud('add_new', Doc)

      fill_in 'Name', with: @doc.name
      page.attach_file 'doc[file]', "#{Rails.root}/test/fixtures/files/computer_text.png", make_visible: true
      click_on 'Create Doc'

      retries = 5
      begin
        retries -= 1
        assert_notice 'Doc successfully created'
      rescue Minitest::Assertion => e
        puts "retries=#{retries}"
        raise e if retries.zero?

        sleep 1
        retry
      end
    end
  end
end
```
More info on rails testing page

# Minitest controllers

Deprecated, use minitest integration tests

# Minitest integration

For integration we use `ActionDispatch::IntegrationTest` which gives methods:
* `get '/'`, `post path, params: {}` (params is required in rails_post_5),
 To set format use `patch path, params: { format: :turbo_stream }`
* also `put`, `patch`, `head`, `delete`
* for ajax use `post path, xhr: true`
* add `as: :json` as third param for json requests so you can use something like
  ```
  get article_path(@article), as: :json
  assert_equal "my title", response.parsed_body["title"]
  ```

* `follow_redirect!`
* `assert_redirected_to post_url(Post.last)`,or you can match only part of url
  using `@response` object
   ```
   assert_match /https:\/\/www.myapp.com\/path/, @response.redirect_url
   ```
* `assert_response :success` (for http codes 200-299), `:redirect` (300-399), `:missing` (404), `:error` (500-599).
* `assert_select 'h1', user.email` . Note that white spaces and new lines are
  ignored
* `assert_difference 'User.count', 1 do`
* You have access to `@request`, `@controller` and `@response` object, but only
  after you call `get`, `post`. You can set `request.remote_ip` by passing
  headers (note that http referrer is written as http referer (single r)
  ```
  get sign_up_path, params: { user: { email: 'new@email.com' } }, headers: { 'REMOTE_ADDR': '123.123.123.123', HTTP_REFERER: 'http://domain.com', 'HTTP_USER_AGENT': 'Mozilla'}
  ```
  also you can authenticate basic auth
  ```
  get my_path, headers: { Authorization: ActionController::HttpAuthentication::Basic.encode_credentials('dhh', 'secret') }
  ```
* to change default host you can use setup block (similar to rspec before) in
  test helper
  ```
  # test/test_helper.rb
  class ActiveSupport::TestCase
    def setup
      host! 'example.com'
    end
  end
  ```
* `assert_select 'h1', 'Welcome'` is used to test view. View can be tested with
  `assert_match /Welcome/', response.body`, for multi line you can use `/m`
  `assert_match /first line.*second line/m, response.body`
  `refute_match "Welcome", response.body` to assert not included text
  There are two forms of *assert_select selector, [equality], [message]* or
  using Nokogiri::XML::Node as element *assert_select element, selector,
  [equality]*.
  selector can be CSS selector as one string, or array with substitutions
  ~~~
  assert_select 'div#123' # exists <div id='123'>
  assert_select "a[href='http://www.example.com/diary/new']"
  assert_select 'a[href=?]', tasks_path
  assert_select 'input[value=?]', username   # substitute username
  assert_select 'input[value*=?]', username   # match when containing username
  assert_select '[data-test=unread-count]', count: 123
  assert_select '[data-test=unread-count]', minimum: 123

  # use custom pseudo class `:match(attribute_name, attribute_value)`
  assert_select "ol>li:match('id', ?)", /item-\d+/       # exists li id='item-1'
  assert_select "div:match('id',?)", /\d+/        # exists div with non empty id

  assert_select 'div', 'Hello' # exists <div>Hello</div>
  assert_select 'div', /hello/ # if div text matches
  assert_select 'div', false # no divs exists
  assert_select 'div', 4 # exactly 4 divs
  assert_select 'div', 4..5 # number of dives is in range

  # multiple attributes
  assert_select "input[type=checkbox][value=messages][checked=checked]", 1
  ~~~

  To perform more than one quality tests in one assert you can use hash
  (assert_selector also use this hash param, like presence items
  `assert_selector '[data-test^=member-profile-]', count:
  Const.limits[:per_page]`)

  ~~~
  # instead of refute_select, for no div with my_name you can use count but put
  # inside hash: text: and count:
  assert_select 'a[href=?]', 'http://link.com', text: /my_name/, count: 0
  # https://apidock.com/rails/ActionController/Assertions/SelectorAssertions/assert_select
   # All input fields in the form have a name
  assert_select "form input" do
    assert_select "[name=?]", /.+/  # Not empty
  end

  # https://apidock.com/rails/HTML/Selector
  # to target all elements data-test="member-profile-1", data-test="member-profile-2"
  assert_select '[data-test^=member-profile-]', 10

  # not sure how to use substitution, why this does not work
  assert_select '[data-test=member-profile-?]', /.+/, count: 10

  assert_select 'div', html: 'p', minimum: 2
  ~~~

  Error like
  ```
Nokogiri::CSS::SyntaxError: unexpected '$' after '[:equal, "id-1"]'
    (eval):3:in `_racc_do_parse_c'
    (eval):3:in `do_parse'
  ```
  occurs when you forget closing brackets like `assert_selector '[data-test=id-1'`

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
    # you can get all respone text with doc.text.split.join(' ')
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
Add to gemfile

```
  # system test
  gem 'capybara' # 3.35.3
  gem 'selenium-webdriver' # 3.142.7
```
and create
```
# test/application_system_test_case.rb
require 'test_helper'

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  # driven_by :selenium, using: :headless_chrome, screen_size: [1400, 1400]
  driven_by :selenium, using: :chrome, screen_size: [1400, 1400]

  # you can use bye bug, but it will stop rails so you can not navigate to other
  # pages or make another requests in chrome while testing
  def pause
    $stderr.write('Press CTRL+j or ENTER to continue') && $stdin.gets
  end
end
```

Use nameing like ROLE_ACTION_test.rb for example *user_shares_card_test.rb*

System tests are not included in default test suite if you run `rails test` (you
should run `rake test:system`), but if you run specific test than it will be run
`rails test test/system/my_test.rb`.

Note that if you use puma with multiple workers than
`ActionMailer::Base.deliveries.size` could be different in rails process and in
test process, so use `workers 0` in `config/puma/test.rb`.
Or insclude ActionMailer::TestHelper and use assert_emails method.

To limit number of bowsers (default is number of cores) use
```
PARALLEL_WORKERS=1 rails test:system
```

In system test you can't mock OmniAuth.mock_auth so use integration test for
that.

Capybara assertions
<http://www.rubydoc.info/gems/capybara/Capybara/Minitest/Assertions>

* `assert_text /dule/` or `assert_text Regexp.new('dule', Regexp::MULTILINE)`
  (instead of `assert_match /dule/, page.body` or in rspec `expect(page).to
  include 'dule'`
* `assert_selector 'a', text: 'duke'`
   you can assert other html attributes with square brackets
   ```
   assert_selector '#my_id[readonly="readonly"][value="My Name"]'
   ```
   or if you use find.attribute
   ```
   assert_match "avatar_female4", find('[data-test="profile-image"]').native.attribute("src")
   ```
* `assert_equal admin_path, page.current_path` page.url ie page.current_url
  contains also http:// so better is to use page path

## Webmock

```
# Gemfile
gem 'webmock'

# config/initializers/webmock.rb
if Rails.env.test?
  require 'webmock'
  # https://github.com/titusfortner/webdrivers/issues/109
  driver_urls = Webdrivers::Common.subclasses.map do |driver|
    Addressable::URI.parse(driver.base_url).host
  end
  WebMock.disable_net_connect!(allow_localhost: true, allow: driver_urls)
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
  include WebmockHelper.

  # t('successfully') instead I18n.t('successfully')
  include AbstractController::Translation

  # devise method: sign_in user
  include Devise::Test::IntegrationHelpers

  # Setup all fixtures in test/fixtures/*.yml for all tests in alphabetical order.
  fixtures :all

  # assert_emails 1
  include ActionMailer::TestHelper

end
```

```
# test/a/webmock_helper.rb
module WebmockHelpers
  def stub_s3
    stub_request(:get, /s3.amazonaws.com/).to_return(status: 200, body: 'text')
    yield if block_given?
  end

  def stub_ip_info(timezone)
    stub_request(:get, /ipinfo.io/)
      .to_return(headers: {content_type: 'application/json'}, body: {"timezone": timezone}.to_json)
  end
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
https://github.com/seattlerb/minitest#mocks
https://stackoverflow.com/questions/10990622/how-do-i-stub-a-method-to-raise-an-error-using-ruby-minitest
~~~
require "minitest/autorun"

class MyModel
  def my_method; end
end

class TestRaiseException < MiniTest::Unit::TestCase
  def test_raise_exception
    my_instance = MyModel.new
    raises_exception = Proc.new { raise ArgumentError, 'hi' }
    my_instance.stub :my_method, raises_exception do
      e = assert_raises(ArgumentError) { my_instance.my_method }
      assert_equal 'hi', e.message
    end
  end
end
~~~


Stub works also with some custom doubles mock. You can define singleton method
on mock `def mock.apply; end` but can not use other methods or variables... in
this case use `mock.expect :method, return_value`

~~~
# test/models/user_test.rb
require 'test_helper'
class UserTest < ActiveSupport::TestCase
  test '#apply_subscription' do
    mock = Minitest::Mock.new
    def mock.apply; true; end
    # or
    mock.expect :apply, true

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
      system "xdg-open #{image_path} &"
      "Opening screenshot: xdg-open #{image_path}"
    end
  end
  ~~~

* test og meta tags for jekyll
* https://gist.github.com/thbar/10be2ea924b81f78d24ab800461bfee3
* if you want to run specific test (not all tests in one test file) than you
  need to use `require 'rails/all` in `config/application.rb` and run by line
  ```
  ./bin/rails test test/controllers/registrations_controller_test.rb:27
  ```
* to set cookies you can simply use
  ```
  class ApplicationControllerTest < ActionDispatch::IntegrationTest
    test '#use_time_zone' do
      user = users(:user)
      cookies['browser.timezone'] = 'UTC'
      get root_path
    end
  end
  ```
* Using rails native helpers instead of timecop
http://edgeapi.rubyonrails.org/classes/ActiveSupport/Testing/TimeHelpers.html
Use `ActiveSupport::Testing::TimeHelpers#travel` (usefull when you want time to
pass) and `travel_to` `travel_back` (time does not move for the duration of the
test).

# Test background jobs

You can test job directly by calling perform.now
```
# test/jobs/update_person_job_test.rb
class UpdatePersonJobTest < ActiveJob::TestCase
  test 'update score' do
    person = people(:one)
    UpdatePersonJob.perform_now(person, 3)
    assert_eqal 3, person.reload.score
  end
end
```

https://api.rubyonrails.org/classes/ActiveJob/TestHelper.html#method-i-assert_enqueued_jobs
You can use
```
assert_enqueued_with job: MyJob, args: [user]
perform_enqueued_jobs
assert_performed_jobs 1
```
`ActiveJob::Base.queue_adapter = :test` will change queue adapter for all
following test.
You can see differences between queue adapters
<http://api.rubyonrails.org/v5.1.4/classes/ActiveJob/QueueAdapters.html>
There is test helpers like `assert_performed_with` http://api.rubyonrails.org/classes/ActiveJob/TestHelper.html#method-i-assert_performed_with
example of use is
<https://github.com/eliotsykes/rspec-rails-examples/blob/master/spec/jobs/headline_scraper_job_spec.rb>
To send email in background Rails use mailers
[ActionMailer::DeliveryJob](https://blog.bigbinary.com/2018/01/15/rails-5-2-allows-mailers-to-use-custom-active-job-class.html)
so to test sending in minitest

~~~
require 'test_helper'

class ProductTest < ActionDispatch::IntegrationTest
  # if you want to actually perform jobs
  include ActiveJob::TestHelper
  test 'mail' do
    perform_enqueued_jobs only: ActionMailer::DeliveryJob do
      ...
    end
  end

  # https://api.rubyonrails.org/v5.1/classes/ActionMailer/TestHelper.html#method-i-assert_emails
  # or you can assert that it is enqueued
  assert_enqueued_jobs 1 do
  # or you can assert how it is called `_with`
  assert_enqueued_with job: ActionMailer::DeliveryJob, args: [1, 'a'] do

  include ActionMailer::TestHelper
  test 'mail' do
    assert_performed_jobs 2, only: ActionMailer::DeliveryJob do
      product.charge(account)
    end
  end

  # for email you can assert
  assert_difference 'ActionMailer::Base.deliveries.size', 1 do
    MyJob.perform_now
  end


  # to test that no jobs are enqueued
  assert_no_enqueued_jobs do
  end
end
~~~

* test unit https://test-unit.github.io and getting started page
  https://github.com/test-unit/test-unit/blob/master/doc/text/getting-started.md
  ```
  # my.rb
  require "test-unit"

  # here is my code or
  # require "my_code"

  class MyTest < Test::Unit::TestCase
    def test_zero
      assert true
    end
  end
  ```
  and run with `ruby my.rb` or using https://github.com/test-unit/test-unit/blob/master/doc/text/getting-started.md#43-rakefile-no-edit
  omit https://test-unit.github.io/test-unit/en/Test/Unit/TestCaseOmissionSupport.html#omit-instance_method
  ```
  def test_omission
    omit
    # Not reached here
  end
  ```
  hooks https://test-unit.github.io/test-unit/en/Test/Unit/TestCase.html
