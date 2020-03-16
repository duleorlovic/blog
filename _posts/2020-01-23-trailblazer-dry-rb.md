---
layout: post
---

# Trailblazer

Docs in three places:
http://trailblazer.to/guides/
http://2019.trailblazer.to/2.1/docs/trailblazer.html
http://trailblazer.to/api-docs/

Concepts (dashboard index, create comment)

Operation implement functions of the app (also called commands) and it use:
* policy - authorizations: `Policy::Guard` macro
* representer - cell(widget) instance methods are used in view model (no need
  for view context)
* validation - forms or contracts: type check and rules
* model - only associations and scopes
* callback

## Operation

http://trailblazer.to/gems/operation/2.0/index.html
Simple example
```
class Song::Create < Trailblazer::Operation
  step :assign_current_user!

  def assign_current_user!(ctx, model:, current_user:)
    model.created_by = current_user
  end
end
```
Operation always return result object. It is called using short notation,
instead `Operation.call(a)` you can write ruby alias for it `Operation.(a)`
```
result = Song::Create.(params.merge current_user: current_user)
result.success?
result['model'] # => #<Song ...>
```

It is using `step` method which can be instance method, lambda or callable model
```
step :model!
step ->(ctx, **) { ctx['model'] = Song.new }
step MyStep

def model!(ctx, **)
  ctx['model'] = Song.new
end

# define Callable
class MyStep
  extend Uber::Callable
  def self.call(ctx, **)
    ctx['model'] = Song.new
    # return value matters
  end
end
```

If step return value is falsey (nil or false) than operation result is marked as
failed and only steps marked as `failure` after failing step will be executed.
You can pass fail_fast `failure :abort, fail_fast: true` if you do not want to
execute other failure steps after this this.
Instead of `step` you can use `success` which does not care about return value.

Step accepts two arguments, `options` or `ctx` (this should act like accumulator
since we can write output to it and it is mean to be mutable). In trailblazer
2.0 first params passed to .call was inside `ctx['params']`. In trb 2.1 you need
to explicitly call using params `result = Memo::Create.(params: params,
current_user: current_user)` (first-value and second-hash params were merged to
`ctx`).

Second argument can extract keys from first, for example, you can write to
options in each step `ctx[:new_result] = SomeClass.new` and you can extract
in following steps with `def step(ctx, new_result:, **)` and so make it
required.
If you state them in method definition for very first step, than those
arguments will be required keys in first hash in trailblazer 2.1 (trailblazer
2.0 it should be second hash argument). double splat `**` means "I know there
are more keyword arguments but I'm not intereseted right now".

## Macro

**Model** http://trailblazer.to/gems/operation/2.0/api.html#model-findby source
is defined in trailblazer-macro gem (not trailblazer-operations)
https://github.com/trailblazer/trailblazer-macro/blob/master/lib/trailblazer/macro/model.rb
```
step Model(Song, :new) # this is the same as ctx[:model] = Song.new
step Model(Song, :find_by) # this is the same as ctx[:model] = Song.find_by params[:id]
# There will be automatic jump to error track if can not find the model
```

**Nested** http://trailblazer.to/gems/operation/2.0/api.html#nested can call
other operations
```
class Update < Trailblazer::Operation
  step Nested( Edit )

  # you can adapt parameters using input param
  step Nested( Multiplier, input: ->(options, some_data:, **) do
    { x: some_data[:pi_constant], y: 2 }
  end)
end
```


**Policy::Guard** http://trailblazer.to/gems/operation/2.0/policy.html#guard
```
step Policy::Guard( :authorize! )
```
and you can access results in
```
result['policy.default']
result['result.policy.default'].success?
```

**Contract::Build** is defined in trailblazer-macro-contract gem
https://github.com/trailblazer/trailblazer-macro-contract/blob/master/lib/trailblazer/operation/contract.rb#L15

## Macro API
http://trailblazer.to/gems/operation/2.0/api.html

Macro is a capitalized name function which returns two element array: actual
step, and default options
```
module Macro
  def self.MyPolicy(allowed_role: "admin")
    step = ->(input, options) { options["current_user"].type == allowed_role }

    [ step, name: "my_policy.#{allowed_role}" ] # :before, :replace, etc. work, too.
  end
end
```

You can use like
```
class Create < Trailblazer::Operation
  step Macro::MyPolicy( allowed_role: "manager" )
  # ..
end
```

## Reform Contract

Example contract form validation using Reform gem
http://trailblazer.to/gems/reform/index.html
http://trailblazer.to/gems/operation/2.0/contract.html
```
gem 'dry-validation'
gem 'reform' # it depends on disposable gem which provide a Twin decorator, and
           # depends on representable gem which provide representer deserializer
           # validation is in dry-validation or Activemodel::Validations
# for rails Activemodel::Validations
gem 'reform-rails'
```
Actual validation can be implemented using Reform (with ActiveModel::Validation
or dry-validation) or using Dry::Schema (without Reform).
It allows mapping to nested models, via composition, to hash fields.
It is initialized with model for which you want to validate data against, and
once read, original model's values will never be accessed. To save to model you
can run `form.save` which will call `sync` and `model.save`.
```
form = Blog::Contract::Create.new Blog.last
form.title # this will return Blog.last.title
form.title = 'new' # this will not change blog title
form.model.title # this old value

result = form.validate title: 'new title'
```

Here is example of reform

```
# app/concepts/song/contract/create.rb
# no need to include if using rails
# require 'reform'
# require 'reform/form/dry'
# include Dry
#
module Song::Contract
  class Create < Reform::Form
    property :title

    # populator is called when validate is triggered, but before deserialisation
    # and actual validation happens
    property :artist, populator: ->(ctx) {}

    # for has_many associations use collections
    # prepopulator must be invoked manually in controller @form.prepopulate!
    # this nested form
    property :user, prepopulator: ->(*) { self.user = User.new },
                    populate_if_empty: ->(*) { User.new } do # this populator
                    # only runs when there is incoming hash, but no
                    # corresponding nested form
      property :email
    end

    validation do
      required(:title).filled
      required(:body).maybe(min_size?: 9)
    end
  end
end
```
and can be used in operation with this API (`initialize`, `validate`, `save`,
`errors`, `sync`, `prepopulate`)
but we can use macros: Build, Validate, Persist
for example in `rails c` you can run
```
res = Venue::Operation::Create.(params: {})
res['contract.default'].errors
```
Using Macros will simplify the code:
```
step Contract::Build( constant: Venue::Contract::Create )
# this will populate contract.default (and result.contract.default) with
# Reform.new ctx[:model]] like this line
# ctx['contract.default'] = Venue::Contract::Create.new(ctx[:model])
# https://github.com/trailblazer/trailblazer-macro-contract/blob/master/lib/trailblazer/operation/contract.rb#L15

step Contract::Validate( key: :venue )
# https://github.com/trailblazer/trailblazer-macro-contract/blob/master/lib/trailblazer/operation/validate.rb
# this will fetch contract.default and valide against key
# reform_contract = ctx["contract.default"]
# result = reform_contract.validate(ctx["params"][:venue])
# error will be available in result['result.contract.default'].errors.messages

step Contract::Persist()
# this will save contract data. Note that it will not change ctx['model']
# reform_contract = ctx["contract.default"]
# reform_contract.save
```

http://trailblazer.to/gems/reform/populator.html has two ways, `prepopulator` is
for rendering and `populator` is on `validate`
Populator function accepts
* `collection` it is indentical to `songs` (or other resources you defined)
* you can access other properties since context is current form

## Representer

http://trailblazer.to/gems/operation/2.0/representer.html
http://trailblazer.to/gems/representable/3.0/api.html

incoming **document** is params, and output is represented object (any object
that expose property setters)
Example
```
Band = Struct.new(:name)
class SongRepresenter < Representable::Decorator
  include Representable::Hash

  property :title
  property :band, class: Band do
    property :name
  end
end
```
usage
```
Song = Struct.new(:title, :band)
song = Song.new
SongRepresenter.new(song).from_hash(title: 'Duke')
# it will do: song.title = params[:title]

input = { "title" => "Let Them Eat War", "band" => { "name" => "Bad Religion" } }
SongRepresenter.new(song).from_hash(input)
song.band  #=> #<struct Band name="Bad Religion">
song.band.name #=> "Bad Religion"
```
or usage in operation in Validate() using `:representer` key
```
  step Contract::Validate( representer: MyRepresenter )
```
Example for rendering (which should be outside of operation)
```
result = Create.(params, document: '')
if result.success?
  result['representer.render.class'].new(result[:model]).to_json
else
end
```
## Custom steps

```
# app/concepts/venue/lib/notification.rb
class Venue::Notification
  def self.call(*)
    true
  end
end
```

Invoke with
```
step :notify!

def notify!
  ctx['result.notify'] = Venue::Notification.(current_user, model)
end
```

## Rails

/home/orlovic/rails/temp/trb_test
Gemfile

```
# https://github.com/trailblazer/reform/issues/500
# reform is not ready for dry-validation ver 1 so fix to 0
gem 'dry-validation', '~> 0.10.6'
gem 'dry-types', '~> 0.10.2'

gem 'trailblazer', '>= 2.0.3'
gem "trailblazer-cells"
gem 'trailblazer-generator', require: false
gem 'trailblazer-rails' # this will include trailblazer-loader

gem 'cells-rails'
gem "cells-slim"
# gem 'cells-hamlit'

gem 'reform-rails'
```

http://trailblazer.to/gems/trailblazer/2.0/rails.html

It will load files from `app/concepts` (trailblazer-loader gem). It will load
files for example `app/concepts/product/operation/create.rb` and you can use
`Product::Create` class name (because trailblazer-loader do not use Rails
autoloader and no need to use `Production::Operation::Create`).
It provides `run Blog::Index` method that will invoke the operation, pass params
https://github.com/trailblazer/trailblazer-rails/blob/68694ccafc58d26dcf1d715c67ac1581cb07a7fa/lib/trailblazer/rails/controller.rb#L41
and if you want, also dependencies as `current_user` using `_run_options`
```
class ApplicationController < ActionController::Base
  def _run_options(options)
    options.merge current_user: current_user
  end
end
```

It sets `@model` (also `@blog` which is identical to `@model`) and `@form` if
exists (if `Contract:Build` was called and initialize in
`result['contract.default']`) and `result` object (by convention operation
should populate `result['model']` object or array for index operation).

If operation can go wrong (success and error track) than you can use the block
syntax `run Venue::Update do |result|` which will be called on success and you
should `return redirect_to path` so the code after `run` will not be executed
(fail path).

Also it extends `render` to allow rendering cells
```
render cell(Blog::Cell::Index, @form), layout: false
```

All file and class names should be singular.
Here is example controller
```
# app/controllers/venues_controller.rb
class VenuesController < ApplicationController
  def new
    run Venue::Create::Present
    render cell(Venue::Cell::New, @form), layout: false
  end

  def create
    run Venue::Create do |result|
      return redirect_to venues_path
    end

    render cell(Venue::Cell::New, @form), layout: false
  end

  def show
    run Venue::Show
    render cell(Venue::Cell::Show, result["model"]), layout: false
  end

  def index
    run Venue::Index
    render cell(Venue::Cell::Index, result["model"]), layout: false
  end

  def edit
    run Venue::Update::Present
    render cell(Venue::Cell::Edit, @form), layout: false
  end

  def update
    run Venue::Update do |result|
      flash[:notice] = "#{result["model"].title} has been saved"
      return redirect_to venue_path(result["model"].id)
    end

    render cell(Venue::Cell::Edit, @form), layout: false
  end

  def destroy
    run Venue::Delete

    flash[:alert] = "Post deleted"
    redirect_to venues_path
  end
end
```

# Test

Use `minitest-rails-capybara` gem for testing 
`rails generate minitest:feature CanAccessHome --no-spec` gives
```
require "test_helper"

class CanAccessHomeTest < Capybara::Rails::TestCase
  def test_homepage_has_content
    visit root_path
    assert page.has_content?("Home#index")
  end
end
```
but I do not see why we will use it when we have system test

## Cells

http://trailblazer.to/gems/cells/
https://github.com/trailblazer/cells

This View-model uses helpers that are not globals, but instance methods. It is
faster than ActionView because there is no code.

```
# inside controller
    render cell(Venue::Cell::New, @form), layout: false
# in console
html = Venue::Cell::New.(Venue.new, current_user: User.last).()
```

You can call `render` to render
`app/concept/venue/view/new.slim` (template name is generated from class name).
Use cells-slim https://github.com/trailblazer/cells-slim
```
# app/concepts/venue/cell/new.rb
module Venue::Cell
  class New < Trailblazer::Cell
    # to use t('hi')
    include ActionView::Helpers::TranslationHelper
    # to use bootstrap_form_for
    include BootstrapForm::ActionViewExtensions::FormHelper
  end
end
```
Extend cell with other methods.
Inside template you can access `model` and `options` object.
```
# app/concepts/venue/cell/edit.rb
module Venue::Cell
  class Edit < New
    def show
      render :new
    end

    def back
      link_to "Back", post_path(model.id)
    end

    def delete
      link_to "Delete Post", post_path(model.id), method: :delete
    end
  end
end
```
You can delegate `title` to `context[:model].title`
```
# app/concepts/venue/cell/show.rb
module Venue::Cell
  class Show < Trailblazer::Cell
    property :title
    property :body

    def current_user
      return ctx[:context][:current_user]
    end

    def time
      model.created_at
    end

    def edit
      link_to "Edit", edit_venue_path(model.id)
    end

    def delete
      link_to "Delete", venue_path(model.id), method: :delete, data: {confirm: 'Are you sure?'}
    end

    def back
      link_to "Back to posts list", venues_path
    end
  end
end
```

```
# app/concepts/venue/cell/item.rb
module Venue::Cell
  class Item < Trailblazer::Cell
    def title
      link_to model.title, model unless model == nil
    end

    property :body

    def created_at
      model.created_at.strftime("%d %B %Y")
    end
  end
end
```

```
# app/concepts/venue/cell/index.rb
module Venue::Cell
  class Index < Trailblazer::Cell
    def total
      return "No posts" if model.size == 0
    end
  end
end
```

When I try to render cell in console it crashes since no context[:controller] is
given and url_options are needed
https://github.com/trailblazer/cells-rails/issues/50
https://github.com/trailblazer/cells/issues/112
```
html = Venue::Cell::New.(Venue.new, current_user: User.last).()
=> "<div id=\"remote-form\"></div>"

# but if I have a root_path than error is raised
NoMethodError (undefined method `[]' for nil:NilClass)

# if I call with blank context than url_options error is given
html = Venue::Cell::New.(Venue.new, current_user: User.last, context: {}).()
NoMethodError (undefined method `url_options' for nil:NilClass)

# so solution is to call using fake controller
html = Venue::Cell::New.(Venue.new, current_user: User.last, context: {controller: OpenStruct.new(url_options:{})}).()
```
To use in test you need to inherit Cell::TestCase and use cell
```
# test/cell/venue/new_test.rb
require 'test_helper'
class VenueCellNewTest < Cell::TestCase
  test 'renders' do
    venue = venues(:novi_sad)
    html = cell(Venue::Cell::New, venue).()
    assert_match /#{venue.name}/, html.to_s
  end
end
```
but I also get error for any line like `root_path`
```
Error:
VenueCellNewTest#test_renders:
NoMethodError: undefined method `url_options' for nil:NilClass
    /home/orlovic/.rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/forwardable.rb:228:in `url_options'
    actionpack (6.0.2.1) lib/action_dispatch/routing/route_set.rb:265:in `call'
```
so when I step into cell
https://github.com/trailblazer/cells/blob/master/lib/cell/testing.rb#L8 which
will call cell_for args so found than it uses context of controller_class
https://github.com/trailblazer/cells/blob/master/lib/cell/testing.rb#L60
can also pass fake controller
```

```
In rails
https://github.com/trailblazer/cells-rails/blob/master/lib/cell/rails.rb#L12

# Dry

Disposable Twins API http://trailblazer.to/gems/disposable/api.html
use dry-types https://dry-rb.org/gems/dry-types/1.2/
```
property :name, type: ::Types::String
# coercion
property :id, type: Types::Form::Int, nilify: true
```

## Dry validations

Debug in console
```
schema = Dry::Validation::Schema() { required(:name).filled }
schema.call name: ''
```
Example or dry-validations https://dry-rb.org/gems/dry-validation/0.13/optional-keys-and-values/
* `.maybe` is used when it is valid that value could be `nil`
* `optional(:f).filled` is when it is not required, but when it is present it
  should be filled (not a nil, empty sting, array or hash)
* `optional(:n).maybe :int?, included_in?: [1, 3]` it is optional and it could
  be present with `nil` value, or 1 or 3 (can not be empty string, maybe is only
  allows to nil, but if you use `type: Types::Params::Integer` than coercion
  will make empty string to nil)
  https://dry-rb.org/gems/dry-validation/0.13/basics/macros/#maybe
  https://dry-rb.org/gems/dry-validation/0.13/basics/built-in-predicates/#code-included_in-code

# What you need to know for using Trailblazer

* you need to learn DSL which change from version to version. You lose if you do
  not follow up documentation.
* errors for trailblazer because of eager loading or something, restart server
  helps
  ```
  uninitialized constant #<Class:0x00007f6b986a5da0>::Index Did you mean? Bindex` .
  NameError (uninitialized constant #<Class:0x000055ab8fc752f0>::Update
  ```
  This can be solved by writing `module [ConceptName}::Operation` and in next
  line `class Update < Trailblazer::Operation` so it is properly loaded with
  rails autoloader, you need to disable trailblazer loader
  ```
  # config/initializers/trailblazer.rb

  YourApp::Application.config.trailblazer.enable_loader = false
  ```
* can not use form partials, for example if you want to render another partial
  https://github.com/trailblazer/cells/issues/460
  https://stackoverflow.com/questions/47069460/rendering-partial-forms-in-trailblazer-cells

# Graph modeling
http://trailblazer.to/api-docs/#what-is-trailblazer 2.1

Instead of subclassing we use compositions. Inside `module` we call
`module_function` so it is more readable and allows to use `method` to attach a
module method to a step
```
module Memo::Update
  extend Trailbazer::Activity::Railway()

  module_function

  def find_model(ctx, id:, **)
    ctx[:model] = Memo.find_by id: id
  end

  step method(:find_model)
end
```

Invoke like
```
ctx = { id: 1, params: { body: 'Awesome' } }
event, (ctx, *) = Memo::Update.( [ctx, {}] )
pp ctx #=>
{ id: 1,
  params: { body: 'Awesome' },
  model: #<struct Test::Memo body=nil>,
  errors: 'body not long enough',
}
```

# Activity

https://2019.trailblazer.to/2.1/docs/activity.html#activity-strategy

For Path, it is automatically wired `:success / Activity::Right` semantic /
signal. You need to manually wire other outputs
```
class Create < Trailblazer::Activity::Path
  step :validate, Output(Activity::Left, :failure) => End(:invalid)
  step :create
  # ...
end
```
You can jump to `End(:db_error)` or back to track `Track(:success)`.
When you use you get semantic of last
```
ctx = {params: nil}
signal, (ctx, flow_options) = Memo::Create.([ctx, {}])

puts signal #=> #<Trailblazer::Activity::End semantic=:invalid>
```

For Railway there are two tracks

```
class Create < Trailblazer::Activity::Railway
  step :validate
  fail :log_error, Output(:success) => Track(:success)
  step :create
  # ...
end
```
You can use `pass :create` so both success and failure will end in `End success`

