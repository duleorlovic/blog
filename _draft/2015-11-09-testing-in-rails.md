Antipaterns

<http://code.tutsplus.com/articles/antipatterns-basics-rails-tests--cms-26011>

Resources:

* http://railscasts.com/episodes/275-how-i-test


Gems:

  gem 'rspec-rails'
  gem 'factory_girl'
  gem 'capybara'
  gem 'guard-rspec', require: false
  gem 'database_cleaner'
  gem 'selenium-webdriver'

  # frontend
  gem 'teaspoon-jasmine'
  gem 'phantomjs'


rails g rspec:install

guard init rspec

# add `require 'capybara/rspec'` to spec/rails_helper.rb

uncomment `Dir[Rails.root.join('spec/support/**/*.rb')].each { |f| require f }`

~~~
# spec/rails_helper.rb
  # https://github.com/DatabaseCleaner/database_cleaner#rspec-with-capybara-example
  config.use_transactional_fixtures = false

  config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation)
  end

  config.before(:each) do |example|
    DatabaseCleaner.strategy = example.metadata[:js] ? :truncation : :transaction
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end
~~~

Write capybara features in spec/features

Support

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

~~~
~~~
# spec/support/login.rb
require 'spec_helper'
include Warden::Test::Helpers

module LoginHelpers

  def manually_log_in
    visit "/"
    user = FactoryGirl.create(:user)
    within "#login-form" do
      fill_in "email", with: user.email
      fill_in "password", with: user.password
    end

    click_button 'Sign in'
  end

  def create_logged_in_user
    user = FactoryGirl.create(:user)
    login(user)
    user
  end

  def login(user)
    login_as user, scope: :user
  end
end

RSpec.configure do |config|
  config.include LoginHelpers
end
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
~~

Usually start with [feature spec](https://www.relishapp.com/rspec/rspec-rails/docs/feature-specs/feature-spec) with capybara [actions](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Actions) and [matchers](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Matchers)

Some of the most used capybara methods [link](https://gist.github.com/duleorlovic/042178b92f1badc09490)

* `visit "/"`
* `click_button "Sign in"`
* `with_in "#login-form" do`
* `fill_in "email", with: 'asd@asd.asd'`
* `find('ng-model="newExpense.amount"').set('123')` [find](http://www.rubydoc.info/github/jnicklas/capybara/Capybara/Node/Finders#find-instance_method) can use css or xpath (see some [xpath examples](scrapper post)) When 
* `find_all('input').first.set(123)`
* confirm dialog
  * selenium `page.driver.browser.switch_to.alert.accept  # can also be .dismiss`
  * webkit `page.accept_confirm { click_link "x" } }` so actions is wrapped with this page.accept_confirm

Than we go in details for frontend controller, backend controller and model test and make sure integration test pass.

We can setup data using `let` or simply inside `it`.
[post](https://robb.weblaws.org/2016/10/20/why-i-dont-use-letlet-in-my-rspec/?)
suggests that `let` should be replaced with instance variable.

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

Controller spec are for each method (CRUD)  in context or authorized and in context of unauthorized user.

Acts as taggable

If you have `acts_as_taggable_on :cuisines` you can create with `_list` method: `FactoryGirl.create :user, cuisine_list: ['American', 'Indian']`.

In fixtures you should put only what is neccesarry to create object (only validated field) so our test do not need to know about validations when they test something different. Define all neccessary stuf (like Tags) in test on the fly. Do not let your tests depend on fixtures. 

Some references:

* guidlines [betterspecs](http://betterspecs.org/)

Devise 

https://github.com/plataformatec/devise/wiki/How-To:-Test-controllers-with-Rails-3-and-4-(and-RSpec)#controller-specs

Controller spec are not rendering view templates. If you need that add `render_views`

Use [clearance backdoor](https://github.com/thoughtbot/clearance#fast-feature-specs) for bypassing login and simply `visit my_profile_path(as: user)


Teaspoon and jasmine

Do not use `it` outside of inner describe block

# some examples 

https://gist.github.com/kyletcarlson/6234923

https://www.youtube.com/watch?v=YjHKetQxk5I

# Minitest

For Minitest you need to install minitest gem and require it. You can `skip`
single test. Run with colors `ruby my_class_test.rb -p`, or `ruby -r
minitest/pride my_class_test.rb` or put `require 'pride'` in test file.

~~~
require 'minitest/autorun'
class MyClassTest < Minitest::Test
  def test_sample
    assert MyClass.new
  end
end
~~~
