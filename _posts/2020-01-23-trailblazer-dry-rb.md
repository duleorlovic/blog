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
https://2019.trailblazer.to/2.1/docs/operation.html
Simple example
```
class Song::Create < Trailblazer::Operation
  step :assign_current_user!
  step :debug

  private

  def assign_current_user!(ctx, model:, current_user:, **)
    model.created_by = current_user
  end

  def debug(ctx, params:, **)
  end
end
```
Operation always return result object. It is called using short notation,
instead `Operation.call a ` you can write ruby alias for it `Operation.(a)`
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
    # return value of step matters and is should not be nil or false
  end
end
```

If step return value is falsey (nil or false) than operation result is marked as
failed and only steps marked as `fail` (`failure` for trb 2.0) after failing
step will be executed.  You can pass fail_fast `failure :abort, fail_fast: true`
if you do not want to execute other failure steps after this this.
Instead of `step` you can use `pass` (2.1 `success`) which does not care about
return value.

Step accepts first param `options` or `ctx` or skill (this should act like
accumulator since we can write output to it and it is mean to be mutable). In
trailblazer 2.0 first params passed to .call was inside `ctx['params']`. In trb
2.1 you need to explicitly call using params `result = Memo::Create.(params:
params, current_user: current_user)`. first-value and second-hash params were
merged to `ctx`. At the end context is converted to result object on which you
can access `result['model']` and other `result['current_user']`.

Second argument can extract keys from first, for example, you can write to
options in each step `ctx[:new_result] = SomeClass.new` and you can extract
in following steps with `def step(ctx, new_result:, **)` and so make it
required.
If you state them in method definition for very first step, than those
arguments will be required keys in first hash in trailblazer 2.1 (trailblazer
2.0 it should be second hash argument). double splat `**` means "I know there
are more keyword arguments but I'm not intereseted right now".

http://trailblazer.to/gems/operation/2.0/api.html
You can use `name: 'build.song` to override step name

## Macro

**Model** http://trailblazer.to/gems/operation/2.0/api.html#model-findby source
is defined in trailblazer-macro gem (not trailblazer-operations)
https://github.com/trailblazer/trailblazer-macro/blob/master/lib/trailblazer/macro/model.rb
```
step Model(Song, :new) # this is the same as ctx[:model] = Song.new
step Model(Song, :find_by) # this is the same as ctx[:model] = Song.find_by params[:id]
step Model(Song, :find) # find params[:id]
# There will be automatic jump to error track if can not find the model
```

**Nested** http://trailblazer.to/gems/operation/2.0/api.html#nested can call
other operations
```
class Update < Trailblazer::Operation
  step Nested( Edit )

  # you can decide which step to perform in runtime
  step Nested( :build! )
  def build!(ctx, current_user:. **)
    current_user.admin? ? Create::Admin ? Create::NeedsModeration
  end

  # you can adapt parameters using input param
  step Nested( Multiplier, input: ->(options, some_data:, **) do
    { x: some_data[:pi_constant], y: 2 }
  end)

  # you can also pick specific items from nested context (not all are needed)
  step Nested( Edit, output: ->(options, mutable_data:, **) do
    {
      'contract.my' => mutable_data['contract.default'],
      'model'       => mutable_data['model']
    }
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
gem 'reform' # it depends on disposable gem which provide a Twin decorator, and
             # depends on representable gem which provide representer
             # deserializer
             # validation is in dry-validation or Activemodel::Validations

gem 'dry-validation'
# for rails Activemodel::Validations
gem 'reform-rails'
```
Actual validation can be implemented using Reform (with ActiveModel::Validation
or dry-validation) or using Dry::Schema (without Reform).
It allows mapping to nested models, via composition, to hash fields.
It is initialized with model for which you want to validate data against, and
once read, original model's values will never be accessed. To save to model you
can run `form.save` which will call `sync` and `model.save`.
Main purpose of form object is to not let form builders (when rendering),
validators (when validating) and writers (when acceptings params) to access the
model.

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
  end
end
```

For hash it use this `property :album, field: :hash`
http://trailblazer.to/gems/disposable/api.html#propertyhash


Properties can be `virtual: true` so that reform will not read, nor write to
model (it is used only in operation run).
`writeable: false` means that it will not write to model. To prevent writing
only on deserializing phase from params, but allow to write in operation step
and to sync with model
```
property :image_meta_data, deserializer: { writeable: false }
```

Populator is called when validate is triggered, but before deserialisation and
actual validation happens and ONLY when that attribute is present in validation
params. `populate_if_empty` is using `populator`.
For collections it will be called for each fragment, it receive options hash,
and `:frament` and `:index` can be extracted. If we have populator for single
value, `fragment` will be string, if we populate collection or object (json
column, or associated object) fragment will be hash or array.
Use `return skip!` to stop deserializing current fragment.
When populate_if_empty is called return value is imporant since it is used to
initialize property (which could be nested form).
Note that populator is not called (triggered) if we call `.validate my_prop:
nil` ie validate again value `nil` or key does not exists, but it will be called
if key exists and value is something not nil (false, string , number).
Inside populator method we need to set the form manually using `self` and
respective setter `self.artist = ` (return value is not used automatically as in
`populate_if_empty`).
```
property :artist, populator: ->(fragment:) { self.artist = Artist.find_by(id:
fragment) }
```
https://github.com/trailblazer/reform/issues/335
To manually populate property which does not exists in params, you can use
parse_pipeline, or `step :setup_params` in operation
```
property :name,
        deserializer: { parse_pipeline: ->(*) { ->(input, represented:, **) {
        represented.name= ... }} }
```

For has_one and has_many associations (which use collections) prepopulators are
used for rendering (for `new` action) and must be invoked manually
`@form.prepopulate!` and since it is invoked manually it will blindly run the
block and create nested form around whatever you instantiated in the block, It
is used for presentation ie show empty fields (not for validation of the form).
```
property :user, prepopulator: ->(*) { self.user = User.new },
```

For validation population use `populate_if_empty` which is run automatically
before deserialisation and actual validation happens. It is called for every
hash fragment that does not have a nested form yet return value of
`populate_if_empty` will be automatically wrapped in a nested form and added to
the main form. Note that params optionally have `_attributes` sufix with STRING
keys for both top properties and nested collection properties for example both
are valid params: `{ song: { 'users_attributes' => { '0' => { 'email' =>
'my@email.com'}}}}`, or `{ song: { users: [{ 'email' => '' } ]} }`.
inside method you can access `options[:fragment]['email']` ie hash for that
part and with string as keys (not `:email` symbols)
```
collection :users,
  skip_if: :all_blank, # dynamically decide whether or not an incoming hash
                       # should be deserialized to populate the form
  populate_if_empty: ->(*) { User.new } # this populator only runs when there
                            # is incoming hash, but no corresponding nested form
  populator: :populate_user! do

  property :email
  validates email, presence: true
end

def populate_user!(fragment:, **)
  self.user = (User.find_by(email: fragment["email"]) or User.new)
end
```

Note that we can use `ctx[:model].build_user` in custom `:populate_association`
step in operation (static way of population) so we don't need `prepopulator` nor
`populate_if_empty`.

To validate properties, you can use groups of `validation`
http://trailblazer.to/gems/reform/validation.html so we run them `after` or `if`
some other group is finished.

Inside `validation` group you can use dry validation schema
`required(:title).filled` see old
https://dry-rb.org/gems/dry-validation/0.13/basics/working-with-schemas/ or new
https://dry-rb.org/gems/dry-schema/1.5/
To define **custom predicates** you need to use `configure`. If you need access
to whole `form` you can add `with: { form: true }`
```
  validation do
    configure with: { form: true } do
      # inside this method we have access to `form`
      def unique?(value)
        Song.where(name: value).not(id: form.model.id).exists?
      end
    end
    required(:title).filled
    required(:body).maybe(min_size?: 9)
  end
```
and can be used in operation with this API (`initialize`, `validate`, `save`,
`errors`, `sync`, `prepopulate`)
but we can use macros: Build, Validate, Persist
for example in `rails c` you can run and see default contract
```
res = Venue::Operation::Create.(params: {})
res['contract.default'].errors
res['contract.default'].users
res['contract.default'].model.users
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
# in reform 2.2.4 in result['contract.default']

step Contract::Persist()
# this will save contract data. Note that it will not change ctx['model']
# reform_contract = ctx["contract.default"]
# reform_contract.save
```

Form is using model persist? method `form.persisted?` is actual
`form[:model].persisted?`.
Sometimes in firefox, when you navigate back, it remembers `_method` input with
different value than `patch` and error `Routing Error No route matches [POST]`
is raised. Solution is to hard refresh
https://github.com/rails/rails/issues/37864

Twins method `form.created?` will return true if decorated model was just
created ie persisted status changed from false to true. To use that we `feature`
```
module Thing::Contract
  class Create < Reform::Form
    feature Disposable::Twin::Persisted
```
We can use `added` for collections to check which was added using `<<` and not
by pulling from db on initialize state. *Imperative Callbacks* is to implicitly
write callback invocation where you need it, twin object graph collects low
level events like adding/deleting/creating/updating/destroying.


Reform is creating object graph from arbitrary input (it can wrap existing or
new models), validate object graph and process it and than sync to models.

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
Instead of `run Operation {block}` you can use oposite method
`Operation.reject() {block}` so block will be executed if operation is failed.

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

    flash[:alert] = "Venue deleted"
    redirect_to venues_path
  end
end
```

To show data from operation in the view, you can use result object
```
# step :a?
def a?(ctx, **)
  if true
    true
  else
    ctx[:error_message] = 'Hi'
    false
  end
end


result = run MyOp
flash[:alert] = result[:error_message]
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
# in controller
render cell(Venue::Cell::New, @form), layout: false

# in console
html = Venue::Cell::New.(Venue.new, current_user: User.last).()

# in view
<%= concept('venue/cell', venue) %>
# this will call: Venue::Cell.new(venue).show

# to render collection which will be joined into one html
concept('venue/cell', collection: venues)
```

Template name is generated from method name, you can call `render :new` to
render `app/concept/venue/view/new.slim`.
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
For different request format (html, js, json) you should have different method
than `show`, for example `concept('venue/cell', venue).(:append)`

```
class Comment::Cell < Cell::Concept
  class Grid < Cell::Concept
    include ActionView::Helpers::JavaScriptHelper

    def show
      # concept('comment/cell', collection: comments).to_s + paginate(comments).html_safe
      render :grid
    end

    def append
      %{
        var el = document.getElementById('next')
        el.outerHTML = "#{j show}"
      }
    end
  end
end
```

Inside methods you can access `model` and `options` object (other parameters
that you passed to cell and also `option[:context][:controller]` where `concept`
was used
https://github.com/trailblazer/cells-rails/blob/master/lib/cell/rails.rb#L12 ).
```
# app/concepts/venue/cell/edit.rb
module Venue::Cell
  class Edit < New
    def show
      render :new
    end

    def back
      link_to "Back", venue_path(model.id)
    end

    def delete
      link_to "Delete venue", venue_path(model.id), method: :delete
    end
  end
end
```
You can delegate `title` to `context[:model].title` using `property` class
method.
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
      link_to "Back to index", venues_path
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
      return "No venues" if model.size == 0
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

Caching in cells http://trailblazer.to/gems/cells/api.html#caching

```
# app/concepts/thing/cell.rb
class Thing::Cell < Cell::Concept
  include Cell::Caching::Notifications

  class Grid < Cell::Concept
    cache :show
  end
end
```

# Dry

Disposable Twins API http://trailblazer.to/gems/disposable/api.html
use dry-types

Twin is an intermediate object (decorator) between model and application params.
```
class AlbumTwin < Disposable::Twin
  property :title
end
```

## Dry types

https://dry-rb.org/gems/dry-types/1.2/ is used by dry schema and dry struct.
Used for value coercions, applying constrains, define complex structs or value
objects.
After installing with `gem install 'dry-types' 'dry-struct'` you can test in irb

```
require 'dry-types'
require 'dry-struct'

module Types
  include Dry.Types()
  # this will provide constants Decimal, Bool, String... Strict::String...
  # Coercible::String...
end

User = Dry.Struct(
  name: Types::String, # Types is our module
  age: Types::Integer,
)
# or
class User < Dry::Struct
  attribute :name, Types::Strict::String # Types is our module
end

User.new name: 'Bob', age: 35
```

You can use `[]` to pass directly without creating an object
`Types::Coercible::Integer['1a'] # Dry::Types::CoercionError (invalid value for
Integer(): "1a")`

To see all types
```
Types::Nominal.constants
 => [:String, :Decimal, :Float, :Hash, :Bool, :DateTime, :Date, :Range, :Class, :Any, :Symbol, :Time, :Integer, :Nil, :True, :Array, :False]

Types.constants - Types::Nominal.constants
 => [:Optional, :Params, :Coercible, :JSON, :Nominal, :Strict]
Types.constants
 ```
Categories of types are
* `Nominal` base type definitions with a primitive class
* `Strict` raise error if passed attribute of wrong type
* `Coercible` attempt to convert to the correct class using Ruby coercion (`'1'`
  will be `1` but `'1a'` will raise an ArgumentError: invalid value for Integer)
* `.optional` attribute value can be `nil` (alternative to `Maybe`)
* `.constrained(gteq: 18)` add custom contraints
  https://dry-rb.org/gems/dry-types/1.2/constraints/
* `.meta(info: 'some info')` add metadata
* `.default('draft')` to set default value if not provided
  https://dry-rb.org/gems/dry-types/1.2/default-values/ Block syntaxt is
  also possible `CallableDateTime = Types::DateTime.default { DateTime.now }` so
  `CallableDateTime[] #=> #<DateTime: 2020-04-01>`
* `|` sum of valid types https://dry-rb.org/gems/dry-types/1.2/sum/ `.optional`
  is implemented with `nil_or_string = Types::Nil | Types::String`

https://dry-rb.org/gems/dry-types/1.2/hash-schemas/
Hash schemas to define a type for a hash with a known set of keys. Add `?` to
mark optional. Other keys are ignored unless `.strict` option is used
```
user_hash = Types::Hash.schema(
  name: Types::String,
  age?: Types::Coercible::Integer
).strict
```
You can transform keys from string to symbols `.with_key_transform(&:to_sym)`.
Also use inheritance by calling `.shema` again
```
with_city = user_hash.schema(city: Types::String)
with_city[name: 'Me', city: 'NS')
```

You can also transform types with a block.

Use enum similar to Rails enum https://dry-rb.org/gems/dry-types/1.2/enum/
but this will raise error if not in list `"a" violates constraints
(included_in?(["text_open", "text_time"], "a") failed)` so it will not coerce.
```
    property :kind, type: Types::String.enum(Element.kinds)
```

Enable maybe extension https://dry-rb.org/gems/dry-types/1.2/extensions/maybe/
`gem install dry-monads` http://dry-rb.org/gems/dry-monads/ and irb
```
require 'dry-types'
Dry::Types.load_extensions(:maybe)
module Types
  include Dry.Types()
end

x = Types::Maybe::Strict::Integer[nil]
x
=> None
x = Types::Maybe::Strict::Integer[1]
x
=> Some(1)
```

## Dry monads

## Dry schema

https://dry-rb.org/gems/dry-schema/1.5/ is used by dry validation and provides
structural validation (validate key presence), pre-coercion filtering, so focus
is in structure validation and data types validation.
After `gem install dry-schema` try in irb

```
require 'dry/schema'

UserSchema = Dry::Schema.Params do
  required(:name).filled(:string)
  required(:age).maybe(:integer, gt?: 18)
  required(:address).hash do
    required(:street).filled(:string)
    required(:city).filled(:string)
  end
end

result = UserSchema.call name: 'Jane', age: nil, address: {city: 'NYC'}
result.succees? # => false
result.errors.to_hash => {:address=>{:street=>["is missing"]}}
```

Three things are happened:
* input keys are coerced to symbols using schema's key map
* input values are coerced based on type specs (for `.maybe(:array)` empty
  string `''` will be coerced to empty array `[]`).
* input keys and values are validated using defined schema rules

Basic macros https://dry-rb.org/gems/dry-schema/1.5/basics/macros/
* **value** is used to provide a list of all predicates that will be AND-ed
  `required(:age).velue(:integer, gt?: 18)` is `required(:age) { int? & gt?(18)
  }`
* **filled** means that value is non-nil and in case of String, Hash and Array,
  is it also not `.empty?`
  ```
  required(:tags).filled(:array) # is array? * filled?
  ```
* **maybe** when value can be nil (`optional` means that key can be ommited, but
  `maybe` requires key, but value can be nil)
  ```
  required(:age).maybe(:integer) # !nil?.then(int?)
  ```
* **hash** value is expected to be a hash
  ```
  required(:tags).hash do # hash? & filled? & schema { required(:name).filled }
    required(:name).filled(:string)
  end
  ```
  You can use that to define Nested data. It is the same as call `value(:hash)`.
  If nested hash is not required simply use `required(:tags).maybe(:hash?)`.

* **schema** similar to `.hash` but without `hash?` predicate
* **array** is used to apply predicate to every element in value that is
  expected to be an array
  ```
  # elements are string
  required(:tags).array(:str?)

  # elements are hashes
  requred(:tags).array(:hash) do
    required(:name).filled(:string)
  end
  ```
  If you need to apply predicates `min_size?: 1` use full form
  ```
  required(:tags).value(:array, min_size?: 1) do
    hash do
      required(:name).filled(:string)
    end
  end
  ```
* **each** is similary to `.array` but does not prepent `array?` predicate

When first argument is a symbol without question mark, it will be set as the
type. Inside `Dry::Schema::Params` for `:integer` will be resolved this class
`Dry::Schema::Params::Integer`.
Array member type
```
required(:number).value(array[:integer], size?: 3)
```

Built-in predicates (simpler than those used in dry-validation v0.13)
https://dry-rb.org/gems/dry-schema/1.5/basics/built-in-predicates/
* `nil?` check if key value is `nil` (similar `:false?` `:true?`)
  ```
  required(:sample).value(:nil?)
  ```
* `type?` check if class is equal given class (shorthand `str?`, `int?`,
  `float?`, `decimal?`, `bool?`, `time?`, `date_time?`, `array?`, `hash?`)
  ```
  required(:sample).value(type?: Integer)
  ```
* `empty?` check that either string, array or hash is empty
  ```
  required(:sample).value(:empty?)
  ```
* `filled?` check that either value is not-nil and in case of string, array or
  hash, is non-empty
  ```
  required(:sample).value(:filled?)
  ```
* `eql?` check if value is equal (similar: `lt?`, `lteq?`, `gt?`, `gteq?`,
  `size?`, `max_size?`, `min_size?`, `bytesize?`
  ```
  required(:sample).value(eql?: 1)
  ```
* `format?` check that string matches a given regular expression
  ```
  required(:sample).value(format: /*a/
  ```
* `included_in?` value is incuded in given array (similar `excluded_from?`)
  ```
  required(:sample).value(included_in?: [1,2])
  ```

Note that you can use predicate logic so invalid state will not crash one of
your rules https://dry-rb.org/gems/dry-schema/1.5/advanced/predicate-logic/
```
schema = Dry::Schema.Params do
  required(:age).value(gt?: 18)
end

# this will raise exception
schema.(age: 'a') # ArgumentError (comparison of String with 18 failed)

schema_with_integer = Dry::Schema.Params do
  required(:age) { int? & gt?(18) }
end
schema_with_integer.(age: 'a')
 => #<Dry::Schema::Result{:age=>"a"} errors={:age=>["must be an integer"]}>
```

You can define your base schema class that contains shared rules
https://dry-rb.org/gems/dry-schema/1.5/basics/working-with-schemas/
```
class AppSchema < Dry::Schema::Params
  define do
    config.messages.load_paths << '/my/app/config/locales/en.yml'
    config.messages.backend = :i18n

    # define common rules, if any
  end
end

# now you can build other schemas on top of the base one:
class MySchema < AppSchema
  # define your rules
end

my_schema = MySchema.new
```

**Optional keys** means that key can be ommited, but if present we can apply
rules https://dry-rb.org/gems/dry-schema/1.5/optional-keys-and-values/ it was
extracted from older dry-validation
https://dry-rb.org/gems/dry-validation/0.13/optional-keys-and-values/
`optional(:age).filled` here `optional` means that KEY `age` is not required,
but when it is present it should be filled (not a nil, empty sting, array or
hash) and has to be `int?` and `gt?: 18`

```
schema = Dry::Schema.Params do
  required(:email).filled(:string)
  optional(:age).filled(:integer, gt?: 18)
end
```

**Optional values** means that value can be `nil`, so in this example `age: nil`
is valid, as `age: 19` is also valid. Can not be empty string, maybe only
permits `nil`, but if you use `type: Types::Params::Integer` than coercion will
make empty string to nil .
```
  optional(:age).maybe(:integer, gt?: 18)
```

You can filter inputs before coercion. This is used in checking if date is in
right format.
https://dry-rb.org/gems/dry-schema/1.5/advanced/filtering/
```
  required(:birthday).filter(format?: /\d{4}-\d{2}-\d{2}/).value(:date)
```

You can also use Abstract syntax tree AST of the schema `schema.to_ast` and its
`schema.key_map` or `schema.info`.

To create custom type `StrippedString` and used it as `:stripped_string`
https://dry-rb.org/gems/dry-schema/1.5/advanced/custom-types/


## Dry validations

https://dry-rb.org/gems/dry-validation/1.5/
https://www.youtube.com/watch?v=nOUPIa7tWpA&list=PLqvlCCuOUZMQAuM6KJk_sWQnZXG00Su1_&index=5
Main reason of implementing dry-validations instead activerecord and
activemodel validations, is that it does not override methods like `:errors` or
`:attributes` (so you can have this names as colomn names). Validation is
performed on input data (not on the activerecord model).
Another reason is that validations are changing (for example adding `validate
:phone, presence: true)`) and if you forgot to migrate data you end up in bugs
when old objects are instantiated (in invalid state).
Another is separation of structural validation and type safety (dr-schema) and
on another side, domain validation rules. So simple first step validation are
inside `params`, `schema`, `json` (they have different coercing rules)  block
but something


Validations are expressed using contract object that are defined by schema with
basic type check and any additional rules that should be applied (it will be
applied after schema is verified).
Main value is separation of schema from domain validation logic which can be
simplified since we do not need to care about types (dry-schema will sanitize,
coerce and type check).
After installing `gem install dry-validation` in `irb` you can
```
require 'dry-validation'

class NewUserContract < Dry::Validation::Contract
  params do
    required(:email).filled(:string)
    required(:age).value(:integer)
  end

  rule(:age) do
    key.failure('must be greater than 18') if value <= 18
  end
end
contract = NewUserContract.new
contract.call(email: 'asd@asd.asd', age: '17')
=> #<Dry::Validation::Result{:email=>"asd@asd,asd", :age=>17} errors={:age=>["must be greater than 18"]}>
```

For defining dry-schema you can use `schema` or `params` (better for HTTP since
it will coerce string to int).
You can reuse schema
```
AddressSchema = Dry::Schema.Params do
  required(:country).value(:string)
end

ContactSchema = Dry::Schema.Params do
  required(:email).value(:string)
end

class NewUserContract < Dry::Validation::Contract
  params(AddressSchema, ContactSchema) do
    required(:name).value(:string)
  end
end
```

Use virtual attribute with default value to validate associations
```
  property :not_used, virtual: true, default: true

  validation do
    configure with: { form: true } do
      def not_used?(_value)
        # we can not proceed if is used
        form.model.songs.empty?
      end
    end
    required(:not_used).filled :not_used?
  end
```

`required(:f).value :none?` check that value is nil (note you can not use
`maybe?` with `none?` predicate)
https://dry-rb.org/gems/dry-validation/0.13/basics/built-in-predicates/#code-none-code

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


You need sometime to access global constant, for example in operation
```
# app/concepts/element/clone.rb
class Element::Create < Trailblazer::Operation
  step a
  step Model(::Element, :new)

  def a(ctx)
    Element # This is Trailblazer::Operation::Element, not Element model
  end
end
```
