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

Integration tests are generic name for any test than combine more than one unit
tests and are often black-box tests, end-to-end, what user see.
Acceptance testing is subset of integration testing and test that behavior is
correct from customer perspective.
There should not be ruby code except setup data or find specific item that we
need to check if it exists on page. We should use only capybara finders and
matchers since we can't see `request` object.
Integration test should not cover special error cases, when data is nil or
implementation details.

~~~
Given [role and state]
When [an event occurs]
Then [the benefit]

or AAA:
Arange - set data
Act - run code
Assert - verify that code did what is expected
# note that multiple assets is OK only if they are related
~~~

# Setup test env

You need to setup db with a command

~~~
bin/rails db:environment:set RAILS_ENV=test
~~~

# Rspec

## Vanilla rspec

~~~
rspec --init
~~~

This will generate `spec/spec_helper.rb` and `.rspec` (option `--require
spec_helper` will automatically load spec_helper so you do not need to write
`require "spec_helper"`). Also rspec will add `spec/` folder to `$LOAD_PATH`

~~~
# spec/simple_test_spec.rb
RSpec.describe "simplest test" do
  it "can check equality" do
    expect(2).to eq(1+1)
  end
end
~~~

You can run simple test example with

~~~
rspec spec/simple_test_spec.rb
~~~

Rspec examples are written using `describe` (ExampleGroup, you can nest multuple
describe blocks), `let` statements and `it` blocks (Example). If you need to
assert count you can use `before {}` (it is `before(:example)` and is run after
`let` statemets) to instantinize all let objects since there are lazy (or you
can use `let!`).
Also you can use `before(:context) { Location.delete_all}` (this is run before
`let` statements) to remove any test data, for example if you have fixtures,
they will be loaded for every example.

~~~
# spec/location_spec.rb
require "location"
describe Location do
  let(:latitude) { 123 }
  let(:longitude) { 321 }
  let(:location) { Location.new latitude: latitude, longitude: longitude }
  before do
    # initiate lazy let when we need to assert count
    [location]
  end
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
  * example groups `describe "..." do` or `context "..." do`. `it "..." do`
  define example. Example group is a class in which `describe` or `context` is
  evaluated, and `it` is evaluated on instance of that class
  * [subject](http://www.relishapp.com/rspec/rspec-core/v/3-5/docs/subject/explicit-subject)
  is used in group scope (`describe`) to define value that is returned by
  `subject` method in example (`it`) scope.
  * [one liner it
  syntax](http://www.relishapp.com/rspec/rspec-core/v/3-5/docs/subject/one-liner-syntax)
  Instead of `it { expect(subject).to be_empty }` you can use
  `it { is_expected.to be_empty }` or `it { should be_empty }` (deprecated). If
  you need to access properties of an object for one liner syntax, than use `it
  { is_expected.to have_attribute :name }`
  * [let](http://www.relishapp.com/rspec/rspec-core/v/3-5/docs/helper-methods/let-and-let)
  is used to memoize value so it is shared inside one example, but not between
  examples
  * [before](http://www.relishapp.com/rspec/rspec-core/v/3-5/docs/hooks/before-and-after-hooks)
  and after hooks are used to execute arbitrary code before and after
  example or context is run. You can use alias `before(:each)` is the same as
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
[have_attributes](https://www.relishapp.com/rspec/rspec-expectations/docs/built-in-matchers/have-attributes-matcher#basic-usage) to check values
* `expect(object).to have_attribute :name` to check just attribute (no value)
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

For pending tests, you can write empty it blocks or just add `skip` or prefix
`x` like `xit` or `xdescribe`

~~~
it "..." do
  skip "pending"
end

xit "skipped" do
end
~~~

For slow test you can mark with `:slow` so running rspec tests with `rspec --tag
~slow` will run only fast tests.

`rspec --profile` will give you list of 10 slowest tests.

## Rspec rails

To install you need to add [rspec-rails](https://github.com/rspec/rspec-rails)
gem and generate `.rspec`, `rails_helper.rb` and `spec_helper.rb`.

~~~
sed -i Gemfile -e '/group :development, :test do/a  \
  gem "rspec-rails"'
bundle
rails generate rspec:install
# you should stub everything rails related so you do not need to load it
# echo "--require rails_helper" >> .rspec
# enable experimental settings, note config.profile_examples = 10 is too noisy
# sed -i spec/spec_helper.rb -e '/=begin/d'
# sed -i spec/spec_helper.rb -e '/=end/d'
git add . && git commit -m "rails generate rspec:install"
~~~

You should write `require "rails_helper"` at the begging of each spec, or you
can add option to `.rspec` (this is perfomance penalty since some tests does not
require rails can perform faster without this option).
Once you put gem in gemfile inside development group, rspec generators will be
available: `rails g model posts` will generate rspec instead of minitest.
Write test file that ends `_spec.rb` in particular folders (so it inherits type
).

You can also use guard, just add to development group

~~~
sed -i Gemfile -e '/group :development do/a  \
  gem "guard-rspec"'
bundle
guard init rspec
git add . && git commit -m "Adding guard init rspec"
~~~

You can run `guard` that will run your specs.

You can use [spring for
rspec](https://github.com/jonleighton/spring-commands-rspec)

~~~
sed -i Gemfile -e '/group :development do/a  \
  gem "spring-commands-rspec"'
bundle
bundle exec spring binstub rspec
~~~

Now you can run tests with `bin/rspec` which will be faster than `rspec`. If you
are using guard, update `guard :rspec, cmd: "bundle exec bin/rspec" do`

## Shoulda matchers

You can use [shoulda matchers](https://github.com/thoughtbot/shoulda-matchers)

~~~
sed -i Gemfile -e '/group :development, :test do/a  \
  gem "shoulda-matchers", "~> 3.1"'
bundle
cat >> spec/rails_helper.rb << HERE_DOC
Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end
HERE_DOC
bundle
git add . && git commit -m "Adding shoulda matchers"
~~~

## RSpec Controller specs

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
only first two. Third is session (used only if you have some multistep process
that use session for continuity) and fourth is flash param (not used in
controller test).
Similar `xhr` (for testing ajax) takes three arguments.

~~~
get :show, params: { id: @task.id }
post :create, logo: fixture_file_upload('/test/data/logo.png', 'image/png')
xhr :post, :create, params: { id: "3" }
~~~

In controller test you have access to `@request`, `@controller` and `@response`
You can validate `response.status` or `expect(response).to
have_http_status(:success)` matcher.  You can use `:success`, `:redirect`,
`:missing`, `:error`.

When you need to create params to for testing, you can use
`ActionController::Parameters.new name: 'Duke'`

## Rspec Requests specs

[request-spec](https://www.relishapp.com/rspec/rspec-rails/docs/request-specs/request-spec)
are used for full stack testing without stubbing. Usually for oauth (doorkeeper
gem)

~~~
# spec/requests/oauth_password_flow_spec.rb
RSpec.describe "OAUTH" do
  let :email_address do
    "test@email.com"
  end
  let :password do
    "asdfasdf"
  end
  let! :user do
    FactoryGirl.create :user, email_address: email_address, password: password
  end

  it "creates a token when credentials are valid" do
    # https://aaronparecki.com/oauth-2-simplified/#password
    post "/oauth/token", params: {
      grant_type: "password",
      username: email_address,
      password: password
    }
    expect(response.status).to eq 200
    expect(JSON.parse(response.body)["access_token"]).not_to be_nil
  end

  it "does not issue token when credentials are invalid" do
    post "/oauth/token", params: {
      grant_type: "password",
      username: email_address,
      password: "wrong_password"
    }
    expect(response.status).to eq 401
    expect(JSON.parse(response.body)["access_token"]).to be_nil
  end
end
~~~

## RSpec Router specs

[router
specs](https://www.relishapp.com/rspec/rspec-rails/v/3-5/docs/routing-specs) is
not so usefull, so you should write only for some the you do not want to exists
or when you have some extra logic in routing.

Usefull matchers are `route_to` and `be_routable`

~~~
# spec/routing/project_routing_spec.rb
require 'rails_helper'
RSpec.describe 'project routing', type: :routing do
  it 'routes projects' do
    expect(get: '/projects/1/edit/').to route_to(
      controller: 'project', action: 'edit', id: 'id')
  end

  it 'does not route by name' do
    expect(get: '/projects/search/duke').not_to be_routable
  end
end
~~~

## RSpec Helper specs

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

## RSpec View specs

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

## RSpec Presenters specs

Bigger logic in view should be replaced with presenter. Ruby core
[SimpleDelegator](https://ruby-doc.org/stdlib-2.1.1/libdoc/delegate/rdoc/SimpleDelegator.html)
delegates all messages to the object passed to the contructor.
We can use test doubles and instead of `rails_helpers` use implicit
`spec_helper`.


~~~
# spec/presenters/project_presenter_spec.rb
RSpec.describe ProjectPresenter do
  let(:project) { Project.new name: 'My Name' }
  let(:project_presenter) { ProjectPresenter.new project }

  it "show name with state" do
    expect(project_presenter.name_with_status).to eq("My Name on_schedule")
  end
end

# app/presenters/project_presenter.rb
class ProjectPresenter < SimpleDelegator
  def name_with_status
    "#{name} on_schedule"
  end
end
~~~

There is `draper` gem

## RSpec Mailer specs

In controller we check if send mail is actually triggered and maybe just one
line of content `email.html_part.body.to_s` matches title

~~~
# spec/controllers/tasks_controller_spec.rb
require 'rails_helper'

RSpec.describe TasksController, type: :controller do
  before(:example) { ActionMailer::Base.deliveries.clear }

  describe "PATCH incomplete update" do
    let(:task) { Task.create! title: "Yo!" }
    it "does not send email" do
      patch :update, id: task.id, task: { size: 3 }
      expect(ActionMailer::Base.deliveries.size).to eq(0)
    end
  end

  describe "PATCH complete update" do
    let(:task) { Task.create! title: "Yo!" }
    it "does send email" do
      patch :update, id: task.id, task: { size: 3, completed: true }
      expect(ActionMailer::Base.deliveries.size).to eq(1)
      email = ActionMailer::Base.deliveries.first
      expect(email.html_part.body.to_s).to match("Yo!")
    end
  end
end
~~~

Full content we check in mailer spec

~~~
# spec/mailers/task_mailer.rb
require "rails_helper"

RSpec.describe TaskMailer, type: :mailer do
  let(:task) { Task.new title: 'Yoo' }
  describe "task_completed_email" do
    let(:mail) { TaskMailer.task_completed_email(task) }

    it "renders the headers" do
      expect(mail.subject).to eq("Task completed email")
      expect(mail.to).to eq(["to@example.org"])
      expect(mail.from).to eq(["from@example.com"])
    end

    it "renders the body" do
      expect(mail.body.encoded).to match("Yoo")
    end
  end
end
~~~

## RSpec Model specs

[model
specs](https://www.relishapp.com/rspec/rspec-rails/docs/model-specs) is a thin
wrapper for `ActiveSupport::TestCase`. Every example is in separate
[transaction](https://www.relishapp.com/rspec/rspec-rails/v/3-5/docs/model-specs/transactional-examples)
You can use
[instance_double](https://www.relishapp.com/rspec/rspec-rails/v/3-5/docs/model-specs/verified-doubles)
If not defined, `subject` is an instance of the model.
Usually have following parts: attributes, relationships, validations.

### Attributes

It is good to write all column names at the top of the test so you can see what
it is about

~~~
# spec/models/auction_spec.rb
RSpec.describe Auction do
  describe "attributes" do
    %w(
      user_id
      ends_at
      time_zone_id
    ).each do |attribute|
      it { is_expected.to have_attribute attribute }
    end

    # or in this one by one form. breakthrough is two. for three and more use
    # above method

    it { is_expected.to have_attribute :ends_at }
    it { is_expected.to have_attribute :time_zone_id }
  end
~~~

### Relationships

~~~
  describe "relationships" do
    it { is_expected.to have_many :auction_admins }
    it { is_expected.to belong_to :user }
  end
~~~

For belongs_to validations it is advised to validate object (`user` not
`user_id`) so you can use `build` strategy in factory girl.

### Validations

You can use [shoulda matchers](https://github.com/thoughtbot/shoulda-matchers)
for simple inline validations (presence for both not null and references
fields), for example:

~~~
  it { is_expected.to validate_presence_of :name }
  it { is_expected.to validate_uniqueness_of :name }
~~~

For complex validation you can test validation as simply `l =
Location.new; expect(l).not_to be_valid` but it's better to go into details of
errors message

~~~
# spec/models/location_spec.rb
require "rails_helper"
RSpec.describe Location do
  describe "validations" do
    before { subject.valid? }
    context "when latitude is missing" do
      subject { Location.new latitude: '' }
      it "should not allow blank latitude and longitude" do
        expect(subject.errors[:latitude]).to include "can't be blank"
        expect(subject.errors[:longitude]).to include "can't be blank"
      end
    end
  end
end
~~~

Validate uniqueness should be explicitly written (not shoulda matchers) with
two objects: `original` (that exists in db) and `duplicate` that is going to be
saved

~~~
  it "validates uniqueness of user_id scoped to auction_id" do
    original = FactoryGirl.create :auction_admin
    duplicate = FactoryGirl.build :auction_admin, user: original.user, auction: original.auction
    duplicate.valid?
    expect(duplicate.errors[:user_id]).to include "has already been taken"
  end
~~~

It is advised to write test for valid factory (so you know that your factory
can be created)

~~~
  it "has a valid factory" do
    expect(FactoryGirl.create(:auction)).to be_persisted
  end

  # or more general

  it "has a valid factory" do
    expect(FactoryGirl.create(described_class.name.underscore)).to be_persisted
  end
~~~

Test enum is with

~~~
  it "has enum fulfillment_typs" do 
    expect(described_class.fulfillment_types).to eq({
      "item" => 0,
      "certificate" => 1,
    })
  end
~~~

Test default values from database and in after hooks.

~~~

~~~

Usually we do not test migration files since we use them later and write test
for that usage.

### Refactoring

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

* session methods you can set expectation for `current_path`
  * `visit "/"`, `visit new_project_path`
  * `within "#login-form" do`
  * [more](http://www.rubydoc.info/github/teamcapybara/capybara/master/Capybara/Session#visit-instance_method)
* node actions target elements by their: id, name, label text, alt text, inner
text. Note that it is case sensitive. You can use substring or you can define
`exact: true`
  * `click_on "Submit"` (both buttons and links) `click_button "Sign in"`,
  `click_link "Menu"`
  * `fill_in "email", with: 'asd@asd.asd'`
  * `check(locator)`, `choose(locator)`, `select(value, from: locator)`, and
  `uncheck`, `unselect`
  * [more](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Actions)
* node finders
  * `find('ng-model="newExpense.amount"').set('123')` [find](http://www.rubydoc.info/github/jnicklas/capybara/Capybara/Node/Finders#find-instance_method) can use css or xpath (see some [xpath examples](scrapper post)) When 
  * `find_all('input').first.set(123)`
  * [more](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Finders)
* node matchers and rspec matchers
  * `expect(page.has_css?('.asd')).to be true`
  * `expect(page).to have_css(".title", text: "my title")`, `have_text`,
  `have_link`, `have_selector("#project_#{project.id} .name", text: 'duke')`
  (`assert_selector` in minitest). `have_no_selector` for opposite. With all you
  can use `text: '...'` and `count: 2` which is number of occurences
  * [more](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Matchers)
  * [rspecmatchers](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/RSpecMatchers)
* node element
  * `find('input').trigger('focus')` (does not work in selenium)
  * [more](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Element)
* confirm dialog
  * selenium `page.driver.browser.switch_to.alert.accept  # can also be .dismiss`
  * webkit `page.accept_confirm { click_link "x" } }` so actions is wrapped with this page.accept_confirm
* `save_and_open_page` to visually inspect the page. It works when `js: false`
and uses `lunchy` gem. It does not load images with relative path (images on
your server).

We can setup data using `let`, or `before` block, or simply inside
`it/scenario`.
[post](https://robb.weblaws.org/2016/10/20/why-i-dont-use-letlet-in-my-rspec/?)
suggests that `let` should be replaced with instance variable.

Fixtures are good for integration tests because they are faster in creating a
lot of objects (in contrast to unit test, where we need clean situation and
a few objects).

Capybara example

~~~
# spec/features/search_spec.rb
require 'rails_helper'

RSpec.feature "Search recipes", js: true do
# write data inside or
  # fixtures :all  # or
  before do
    Recipe.create! name: 'Baked Potato'
  end
  scenario "successfully" do
    visit "/"
    fill_in "keywords", with: "baked"

    click_button "Search"

    expect(page).to have_content("Baked Potato")
    expect(page).to_not have_content("Garlic")
    expect(page.has_link?('Sign up')).to be true
  end
end
~~~

# Debug with selenium


# Rspec Helpers

You an define your helpers and macros in separate file and include it in Rspec
config.

Imporant note that helpers can be used only inside examples (not example groups)

## Custom matchers

~~~
RSpec::Matchers.define :be_able_to_see do |*projects|
  match do |user|
    expect(user.visible_projects).to eq(projects)
    projects.all? { |p| expect(user.can_view?(p)).to be_truthy }
    (all_projects - projects).all? { |p| expect(user.can_view?(p)).to be_falsy }
  end
end

let(:all_projects) { [project_1, project_2] }
expect(user).to be_able_to_see(project_1, project_2)
~~~

## Login helper

In controller tests you can use devise helpers.

~~~
RSpec.describe ProjectsController, type: :controller do
  let(:user) { FactoryGirl.create :user }
  describe "authenticated user" do
    before(:example) do
      sign_in user
    end

    # Note that you can change user and it will affect current_user
    describe "has some projects" do
      before { user.projects << project }
      it "user has project" do
        expect(controller.current_user.projects.count).to eq(1)
      end
    end
~~~

Alternatively, you can stub `current_user`.

In feature tests there we can't use devise helpers but we can manually log in or
use `login_as` warden method which is outside of rails stack

~~~
# spec/support/login_helpers.rb
RSpec.configure do |config|
  # this enable sign_in, sign_out devise methods in controller and view specs
  config.include Devise::Test::ControllerHelpers, type: :controller
  config.include Devise::Test::ControllerHelpers, type: :view
end

include Warden::Test::Helpers
module LoginHelpers
  def create_logged_in_user
    user = FactoryGirl.create(:user)
    # login_as is warden method
    login_as user
    user
  end

  def manually_log_in(user = FactoryGirl.create(:user))
    visit new_user_session_path

    within "#new_user" do
      fill_in "Email", with: user.email
      fill_in "Password", with: user.password
    end

    click_button 'Log in'
  end
end
RSpec.configure do |config|
  config.include LoginHelpers, type: :feature
end
~~~

## Mailer helper

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
  # you can call this manually for specific test
  # config.before(:each) { reset_email }
end
~~~

## Pause helpers

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
{{ site.baseurl }} {% post_url 2017-01-10-get-syntax-right-in-jade-yaml %}#yaml)

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
* fixtures define database column so you need erb to define password hash.

  ~~~
  user:
    email: "test@example.com"
    encrypted_password: <%= User.new.send(:password_digest, 'password') %>
  ~~~

  In factory girl you can use model methods.

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

[Getting
started](https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md#configure-your-test-suite)

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
use when it is used. Note that you do not use `_id` for field name, use class
name as usuall. In validation, you can validate presence for object but not id
`validates :user, presence: true` (but not `validates :user_id, presence: true`)

  ~~~
  factory :comment do
    user
    # or
    association :user
    # if you want that user is build but not save to database
    association :user, strategy: :build
    # you can define class and additional attributes
    association :updated_by, factory: :user, name: "Admin User"
  end
  ~~~

* transient attributes can be used to set some configuration for dynamic
attributes, that is not part of model attributes. Mostly used for has_many
associations and *create_list* method

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
  preferred way of creating data.

* multiple records can be `build_list :user, 25` or `create_list :user, 25` or
`build_stubbed_list :user, 25`

You can debug in rails console. After debugging test data will stay in database
so you need to clean it manually

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
do should not define all attributes.

Use fixtures for integration or complex controller tests. Use factories for unit
tests.

# Minitest

Minitest is included in ruby.

Rails `TestCase`s (like `ActiveSupport::TestCase`) inherits from
`Minitest::Test` so you can use it `def test_password`, but Rails adds `test`
method so it can be used like `test "my password" do`.  In minitests you can not
have same test descriptions since it will be converted to same
`test_description`.

You can run specific line `rails test test/models/article_test.rb:6` or name
`rails test -n test_example`
or using ENV variables `rake test TEST=test/machine_with_callbacks_test.rb
TESTOPTS="--name=test_should_run_validations_for_specific_state -v"`

Model test looks like

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
* `assert_raises(NameError) do end`
* `assert_match(expected, actual)`
* `assert_includes(collection, object)`
* `assert_equal(expected, actual)`

They all have oposite `refute_` methods.
They all accept additional string param that will be error message.

Rails also defines `assert_difference`, `assert_blank`, `assert_presence`

To run test you can `rake test` or `rake test
test/controllers/posts_controller_test.rb`.

Each test run will load all fixture data, run setup blocks, run test method, run
teardown block,rollback fixtures.

## Minitest rails controllers

You can use `ActiveSupport::Test` or `ActionDispatch::IntegrationTest` which
gives you `assert_redirected_to post_url(Post.last)` and `post`, `get`...
methods. You have access to `@request`, `@controller` and `@response` object.
Also `assert_response :success` (for http codes 200-299), `:redirect` (300-399),
`:missing` (404), `:error` (500-599). `assigns` and `assert_template` also
exists.

~~~
# test/controllers/projects_controller_test.rb
require 'test_helper'

class ProjectsControllerTest < ActionDispatch::IntegrationTest
  test "the create method creates project" do
    post projects_url, params: { project: { name: 'Duke' } }
    assert_redirected_to projects_path
    # assert_equal "Duke", assigns[:action].project.name
  end
end
~~~

## Minitest views

View can be tested with `assert_select` similar to capybara `have_selector` but
separate implementation [html
selector](http://edgeapi.rubyonrails.org/classes/HTML/Selector.html)

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

## Minitet mocha feature tests

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

# System testing

System tests are not included in default test suite, you should run `rake
test:system`

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

A test double is fake object or method used in place of real when it is:

* difficult to create it (network failure)
* to isolate env so it tests only specific method
* to test behavior during this test (what is called and how) and not just end
results.

A **STUB** is test double that returns predetermined value for a method call
(without calling actual method on actual object). It is defined using
`allow().to receive().and_return()`. It can also use `and_raise(Exception,
"message")`.

~~~
allow(thing).to receive(:name).and_return("Duke")
~~~

A **MOCK** is similar to stub, but it sets expectation that method will actually
be called. It use `expect().to receive.and_return`. If it is not called mock
object triggers test failure. We can also set with which arguments it should be
called ([more about with
arguments](https://relishapp.com/rspec/rspec-mocks/v/3-5/docs/setting-constraints/matching-arguments))

~~~
expect(thing).to receive(:name).and_return("Duke")

service_double = instance_double(CreatesProject, create: true)
expect(CreatesProject).to receive(:new)
  .with(name: 'Duke', tasks_string: nil)
  .and_return(service_double)
~~~

A **SPY** is similar to mock but we set expectation later using `allow().to
receive` and `expect().to have_received()` (so we follow
Given/When/Then structure)

~~~
allow(thing).to receive(:name).and_return("Duke")
# body of test
expect(thing).to have_received(:name)
~~~

**PARTIAL DOUBLE** is when you stub particular methods of real object, a **FULL
DOUBLE** is when you use totally fake object that responds only to specified API
(so we do not care about behavior, only about public interface). RSpec defines
full double with `double`. If you do not want that exception is raised when some
other method is called than you can `double(:user).as_null_object` or
`spy(:user)`

~~~
# this will raise exception for user.country
user = double(:user, name: 'Duke')
# this will not raise exception for user.country
user = spy(:user)
~~~

When you want to mimic specific object you can use **VERYFING DOUBLE** to check
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

# Testing External service

* **client** is our app, **server** is external service, **adapter** is between
client and server, **fake server** returns a fake response
* *smoke test* is using real server, not used often, but could be helpfull
* *integration test* is using fake server
* *client unit test* ends at adapter

# When not to write tests

* when things are too hard to test, for example setting up model that external
services need to use
* too trivial tests
* overtesting when we use same model method on severall controllers
* exploratory coding (code that will not go to production)

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
* big method does not clearly separate individual steps, depend on side effects
rher than return values.
* first write initial state, than simple successful path than alternate
successful paths and than error edge cases that break code (one by one)
* there is a difference between integration test (when we do not use doubles)
and top level unit test, when we use stubs since we only test that method is
calling other methods, which will be unit tested separately. We could write unit
test for those other methods first, that is bottom up approach, we need to fake
data. Symetrically, top down approach is when we first write top level unit test
and fake calls to other methods. You should cover edge cases in top level unit
test (using doubles), and not writting a lot of integration tests which are slow
and more likely to fail as it needs to call all other methods.
* on legacy app, when code is intertwined, there are a lot of dependencies
between parts of the code, it is hard to write unit test. You can start with
feature tests, than write characterization test `(assert(result).to
eq('correct')` that are used only to check old behavior, than use TDD to write
new tests and code, and than refactor while characterization tests keep old
behavior.
* do not use test doubles too much when code has a lot of depencencies. It is
better that test is fragile than writting/updating test double setup

> Overly aggressive test doubles that set unneeded expectations are also a
> common cause of test fragility, so if a test fails because of an expectation
> on the double, it’s useful to ask whether the expectation needs to be there in
> the first place.

* you do not need to write tests for part of the framework.
* YAGNI is not used in tests
* repeat Red-Green-Refactor cycle in small increments, because a lot of changes
will have a lot of places where code could broke. Refactoring step should not
need to change any tests, just code.

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

# Opensource examples for tests

* <https://github.com/discourse/discourse>
* <https://github.com/manshar/manshar> rails and angular
* <https://github.com/scottwillson/racing_on_rails> minitests
* <https://github.com/bikeindex/bike_index> rspec, api, swagger
* <http://stackoverflow.com/questions/4421174/what-are-some-good-example-open-source-ruby-projects-that-use-cucumber-and-rspec>

# some examples

https://gist.github.com/kyletcarlson/6234923

[olympic](https://github.com/mabranches/Olympic) swagger, jsonapi,

[Mocking Luca Pradovera](https://www.youtube.com/watch?v=YjHKetQxk5I)
[Carlos Souza](https://www.youtube.com/watch?v=CDC7zA8a-mA)

# Live Coding

* [Live coding Phil
Sturgeon list](https://www.youtube.com/watch?v=Q00H6BPVNQI&list=PL6XWIjBFDFOIiJyDK61thx6ILIbaUMEAb) author of Building APIs your won't hate
* [gregg pollack](https://www.youtube.com/watch?v=jk016koiMAI)
[carlos souza](https://www.youtube.com/watch?v=GV37oAofoak)
* [Sean Devine Live code a charity auction
application](https://www.youtube.com/playlist?list=PL93_jRSrU7hzEUNmevlnMeUx6F35Za638)
* [thoughbot workshop](https://www.youtube.com/watch?v=sj5TXzgZ1Sk) also https://www.youtube.com/watch?v=8368hGNIJMQ
* [matt smith](https://www.youtube.com/watch?v=H1Czc3NL4c4)
* [Paul Hiatt](https://www.youtube.com/watch?v=9f08KzNO4qo)

Rspec book <http://www.betterspecs.org/>

TODO

https://www.youtube.com/watch?v=yTkzNHF6rMs
https://vimeo.com/44807822
https://www.youtube.com/watch?v=9f08KzNO4qo
https://m.patrikonrails.com/how-i-test-my-rails-applications-cf150e347a6b
https://robots.thoughtbot.com/headless-feature-specs-with-chrome
https://building.buildkite.com/5-ways-weve-improved-flakey-test-debugging-4b3cfb9f27c8
[aceptance
testing](http://blog.arkency.com/2017/06/acceptance-testing-ruby-using-actors-personas)

