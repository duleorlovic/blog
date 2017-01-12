---
layout: post
title: Testing in Rails
---

Usually we start with integration test, than we go in details for frontend
controller (Angular), than backend controller, than model ... all to make
sure integration (feature) test pass.
You should follow [style guide for
testing](https://github.com/thoughtbot/guides/tree/master/style/testing) and
read some [best
practices](https://github.com/thoughtbot/guides/tree/master/best-practices).

Integration tests usually test only hapy path. All possible edge cases should be
unit tested. External api should be stubbed and repeatable.

Acceptance testing is synonym for integration testing, but focus is only on that
user see. There should not be ruby code except setup data, or find specific item
that we need to check if it exists on page. We should use only capybara finders
and matchers since we can't see `request` object.

~~~
Given [role and state]
When [an event occurs]
Then [the benefit]

or AAA: Arange Act and Assert
~~~

# Setup test env

You need to setup db with a command

~~~
bin/rails db:environment:set RAILS_ENV=test
~~~

# Rails default mini testing

Rails `TestCase`s (like `ActiveSupport::TestCase`) inherits from
`Minitest::Test` so you can use it `def test_password`, but Rails adds `test`
method so it can be used like `test "my password" do`

You can run specific line `rails test test/models/article_test.rb:6` or name
`rails test -n test_example`

Rails calls "integration tests" as "request tests" and it covers both controller
and view code. If logic is more complex, it should be written in model anyway,
and use lower level tests.

# Rspec

## Vanilla rspec

~~~
rspec --install
~~~

This will generate `spec/spec_helper.rb` and `.rspec` (option `--require
spec_helper` will automatically load spec_helper so you do not need to write
`require "spec_helper"`). Also rspec will add `spec/` folder to `$LOAD_PATH`

Rspec examples are written using `describe` (ExampleGroup, you can nest multuple
describe blocks), `let` statements and `it` blocks (Example). You can use
`before {}` to instantinize all let objects since there are lazy.  Test looks
like

~~~
# spec/location_spec.rb
require "location"
describe Location do
  let(:latitude) { 123 }
  let(:longitude) { 321 }
  let(:location) { Location.new latitude: latitude, longitude: longitude }
  subject { location }
  describe "#initialize" do
    it { should have_attributes latitude: latitude }
    it { should have_attributes longitude: longitude }
  end
  describe "#near" do
    context "for near location" do
      let(:near_location) do
        Location.new(latitude: latitude + 1, longitude: longitude + 1)
      end
      it { should be_near(near_location, 2) }
    end
    context "for not near location" do
      let(:not_near_location) { Location.new latitude: 1, longitude: 2 }
      it { should_not be_near(not_near_location, 2) }
    end
    it "raise exception for negative radius" do
      expect { location.near?(location, -1) }.to raise_error ArgumentError
    end
  end
end
~~~

* [rspec-core](http://www.relishapp.com/rspec/rspec-core/docs) define:
  * example groups `describe "..." do context "..." do it "..." do`. `it` define
  example. Example group is a class in which `describe` or `context` is
  evaluated, and `it` is evaluated on instance of that class
  * [subject](http://www.relishapp.com/rspec/rspec-core/v/3-5/docs/subject/explicit-subject)
  is used in group scope to define value that is returned by `subject` method in
  example (`it`) scope.
  * [one liner it
  syntax](http://www.relishapp.com/rspec/rspec-core/v/3-5/docs/subject/one-liner-syntax)
  Instead of `it { expect(subject).to be_empty }` you can use
  `it { is_expected.to be_empty }` or `it { should be_empty }`. If
  you need to access properties of an object for one liner syntax, than use `it
  { should have_attributes name: 'duke' }`
  * [let](http://www.relishapp.com/rspec/rspec-core/v/3-5/docs/helper-methods/let-and-let)
  is used to memoize value so it is shared inside one example, but not between
  examples
  * [before](http://www.relishapp.com/rspec/rspec-core/v/3-5/docs/hooks/before-and-after-hooks)
  and after hooks are used to execute arbitrary code before and after
  example/context is run. You can use alias `before(:each)` is the same as
  `before(:example)` and `before(:all) == before(:context)`

Some [rspec
expectations](http://www.relishapp.com/rspec/rspec-expectations/docs) built in
matchers

* `expect(object).to be(expected)`, `expect(object).to be >
3`, `expect(object).to eq(expected)`...
  * `be` is object identity `a.equal? b` (refer to the same object)
  * `eq` is object equivalence with type conversion `a == b` (same value)
* [predicate
matchers](http://www.relishapp.com/rspec/rspec-expectations/docs) for any
method that begin with`has_` or ends with `?` you can use `have_` and `be_`
`expect(object).not_to be_empty` or `be_near near_location`
* `expect { do_something }.to change { object.attribute }`
[change](http://www.relishapp.com/rspec/rspec-expectations/v/3-5/docs/built-in-matchers/change-matcher)
* `expect(object).to have_attributes name: 'duke'`
[have_attributes](https://www.relishapp.com/rspec/rspec-expectations/docs/built-in-matchers/have-attributes-matcher#basic-usage)
* `expect { do_something }.to raise_error ArgumentError`
* `expect(object).to match /expression/`
* `expect(object).to be_a(Class)`
* `expect(object).to satisfy { block }`

You can use `not_to` for all matchers (expect raise_error).


Instead of multiple `expect` statements inside `it`, you could change it to
`describe` and use inline `it` statements (pros: all assertions will be checked,
cons: it could be slower because of setup and dificult to read to know how test
was setup). Another solution is to use compound matchers.
All matchers have alternate form (an_object_having...) so it reads better when
using compose (or compound) matchers

~~~
expect(a[0]).to eq(5)
expect(a[1]).to eq(6)
# is the same as
expect(a).to match([an_object_eq_to(5), an_object_eq_to(6)])

# or

describe "build three tasks" do
  let(:tasks_string) { "My Task:2\nYour Task:3\nHis Task:1\n" }
  it do
    expect(tasks.map(&:title)).to eq(['My Task', 'Your Task', 'His Task'])
  end
  it { expect(tasks.map(&:size)).to eq([2, 3, 1]) }
end
# is the same as
describe "build three tasks" do
  let(:tasks_string) { "My Task:2\nYour Task:3\nHis Task:1\n" }
  it do
    expect(tasks).to match(
      [an_object_having_attributes(title: 'My Task', size: 2),
       an_object_having_attributes(title: 'Your Task', size: 3),
       an_object_having_attributes(title: 'His Task', size: 1)]
    )
  end
end
~~~

You can use `.or` and `.and`

~~~
expect(a).to include("a").and match(/.*1.*/)
~~~

[rubydoc
matchers](http://www.rubydoc.info/github/rspec/rspec-expectations/RSpec/Matchers)

For pending tests, you can write empty it blocks or just add skip or x

~~~
it "find" do
  skip "broken"
end

xit "skipped" do
end
~~~

## Rspec rails

To install you need to add [rspec-rails](https://github.com/rspec/rspec-rails)
gem and generate `.rspec`, `rails_helper.rb` and `spec_helper.rb`.

~~~
sed -i Gemfile -e '/group :development, :test do/a  \
  gem "rspec-rails"'
bundle
rails generate rspec:install
echo "--require rails_helper" >> .rspec
git add . && git commit -m "rails generate rspec:install"
~~~

Once you put gem in gemfile inside development group, rspec generators will be
available: `rails g model posts` will generate rspec instead of minitest.
Write test file that ends `_spec.rb` in particular folders (so it inherits type
).
You should stub everything rails related so you do not need to load it and it
will be faster.
You should also use guard, just add to development group

~~~
sed -i Gemfile -e '/group :development do/a  \
  gem "guard-rspec"'
bundle
guard init rspec
git add . && git commit -m "Adding guard init rspec"
~~~

You should write `require "rails_helper"` at the begging of each spec, or you
can add option to `.rspec` (this is perfomance penalty since some tests does not
require rails can perform faster without this option).

## Router specs

[router
specs](https://www.relishapp.com/rspec/rspec-rails/v/3-5/docs/routing-specs) is
not so usefull, so you should write only for some the you do not want to exists

~~~
RSpec.describe "routes", type: :routing do
  it "home should route to pages" do
    expect(get("/")).to route_to("pages#index")
  end
end
~~~

## Controller specs

[controller
specs](https://www.relishapp.com/rspec/rspec-rails/docs/controller-specs) can
use `get`,`post` and validate `response` (to `render_template :new`, to
`redirect_to posts_path`).
Testing `assings` and `render_template` is deprecated.

~~~
subject { get "/posts" }
expect(subject).to render_template(:index)
expect(assigns(:posts)).to be([post])
~~~

>  assigns has been extracted to a gem. To continue using it, add `gem
>  'rails-controller-testing'` to your Gemfile.

Move all business logic and ActiveRecord calls out from controller and test only
triggers to services or model methods (using mocks) and check responses. Testing
routes is done separately.

Controller spec should be for each method (CRUD) in context of authorized and in
context of unauthorized user (testin security), and for valid and some invalid
requests.

`get`, `delete`, `head`, `patch`, `post` and `put` takes 4 params, usually used
only first two. Third is session (used only is you have some multistep process
that use session for continuity) and fourth is flash param (not used in
controller test).
Similar `xhr` (for testing ajax) takes three arguments.

~~~
get :show, params: { id: @task.id }
post :create, logo: fixture_file_upload('/test/data/logo.png', 'image/png')
xhr :post, :create, params: { id: "3" }
~~~

In controller test you have access to `@request`, `@controller` and `@response`
You can validate `response.status` with `have_http_status(:success)` matcher.
You can use `:success`, `:redirect`, `:missing`, `:error`.

## Router specs

Usually do not need to write router tests but if you have some extra logic in
routing than you should. Usefull matchers are `route_to` and `be_routable`

~~~
# spec/routing/project_routing_spec.rb
require 'rails_helper'
RSpec.describe 'project routing' do
  it 'routes projects' do
    expect(get: '/projects/1/edit/').to route_to(
      controller: 'project', action: 'edit', id: 'id')
  end

  it 'does not route by name' do
    expect(get: '/projects/search/duke').not_to be_routable
  end
end
~~~

## Helper specs

You have access to `helper` object so you can call methods on it.

~~~
# spec/helpers/project_helper_spec.rb
require 'rails_helper'
RSpec.describe ProjectsHelper do
  let(:project) { Project.new name: 'Duke' }
  it "adds class if not finished" do
    allow(project).to receive(:on_schedule?).and_return(true)
    actual = helper.name_with_status(project)
    expect(actual).to have_selector('span.on_schedule', text: 'Duke')
  end
end

# app/helpers/projects_helper.rb
module ProjectsHelper
  def name_with_status(project)
    content_tag(:span, project.name, class: 'on_schedule')
  end
end
~~~

Helper tests does not go to views, so to test if it actually works write some
view tests

## View specs

You can set instance variables and call `render` (which will render outermost
describe block name or you can call `render 'projects/index'` or `render
partial: 'projects/_form', locals: { admin: true }`).
We can stub helpers using view object like
`view.stub(:current_user).and_return(User.new)`
Output is available in `rendered` object.
We can use capybara `have_selector`, `have_no_selector` instead of default
`match` rspec matcher.
We can use rails `assert_select`
[assert_select](http://edgeapi.rubyonrails.org/classes/ActionDispatch/Assertions/SelectorAssertions.html#method-i-assert_select)

~~~
# spec/views/projects/index.html.erb_spec.rb
require 'rails_helper'

RSpec.describe "projects/index" do
  let(:on_schedule) { Project.create! name: 'Dule' }
  it "show on schedule" do
    @projects = [on_schedule]
    render
    expect(rendered).to have_selector(
      "#project_#{on_schedule.id} .on-schedule",
      text: on_schedule.name,
      count: 1
    )
    assert_select ".on-schedule", count: 1
  end
end
~~~

## Model specs

[model
specs](https://www.relishapp.com/rspec/rspec-rails/docs/model-specs) is a thin
wrapper for `ActiveSupport::TestCase`. Every example is in separate
[transaction](https://www.relishapp.com/rspec/rspec-rails/v/3-5/docs/model-specs/transactional-examples)
You can use
[instance_double](https://www.relishapp.com/rspec/rspec-rails/v/3-5/docs/model-specs/verified-doubles)

* validations. You can test validation as simply `l =
Location.new; expect(l).not_to be_valid` but it's better to go into details of
errors property

~~~
RSpec.describe Location do
  describe "validations" do
    before { subject.valid? }
    context "when latitude is missing" do
      subject { Location.new latitude: '' }
      it "should not allow blank latitude and longitude" do
        expect(subject.errors[:latitude]).to include("can't be blank")
        expect(subject.errors[:longitude]).to include("can't be blank")
      end
    end
  end
end
~~~

Refactoring in 3 types:

* break up complexity: boolean logic `has_purchased_before?`, convert local
variables to methods and extract other methods from comments (part of methods)
* combine duplication: duplication of fact (use class constant or instance
method), duplication of logic (use before actions), find missing abstractions
(replace some group of methods that are used together as parameters, with a
new class with those instance methods, like value objects)

~~~
class User < ActiveRecord::Base
delegate :full_name, :sort_name, to: :name
def name
  Name.new(first_name, last_name)
end

class Name
  attr_accessor :first_name, :last_name
  def initialize(first_name, last_name)
    @first_name, @last_name = first_name, last_name
  end
  def full_name
    "#{first_name} #{last_name}"
  end
  def sort_name
    "#{last_name}, #{first_name}"
  end
end
~~~

* usually do not test associations
* testing class methods, do not create more objects than it is needed
  * when testing filter, than create two objects, one that satisfy and other
  does not

  ~~~
  it "finds completed tasks" do
    complete = Task.create!(completed_at: 1.day.ago, title: 'Completed')
    incomplete = Task.create!(completed_at: nil, title: 'Incompleted')
    expect(Task.complete.map(&:title)).to eq(['Completed'])
    # or
    expect(Task.complete).to match(
      [an_object_having_attributes(title: 'Completed')]
    )
  end
  ~~~

  * when testing sort method, create three objects with first in the middle

* testing included modules and concerns is done with [shared
examples](https://www.relishapp.com/rspec/rspec-core/docs/example-groups/shared-examples)
so we test with `it_behaves_like "linkable"` or `include_examples 'linkable'`
* class methods could be stubbed

~~~
require 'rails_helper'
RSpec.describe User do
  context ".latest" do
    let(:user) { build :user, :published }
    before { allow(User).to receive(:latest).and_return([:user]) }
    it { expect(User.latest).to include user }
  end
end

RSpec.describe "Post #create" do
  context 'when password is invalid' do
    it 'renders the page with error' do
      user = create(:user)
      post :create, session: { email: user.email, password: 'invalid' }
      expect(response).to render_template(:new)
      expect(flash[:notice]).to match(/^Email and password do not match/)
    end
  end
end
~~~

# Capybara

Instead of using `get` and `response.body` we can use only what end user see
`visit`, `fill_in` and `page.should`. Also it can use the
same DSL to drive browser (selenium-webdriver or capybara-webkit) or headless
drivers (Rack::Test or phantomjs).
To run in browser javascript use `js: true`. Note that in this mode, drop down
links are not visible, you need to click on dropdown. Also `data-confirm` will
be ignored.
Note that in this mode you can't use `rails-rspec` default
`config.use_transactional_fixtures` since selenium can't know that refresh or
navigation to another page should be in the same transaction, so you need to use
`database_cleaner` as we do configuration below.
For newer Firefox I needed to download
[geckodriver](https://github.com/mozilla/geckodriver/releases) and put it
somewhere like `/user/local/bin/geckodriver`. Also [Firefox
47.0.1](https://ftp.mozilla.org/pub/firefox/releases/47.0.1/) is suggested, but
my ver 50 also works.

Capybara is used only with
[feature
spec](https://www.relishapp.com/rspec/rspec-rails/docs/feature-specs/feature-spec)
Just install the gem in require it:

~~~
sed -i Gemfile -e '/group :development, :test do/a  \
  gem "capybara"\
  gem "selenium-webdriver"\
  # for save_and_open_page in browser\
  gem "launchy"\
  # to clean db when using js mode\
  gem "database_cleaner"'
bundle
sed -i spec/rails_helper.rb -e '/require .rspec.rails/a  \
require "capybara/rspec"'

sed -i spec/rails_helper.rb -e '/use_transactional_fixtures/c  \
  # http://stackoverflow.com/questions/12326096/capybara-selenium-fault-and-redirect-example-com-when-without-everything-is-gre/12330557#12330557\
  config.use_transactional_fixtures = false\
  config.before :each do\
    DatabaseCleaner.strategy = if Capybara.current_driver == :rack_test\
                                 :transaction\
                               else\
                                 # this is when metadata js: true\
                                 :truncation\
                               end\
    DatabaseCleaner.start\
  end\
  config.after do\
    DatabaseCleaner.clean\
  end'
~~~

Capybara adds some aliases:

* `feature` is alias for `describe ..., type: :feature`
* `scenario` is an alias for `it`
* `given` is alias for `let`

Some of the most used capybara methods
[link](https://gist.github.com/duleorlovic/042178b92f1badc09490) or [cheat
sheet](https://thoughtbot.com/upcase/test-driven-rails-resources/capybara.pdf)

* [session
methods](http://www.rubydoc.info/github/teamcapybara/capybara/master/Capybara/Session#visit-instance_method)
  * `visit "/"`
  * `within "#login-form" do`
* [node
actions](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Actions)
  * `click_button "Sign in"`
  * `fill_in "email", with: 'asd@asd.asd'`
* [node
finders](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Finders)
  * `find('ng-model="newExpense.amount"').set('123')` [find](http://www.rubydoc.info/github/jnicklas/capybara/Capybara/Node/Finders#find-instance_method) can use css or xpath (see some [xpath examples](scrapper post)) When 
  * `find_all('input').first.set(123)`
* [node
matchers](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Matchers)
and similar
[rspecmatchers](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/RSpecMatchers)
  * `expect(page.has_css?('.asd')).to be true`
  * `expect(page).to have_css(".asd")`, `have_text`, `have_link`,
  `have_selector("#project_#{project.id} .name", text: 'duke')`
* [node
element](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Element)
  * `find('input').trigger('focus')` (does not work in selenium)
* confirm dialog
  * selenium `page.driver.browser.switch_to.alert.accept  # can also be .dismiss`
  * webkit `page.accept_confirm { click_link "x" } }` so actions is wrapped with this page.accept_confirm
* `save_and_open_page` to visually inspect the page (it works when js: false)


We can setup data using `let` or simply inside `it`.
[post](https://robb.weblaws.org/2016/10/20/why-i-dont-use-letlet-in-my-rspec/?)
suggests that `let` should be replaced with instance variable.

## Capybara example

~~~
# spec/features/search_spec.rb
require 'rails_helper'

RSpec.feature "Search recipes", js: true do
  before do
    Recipe.create! name: 'Baked Potato'
  end
  scenario "find recipes" do
    visit "/"
    fill_in "keywords", with: "baked"

    click_button "Search"

    expect(page).to have_content("Baked Potato")
    expect(page).to_not have_content("Garlic")
    expect(page.has_link?('Sign up')).to be true
  end
end
~~~

## Helpers

You an define your helpers and macros in separate file and include it in Rspec
config

~~~
# spec/support/login_helpers.rb
include Warden::Test::Helpers

module LoginHelpers
  def create_logged_in_user
    user = FactoryGirl.create(:user)
    login(user)
    user
  end

  def manually_log_in
    visit "/"
    user = FactoryGirl.create(:user)
    within "#login-form" do
      fill_in "email", with: user.email
      fill_in "password", with: user.password
    end

    click_button 'Sign in'
  end

  private

  def login(user)
    login_as user, scope: :user
  end
end

RSpec.configure do |config|
  config.include LoginHelpers
end
~~~

~~~
# spec/support/mailer_macros.rb
module MailerMacros
  def last_email
    ActionMailer::Base.deliveries.last
  end
  def reset_email
    ActionMailer::Base.deliveries = []
  end
end
RSpec.configure do |config|
  config.include(MailerMacros)
  config.before(:each) { reset_email }
end
~~~

~~~
# spec/support/pause_helpers.rb
module PauseHelpers
  def pause
    $stderr.write 'Press enter to continue'
    $stdin.gets
  end
end
RSpec.configure do |config|
  config.include PauseHelpers, type: :feature
end
~~~

# Fixtures

All data is cleared between tests. Although we touch database (huge third party
dependency) we call it *unit* test. All data is available on each tests. Rails
starts db transaction at the begining of each tests and roll back at the end of
test (if you do not want transaction you can disable
`config.use_transactional_fixtures = false`)

We define it using [yml file](
{{ site.baseurl }} {% post_url 2017-01-10-get-syntax-right-in-jade-yaml %}#yaml))

~~~
# test/fixtures/projects.yml
my_project:
  name: My Project Name
  due_date: 2017-01-01
~~~

In tests you can access those data with `projects(:my_project)`.
Fixture loading mechanism bypasses ActiveRecord validations.
You can use yaml identifier to create associations

~~~
simple_task:
  title: Simple task
  project: my_project
~~~

You can use erb to add dynamic attributes or to specify multiple entries

~~~
<% 10.times do |i| %>
task_<%= i %>:
  name: "Task <%= i %>"
<% end %>
~~~

Pros:

  * fixtures are usefull for global semi static data (like job_types)
  * very fast since all data is loaded at once. It's better than factory girl
  since `FactoryGirl.create :task` 10 times will create 10 projects (association
  objects) as well.

Cons:

 * fixtures are global so all edge cases will increase database data for each
 test
 * fixture files are for specific models, so sometime it is hard to track
 complex setup (comment for that task for this project for that owner)
 * fixtures are distant and far away of test so you need look into it to figure
 out actual test setup
 * fixtures define database column so you need erb to define password hash. In
 factory girl you can use model methods.

# Factory Girl

After you install

~~~
sed -i Gemfile -e '/group :development, :test do/a  \
  gem "factory_girl_rails"'
bundle
mkdir spec/support
cat >> spec/support/factory_girl.rb << HERE_DOC
# so we do not need to prefix with FactoryGirl.create :user
RSpec.configure do |config|
  config.include FactoryGirl::Syntax::Methods
end
HERE_DOC

sed -i spec/rails_helper.rb -e '/require .spec_helper/a  \
require "support/factory_girl"'
~~~

You can create factories in `spec/factories.rb` or `spec/factories/*.rb`. It is
recomended to use only one file since it should be bare minimum, and should not
change while the application grows (although it could get bigger).
So write minimum that meets validation and use inheritance to create variations
of data.

Note that
[link](https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md#configure-your-test-suite)

* factory name is used to determine the class (it has to be same or you need to
use `class: Project`), dynamic attributes are defined with a block
`date_of_birth { 11.years.ago }` in which you can use other attributes

~~~
# spec/factories.rb
FactoryGirl.define do
  factory :project do
    full_name { "exiting #{name}" }
    name "My Project"
    due_date Date.parse("2017-01-10")
  end
end
~~~

* belongs_to associations attributes, you do not need to write any attributes if
it match the class name, or you can specify which factory and which strategy to
use when it is build

   ~~~
   factory :comment do
     user
     # or
     association :updated_by, factory: :user, name: "Admin User"
   end
   ~~~

* transient attributes can be used to set some configuration for dynamic
attributes, that is not part of model attributes. Mostly used for has_many
associations

  ~~~
  factory :post do
    user
  end

  factory :user do
    transient do
      posts_count 5
    end

    # the after(:create) yields two values; the user instance itself and the
    # evaluator, which stores all values from the factory, including transient
    # attributes; `create_list`'s second argument is the number of records
    # to create and we make sure the user is associated properly to the post
    after(:create) do |user, evaluator|
      create_list(:post, evaluator.posts_count, user: user)
    end
  end
  ~~~

* callbacks `after(:create) do`, `after :build`, `before :create`, `after :stub`
* inheritance: nest multiple `factory :user_with_posts` or use `parent: :user`
attribute
* sequences are used to generate values (not AR objects) which will be
incremented every time it is called during test (test sessions is whole `rake
test` process or `rails c -e test` session). It is used for columns that
need uniqueness like email field. Usually only one field in table can be
sequence, other can use it with dynamic attributes

  ~~~
  # seq could be defined outside factory and reused
  sequence :username { |n| "username#{n}" }
  factory :user do
    username
    another_uniq_field { generate :username }
    sequence(:email) { |n| "user#{n}@asd.asd" }
  end
  ~~~

* traits are used to group attributes so you can apply them to any factory. They
can be used with hash also `build :user, :admin, name: 'Mike'`

  ~~~
  factory :user do
    name "Duke"
    username { name }

    trait :admin do
      username { "admin-#{name}" }
      is_admin true
    end

    factory :user_admin do
      admin
    end
  end
  ~~~

* 4 ways of using: you can pass the block to any of it, or you can override with
hash attributes `build(:user, name: 'My Name')`
  * `build(:user)` it is not saved but if you have associated `belongs_to
  :project` than it will trigger `create(:project)` to get `project_id` and all
  its associated objects... you can try with `strategy: :build` but that could
  be a problem since associated_ids do not exists. Solution is to use
  `build_stubbed` or do not define association at all and define them in tests
  (code need to be testable without associations)
  * `create(:user)` it is saved. Use this only when need to test find/query db.
  * `attributes_for(:user)` get hash of attributes
  * `build_stubbed(:user)` object with all AR attributes stubbed out (like
  `save`). It will raise an exception if they are called. Very fast since we do
  not create AR objects. It has rails ID and we can use associations. This is
  the best way of creating data.

* multiple records can be `build_list :user, 25` or `create_list :user, 25` or
`build_stubbed_list :user, 25`

You can debug in rails console. Note that data will stay in database so you need
to clean it manually

~~~
RAILS_ENV=test rake db:drop db:create db:migrate
rails c -e test
require "factory_girl";include FactoryGirl::Syntax::Methods
#  require "./spec/factories";
build :user, name: 'Dule'
~~~

Factory girl can't be used as seed file as suggested [thoughtbot
post](https://robots.thoughtbot.com/factory_girl-for-seed-data) mainly because
it is used to create new data (not to use existing) and used in a way that you
do not need to define attributes.

Use fixtures for integration or complex controller tests. Use factories for unit
tests.

# Minitest

For Minitest you need to install minitest gem and require it.
Run with colors `ruby my_class_test.rb -p`, or `ruby -r
minitest/pride my_class_test.rb` or put `require 'pride'` in test file.

~~~
require 'minitest/autorun'
class MyClassTest < Minitest::Test
  def test_sample
    assert MyClass.new
  end

  def test_that_is_skipped
    skip "later"
  end
end
~~~

Assertions: `assert true`, `assert_raises(NameError) do` ...
[all](http://docs.seattlerb.org/minitest/Minitest/Assertions.html)

# Opensource examples for tests

* <https://github.com/discourse/discourse>
* <https://github.com/manshar/manshar> rails and angular
* <https://github.com/scottwillson/racing_on_rails> minitests
* <https://github.com/bikeindex/bike_index> rspec, api, swagger
* <http://stackoverflow.com/questions/4421174/what-are-some-good-example-open-source-ruby-projects-that-use-cucumber-and-rspec>

# Testing time and date

Use `ActiveSupport::Testing::TimeHelpers#travel` (usefull when you want time to
pass) and `travel_to` `travel_back` (time does not move for the duration of the
test).

Most ActiveSupport methods (such as `6.days.ago`) returns `DateTime`. If you
want to compare with `Date` objects than you need to convert `.to_date`, or
use `.to_time` or `.to_datetime`. The most robust way to compare is to use
`to_s(:db)` for example `5.days.ago.to_date.to_s(:db)`

When you want to set particular `created_at` than you can do like for any other
attribute, in fixture or update method. Sometimes for `updated_at` you need to
prevent rails callbacks with `Project.record_timestamps = false;
@project.updated_at = 5.days.ago; @project.save; Project.record_timestamps =
true`

# Using test doubles as mocks and stubs

A test double is fake object or method used in place of real when it is
difficult to create it (network failure), to isolate env so it tests only
specific method, and to test behavior during this test (what is called and how)
and not just end results.

A stub is test double that returns predetermined value for a method call
(without calling actual method on actual object). It can also use
`and_raise(Exception, "message")`.

~~~
allow(thing).to receive(:name).and_return("Duke")
~~~

A mock is similar to stub, but it sets expectation that method will actually be
called. If it is not called mock object triggers test failure. We can also set
with which arguments it should be called [more about with
arguments](https://relishapp.com/rspec/rspec-mocks/v/3-5/docs/setting-constraints/matching-arguments)

~~~
expect(thing).to receive(:name).and_return("Duke")

service_double = instance_double(CreatesProject, create: true)
expect(CreatesProject).to receive(:new)
  .with(name: 'Duke', tasks_string: nil)
  .and_return(service_double)
~~~

A spy is similar to mock but we set expectation later (so we follow
Given/When/Then structure)

~~~
allow(thing).to receive(:name).and_return("Duke")
# body of test
expect(thing).to have_received(:name)
~~~

*Partial double* is when you stub particular methods of real object, a *full
double* is when you use totally fake object that responds only to specified API
(so we do not care about behavior, only about public interface). RSpec defines
full double with `double`. If you do not want raised exception when some other
method is called than you can `double(:user).as_null_object` or `spy(:user)`

~~~
# this will raise exception for user.country
user = double(:user, name: 'Duke')
# this will not raise exception for user.country
user = spy(:user)
~~~

When you want to mimic specific object you can use *verifying double* to check
if message passed to double is actually real method on real object.

~~~
instance_user = instance_double(User) # only `method_defined?` messages
class_user = class_double(User) # only `repond_to?` messages
object_user = object_double(User.new) # only `respond_to?` messages
# usefull since it see dynamically defined methods with `method_missing`
~~~

All those have spy forms `instance_spy`, `class_spy` and `object_spy` giving
verification that message exists without having to specify a return value.

In RSpec 3 configuration `mocks.verify_partial_doubles = true` will verify all
partial doubles so you can not stub non existing method.

You can use mocks for controller tests, so you do not need to know valid and
invalid params and a bug in CreatesProject will not fail this test. Usually when
I use double to mock some class, than use small integration test I cover both
classes (so I know that change of class API is not an issue here). It is fine
that stubbed method returns a stub or double, but is not recomended that double
contains other stub or doubles (`double(create: true, project: double(name:
'Duke', tasks: [double(title: 'Hi')]))`).

~~~
RSpec.describe ProjectsController, type: :controller do
  it "create a project" do
    fake_action = instance_double(CreatesProject, create: true)
    expect(CreatesProject).to receive(:new)
      .with(name: 'Duke', tasks_string: nil)
      .and_return(fake_action)
    post :create, params: { project: { name: 'Duke' } }
    expect(response).to redirect_to projects_path
  end
  it "goes back to the form on failure" do
    action_stub = double(create: false, project: Project.new)
    expect(CreatesProject).to receive(:new).and_return(action_stub)
    post :create, params: { project: { name: 'na' } }
    expect(response).not_to redirect_to projects_path
  end
  it "fails update gracefully" do
    sample = Project.create!(name: "Test Project")
    expect(sample).to receive(:update_attributes).and_return(false)
    allow(Project).to receive(:find).and_return(sample)
    patch :update, id: sample.id, project: {name: "Fred"}
    expect(response).to render_template(:edit)
  end
end
~~~

* usually do not stub methods that you have not written. Extract external
service to separate classes. Usually test double could still works but actual
object does not receive or return reasonable value. If you mock external code
which could easilly change, than your tests will still pass. One way to deal
with it is to create wrappers

  ~~~
  class Project
    def self.create_from_controller(params)
      create(params)
    end
  end

  it "creates a project" do
    allow(Project).to receive(:create_from_controller).and_return(Project.new)
  end
  ~~~

* if you need big fake objects it's better to use stubs rather than mocks. Do
not need to set expectation for current test (cover that object separatelly). We
just stub external call and use some returned value. Use mock only when you need
to trigger sending email.

# Tips

* write tests that descibe behavior not implementation, so it does not change
when implementation changes (completed? could be boolean true/false, or presence
of completed_at),
* only when dealing with edge cases you can look at implementation (time/dates
and off-by-one error)
* all unit tests should be (Rails 4 Test Prescriptions book):
  * Straightforward: since method should do one thing we should split tests into
  smaller semantically meaningful parts which are easy to understand. Do not use
  tricks to write short test code, since it should be simple.
  * Well defined: repeatability could be problem in case of datetime, random and
  third-party calls.
  * Independent: does not depend on other tests (single line of code breaks
  multiple tests or order of test cause failure) or external data like fixtures
  * Fast: big startup time, dependencies that create a lot of objects, database
  usage
  * Truthful: do not fail while code still works (in view tests we can replace
  exact match "Some title" with "#id.class" that rarely change). or do not pass
  while code has an issue (mock objects)
* if you find yourself writting tests that already pass, that means you are
writing too much code. You shoult apply "sliming", write only code that make
makes test pass
* first write initial state, than simple successful path than alternate
successful paths and than error edge cases that break code (one by one)

# TODO:

Antipaterns

<http://code.tutsplus.com/articles/antipatterns-basics-rails-tests--cms-26011>

Resources:

* http://railscasts.com/episodes/275-how-i-test



~~~
# spect/support/geocoder.rb
Geocoder.configure(lookup: :test)

Geocoder::Lookup::Test.set_default_stub(
  [
    {
      'latitude'     => 40.7143528,
      'longitude'    => -74.0059731,
      'address'      => 'New York, NY, USA',
      'state'        => 'New York',
      'state_code'   => 'NY',
      'country'      => 'United States',
      'country_code' => 'US',
      'city'         => 'New York',
    }
  ]
)
~~~

Acts as taggable

If you have `acts_as_taggable_on :cuisines` you can create with `_list` method: `FactoryGirl.create :user, cuisine_list: ['American', 'Indian']`.

In fixtures you should put only what is neccesarry to create object (only validated field) so our test do not need to know about validations when they test something different. Define all neccessary stuf (like Tags) in test on the fly. Do not let your tests depend on fixtures. 

Some references:

* guidlines [betterspecs](http://betterspecs.org/)

Devise 

https://github.com/plataformatec/devise/wiki/How-To:-Test-controllers-with-Rails-3-and-4-(and-RSpec)#controller-specs


Use [clearance backdoor](https://github.com/thoughtbot/clearance#fast-feature-specs) for bypassing login and simply `visit my_profile_path(as: user)`


# some examples 

https://gist.github.com/kyletcarlson/6234923

https://www.youtube.com/watch?v=YjHKetQxk5I
