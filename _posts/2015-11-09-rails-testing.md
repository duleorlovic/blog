---
layout: post
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

As a instructor
I want to be able to sign in/sign/out/reset my password

- Standard password reset via email
- Login via social buttons

or AAA:
Arange - set data - SETUP
Act - run code - EXERCISE
Assert - verify that code did what is expected - VERIFICATION
Teardown - eventual TEARDOWN
# note that multiple assets is OK only if they are related
~~~

# Setup test env

You need to setup db with a command

~~~
bin/rails db:environment:set RAILS_ENV=test
~~~

or use `rake db:test:prepare`. I was trying to do `RAILS_ENV=test rake db:drop
db:create db:migrate` but it did not work because of some missing tables.

Capybara javascript test also rely on precompiled assets so you need to run
`rake assets:precompile` or put `config.assets.compile = true` in
`conig/environments/test.rb` (this could slow down).
Somehow when running with headless driver than I do not need to precompile
assets for those js tests (this is when I run `rake`).
But when I run single test `rspec ./spec/features/my_spec.rb` I need to `rm -rf
public/assets` (do not know how they appear here) and also `spring stop` so than
it will use fresh code (not precompiled)

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
rspec -b # to show full backtrace in case of exceptions
~~~

When you want to stop on first failed test (and not to wait whole test suite to
finish) you can use

~~~
rspec spec --fail-fast
~~~

Rspec examples are written using `describe` (ExampleGroup, you can nest multuple
describe blocks), `let` statements and `it` blocks (Example). If you need to
assert count you can use `before {}` (it is `before(:example)` and is run after
`let` statemets) to instantinize all let objects since there are lazy (or you
can use `let!`).
Also you can use `before(:context) { Location.delete_all}` (this is run before
`let` statements) to remove any test data, for example if you have fixtures,
they will be loaded for every example.
[thoughtbot](https://github.com/thoughtbot/guides/blob/master/style/testing/avoid_let_spec.rb)
do not suggest using let, but to extract methods.

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

[rspec-core](http://www.relishapp.com/rspec/rspec-core/docs) and [style
guide](https://github.com/reachlocal/rspec-style-guide)
* example groups `describe "..." do` or `context "..." do`.
Describe use hash `#method_name` and dot `.class_method_name`
Context is alias for describe, but it is used to group examples with same state
(for example `context 'when not logged in` or `context 'when resource is not
found'`). Context description always start with `when` and always have a oposite
context.
* example is `it "..." do`. `it` description should not end with conditional
like this bad example: `it 'returns name if it is present'` (better `context
'when name is present' it 'returns name'`)
* Example group is a class in which `describe` or `context` is evaluated, and
`it` is evaluated on instance of that class
* [subject](http://www.relishapp.com/rspec/rspec-core/v/3-5/docs/subject/explicit-subject)
is used in group scope (`describe`) to define value that is returned by
`subject` method in example (`it`) scope.
* [one liner it
syntax](http://www.relishapp.com/rspec/rspec-core/v/3-5/docs/subject/one-liner-syntax)
Instead of `it { expect(subject).to be_empty }` you can use
`it { is_expected.to be_empty }` or `it { should be_empty }` (deprecated). If
you need to access properties of an object for one liner syntax, than use `it
{ is_expected.to have_attribute :name }`. So do not use `subject` inside `it`
block.
* [let](http://www.relishapp.com/rspec/rspec-core/v/3-5/docs/helper-methods/let-and-let)
is used to memoize value so it is shared inside one example, but not between
examples
* [before](http://www.relishapp.com/rspec/rspec-core/v/3-5/docs/hooks/before-and-after-hooks)
and after hooks are used to execute arbitrary code before and after
example or context is run. You can use alias `before(:each)` is the same as
`before(:example)` and `before(:all) == before(:context)`

DRY is accomplished with `shared_context` (and `shared_examples`) so you can
share contex so do not need to repeat let and before.

~~~
RSpec.describe User do
  shared_context 'one_user' do
    let(:user) { create :user }
  end

  describe do
    include_context 'one_user'
  end
end
~~~

Testing included modules and concerns is done with
[shared_examples](https://www.relishapp.com/rspec/rspec-core/docs/example-groups/shared-examples)
so we test with `it_behaves_like "linkable"` or `include_examples 'linkable'`

~~~
RSpec.shared_examples 'payment check_no' do
  it 'allows long check_no' do
    check_no = '356431 575017004 201705 11'
    customer_payment = create :customer_payment, check_no: check_no
    expect(customer_payment.new_record?).to be_falsy
  end
end

RSpec.describe CustomerPayment do
  describe 'validations' do
    it_behaves_like 'payment check_no'
  end
end
~~~

Also valid factory can be shared between tests

~~~
# spe/models/user_spec.rb
RSpec.describe User do
  it_behaves_like 'has_valid_factory'
end

# spec/support/factory_bot.rb
RSpec.shared_examples 'has_valid_factory' do
  it 'has a valid factory' do
    expect(create(described_class.name.underscore)).to be_valid
  end
end
RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
end
~~~

Some [rspec
expectations](http://www.relishapp.com/rspec/rspec-expectations/docs) built in
matchers

* `expect(object).to be(expected)`, `expect(object).to be >
3`, `expect(object).to eq(expected)`...
  * `be` is object identity `a.equal? b` (refer to the same object)
  * `eq` is object equivalence with type conversion `a == b` (same value)
* [predicate
matchers](http://www.relishapp.com/rspec/rspec-expectations/docs) for any
method that begin with `has_` or ends with `?` you can use `have_` and `be_`
`expect(object).not_to be_empty` or `be_near near_location`
* [type
matchers](https://relishapp.com/rspec/rspec-expectations/v/3-7/docs/built-in-matchers/type-matchers)
  `expect(obj).to be_kind_of(Type)` or `be_a` type of Type

* `expect { do_something }.to change { object.attribute }`.
  You can also specify values `expect {}.to change
  {session[:me]}.from(nil).to("dule")`
  It is also possible to detect changes in two tables
  <http://www.relishapp.com/rspec/rspec-expectations/v/3-5/docs/built-in-matchers/change-matcher>

  ~~~
  it "should increment the counters" do
    expect { Foo.bar }.to change { Counter.count }.by(1).and \
                          change { AnotherCounter.count }.by(1)
  end
  ~~~

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
it '...', :skip do
end

it "..." do
  skip "pending"
end

xit "skipped" do
end
~~~

For slow test you can mark with `:really_slow` and exclude specific tags with
`rspec --tag ~really_slow`. Or you can add to add to `spec/spec_helper.rs`

~~~
# spec/spec_helper.rb
RSpec.configure do |config|
  config.filter_run_excluding(really_slow: true)
end
~~~

Note that if you run specific line `bin/rspec spec/features/my_spec.rb:10` than
it will not be excluded. Only if you run all or speficic file `bin/rspec
spec/features/my_spec.rb`

If you want to run specific task then than add tag `rspec --tag really_slow`. If you
want to run all and specific tasks than use env variable:

~~~
# spec/spec_helper.rb
RSpec.configure do |config|
  # If you really want to run then than add tag `bin/rspec --tag really_slow` to run
  # only that tag, or to run all and some tags use
  # INCLUDING_TAGS=chrome_remote_selenium bin/rspec
  %i(
    really_slow chrome_remote_selenium
    headless_chrome_remote_selenium edit_hosts
  ).each do |tag|
    config.filter_run_excluding(tag) unless ENV['INCLUDING_TAGS'].to_s.include? tag.to_s
  end
end
~~~

You can exclude specific folder
<https://relishapp.com/rspec/rspec-core/v/3-3/docs/configuration/exclude-pattern>
`rspec --exclude-pattern "spec/features/*"` I do not know why quotes are needed
here, but it works only with quotes.

`rspec --profile` will give you list of 10 slowest tests.

Rspec config before each example for particular group

~~~
# spec/rails_helper.rb
RSpec.configure do |config|
  config.before :each do
    DatabaseCleaner.start
  end
  config.before :each, type: lambda {|v| v != :feature } do
    allow_any_instance_of(Paperclip::Attachment).to receive(:save).and_return(true)
  end
  config.include Warden::Test::Helper, type: :features
end
~~~

Better is you include `config.before` only for specific tests using filter
symbols
https://relishapp.com/rspec/rspec-core/v/3-6/docs/hooks/filters#filtering-hooks-using-symbols

~~~
RSpec.configure do |config|
  config.before :example, :setup_xxx do
    xxx
  end
end
RSpec.describe 'invoke xxx', :setup_xxx do
  it 'in all nested examples' do
  end
end
~~~

Change capybara driver for specific tests

~~~
RSpec.configure do |config|
  config.before(:each, remote_selenium: true) do
    Capybara.current_driver  = :remote_selenium
  end
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
Also there are generators like `rails g integration_test` and rspec will
generate `spec/requests/password_resets_spec.rb`

Write test file that ends `_spec.rb` in particular folders (so it inherits type
).

# Guard

You can also use guard, just add to development group

~~~
sed -i Gemfile -e '/group :development do/a  \
  gem "guard-rspec"'
bundle
guard init rspec
git add . && git commit -m "Adding guard init rspec"
~~~

You can run `guard` that will run your specs.

More options for
[guard](https://github.com/guard/guard-rspec#list-of-available-options)
My favorite is `failed_mode: :focus` which reruns last failed test.
Also you can use [spring for
rspec](https://github.com/jonleighton/spring-commands-rspec)

~~~
sed -i Gemfile -e '/group :development do/a  \
  gem "spring-commands-rspec"'
bundle
bundle exec spring binstub rspec
~~~

Now you can run tests with `bin/rspec` which will be faster than `rspec`. If you
are using guard, use this line

~~~
# Guardfile
guard :rspec, cmd: "bundle exec bin/rspec", failed_mode: :focus do
end
~~~


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
context of unauthorized user (testing security), and for valid and some invalid
requests.

`get`, `delete`, `head`, `patch`, `post` and `put` takes 4 params, usually used
only first two. Third is session (used only if you have some multistep process
that use session for continuity) and fourth is flash param (not used in
controller test).
(in `ActionController::TestCase` you can use `session:` keyword arg like `get
:show, params:{ id: 3}, session: { 'user_id': 3}`, but in
`ActionDispatch::IntegrationTest` you can not set session (no `@session`))
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

To set a [host name](https://stackoverflow.com/a/29037481)
* `host! "my.awesome.host"` for integration test [available
helpers](http://guides.rubyonrails.org/testing.html#helpers-available-for-integration-tests)
* in controller specs use `@request.host = 'my.awesome.host'`
(`ActionController::TestCase`)
* In view specs use `controller.request.host = "my.awesome.host"`
* In capybara use `Capybara.default_host = "http://my.awesome.host"`,

When you need to create params to for testing, you can use
`ActionController::Parameters.new name: 'Duke'`

~~~
# spec/controllers/users_controller_spec.rb
RSpec.describe UsersController do
  describe 'new' do
  end
end
~~~

## Rspec Request specs

[requests spec](https://www.relishapp.com/rspec/rspec-rails/docs/request-specs/request-spec)
are used for full stack testing without stubbing.
It is faster than system but can not use capybara...
Usually for oauth (doorkeeper gem).
If request is not performed (`get` `xhr`) than something is different (current
user is not initialized, or something).
`post url, name: 'my name'` is used without `params` key
To set json request use `get path, format: :json`.
To set headers you can use third param

```
  get login_path, nil, { 'Authentication' => 'MyToken' }

```

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
    FactoryBot.create :user, email_address: email_address, password: password
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

  it 'redirect' do
    post '/'
    expect(response).to redirect_to error_path
    follow_redirect!
  end
end

expect(response.body).to include 'Logout'
# use html encoding when you have quote, so you do not need to match &#39;
expect(response.body).to include ERB::Util.html_escape "Some text with ' quote"
expect(response.body.scan(/<tr>/).size).to be 3
expect(response.body).to match /button.*Publish Package.*\/button/
# for multiline math response.body text you can use modifier m
expect(response.body).to match /dl.*dl/m
~~~

Sometimes I get error if you use `sign_in user` but do not perform `get` or
`post` requests, ie you use empty `it 'bla bla' do;end`. Than it pass when
specific tests are run, but fail when all test from file are run.

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
  before { ActionMailer::Base.deliveries.clear }

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
Usually have following parts: attributes, relationships, validations,
hooks(callbacks), scopes and other methods.

[model cheatsheet](https://gist.github.com/kyletcarlson/6234923)

### Attributes

It is good to write all column names at the top of the test so you can see what
it is about

~~~
# spec/models/auction_spec.rb
RSpec.describe Auction do
  describe "attributes" do
    %w[
      user_id
      ends_at
      time_zone_id
    ].each do |attribute|
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
`user_id`) so you can use `build` strategy in factory bot.

### Validations

You can install [shoulda
matchers](https://github.com/thoughtbot/shoulda-matchers) for simple inline
validations:

~~~
# Gemfile
  gem 'shoulda-matchers', '~> 3.1'

# spec/rails_helper.rb
require 'shoulda/matchers'
Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end
~~~

You should check for presence for both not null and references fields, with
simple line:

~~~
  descibe 'validations' do
    subject { build :sport } # we need to use factory bot since implicit subject is empty
    it { is_expected.to validate_presence_of :name }
    it { is_expected.to validate_uniqueness_of :name }
  end
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
    original = FactoryBot.create :auction_admin
    duplicate = FactoryBot.build :auction_admin, user: original.user, auction: original.auction
    duplicate.valid?
    expect(duplicate.errors[:email]).to include "has already been taken"
  end
~~~

It is advised to write test for valid factory (so you know that your factory
can be created)

~~~
  it "has a valid factory" do
    expect(FactoryBot.create(:auction)).to be_persisted
  end

  # or more general

  it 'has a valid factory' do
    # expect(create(described_class.name.underscore)).to be_persisted
    expect(build(described_class.name.underscore)).to be_valid
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

## Test migration

You can test data migration. Do not perfom migration, come just before it

~~~
rake db:drop db:create db:migrate VERSION=20180503095528
rake db:migrate:status
You have 1 pending migration:
  20180508073700 RemoveLocationVoucherType
# Run `rake db:migrate` to update your database then try again.
# I tried to ignore with `config.active_record.migration_error = false` but it
# gives me error message. You should move or rename migration file while you run
rake db:drop db:create db:migrate && bin/rspec spec/migrations/my_migration_specc.rb
~~~~

You should also rename test file so it is not run with all other, or ignore
specific tag metadata.

~~~
class MyMigration < ActiveRecord::Migration
  # this is local Location class used only inside this migration
  class Location < ActiveRecord::Base
  end
  def up
    rename_column :locations, :name, :name2
    # this is needed since renamed columns will be ignored on "save"
    Location.reset_column_information
    Location.find_each do |location|
      location.name2 = 'updated in migration'
      location.save!
    end
  end
end
~~~

I tried with
[Migrator.migrate](https://github.com/xevix/activerecord_migration_testing_example/blob/53ee8c52f5c4d21a616f4a2f40cdc2481ccbd73c/spec/migrations/move_sales_price_to_user_item_spex.rb)
but nothing happens (no sql migration commands), so I simply run
`MyMigration.new.up`

~~~
# spec/migrations/my_migration_spec.rb
load 'spec/spec_helper.rb'
load 'tmp/20180508073700_remove_location_voucher_type.rb'
# run test with this command
# rake db:drop db:create db:migrate STEP=20180503095528 && bin/rspec spec/migrations/remove_location_voucher_type_spec.rb --tag migration_specs
RSpec.describe RemoveLocationVoucherType, migration_specs: true do

  it 'up' do
    # ActiveRecord::Migrator.migrate(migrations_paths, previous_version)
    my_user = create :user
    MyMigration.new.up
    my_user.reload
    expect(my_user.name2).to be 'updated in migration'
  end
end
~~~

If there is an error than skip DatabaseCleaner

~~~
# error

# spec/support/database_cleaner.rb
RSpec.configure do |config|
  config.around(:each) do |example|
    if example.metadata[:migration_specs]
      example.run
    else
      # Start transaction
      DatabaseCleaner.cleaning do
        example.run
      end
    end
  end
end
~~~


# Refactoring

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

You can stub request.remote_ip

~~~
allow_any_instance_of(ActionDispatch::Request).to receive(:remote_ip).and_return('192.168.0.1')
~~~

Also you can stub helper methods

~~~
allow_any_instance_of(ApplicationHelper).to receive(:generate_password).and_return '111000'
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

Instead of using `get` and `response.body` we can use only what end user see
`visit`, `fill_in` and `page.should`. Also it can use the
same DSL to drive browser (selenium-webdriver, chrome-driver or capybara-webkit)
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

Open new tab

~~~
# http://www.rubydoc.info/github/jnicklas/capybara/Capybara/Window
    old_window = page.driver.browser.window_handles.last
    new_window = page.open_new_window
    page.switch_to_window new_window
    # page.current_window.close
    page.driver.browser.close # this will close tab, not whole window
    page.switch_to_window old_window
~~~

# Capybara

Capybara is used only with
[feature
spec](https://www.relishapp.com/rspec/rspec-rails/docs/feature-specs/feature-spec)
Just install the gem in require it:

~~~
sed -i Gemfile -e '/group :development, :test do/a  \
  gem "capybara"\
  gem "capybara-email"\
  gem "selenium-webdriver"\
  # for save_and_open_page in browser\
  gem "launchy"\
  # to clean db when using js mode\
  gem "database_cleaner"'
bundle
sed -i spec/rails_helper.rb -e '/require .rspec.rails/a  \
require "capybara/rspec"\
require "capybara/email/rspec"'

cat > spec/support/database_cleaner.rb << HERE_DOC
RSpec.configure do |config|
  # If you're not using ActiveRecord, or you'd prefer not to run
  # each of your examples within a transaction, remove the following
  # line or assign false instead of true.
  config.use_transactional_fixtures = false

  # Clean up and initialize database before
  # running test exmaples
  config.before(:suite) do
    # Truncate database to clean up garbage from
    # interrupted or badly written examples
    DatabaseCleaner.clean_with(:truncation)

    # Seed dataase. Use it only for essential
    # to run application data.
    load "#{Rails.root}/db/seeds.rb"
  end

  config.around(:each) do |example|
    # Use really fast transaction strategy for all
    # examples except 'js: true' capybara specs
    # that is Capybara.current_driver != :rack_test
    # https://github.com/DatabaseCleaner/database_cleaner#what-strategy-is-fastest
    DatabaseCleaner.strategy = example.metadata[:js] ? :truncation : :transaction

    # Start transaction
    DatabaseCleaner.cleaning do

      # Run example
      example.run
    end

    load "#{Rails.root}/db/seeds.rb" if example.metadata[:js]

    # Clear session data
    Capybara.reset_sessions!
  end
end
HERE_DOC

sed -i spec/rails_helper.rb -e '/use_transactional_fixtures/c  \
  config.use_transactional_fixtures = false'

git add . && git commit -m "add capybara"
~~~

[capybara-email](https://github.com/DockYard/capybara-email) is using
`open_email 'my@email.com'` to set `current_email` for which you can
`current_email.click_link`

Capybara adds some aliases:

* `feature` is alias for `describe ..., type: :feature` or `context`
* `background` is alias for `before`
* `scenario` is an alias for `it`
* `given` is alias for `let`

Some of the most used capybara methods
[link](https://gist.github.com/duleorlovic/042178b92f1badc09490) or [cheat
sheet](https://thoughtbot.com/upcase/test-driven-rails-resources/capybara.pdf)

**Session methods** you can set expectation for `current_path` or `current_url`
`page` is actually `Capybara::Session` class so better is to use `sessions`
name. When you find some element you get `Capybara::Node::Element`, but you can
create new node without going to selenium, using html text

~~~
node = Capybara.string <<-HTML
  <ul>
    <li id="home">Home</li>
    <li id="projects">Projects</li>
  </ul>
HTML
node.class # => Capybara::Node::Simple
node.find('#projects').text # => 'Projects'
~~~

* `visit "/"`, `visit new_project_path`. Remember that if you want to go to
  different domain than `Capybara.app_host` than you need to use full url (with
  protocol) so no `visit 'www.google.com` but rather `visit
  'http://google.con'`. Any relative url will use `Capybara.app_host` just note
  that it also needs protocol `Capybara.app_host = 'http://...'` otherwise error
  `undefined method  +  for nil:NilClass`
* `within "#login-form" do`
* generate capybara post request using submit (this does not work for `js:
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
* [more](http://www.rubydoc.info/github/teamcapybara/capybara/master/Capybara/Session#visit-instance_method)

**Node actions** target elements by their: id (without `#`), name, label text,
alt text, inner text
[more](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Actions)
Note that locator is case sensitive. You can NOT use css or xpath (this is
only for finders). You can use substring or you can define `exact: true`

* `click_on "Submit"` (both buttons and links) `click_button "Sign in"`,
`click_link "Menu"`. Find can be used for click, for example
`find('.class').click` but I prefer to enable aria labels and use that
`Capybara.enable_aria_label = true` and `click 'my-aria-label'`
If there are many buttons, you can use `click_on 'Edit', match: :first`
To check if element exists you can use `has_css?('.sm', text: 'my', wait: 0)` do
not wait if it does not exists (for nokogiri it is
`response.at('.sm:contains("my")')`
* `fill_in "email", with: 'asd@asd.asd'` locator is input name, id, test_id,
  placeholder, label text. Note that it is case sensitive.
  alternative is find set `find("input[name='cc']").set 'asd@asd.asd'`, or using
  javascript `page.execute_script "$('#my-id').val('asd@asd.asd')"`
* to click on select2 I use `find('#original-select-id+span').click` so we find
  first next sibling of original select which was disabled and replaced by
  select2 spans. Also works `find('li', text: select.label).click` but I can not
  find that `li` in dom.
* `check 'my checkbox'` (or by id but without # `check 'my_check_id'`), `choose
 'my radio button'`, `select 'My Option or Value', from: 'My Select Box'`, also
 uncheck `uncheck 'my checkbox'`, `unselect`, `attach_file 'Image',
 '/path/to/image.jpg'`

**Node finders** [find](http://www.rubydoc.info/github/jnicklas/capybara/Capybara/Node/Finders#find-instance_method) can use css, xpath, or text (see some [xpath examples](scrapper post))

* `find 'th', text: 'Total Customers'`
* `find('ng-model="newExpense.amount"').set('123')`
* `find_all('input').first.set(123)` but I think it is better to use
  `find('input', match: :first)` since it do not need to find all
* `find('[data-test="id"]', visible: false)` to find invisible element
* `find('#selector').find(:xpath, '..')` find parent node of selector
  `.find(:xpath, '../..')` is parent of parent (grandparent).
* label as child of next adjacent
  `<h3>Name2</h3><div><label>enabled</label><label>disabled></div>` is
  ```
  find(:xpath, "//h3[contains(text(),'Name2')]/following-sibling::div/label[contains(text(),'enabled')]")
  ```

If you need to fill_in iframe than you can access it by id or number

~~~
  within_frame 0 do
  end
~~~

If you need to switch jump into new window opened by target `_blank` than you
can
~~~
new_window = window_opened_by { click_link 'Something' }
# or
new_window = page.driver.browser.window_handles.last

page.within_window new_window do
  # code
end
~~~

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
  * `expect(page).to have_selector("input[value='John']")`
  * `expect(page).to have_field('Your name', with: 'John')` this does not match
  disabled input field. of you want to match disabled use `have_field('Your
  name', disabled: true, with: 'John')`

Node element [more](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Element)
* `find('input').trigger('focus')` (does not work in selenium)

Confirm dialog
* selenium `page.driver.browser.switch_to.alert.accept  # can also be .dismiss`
* webkit `page.accept_confirm { click_link "x" } }` so actions is wrapped with
this `page.accept_confirm`

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

## Oauth specs

https://github.com/elizabrock/coursewareofthefuture/blob/25372c671f73525e09927344d8f2031b6acf82d0/spec/support/features/authentication_helper.rb

Controller tests https://gist.github.com/jittuu/792715
Helper https://gist.github.com/spyou/1200365/b0bb95e5cf9144b5a9a58bb6c1fc33aee4c34e47


# Rspec Helpers

You an define your helpers in separate file and include it in Rspec
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

~~~
# spec/a/equal_when_sorted_by_id.rb
require 'rspec/expectations'

# When using `Timecop.freeze Date.parse('2018-06-06 10:00:00') do` than ordering
# from database could be random (since created_at are all the same) so you might
# get failed tests when comparing some scopes.
RSpec::Matchers.define :equal_when_sorted_by_id do |expected|
  match do |actual|
    actual.sort_by(&:id) == expected.sort_by(&:id)
  end
end
~~~

For flash messages in feature tests

~~~
# spec/a/flash_message_feature_helpers.rb
# instead of expect(page).to have_selector 'div', text: 'Flash message', visible: false
# you should use: expect(page).to have_flash_message 'Flash message'

require 'rspec/expectations'

RSpec::Matchers.define :have_flash_message do |expected|
  match do |page|
    expect(page).to have_selector 'div', text: expected, visible: false
  end
  failure_message do |page|
    "expected find text '#{expected}' in #{page.body}"
  end
end
~~~

## Login helper

In controller tests you can use devise helpers.

~~~
RSpec.describe ProjectsController, type: :controller do
  let(:user) { FactoryBot.create :user }
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

Since in Rails 5, instead of controller tests, we should use integration helper
and use `sign_in user`

~~~
class ActiveSupport::TestCase
  # provide `sign_in user`
  include Devise::Test::IntegrationHelpers
end
~~~

Alternatively, you can stub `current_user`.

https://github.com/plataformatec/devise/wiki/How-To:-Test-controllers-with-Rails-3-and-4-(and-RSpec)#controller-specs

Use [clearance
backdoor](https://github.com/thoughtbot/clearance#fast-feature-specs) for
bypassing login and simply `visit my_profile_path(as: user)`


In feature tests there we can't use devise helpers but we can manually log in or
use `login_as` warden method which is outside of rails stack

~~~
# spec/support/features/authentication_helpers.rb
module Features
  module AuthenticationHelpers
    def user_manual_sign_in(email, password)
      visit new_user_session_path
      fill_in 'Email', with: email
      fill_in 'Password', with: password
      click_button 'Sign In'
    end

    def create_logged_in_user
      user = FactoryBot.create(:user)
      login_as user
      user
    end

    def mock_facebook_auth(user = {})
      # info https://github.com/omniauth/omniauth/wiki/Integration-Testing
      auth = {
        provider: 'facebook',
        uid: user.try(:facebook_uid) || '1234567',
        info: {
          email: user.try(:email) || 'joe@bloggs.com',
          name: user.try(:name) || 'Joe Bloggs',
          first_name: 'Joe',
          last_name: 'Bloggs',
          image: 'http://graph.facebook.com/1234567/picture?type=square',
          verified: true
        },
        credentials: {
          token: 'ABCDEF...', # OAuth 2.0 access_token, which you may wish to store
          expires_at: 1321747205, # when the access token expires (it always will)
          expires: true # this will always be true
        },
      }

      OmniAuth.config.test_mode = true
      OmniAuth.config.add_mock(:facebook, auth)
    end

    def mock_facebook_invalid_auth
      OmniAuth.config.test_mode = true
      OmniAuth.config.mock_auth[:facebook] = :invalid_credentials
    end

    def silence_omniauth
      previous_logger = OmniAuth.config.logger
      OmniAuth.config.logger = Logger.new("/dev/null")
      yield
    ensure
      OmniAuth.config.logger = previous_logger
    end
  end
end

RSpec.configure do |config|
  config.include Features::AuthenticationHelpers, type: :feature
end
~~~

~~~
# spec/support/devise.rb
RSpec.configure do |config|
  # this enable sign_in, sign_out devise methods in controller and view specs
  config.include Devise::Test::ControllerHelpers, type: :controller
  config.include Devise::Test::ControllerHelpers, type: :view
  # for devise <= 4.1.0
  # config.include Devise::TestHelpers
end
# configure only for features
RSpec.configure do |config|
  # login_as is warden method
  config.include Warden::Test::Helpers
end
~~~

I got error: `UncaughtThrowError: uncaught throw :warden` and that is because
user was unconfirmed. Better is be by default confirmed:

~~~
  factory :user do
    sequence(:email) { |n| "email#{n}@trk.in.rs" }
    password 'asdfasdf'
    confirmed_at { Time.current }

    factory :unconfirmed_user do
      confirmed_at nil
    end
  end
~~~

Note that devise confirmation email is sent
[after_commit](https://github.com/plataformatec/devise/blob/4-1-stable/lib/devise/models/confirmable.rb#L48)
which is not happen if you are using database_cleaner. Solution is to jump to
`js` or manually initiate sending email confirmation after create
[link](https://github.com/plataformatec/devise/issues/2688#issuecomment-187173433)

## Mailer helper

~~~
# spec/support/mailer_helpers.rb
module MailerHelpers
  # you should call this manually for specific test
  # not for all tests like: config.before(:each) { reset_email }
  def clear_emails
    ActionMailer::Base.deliveries = []
  end

  def last_email
    fail 'please use give_me_last_mail_and_clear_mails'
    ActionMailer::Base.deliveries.last
  end

  # some usage is like:
  # mail = give_me_last_mail_and_clear_mails
  # assert_equal [email], mail.to
  # assert_match t('user_mailer.landing_signup.confirmation_text'), mail.html_part.decoded
  # confirmation_link = mail.html_part.decoded.match(
  #   /(http:.*)">#{t("confirm_email")}/
  # )[1]
  # visit confirmation_link
  def give_me_last_mail_and_clear_mails
    email = ActionMailer::Base.deliveries.last
    ActionMailer::Base.deliveries = []
    email
  end
end
RSpec.configure do |config|
  config.include(MailerHelpers)
end
~~~

Testing emails rspec is different for Devise 3 and Devise 4 (there is a problem
that I can't open regitration email in devise 4).

## Pause helpers

~~~
# spec/support/pause_helpers.rb
module PauseHelpers
  # you can use byebug, but it will stop rails so you can not navigate to other
  # pages or make another requests in chrome while testing
  def pause
    $stderr.write 'Press CTRL+J or ENTER to continue'
    $stdin.gets
  end
end
RSpec.configure do |config|
  config.include PauseHelpers, type: :feature
end
~~~

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

## Debug

Debug capybara
* using byebug you can `page.find('[data-id]')`
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

If you want to show immediatelly errors while whole test suite is running you
can use <https://github.com/grosser/rspec-instafail>

~~~
# Gemfile
group :development, :test do
  gem 'rspec-instafail', require: false
end

# .rspec
--require rspec/instafail
--format RSpec::Instafail
--format progress # to keep dots ap
~~~

## Waiting ajax

<https://github.com/teamcapybara/capybara#asynchronous-javascript-ajax-and-friends>
capybara is smart enough to wait if some ajax is called and text is not found.
So it will retry (default_max_wait_time=2) untill failure is not raised. Note
that `!page.has_xpath?('a')` is not the same as `page.has_xpath?('a')` in
example where you are removing `a` in ajax. First will fail since it find `a`
negate (it does not wait when capybara is success). Second will wait untill it
is removed. So use expectations which are goint to be met untill after ajax.

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

## Helpers inside spec file

You can add methods to your spec file, usually for integration specs using
`yield` (click some button on some page) that can be customized for each test
example

~~~
# spec/features/user_forgot_password_spec.rb
require 'rails_helper'

RSpec.feature 'User forgot password' do
  let(:email) { 'my@email.com' }
  before do
    create :user, email: email
    reset_email
    Rails.cache.clear
  end

  def request_forgot_password(&block)
    visit new_user_session_path
    click_link 'Forgot your password?'

    fill_in 'Email', with: 'user@test.com'
    yield if block_given?

    click_button 'Send me reset password instructions'
  end

  scenario 'with existing email' do
    request_forgot_password do
      fill_in 'Email', with: email
    end
    expect(page).to have_content 'You will receive an email with instructions about how to reset your password in a few minutes.'
    expect(last_email.to).to include email
    expect(all_emails.size).to eq 1
  end
end
~~~

# Fixtures

All data is cleared between tests. Although we touch database (huge third party
dependency) we call it *unit* test. All data is available on each tests. Rails
starts db transaction at the begining of each tests and roll back at the end of
test (if you do not want transaction you can disable
`config.use_transactional_fixtures = false`)

We define it using [yml file](
{{ site.baseurl }} {% post_url 2017-01-10-get-syntax-right-in-jade-yaml-haml %}#yaml)

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
  * very fast since all data is loaded at once. It's better than factory bot
  since `FactoryBot.create :task` 10 times will create 10 projects (association
  objects) as well.

Cons:

* fixtures are global so all edge cases will increase database data for each
 test
* fixture are for specific models, so sometime it is hard to track
 complex setup (comment for that task for this project for that owner)
* fixtures are distant and far away of test so you need look into it to figure
 out actual test setup
* fixtures define database column so you need erb to define password hash.

  ~~~
  user:
    email: "test@example.com"
    encrypted_password: <%= User.new.send(:password_digest, 'password') %>
  ~~~

  In factory bot you can use model methods.

# Factory Bot

(former Factory girl)
After you install

~~~
sed -i Gemfile -e '/group :development, :test do/a  \
  gem "factory_bot_rails"'
bundle
mkdir spec/support
cat >> spec/support/factory_bot.rb << HERE_DOC
# so we do not need to prefix with FactoryBot.create :user
RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
end
HERE_DOC

sed -i spec/rails_helper.rb -e '/require .spec_helper/a  \
require "support/factory_bot"'
~~~

To use factories in console you can simply use: `FactoryBot.create :user`

You can create factories in `spec/factories.rb` or `spec/factories/*.rb`. It is
recomended to use only one file since it should be bare minimum, and should not
change while the application grows (although it could get bigger).
So write minimum that meets validation and use inheritance to create variations
of data.

[Getting
started](https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md#configure-your-test-suite)

* factory name is used to determine the class (it has to be same or you need to
use `class: Project`), dynamic attributes are defined with a block
`date_of_birth { 11.years.ago }` in which you can use other attributes

~~~
# spec/factories.rb
FactoryBot.define do
  factory :project do
    name "My Project"
    full_name { "exiting #{name}" }
    due_date Date.parse("2017-01-10")
    # note that Time.zone.now is evaluated at begining of test suite so that
    # expired_at = Time.zone now - 20.mins if you test suite lasts 20.mins
    expired_at Time.zone.now
    # here is evaluated when we create :project
    expired_at { Time.zone.now }
    # also
    before :create do |project|
      project.expired_at = Time.zone.now
    end
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
    # if you want that user is built but not save to database
    association :user, strategy: :build
    # you can define class and additional attributes
    association :updated_by, factory: :user, name: "Admin User"
    # another form
    user.association :user, name: 'My User'
  end
  ~~~

  If you have circular dependency ie two associations that are not independent,
  than you can setup object in after build block (one of them is less important
  and another is more, which will always used as parameter and not used as
  generated, for example it will not happen `create :location_ticket, location:
  location` without customer it will generate customer from some other
  location).

  ~~~
  # location_ticket --------> location
  #                 --> customer --^
  factory :location_ticket do
    # location - we set in after build because of circular dependency
    customer
    after(:build) do |location_ticket|
      location_ticket.location = location_ticket.customer.location
    end
  end

  # if you have three dependencies than you can set both in after block.
  factory :location_package_sale do
    # location - we set in after build because of circular dependency
    # customer - we set in after build because of circular dependency
    # location_package - we set in after build because of circular dependency
    after(:build) do |location_package_sale|
      location = location_package_sale.location_package&.location
      location ||= location_package_sale.customer&.location
      unless location
        location = create :location
      end
      location_package_sale.location = location
      unless location_package_sale.customer.present?
        location_package_sale.customer = create :customer, location: location
      end
      unless location_package_sale.location_package.present?
        location_package_sale.location_package = create :location_package, location: location
      end
    end
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

* callbacks `after(:create) do`, `after(:build)`, `before(:create)`,
`after(:stub)`. Note that there is no `before(:build)`
~~~
  password 'asdfasdf'
  confirmed_at { Time.current }

  factory :unconfirmed_user do
    confirmed_at nil
  end
  ~~~

* if you need to use existing record and not create new, you can like this
~~~
factory :company do
  country { Country.first || create(:country) }
  created_by { User.first || create(:user) }
end
~~~
* if you need polymorphic, than use `factory: :model` attribute. Also you can
put params in `association`
https://robots.thoughtbot.com/aint-no-calla-back-girl

~~~
class SmsAlert < ActiveRecord::Base
  belongs_to :smsable, polymorphic: true
end
class Customer < ActiveRecord::Base
  has_many :sms_alerts, as: :smsable
end
factory :sms_alert do
  message 'a'
  factory :customer_sms_alert do
    smsable factory: :customer
  end
end
~~~

* nested factory is usefull if you need different kind of objects. It is
  similar to inheritance. Note that if you have model `UserWithPosts` nested
  factory will use parent factory class `User`, not the `UserWithPosts`.
  You can also define as `parent: :user` attribute

~~~
factory :user_without_profile do
  factory :user do
    after :create do |user|
      create :profile, user: user
    end
  end
end
~~~

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

    # do not create profile for all users since that is not required in all test
    trait :with_profile do
      after :create do |user|
        create :user_profile, user: user
      end
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
  * `attributes_for(:user)` get hash of attributes so you can use in params_for
  for example: `xhr :post, users_path, user: attributes_for(:user)`
  * `build_stubbed(:user)` object with all AR attributes stubbed out (like
  `save`). It will raise an exception if they are called. Very fast since we do
  not create AR objects. It has rails ID and we can use associations. This is
  preferred way of creating data.

* multiple records can be `build_list :user, 25` or `create_list :user, 25` or
`build_stubbed_list :user, 25`

You can debug factory bot in rails console. After debugging test data will stay
in database so you need to clean it manually

~~~
RAILS_ENV=test rake db:drop db:create db:migrate
rails c -e test # Better to work in test so you can drop db

# require "factory_bot" no needed if you have gem factory_bot_rails
require "./spec/factories"
FactoryBot.build :user

include FactoryBot::Syntax::Methods
build :user, name: 'Dule'

RAILS_ENV=test rake db:drop db:create db:migrate
~~~

Factory bot can't be used as seed file as suggested [thoughtbot
post](https://robots.thoughtbot.com/factory_girl-for-seed-data) mainly because
it is used to create new data (not to use existing) and used in a way that you
do should not define all attributes.

Use fixtures for integration or complex controller tests. Use factories for unit
tests.

# Testing uploading files

~~~
joe.photo = File.new(File.join(Rails.root, 'spec', 'support', 'files', 'pookie.jpg'))
joe.save!
joe.photo_before_type_cast.should == "pookie.jpg"
~~~

For files you can use
~~~

~~~

# Testing time and date

Using rails native helpers
http://edgeapi.rubyonrails.org/classes/ActiveSupport/Testing/TimeHelpers.html
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

Use gem Timecop https://github.com/travisjeffery/timecop timecoop
Similar to mock which will return same value for all subsequent calls...

~~~
assert now
Timecop.freeze(Date.today + 30) do
  assert in one month
end

# Recomended is to use block syntax since
Timecop.freeze(Date.today + 30)
Time.now # will always return same value
Time.now # will always return same value even in different test
Timecop.return

# or travel with block syntax
Timecop.travel(Date.today + 30) do
  Time.now # will show Date.today + 30 + 1s...
end

# Note that factory bot definitions are used before timecop, so if you have
factory :user do
  renewed_at: Time.zone.today
end

Timecop.freeze Date.parse('2018-01-15') do
  user = create :user
  user.renewed_at # => 2018-05-05
end
~~~

# Using test doubles as mocks and stubs

<https://relishapp.com/rspec/rspec-mocks>
A test double is fake object (`double`) or method (`allow` and
`receive`) used in place of real when it is:

* difficult to create it (network failure)
* to isolate env so it tests only specific method
* to test behavior during this test (what is called and how) and not just end
results.

We can set expectation on arguments with `with` methods.

A **TEST DOUBLE** (serbian "kaskader" or "dvojnik") is a term for any object
that stands in for a real object. You can create with `double` method and by
default they are strict: any message you have not allowed or expected will
trigger an error.  If you do not want that exception is raised when some other
method is called than you can `double(:user).as_null_object` or `spy(:user)`.

~~~
# this will raise exception for user.country, but not for user.name
user = double(:user, name: 'Duke')
# this will not raise exception for user.country
user = spy(:user)
~~~

**VERYFING DOUBLE** is used to mimic specific object and check if message passed
to double is actually real method on real object and that arguments arity is the
same.

~~~
instance_user = instance_double(User)
~~~

There are also `class_double(User)` and `object_double(User.new)`.

All those have spy forms `instance_spy`, `class_spy` and `object_spy` giving
verification that message exists without having to specify a return value.

In RSpec 3 configuration `mocks.verify_partial_doubles = true` will verify all
partial doubles so you can not stub non existing method.

A **STUB** is test double that returns predetermined preconfigured value for a
method call (without calling actual method on actual object). It is defined
using `allow().to receive().and_return()`. It can also use `and_raise(Exception,
"message")` or to provide a block to specify return value, verify arguments,
perform calculation.
<https://relishapp.com/rspec/rspec-mocks/v/3-7/docs/configuring-responses>

~~~
allow(thing).to receive(:name).and_return("Duke")
# you can define methods and return values in bulk
allow(thing).to receive(first_name: 'Duke', last_name: 'Orl')
~~~

Stubs are used to prepare test data, and mocks are used to expect some calls.
A **MOCK** is similar to stub, but it sets expectation (`expect` instead
`allow`) that method will actually **be called** till the end of a example. It
use `expect().to receive.and_return`. If it is not called, mock object triggers
test failure. We can also set with **which arguments it should be called**
<https://relishapp.com/rspec/rspec-mocks/v/3-5/docs/setting-constraints/matching-arguments>

~~~
expect(thing).to receive(:name).and_return("Duke")

service_double = instance_double(CreatesProject, create: true)

expect(CreatesProject).to receive(:new)
  .with(name: 'Duke', tasks_string: nil)
  .and_return(service_double)
post ...
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
(`double`) (so we do not care about behavior, only about public interface).

You can use mocks for controller tests, so you do not need to know valid and
invalid params and a bug in CreatesProject will not fail this test. Usually when
I use double to mock some class, than use small integration test to cover both
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

* if you need to change behavior of real objects, and can't do dependency
injection (`process(date, validator)` and stub/mock on `validator`, than you can
use `allow_any_instance_of` and `expect_any_instance_of`:

~~~
  context 'with invalid data' do
    it 'raises Error' do
      allow_any_instance_of(Validator).to receive(:valid?).and_return(false)
      expect { processor.process('foo') }.to raise_error(DataProcessor::Error)
    end

    it 'call disconnect' do
      expect_any_instance_of(Subscriber).to receive(:delayed_disconnect)
    end
  end
~~~

There is a gem than can create stub for any activerecord model
<https://github.com/zeisler/active_mocker>

# Testing External service

* **client** is our app, **server** is external service, **adapter** is between
client and server, **fake server** returns a fake response
* *smoke test* is using real server, not used often, but could be helpfull
* *integration test* is using fake server
* *client unit test* ends at adapter

## Webmock
After installing and requiring [webmock](https://github.com/bblimke/webmock)

~~~
sed -i Gemfile -e $'/group :development, :test do/a  \
  gem \'rspec-rails\'\
  gem \'webmock\''
bundle
sed -i spec/rails_helper.rb -e '/require .spec_helper/a  \
require \'support/factory_bot\''
~~~

you can not make any external request `WebMock::NetConnectNotAllowedError:` will
be raised and information how to stub requests will be shown which you can use
in your `before` of `setup` block, or in some helper like
https://github.com/nebulab/cangaroo/blob/4effc172c6ee36ccf2b844e90dcc7041035d49cc/spec/support/spec_helpers.rb#L11-L30
You can also save real response in  a file `curl -is
http://api.twitter.com/1/users/show/marnen.json > tests/responses/canned_response.json`
and `stub_request(:get, "api.twitter.com/1/users/show/marnen.json").to_return(File.new 'canned_response.json'`
You can match partial query
[hash_including](https://github.com/bblimke/webmock#matching-partial-query-params-using-hash)
or using a block `{}` (`do end` wont work).
* `stub_request(method, uri)`, uri needs to be full (including query string) or
use regexp
*  uri, body and headers and query params can be matched agains regexp
~~~
stub_request(:get, '/webmock/')
stub_request(:get, '
`.with(body: //, headers:
{ "Content-Type": // )`
~~~

~~~
# spec/a/webmock_helper.rb
module WebmockHelper
  SMS_URI = /control.msg91.com/
  def stub_sms_to(mobile = "1111111111", messageRegexp = nil)
    if messageRegexp
      stub_request(:get, SMS_URI)
        .with(query: hash_including(mobiles: mobile)) { |request| request.uri.query_values['message'] =~ messageRegexp }
        .to_return(status: 200, body: "376967743076313037373133", headers: {}).times(1)
    else
      stub_request(:get, SMS_URI)
        .with(query: hash_including(mobiles: mobile))
        .to_return(status: 200, body: "376967743076313037373133", headers: {})
    end
  end

  def stub_sms_and_raise(mobile = "1111111111", error = Net::ReadTimeout)
    stub_request(:get, SMS_URI)
      .with(query: hash_including(mobiles: mobile))
      .to_raise(error)
  end
end
RSpec.configure do |config|
  config.include(WebmockHelper)
end
~~~

Last reponse is repeated infinitely (times) and you can specify number of times
given response should be returned. You can set expecation on how many
times request has been made.

Error like `stub_request(:get, "http://127.0.0.1:9516/shutdown").` is issue with
[spring](https://github.com/bblimke/webmock/issues/163#issuecomment-37257333)
Also the error
~~~
WebMock::NetConnectNotAllowedError: Real HTTP connections are disabled. Unregistered request: POST http://127.0.0.1:9515/session with body '{"desiredCapabilities":{"browserName":"chrome","version":"","platform":"ANY","javascriptEnabled":true,"cssSelectorsEnabled":true,"takesScreenshot":false,"nativeEvents":false,"rotatable":false},"capabilities":{"firstMatch":[{"browserName":"chrome"}]}}' with headers {'Accept'=>'application/json', 'Accept-Encoding'=>'gzip;q=1.0,deflate;q=0.6,identity;q=0.3', 'Content-Length'=>'250', 'Content-Type'=>'application/json; charset=UTF-8', 'User-Agent'=>'selenium/3.14.1 (ruby linux)'}
~~~

Also when I run system tests
```
An HTTP request has been made that VCR does not know how to handle:
  GET http://127.0.0.1:9522/shutdown

  GET http://127.0.0.1:9522/session

  GET https://chromedriver.storage.googleapis.com/LATEST_RELEASE_79.0.3945

There is currently no cassette in use. There are a few ways
```
To fix you need to write initializer that call `disable_net_connect!`
~~~
# config/initializers/webmock.rb
if Rails.env.test?
  require 'webmock'
  WebMock.disable_net_connect!(allow_localhost: true)
end
~~~

You can not use webmock for requests in javascript (capybara tests).

## VCR

VCR records **outgoing** HTTP requests.
VCR is using cassetes so you do not need to manualy stub requests using curl.

~~~
# spec/support/vcr.rb
VCR.configure do |config|
  config.cassette_library_dir = "spec/vcr_cassettes"
  config.hook_into :webmock
  # config.allow_http_connections_when_no_cassette = true
  config.ignore_localhost = true

  # https://github.com/titusfortner/webdrivers/wiki/Using-with-VCR-or-WebMock
  driver_hosts = Webdrivers::Common.subclasses.map { |driver| URI(driver.base_url).host }
  driver_hosts += ['googleapis.com'] # chromedriver webdriver downloads
  driver_hosts += ['192.168.5.56'] # mikrotik test router
  config.ignore_hosts(*driver_hosts)

  # custom matcher ignore message
  config.default_cassette_options = {
    match_requests_on: [ :method,
      VCR.request_matchers.uri_without_params(:message)]
  }
end

# we can use implicit form with macros use_vcr_cassete, but that is deprecated,
# use metadata
# https://relishapp.com/vcr/vcr/v/3-0-3/docs/test-frameworks/usage-with-rspec-metadata
~~~

Latest rails uses `gem 'webdrivers'` https://github.com/titusfortner/webdrivers
Since it automatically updates, it will be prevented by VCR
You can ignore those requests
https://github.com/titusfortner/webdrivers/wiki/Using-with-VCR-or-WebMock
or manually trigger update with
https://github.com/titusfortner/webdrivers#rake-tasks

```
RAILS_ENV=test rails webdrivers:chromedriver:update
```

If you really need external requests you can

~~~
# spec/a/webmock.rb
require 'webmock/rspec'
WebMock.disable_net_connect!
RSpec.configure do |config|
  config.around :example, :live do |example|
    WebMock.allow_net_connect!
    VCR.turn_off!
    example.run
    VCR.turn_on!
    WebMock.disable_net_connect!
  end
end
~~~

Use cassete inside `it` example block

~~~
RSpec.describe "Location direct sms", js: true do
  it 'sends successfully' do
    VCR.use_cassette "sms_success_1111111111_cassette" do
      visit customer_path customer, open_chat: true
      click_button 'Send', visible: true
    end
  end
end
~~~

If you allow multiple requests (for example uploading csv with multiple users
with same mobile number) than add `allow_playback_repeats: true`
<https://relishapp.com/vcr/vcr/v/3-0-3/docs/request-matching/playback-repeats>

If you are not sure if sms will be sent (it is based on some other
configuration) than `allow_unused_http_interactions: false`


# Testing Rails.cache

By default in `config/environments/test.rb` we have
`config.action_controller.perform_caching = false`. Note that this applies only
for `action_controller` so any page, fragment or custom caching still works.
You can disable all caching, ie caching is enabled but any write to cache is
discarded with:

~~~
# config/environments/test.rb
Rails.application.configure do
  config.cache_store = :null_store
end
~~~

I you need to test some caching (if rails state depend on cache value) than you
can clear (purge) cache before example

~~~
before :each do
  Rails.cache.clear
end
~~~

# When not to write tests

* when things are too hard to test, for example setting up model that external
services need to use
* too trivial tests
* overtesting when we use same model method on severall controllers
* exploratory coding (code that will not go to production)

# Coverage

<https://github.com/colszowka/simplecov>

`rake stats` can give you LOC  Code to Test ratio, which should be 1:2 - 1:3.

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
* simulate network timeouts with

  ~~~
  # if you perform request with `res = Net::HTTP.get_response(uri)`
  expect(Net::HTTP).to receive(:get_response).and_raise(Net::OpenTimeout)
  click_button 'Make Request'
  ~~~

# TODO:

Antipaterns

<http://code.tutsplus.com/articles/antipatterns-basics-rails-tests--cms-26011>



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

If you have `acts_as_taggable_on :cuisines` you can create with `_list` method: `FactoryBot.create :user, cuisine_list: ['American', 'Indian']`.

In fixtures you should put only what is neccesarry to create object (only validated field) so our test do not need to know about validations when they test something different. Define all neccessary stuf (like Tags) in test on the fly. Do not let your tests depend on fixtures. 

Some references:

guidlines [betterspecs](http://betterspecs.org/)
[thoughtbot](https://github.com/thoughtbot/guides/tree/master/best-practices#testing)
* avoid using instance variables in tests
* do not test private methods
* use [stubs and spies](https://robots.thoughtbot.com/spy-vs-spy) instead of
mocks since it clearly separate SETUP, EXERSIZE and VERIFICATION phase.

# Parallel tests

https://github.com/grosser/parallel_tests

# Opensource examples with tests

Best source is real word rails applications
<https://github.com/eliotsykes/real-world-rails>
and real world rspec examples with prescriptions
<https://github.com/eliotsykes/rspec-rails-examples>

* <https://github.com/discourse/discourse>
* <https://github.com/manshar/manshar> rails and angular
* <https://github.com/scottwillson/racing_on_rails> minitests
* <https://github.com/bikeindex/bike_index> rspec, api, swagger
* <https://github.com/mabranches/Olympic> swagger, jsonapi
* <http://stackoverflow.com/questions/4421174/what-are-some-good-example-open-source-ruby-projects-that-use-cucumber-and-rspec>

clean https://github.com/ni3t/tweetfire
https://github.com/elizabrock/coursewareofthefuture exaples for all tests

# Live Coding and other video tutorials

* [Advance API Phil
Sturgeon list](https://www.youtube.com/watch?v=Q00H6BPVNQI&list=PL6XWIjBFDFOIiJyDK61thx6ILIbaUMEAb) author of Building APIs your won't hate
* [gregg pollack and carlos souza](https://www.youtube.com/watch?v=jk016koiMAI)
minispec rails api
* [Sean Devine Live code a charity auction
application](https://www.youtube.com/playlist?list=PL93_jRSrU7hzEUNmevlnMeUx6F35Za638)
[source code](https://github.com/barelyknown/charity_auction-server)
* [thoughbot workshop intro to tdd](https://www.youtube.com/watch?v=sj5TXzgZ1Sk) also [mocking](https://www.youtube.com/watch?v=8368hGNIJMQ)
* [matt smith](https://www.youtube.com/watch?v=H1Czc3NL4c4)
* [Paul Hiatt](https://www.youtube.com/watch?v=9f08KzNO4qo)
* [Railscasts](http://railscasts.com/episodes?utf8=%E2%9C%93&search=test)
* [Mocking Luca Pradovera](https://www.youtube.com/watch?v=YjHKetQxk5I)
* [Carlos Souza](https://www.youtube.com/watch?v=CDC7zA8a-mA)

Rspec book <http://www.betterspecs.org/>

* [sandy metz](https://www.youtube.com/watch?v=URSWYvyc42M)

* fake sms client external test with adapter https://thoughtbot.com/blog/faking-external-services-in-tests-with-adapters
Improve speed of the tests

https://github.com/palkan/test-prof

TODO

https://player.fm/series/series-1401837/46-joe-ferris-test-driven-rails
https://www.youtube.com/watch?v=yTkzNHF6rMs
https://vimeo.com/44807822
https://www.youtube.com/watch?v=9f08KzNO4qo
https://m.patrikonrails.com/how-i-test-my-rails-applications-cf150e347a6b
https://robots.thoughtbot.com/headless-feature-specs-with-chrome
https://building.buildkite.com/5-ways-weve-improved-flakey-test-debugging-4b3cfb9f27c8
[aceptance
testing](http://blog.arkency.com/2017/06/acceptance-testing-ruby-using-actors-personas)
easy system testings https://rlafranchi.github.io/system_tester/

