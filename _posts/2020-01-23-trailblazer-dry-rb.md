---
layout: post
---

# Trailblazer

http://trailblazer.to/guides/

Concepts (dashboard index, create comment)

Operation implement functions of the app, using the:
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

  def assign_current_user!(options, model:, current_user:)
    model.created_by = current_user
  end
end
```
Operation always return result object
```
result = Song::Create.(params, current_user: current_user)
result.success?
result['model'] # => #<Song ...>
```

It is using `step` method which can be instance method, lambda or callable model
```
step :model!
step ->(options, **) { options['model'] = Song.new }
step MyStep

def model!(options, **)
  options['model'] = Song.new
end

# define Callable
class MyStep
  extend Uber::Callable
  def self.call(options, **)
    options['model'] = Song.new
  end
end
```

If step return value is falsey (nil or false) than operation result is marked as
failed and only steps marked as `failure` after failing step will be executed.
Instead of `step` you can use `success` which does not care about return value.

Step accepts two arguments, `options` (first params passed to .call will be
inside `options['params']`) and it is mean to be mutable (you can write output
to it and it will be merged to result object) and second is hash `**` which
accepts other params and can be extracted from first param options. If you state
them in method definition for very first step, than those arguments will be
required as second hash (can not be extracted from first params). Since you can
write to it in each step `options[:new_result] = SomeClass.new` you can extract
in following step `def step(options, new_result:)` and make it required.

## Macro

**Model** http://trailblazer.to/gems/operation/2.0/api.html#model-findby source
is defined in trailblazer gem (not trailblazer-operations)
https://github.com/trailblazer/trailblazer-macro/blob/master/lib/trailblazer/macro/model.rb
```
step Model(BlogPost, :new) # this is the same as options[:model] = BlogPost.new
step Model(BlogPost, :find_by) # this is the same as options[:model] = BlogPost.find_by params[:id]
# There will be automatic jump to error track if can not find the model
```

**Nested** http://trailblazer.to/gems/operation/2.0/api.html#nested can call
other operations
```
step Nested( Edit )
```

Policy::Guard http://trailblazer.to/gems/operation/2.0/policy.html#guard

## Reform Contract

Example contract form validation using Reform gem
http://trailblazer.to/gems/reform/index.html
```
gem 'dry-validation'
gem 'reform' # it depends on disposable gem which provide a Twin decorator, and
           # depends on representable gem which provide representer deserializer
           # validation is in dry-validation or Activemodel::Validations
# for rails Activemodel::Validations
gem 'reform-rails'
```
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
# app/concepts/blog_post/contract/create.rb
require 'reform'
require 'reform/form/dry'

module BlogPost::Contract
  class Create < Reform::Form
    include Dry
    property :title
    # populator is called when validate is triggered, but before deserialisation
    # and actual validation happens
    property :artist, populator: ->(options) {}

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
```
step Contract::Build( constant: BlogPost::Contract::Create )
# this will populate contract.default
# options['contract.default'] = BlogPost::Contract::Create.new(options[:model])

step Contract::Validate( key: :blog_post )
# this will fetch contract.default and valide against key
# reform_contract = options["contract.default"]
# result = reform_contract.validate(options["params"][:blog_post])

step Contract::Persist()
# this will save contract data. Note that it will not change options['model']
# reform_contract = options["contract.default"]
# reform_contract.save
```

http://trailblazer.to/gems/reform/populator.html has two ways, `prepopulator` is
for rendering and `populator` is on `validate`
Populator function accepts
* `collection` it is indentical to `songs` (or other resources you defined)
* you can access other properties since context is current form

## Custom steps

```
# app/concepts/blog_post/lib/notification.rb
class BlogPost::Notification
  def self.call(*)
    true
  end
end
```

Invoke with
```
step :notify!

def notify!
  options['result.notify'] = BlogPost::Notification.(current_user, model)
end
```

## Rails

Gemfile

```
gem 'dry-validation', '~> 0.10.6'
gem 'dry-types', '0.10.2'
gem 'trailblazer', '>= 2.0.3'
gem 'trailblazer-rails' # this will include trailblazer-loader
gem "trailblazer-cells"
gem 'cells-rails'
gem 'cells-hamlit'
# gem "cells-slim"

gem 'reform-rails'
```

http://trailblazer.to/gems/trailblazer/2.0/rails.html

It will load files from `app/concepts` (trailblazer-loader gem).
Also it will Add `run Blog::Index` method that will invoke the operation, pass
dependencies as `current_user`, and set `@operation`, `@model` (also `@blog`
which is identical to `@model`) and `@form` (if exists, if `Contract:Build` was
called, it is equal to `result['contract.default']` ) and `result`
object (by convention operation should populate `result['model']` object or
array for index operation). If operation can go wrong (success and error track)
than you can use the block syntax `run BlogPost::Update do |result|` which will
be called on success and you should `return redirect_to path` so the code after
`run` will not be executed (fail path).
Also it provides `render cell(Blog::Cell::Index, @form)` method to render.

All file and class names should be singular.
Here is example controller
```
# app/controllers/blog_posts_controller.rb
class BlogPostsController < ApplicationController
  def new
    run BlogPost::Create::Present
    render cell(BlogPost::Cell::New, @form), layout: false
  end

  def create
    run BlogPost::Create do |result|
      return redirect_to blog_posts_path
    end

    render cell(BlogPost::Cell::New, @form), layout: false
  end

  def show
    run BlogPost::Show
    render cell(BlogPost::Cell::Show, result["model"]), layout: false
  end

  def index
    run BlogPost::Index
    render cell(BlogPost::Cell::Index, result["model"]), layout: false
  end

  def edit
    run BlogPost::Update::Present
    render cell(BlogPost::Cell::Edit, @form), layout: false
  end

  def update
    run BlogPost::Update do |result|
      flash[:notice] = "#{result["model"].title} has been saved"
      return redirect_to blog_post_path(result["model"].id)
    end

    render cell(BlogPost::Cell::Edit, @form), layout: false
  end

  def destroy
    run BlogPost::Delete

    flash[:alert] = "Post deleted"
    redirect_to blog_posts_path
  end
end
```

## Cells

view model uses helpers that are not globals, but instance methods. It is faster
than ActionView because there is no code.
Inside template you can access `model`

```
# app/concepts/blog_post/cell/new.rb
module BlogPost::Cell
  class New < Trailblazer::Cell
    include ActionView::RecordIdentifier
    include ActionView::Helpers::FormOptionsHelper
    include SimpleForm::ActionViewExtensions::FormHelper
  end
end
```

```
# app/concepts/blog_post/cell/edit.rb
module BlogPost::Cell
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

```
# app/concepts/blog_post/cell/show.rb
module BlogPost::Cell
  class Show < Trailblazer::Cell
    property :title
    property :body

    def current_user
      return options[:context][:current_user]
    end

    def time
      model.created_at
    end

    def edit
      link_to "Edit", edit_blog_post_path(model.id)
    end

    def delete
      link_to "Delete", blog_post_path(model.id), method: :delete, data: {confirm: 'Are you sure?'}
    end

    def back
      link_to "Back to posts list", blog_posts_path
    end
  end
end
```

```
# app/concepts/blog_post/cell/item.rb
module BlogPost::Cell
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
# app/concepts/blog_post/cell/index.rb
module BlogPost::Cell
  class Index < Trailblazer::Cell
    def total
      return "No posts" if model.size == 0
    end
  end
end
```

http://trailblazer.to/api-docs/#what-is-trailblazer

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

# What you loose by using Trailblazer

* you need to learn DSL which change from version to version (for example does
  failure continue or not, to the other failure steps)
* errors for trailblazer because of eager loading or something, restart server
  helps
  ```
  uninitialized constant #<Class:0x00007f6b986a5da0>::Index Did you mean? Bindex` .
  NameError (uninitialized constant #<Class:0x000055ab8fc752f0>::Update
  ```
