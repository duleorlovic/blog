---
layout: post
title: State machines audit trails gem
---

# Old gem state_machine

Definition with:

~~~
class Objects
  state_machine :state, initial: :my_state do
  ...
  end
~~~

Inside definition you can use:

* `before_transition on: :my_state`, `after_transition on: :my_state`
* `after_failure on: :my_state`
* `around_transition`

* `event :my_event { transition :my_state, :my_other_state, if: Proc.new {
|my_object| my_object.valid? } }`
* `state :my_state do end`

In transitions you can use keyword `all` or `any` to match all/any states, and
`same` to match same state (usefull when you have list of matched states).
You can use `object.my_state?` to check if it is currently in that `my_state`.
Also you can use `object.can_my_event?` to check if it can change state using
this event. Also `object.fire_state_event(:my_event)` and
`object.state?(:my_state)`. You can use bang method to raise error if event
fails `object.my_event!`. List of all possible events is `object.state_events`
(or `object.status_events` to list all events for status column).

To list of all states you can use `Post.state_machines[:status].states.map
&:name`

You can use namespace `state_machine :alarm_state, initial: :active, namespace:
:alarm do` when all events and states have prefix `alarm_my_state` and suffix
`my_event_alarm`.

Instead of calling the event explicitly you can use `object.state_event =
"my_event"; object.save` to trigger event. You can manually change state but
then you will break state machine, so better is to update `state_event`. On
creation, initial state event is triggered, so you will have two events (from
nil to initial, and from initial to the value of state_event).

You can define transitions inside the context of an event. First transition that
matches the state and passes its conditions will be used.

~~~
event :repair do
  transition stalled: :parked, unless: :auto_shop_busy
  transition stalled: same
end
~~~

but also you can define transition inside the [context of
state](https://github.com/pluginaweek/state_machine#transition-context). Than
you do not need to use `:from` (since we know the state) but you need `:on`

~~~
state :parked do
  transition to: :idling, on: [:ignite, :shift_up], if: :seatbelt_on?
  def speed
    0
  end
end
~~~

When defining transitions instead of using hash rocket syntax you can use
`:from`, `:except_from`, `:to` and `:except_to` options

~~~
before_transition :parked => any - :parked, :do => :put_on_seatbelt
# is the same as
before_transition :from => :parked, :except_to => :parked, :do => :put_on_seatbelt
~~~

You can generate
[graph](https://github.com/pluginaweek/state_machine/blob/master/examples/Vehicle_state.png) with

~~~
sudo apt-get install graphviz
echo "gem 'ruby-graphviz', require: 'graphviz'" >> Gemfile
rake state_machine:draw CLASS=Customer,Location HUMAN_NAMES=true
gnome-open Location_status.png
~~~

# Old gem state_machine-audit_trail

~~~
cat "gem 'state_machine-audit_trail'" >> Gemfile
rails generate state_machines:audit_trail Post status
~~~

This will generate model and migration which you can update

~~~
# app/models/post_status_transition.rb
class PostStatusTransition < ActiveRecord::Base
  belongs_to :post
end
~~~

~~~
# db/migrate/123123123123_create_post_status_transitions.rb
class CreatePostStatusTransitions < ActiveRecord::Migration[5.1]
  def change
    create_table :post_status_transitions do |t|
      t.references :post, foreign_key: true
      # t.string :namespace # this is for new version of state_machines
      t.string :event
      t.string :from
      t.string :to
      # add your custom fields
      t.string :change_description
      t.text :comment
      t.timestamp :created_at
    end
  end
end
~~~

If you need more data you need to add more columns, for example
`change_description` (by server) and `comment` (by user) which are defined as
`attr_accessor :change_description, :comment` in model and populated in form or
before_transition. For other fields that you want to track, those columns need
to exists in table.

~~~
class Post < ApplicationRecord
  state_machine :status, initial: :walk do
    store_audit_trail context_to_log: :comment
    event :start do
      transition all => :run
    end
    event :stop do
      transition all => :walk
    end
  end
end
~~~

# Upgrade gem state_machines

New gem have some breaking changes. You need is to include additional
gem that will use hooks to initialize data

~~~
cat >> Gemfile << HERE_DOC
gem "state_machines"
gem 'state_machines-activerecord'
HERE_DOC
~~~

Before audit trail was adding description on `.save`, but in new gem, it is
adding on `.new` action and values are memoized

<https://github.com/state-machines/state_machines-audit_trail/issues/22>

Also, before we could use `Post.where(name: 'my name', status:
:enabled).first_or_create` event we had `state_machine :status, initial:
:created_new`. Now post will create with `Post.last.status # => 'initial'`

For audit trail you need to do some renaming
<https://github.com/state-machines/state_machines-audit_trail/wiki/Converting-from-former-state_machine-audit_trail-to-state_machines-audit_trail>

~~~
cat >> Gemfile << HERE_DOC
gem 'state_machines-audit_trail'
HERE_DOC
~~~

New version `rails g state_machines:audit_trail Post status` will generate same
migration with additional column: `t.string :namespace`. Note that you should
use singular `Post` and not plural `Posts`

For additional fiels that you want to store, you can use real columns, but its
easier to use some `description` which you can generate from other fields.

~~~
  before_transition :on => any do |object, transition|
    case transition.event
    when :start
      self.description = "My object started"
    end
~~~

In model you need

~~~
class Post < ApplicationRecord
  attr_accessor :change_description, :comment
  state_machine :status, initial: :walk do
    audit_trail context: [:change_description, :comment, :name]
    event :start do
      transition all => :run
    end
    event :stop do
      transition all => :walk
    end
  end

  def description=(text)
    @description = "My associated #{page.name} : #{text}"
  end

  def description
    # note that this is called on Post.new, so you should have page parameter
    # Post.new; Post.new; will call this twice, but without associated page
    # Post.new post_params.merge(page_id: current_page)
    @description ||= "My associated #{page.name}"
  end
end
~~~

In your view

~~~
  <% post.status_events.each do |event| %>
    <%= link_to event, action_post_path(post, event: event), method: :post %>
  <% end %>

  # or on update form
  <%= form.select :status_event, post.status_events %>

  # of in hidden field on form object
  <%= form.hidden_field :status_event, value: :edit %>

  # or plain input (no need for :value key)
  <%= hidden_field_tag :status_event, :edit %>
~~~

And controller

~~~

  def action
    @post.fire_status_event params[:event]
    redirect_to posts_path
  end

  # or on update form
  params.require(:post).permit(
    :name, :description, :status_event
  ).merge(
    updated_from_ip: request.remote_ip,
    updated_by: current_user,
  )
~~~

You can use `Constant.POST_STATUSES # { CREATED_NEW: "Created new" }` to list
all statuses and events in one file. Than in transition you can use `transition
all - Constant.POST_STATUSES.slice(:CREATED_NEW) => same`

Conditional validation callbacks hooks works fine if it is rails `validates` but
if it is customer validation `validate :my_method` than it won't work in
multiple states
<https://github.com/state-machines/state_machines-activerecord#state-driven-validations>
so instead of

~~~
state :first_gear, :second_gear do
  validate :speed_is_legal
end
~~~

~~~
state :first_gear, :second_gear do
  validate {|vehicle| vehicle.speed_is_legal}
end
~~~

Note that whole event change is wrapped in transaction, so if state validation
fails everything will be rolledback.

(short notation might rise expection 'no receiver given')
```
# if this line raises exception
validate(&:check?)
# than use longer syntax
validate { |location| location.check? }
```

If you want to validate specific event `advance_renewal` (event without
corresponding state) so you can not use `event :advance_renewal; transition all
=> same, if: Proc.new { |m| m.has_advance_package? }` since at the show time if
does not have advance package so `m.can_advance_renewal #=> false`. So you need
to use rails validate `validate  :advance_renewal_settings, on: [:update], if:
:advance_renewal?` and `def advance_renewal_settings;
errors.add(:advance_renewal_package, "Please select Advance Renewal
Package.")unless advance_renewal_package.present?; end`.

`before_transition`  and `after_transition` callbacks

# Other audit gems

* paper_trail
  ~~~
  bundle exec rails g paper_trail:install --with-changes
  bundle exec rake db:migrate
  ~~~
  in model
  ```
  has_paper_trail
  ```

  You can set custom event
  ```
  a.paper_trail_event = 'update title'
  a.update title: 'new'
  a.versions.last.event # 'update title'
  ```
  To see changes
  ```
  m.versions.last.changeset
  # { changed_column => [before, after], updated_at => DateTime }
  # {"full_name"=>["member_profile", "first change"], "updated_at"=>[Tue, 27 Jul 2021 06:27:31.493388000 UTC +00:00, Tue, 27 Jul 2021 06:28:07.407190000 UTC +00:00]}

  # to see all changes
  puts PaperTrail::Version.all.map(&:changeset).join"\n"

  # to see all changes for particular object
  o.versions.map {|v| v.changeset.each {|k,v| puts "#{k}: #{v.first.nil? ? 'nil' : v.first} -> #{v.second.nil? ? 'nil' : v.second}" } } && nil

  ```

  In test use helper https://github.com/paper-trail-gem/paper_trail#7-testing
  ```
  # in config/environments/test.rb
  config.after_initialize do
    PaperTrail.enabled = false
  end

  # in test/test_helper.rb
  def with_versioning
    was_enabled = PaperTrail.enabled?
    was_enabled_for_request = PaperTrail.request.enabled?
    PaperTrail.enabled = true
    PaperTrail.request.enabled = true
    begin
      yield
    ensure
      PaperTrail.enabled = was_enabled
      PaperTrail.request.enabled = was_enabled_for_request
    end
  end

  test 'something that needs versioning' do
    with_versioning do
      # your test
    end
  end
  ```

  When using uuid you need to change type for item_id
  https://github.com/paper-trail-gem/paper_trail/issues/363#issuecomment-249080184
  ```
  def change
    create_table :versions do |t|
      t.string   :item_type, null: false
      t.uuid     :item_id,   null: false
  ```
  Sometimes new versions is not save, I do not know why I got `o.versions # =>
  []` or there exists versions that I have not created.
  ```
  PaperTrail::Version.all.map &:item_type
  PaperTrail::Version.all.map &:item_id
  PaperTrail::Version.all.map &:item
  ```
