---
layout: post
---

[7-ways-to-decompose-fat-activerecord-models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
suggestions for code organization.

You can organize domain (controller, model and view) into separate folders
[drawers](https://github.com/NullVoxPopuli/drawers)

https://thoughtbot.com/upcase/intermediate-ruby-on-rails
https://code.tutsplus.com/articles/a-beginners-guide-to-design-patterns--net-12752

Three basic kind of design pattern:
* structural
* creational
* behavioral


https://www.youtube.com/watch?v=3SGRjUWScRw&feature=youtu.be

Instead of

~~~
class Member < ActiveRecord::Base
  has_many :email_addresses
  has_one :primary_email_address, conditions: { primary: true }
end
~~~

Use belongs_to on members

~~~
class Member < ActiveRecord::Base
  has_many :email_addresses
  belongs_to :primary_email_address, class_name: "EmailAddress", foreign_key: :email_address_id
end
~~~

https://thoughtbot.com/blog/handling-associations-on-null-objects

# DHH

[dhh tips for rails](https://www.youtube.com/watch?v=D7zUOtlpUPw)
* epipsode 1: do not use comments but method names or constants... follow
  table of content: method definition should be in same order as they are used
  in the class (in before callbacks)
  administered concern define methods on associations. Also use `or` `|` to join
  two arrays

~~~
module Account::Administered
  extend ActiveSupport::Concern

  included do
    has_many :administratorships, dependent: :delete_all do
      def grant(person)
        create_or_find person: person
      end

      def revoke(person)
        where(person_id: person.id).destroy_all
      end
    end

    has_many :administrators, through: :administratorships, source: :person
  end

  def all_administrators
    administrators | all_owners
  end

  def administrator_candidates
    people.users.
      where.not(id: administratorships.pluck(:person_id)).
      where.now(id: ownerships.pluck(:person_id)).
      where.not(id: owner_person.id)
  end
end
~~~
* episode 2: use callbacks to initiate background jobs
* episode 3: use globals in request/response cycle. For background jobs you need
  to pass them as params.
* episode 4


# Rails Patterns

Pawel Daborwski
Patterns are used for benefit of:
* isolation: db/payment should be tested separately
* readability: class name should tell what given code is doing
* extendability: it should be easy to modify by changing in single place
* single responsibility: one method for one action

## Policy object

It is used to check if someone is allowed to do the action. File name should use
`_policy` suffix. Return only boolean value. Inside method, only call methods on
passed objects.

```
class UsersPolicy
  def initialize(user)
    @user = user
  end
  def admin?
    @user.role == 'admin'
  end
end
```

## Query object

Complex queries should be extracted and tested separatelly. File name could be
`app/queries/users/list_active_users_query.rb` for single query, or
`app/queries/users_query.rb` for multiple `#active` and `#suspended` methods.
For single query you can use class methods
```
module Users
  class ListActiveUsersQuery
    def self.call
      User.where(status: 'active').where.not(email: nil)
    end
  end
end
```
for multiple you can use poro
```
class UsersQuery
  def initialize(users = User.all)
    @users = users
  end
  def active
    @users.where(active: true)
  end
  def pending
    @users.where(pending: true)
  end
end
```
and you can chaining `query = UsersQuery.new` and `query.active.pending`.
You can use https://apidock.com/rails/ActiveRecord/QueryMethods/extending to
extend scopes.

## Service objects

Always define only one public method: `call`, `perform` or `process`

~~~
# app/services/match_posts.rb
def MatchPosts
  def initialize(posts)
    @posts = posts
  end

  def perform
  end
end
~~~

You should return object that can be success and hold some data

```
# app/models/result.rb
class Result
  attr_accessor :message, :data

  # you can return in service like:
  #   return Result.new 'Next task created', next_task: next_task
  # and use in controller:
  #   if result.success? && result.data[:next_task] == task
  def initialize(message, data = {})
    @message = message
    @data = data
  end

  def success?
    true
  end
end

# app/models/error.rb
class Error < Result
  def success?
    false
  end
end
```

and you can rescue from exceptions:

~~~
# app/services/my_service.rb
class MyService
  # Some custom exception if needed
  class ProcessException < Exception
  end

  def initialize(h)
    @user = h[:user]
  end

  def process(posts)
    success_message = do_something posts
    Result.new success_message
  rescue ProcessException => e
    Error.new e.message
  end

  private

  def do_something(posts)
    raise ProcessException, "Error: empty posts" unless posts
    "Done with do_something"
  end
end
~~~

~~~
# main.rb
require './my_service.rb'

my_service = MyService.new user: 'me'
puts my_service.process(1).success? # true
puts my_service.process(1).message # Done with do_something
puts my_service.process(false).success? # false
puts my_service.process(false).message # empty posts
~~~

## Observer

We an use observable objects to send notifications [implementation in 3
languages](http://www.diatomenterprises.com/observer-pattern-in-3-languages-ruby-c-and-elixir/)
It could looks like [action as a
distance](https://en.wikipedia.org/wiki/Action_at_a_distance_(computer_programming))
antipattern but if we explicitly add than is it fine.
It is called publish/subscriber pattern. It allows you to observe one object
and when it's changed then trigger certain code in other objects that are
subscribing the main object.

~~~
module ObservableImplementation
  def observers
    @observers ||= []
  end

  def notify_observers(*args)
    observers.each do |observer|
      observer.update(*args)
    end
  end

  def add_observer(object)
    observers << object
  end
end

class Task
  include ObservableImplementation
  attr_accessor :counter
  def initialize
    self.counter = 0
  end

  def tick
    self.counter += 1
    notify_observers(counter)
  end
end

class PutsObserver
  def initialize(observable)
    observable.add_observer self
  end

  def update(counter)
    puts "Count has increased by #{counter}"
  end
end

class DotsObserver
  def initialize(observable)
    observable.add_observer self
  end

  def update(counter)
    puts "." * counter
  end
end

task = Task.new
task.tick
DotsObserver.new(task)
task.tick # ..
PutsObserver.new(task)
task.tick # ...
# Count has increased by 3
~~~

In ruby there are
https://docs.ruby-lang.org/en/2.2.0/Observable.html
where you call `changed` and `notify_observers params` (which will set changed
to false).
You need to call `task.add_observer DotsObserver.new`.


## Interactor

Service object is similar to Command pattern which is implemented in gem
<https://github.com/collectiveidea/interactor>

When does its single purpose, it affects tis given context
```
context.user = user
context.fail! error: 'Boom'
# .call will swallow exception Interactor::Failure
context.success? # false
```

Before and after hooks for preparing data. Before hooks are invoked in the order
they were defined.
```
before do
  context.emails_sent = 0
end
after :reload # note that this will not be run if `fail!` is called
```

Example
```
# app/interactors/authenticate_user.rb
class AuthenticateUser
  include Interactor

  def call
    if user = User.authenticate(context.email, context.password)
      context.user = user
      context.token = user.secret_token
    else
      context.user = User.new email: context.email
      context.user.errors.add :email, 'wrong email or password'
      context.fail!(message: "authenticate_user.failure")
    end
  end
end
```
and use in controller like
```
  result = AuthenticateUser.call(session_params)
  if result.success?
    session[:user_token] = result.token
    redirect_to dashboard_path
  else
    @user = result.user
    render :new
  end
```

Second type of interactor is Organizer which is used to run other interactors.

```
class PlaceOrder
  include Interactor::Organizer

  organize CreateOrder, ChargeCard
end
```

In this case then `ChargeCard` fails we can use `CreateOrder#rollback` method to
revert changes.

In test, you can stub other method calls and cover 100% of unit interactor tests
than write integration or acceptance tests to test wires.

# Decorator

It is used to decorate object, ie catch method missing and send to object in
initializer on runtime. This is different than subclassing since it happens in
run-time.

```
# app/decorators/user_profile_decorator < SimpleDelegator
class UserProfileDecorator < SimpleDelegator
  def model
    __getobj__
  end
  def name
    "#{model.first_name} #{model.last_name}"
  end
end
```
usage
```
user = Struct.new(:first_name, :last_name).new("John", "Doe")
decorator = UserProfileDecorator.new(user)
decorator.first_name # => "John"
decorator.name # => "John Doe"
```

Gem that supports decorating collections and associations
https://github.com/drapergem/draper

Another use case if for dependency injection, for example you have two pdf
generators than create graphs and you want to create another pdf that include
both graphs.
https://www.honeybadger.io/blog/decoupling-ruby-delegation-dependency-injection/
```
class PrawnWrapper < SimpleDelegator
  def initialize(document: nil)
    document ||= Prawn::Document.new(...)
    super(document)
  end
end
```
and call others using injection
```
class OverviewReport < PrawnWrapper
  ...
  def render
    sales = SaleReport.new(..., document: self)
    sales.sales_table
    costs = CostReport.new(..., document: self)
    costs.costs_pie_chart
    ...
  end
end
```

# Facade pattern

page 37
