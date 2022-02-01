---
layout: post
---

# Cancan

use cancancan and define all your actions in app/models/ability. If you want to
use
[load_resource](https://github.com/ryanb/cancan/wiki/authorizing-controller-actions#load_resource),
you should separately define: index (with hash), show/edit/update/destroy (with
block or hash), new/create (with hash). For nested resources just write parent
association

Note that following next `cannot` rule will override a previous `can` rule, so
it is enough to set `can :manage, :all` and than write what `cannot :destroy,
Project`

Define abilities
https://github.com/CanCanCommunity/cancancan/wiki/defining-abilities
```
can :read, Article # autorize! :read, Article will return true
can :crud, Article # authorize! :read/:create/:update/:destroy, Article will return true
# https://github.com/CanCanCommunity/cancancan/wiki/Action-Aliases
aliase_action :index, :show, to: :read | :new, to: :create | :edit. to: :update
can :manage, Article # authorize! :any_action, Article will return true
can [:update, :destroy], [Post, Comment] # array notation
```
Hash of conditions
```
# can read only posts that belongs to user
can :read, Post, user_id: user.id
```
Defining with a block is evaluated only when instance is passed (for example not
in index action when class is used, ie if you call `can? :update, Project` it
will return true)
```
can :update, Project do |project|
  false
end
```

Defining with a block without other arguments can be used for defining
Abilities and roles in database
https://github.com/CanCanCommunity/cancancan/wiki/Abilities-in-Database
```
rails g model Permission user_id:integer name:string subject_class:string subject_id:integer action:string description:text

# app/models/ability.rb
class Ability
  include CanCan::Ability
  can do |action, subject_class, subject|
    # action: :read, subject_class: User, subject: 1 or nil
  end
end
```

Example that only admins can change, for example company_id, with
can_change_user_company_id
```
# in models/ability.rb
can :can_change_user_company_id, User if user.admin?

<!-- users/_form.html.erb -->
<% if can? :can_change_user_company_id, @user %>
  <!-- role checkboxes go here -->
<% end %>

# users_controller.rb
def update
  authorize! :can_change_user_company_id, @user if params[:user][:company_id]
  # ...
end
```


# Pundit

https://github.com/varvet/pundit

Install with gem and include in controller
```
# Gemfile
gem 'pundit'

# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  include Pundit
end

# generate default app/policies/application_policy.rb
rails g pundit:install
```

Sample ruby poro class
```
class PostPolicy
  attr_reader :user, :post

  def initialize(user, post)
    @user = user
    @post = post
  end

  def update?
    user.admin? or not post.published?
  end
end
```
Pundit will assume that:
* class has the same name as model + suffix `Policy`
* first argument is a `current_user` (called where it was invoked) and stored in
  `user`.
* second argument is object and stored in `record` (if you use generated
  ApplicationPolicy).
* define methods like method name plus ?

When using `authorize` you can set first argument as a class `authorize Post` or
a symbol for headless policy. Second argument could be action name: `authorize
@post, :destroy?`. To specify Policy class you can use `authorize @post,
policy_class: PostPolicy`

In controller `authorize @post` inside `def update` action will instantiate
policy with `current_user` and call method with `?` at the end, something like
```
unless PostPolicy.new(current_user, @post).update?
  raise Pundit::NotAuthorizedError, "not allowed to update? this #{@post.inspect}"
end
```

In view, you can use `if policy(@post).update?` method to check if current_user
is authorized.
You can rescue from not authorized requests
https://github.com/varvet/pundit#rescuing-a-denied-authorization-in-rails
```
def new
  authorize @user
  rescue Pundit::NotAuthorizedError => e
    flash.now[:alert] = e.message
    render :error
  end
end
```

## Headless policies

Headless policies, if you do not have corresponding model.
```
# app/policies/dashboard_policy.rb
class DashboardPolicy < Struct.new(:user, :dashboard)
  # ...
end
```
and use like
```
<% if policy(:dashboard).show? %>
  <%= link_to 'Dashboard', dashboard_path %>
<% end %>
```

## Scopes

To define scope on a model for which current_user have access, punding assumes:
* class has name `Scope` and is nested under the policy class
* first argument is user and second argument is a scope

```
class PostPolicy < ApplicationPolicy
  class Scope < Scope
    def resolve
      if user.admin?
        scope.all
      else
        scope.where(published: true)
      end
    end
  end

  def update?
    user.admin? or not record.published?
  end
end
```
and you can use like
```
def index
  # @posts = PostPolicy::Scope.new(current_user, Post).resolve
  @posts = policy_scope(Post)
end

def show
  @post = policy_scope(Post).find(params[:id])
end
```

To check if `authorize` is called you can use `verify_authorized` so it raises
error if `authorize` is not called.
```
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  include Pundit
  after_action :verify_authorized
end
```

Also you can check if user exists in ApplicationPolicy (so you not need to check
in other policies).

```
class ApplicationPolicy
  def initialize(user, record)
    raise Pundit::NotAuthorizedError, "must be logged in" unless user
    @user   = user
    @record = record
  end

  class Scope
    attr_reader :user, :scope

    def initialize(user, scope)
      raise Pundit::NotAuthorizedError, "must be logged in" unless user
      @user = user
      @scope = scope
    end
  end
end
```

Use alias after method are defined
```
class MyPolicy < ApplicationPolicy
  def create?
  end
  alias new? create?
end
```

# Action Policy

https://github.com/palkan/action_policy is similar to pundit

Generate application policy
```
rails generate action_policy:install
rails generate action_policy:policy post
```

Use in controller (policy class is infered from model name, method infers from
action name and current_user becomes user).
```
  # You can check in before action
  def _set_post
    authorize! @post
  end

  # or with specific method
  authorize! @post, to: :update?
  # or with specific policy
  authorize! @post, with: CustomPostPolicy

  # bang will raise error if not allowed
  rescue_from ActionPolicy::Unauthorized do |ex|
    # Exception object contains the following information
    ex.policy #=> policy class, e.g. UserPolicy
    ex.rule #=> applied rule, e.g. :show?
  end

  # make sure we call authorize! during the action, use this after_action
  verify_authorized
```

or use in view
```
<% if allowed_to? :edit?, post %>
```

For non models, services you need to pass any object and policy class
```
<% if allowed_to? :index?, Object, with: TranslationsPolicy %>
  <%= link_to 'Translations', translations_path %>
<% end %>
```

Define policy using `user` and `record` instance methods. Remember to use `?` as
suffix for method names.
```
class CommentPolicy < ApplicationPolicy
  # https://actionpolicy.evilmartians.io/#/authorization_context
  # If you have some pages without user, you can define that with allow_nil: true
  # do not define on ApplicationPolicy since you need to check user.nil? in
  # every method... For view, you can check current_user before policy check
  authorize :user, allow_nil: true

  # you can call other policies, synonim as `check?`
  def update?
    user.admin? || allowed_to?(:update?, record.post)
  end

  # pass fast or fail fast with `allow` and `deny`
  def show?
    allow! if user.admin?

    user.public?
  end

  # alias synonims so you do not need to repeat dry, NOTE that is will call if
  # method exists, so alias is used only when method missing
  # By default, ActionPolicy::Base adds one alias: alias_rule :new?, to: :create?.
  alias_rule :edit?, :destroy?, to: :update?

  # default rule is :manage (not matching anything, defaults are index? create?
  # https://github.com/palkan/action_policy/blob/master/docs/custom_policy.md
  def manage?
    true
  end
end
```

