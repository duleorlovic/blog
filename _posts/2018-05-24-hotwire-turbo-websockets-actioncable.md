---
layout: post
---

# Action Cable

https://www.sitepoint.com/create-a-chat-app-with-rails-5-actioncable-and-devise/
example
https://github.com/duleorlovic/premesti.se/commit/ad7cefe192cbbb6b97c13860eb6410d1b02cb5fc

`Consumer` is a client of a web socket connection that can subscribe to one or
multiple channels. Each ActionCable server may handle multiple connections (it
is created per browser tab, so one user can open two tabs to make two
consumers). When consumer is subscribed to a channel they act as a subscriber.
Each channel is streaming zero or more broadcastings. Anything that is
transmitted by broadcaster is sent to channel subscribers.

~~~
rails g channel chat
    test/channels/chat_channel_test.rb
    app/channels/chat_channel.rb
    app/javascript/channels/index.js
    app/javascript/channels/consumer.js
    app/javascript/channels/chat_channel.js
~~~

If you `stream_for chat` you can broadcast with

~~~
ActionCable.server.broadcast("chat:#{chat.id}", data: data)
ChatChannel.broadcast_to chat, data: data
~~~

Note that you should use redis instead of async if you want to send from console
```
# config/cable.yml
development:
  adapter: redis
  url: <%= ENV.fetch("REDIS_URL") { "redis://localhost:6379/1" } %>
  channel_prefix: my_app_production

test:
  adapter: async

production:
  adapter: redis
  url: <%= ENV.fetch("REDIS_URL") { "redis://localhost:6379/1" } %>
  channel_prefix: my_app_production
```

or in javascript

~~~
App.chat = App.cable.subscriptions.create 'ChatChannel'
App.chat.send({data: data})
~~~

You can call server method with `@perform 'away', my_id:
$('main').data('my-id')`

You can not use devise `current_user` when rendering for actioncable, there will
be an error

~~~
ActionView::Template::Error (Devise could not find the `Warden::Proxy` instance on your request environment.
Make sure that your application is loading Devise and Warden as expected and that the `Warden::Manager` middleware is present in your middleware stack.
If you are seeing this on one of your tests, ensure that your tests are either executing the Rails middleware stack or that your tests are using the `Devise::Test::ControllerHelpers` module to inject the `request.env['warden']` object for you.):
~~~

https://gorails.com/episodes/realtime-ssh-logs-with-actioncable

# Hotwire

https://hotwire.dev/

```
gem 'hotwire-rails'
# this includes stimulus-rails and turbo-rails gems

rails hotwire:install
# this will install stimulus and turbo https://github.com/hotwired/hotwire-rails/blob/28d25901c0b0b4492e473478e7e10ca9fc94213e/lib/tasks/hotwire_tasks.rake#L3
# there are two ways, using asset pipeline and webpacker
# Removed from Gemfile turbolinks and from packs/application.js

rails turbo:install:asset_pipeline
# added to app/views/layouts/application.html.erb
# <%= turbo_inlude_tags %> this is old
<%= yield :head %>
<%= javascript_include_tag "turbo", type: "module" %>

rails turbo:install:webpacker
# Please check if this are added to app/javascript/packs/application.js
import '@hotwired/turbo-rails'
# There is an issue when we import turbo in javascript, double click
# I have a problem with double request it seems that turbo frames does not work
# (it triggers only once) second click on edit does nothing
# one way to solve is to add response in controller
  format.turbo_stream { render turbo_stream: turbo_stream.replace("#{_params_step}-edit", template: "profile/#{_params_step}") }
```
https://github.com/hotwired/turbo-rails/issues/180

GET are HTML and PATH/POST are TURBO_STREAM requests for ALL forms on the page.
You should be able to edit inline twice double (at this stage, without importing
turbo in js... if you import than add turbo_stream.replace response)
There is an error `Error: Form responses must redirect to another location` if
we submit a form without `turbo_stream_tag`
https://github.com/hotwired/turbo-rails/issues/12#issuecomment-749857225


https://turbo.hotwire.dev/handbook/introduction

## Turbo frames

Frames is used to scope navigation of that independent segments, keeping the
rest of the page inact. Note that links to other pages need to have target
`data-turbo-frame': '_top'` (similar as with iframes)
Turbo frame tag with block
  ```
  # show.html.erb
  <%= turbo_frame_tag 'room' do %>
    <p>
      <strong>Name:</strong>
      <%= @room.name %>
    </p>

    <p>
      <%= link_to 'Edit', edit_room_path(@room) %> |
      <%= link_to 'Back', rooms_path, 'data-turbo-frame': '_top' %>
    </p>
  <% end %>

  # edit.html.erb
  <%= turbo_frame_tag 'room' do %>
    <%= render 'form', room: @room %>
  <% end %>
  ```

Turbo frame tag inline (laizy lazily load) with `src: template_path` attribute
Use `target: '_top'` and `redirect_to @room` on server, to reload the page
Note that usually target attribute on turbo_frame_tag in response to server does
not have an effect, always define on tag that is first rendered.
```
<%= turbo_frame_tag 'new_message', src: new_room_message_path(@room), target: '_top' %>
```
You can also add class to turbo_frame, but be aware that you need to use on
initial frame `<%= turbo_frame_tag 'new_message', class: 'my-class' do %>`

To run javascript on when turbo frame is loaded you can listen to events
https://turbo.hotwire.dev/reference/events
or you can use stimulus controller connect event.
For example reset input when turbo finishes
```
  // <%= form_with ..., html: { 'data-controller': 'forms', 'data-action': 'turbo:submit-end->forms#reset' } do |f| %>
  reset() {
    console.log('forms#reset')
    this.element.reset()
  }
```

Another example is to automatically open modal, and close on submit
```
<%= turbo_frame_tag 'modal' do %>
  <div class="modal fade" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true" data-controller='start-modal-on-connect'>
    <button type="button" class="close" data-dismiss="modal" aria-label="Close"> <span aria-hidden="true">&times;</span></button>
    <%= button_to 'Ok', interests_path(to_member_profile_id: @interest.to_member_profile.id), class: 'btn btn-primary mb-3', 'data-action': 'start-modal-on-connect#close' %>

// app/javascript/controllers/start_modal_on_connect_controller.js
import { Controller } from 'stimulus'

export default class extends Controller {
  connect() {
    console.log('start-modal-on-connect')
    $(this.element).modal()
  }

  close() {
    console.log('start-modal-on-connect#close')
    $(this.element).modal('hide')
  }
}
```
Usually when modal is rendered from server, I use target value to `target
"modal-123"` and inside modal I use `target: "express_interest_buttons-123"`
```
# app/views/interets/index.html.erb
            <%= turbo_frame_tag "modal-#{member_profile.id}", target: "express_interest_buttons-#{member_profile.id}" %>
            <%= render 'interests/express_interest_buttons', member_profile: member_profile %>
              which is:
              <%# turbo_frame_tag "express_interest_buttons-#{member_profile.id}", target: "modal-#{member_profile.id}" do %>
                <%= link_to new_cancel_interest_path(interest), class: "btn btn-primary interest-shown interestbtn" do %>

# app/controllers/interests_controller.rb
  def new_cancel
    authorize @interest
  end

  def cancel
    authorize @interest
    @interest.cancelled!
    render partial: 'express_interest_buttons', locals: { member_profile: @interest.to_member_profile }
  end

# app/views/interes/new_cancel.html.erb
<%= turbo_frame_tag "modal-#{@interest.to_member_profile.id}", target: "express_interest_buttons-#{@interest.to_member_profile.id}" do %>
  <%= button_to 'Ok', cancel_interest_path(@interest), class: 'btn btn-primary mb-3', 'data-action': 'start-modal-on-connect#close', method: :patch %>
```

Turbo Stream with 5 CRUD actions: append, prepend, replace, update, remove to
target element with #id. It is limited to DOM changes, for javascript invocation
use stimulus.


```
<turbo-stream action="append" target="messages">
  <template>
    <div id="message_1">My new message!</div>
  </template>
</turbo-stream>

<turbo-stream action="replace" target="message_1">
  <template>
    <div id="message_1">This changes the existing message!</div>
  </template>
</turbo-stream>
```

In in response we use helper `turbo_stream.append 'id', partial: 'edit'` for
example
```
# app/views/posts/create.turbo_stream.erb
<%= turbo_stream.append 'messages', @message %>
```
or in controller

```
def create
  respond_to do |format|
    format.turbo_stream do
      render turbo_stream: turbo_stream.append 'id', partial: 'edit', locals: { post: @post }
    end
  end
```

Use tag to create websocket connection with channel dom_id @room
```
<%= turbo_stream_from @room %>
```
If you do not see cable connection that try to `rm -rf public/packs`.
and in model instead of `after_create_commit -> { broadcast_append_to room }`
you can use async job to broad cast all changes using partial
`app/messages/_message.html.erb` to `message.room` stream channel.
```
class Message < ApplicationRecord
  belongs_to :room
  broadcasts_to :room
end
```

Note that you do not have access to request when we broadcast over websocket so
to differentiate between current_user you need to use data attributes or meta
tag with `current_user.id` and add class in js if matches `author.id`
https://discuss.hotwire.dev/t/how-to-pass-current-user-id-to-a-controller/287/4
https://discuss.hotwire.dev/t/building-a-real-time-chat-system-having-trouble-with-chat-bubble-styling/2534/6
```

// app/views/layouts/application.html.erb
    <meta content="Free Matrimonial website for Indian Community in US & Canada" name="description" />

// app/javascript/controllers/message_chat_controller.js
import { Controller } from 'stimulus'

export default class extends Controller {
  // <li class="send-msg" data-controller='single-message' data-single-message-member-profile-id='<%= message.member_profile.id %>' hidden>
  connect() {
    console.log('message-chat#connect')
    if (this.currentMemberProfileId == this.memberProfileId)
      this.element.classList.add('message--righted')

    this.element.hidden = false
  }

  get currentMemberProfileId() {
    return document.querySelector("[name=current-member-profile-id]").content
  }

  get memberProfileId() {
    return this.data.get('memberProfileId')
  }
}
```

Redirect inside turbo_frame_tag
  https://discuss.hotwire.dev/t/form-redirects-not-working-as-expected/2058/6
Solution is using stimulus event
https://github.com/hotwired/turbo/issues/138#issuecomment-802171574
```
// app/javascript/controllers/turbo_form_submit_redirect_controller.js
import { Controller } from "stimulus"
import * as Turbo from "@hotwired/turbo"

export default class extends Controller {
  connect() {
    this.element.addEventListener("turbo:submit-end", (event) => {
      this.next(event)
    })
  }

  next(event) {
    if (event.detail.success) {
      Turbo.visit(event.detail.fetchResponse.response.url)
    }
  }
}
```
Usage is with

```
<turbo-frame id='phone' data-controller='turbo-form-submit-redirect'>
```
note that it needs to be on first frame in case you have steps that replaces
several templates

To disable turbo drive on specific element using `data-turbo="false"`
https://turbo.hotwire.dev/handbook/drive#disabling-turbo-drive-on-specific-links-or-forms
for example on form
```
<%= bootstrap_form_with model: @member_profile, url: profile_update_path, method: :patch, html: { 'data-turbo': false } do |f| %>
```

Error when adding stimulus controller that use importing in js like
`import * as Turbo from "@hotwired/turbo"`
```
Uncaught DOMException: Failed to execute 'define' on 'CustomElementRegistry': the name "turbo-frame" has already been used with this registry
```
and I see that `cable` connection is not created when this error is present.
Solution is to import in main pack anywhere (below or above `import
"controllers"`). Error will remain, but at least `cable` connection is created
```
app/javascript/packs/application.js
// without this line, cable websocket connection will fail if you use turbo in
// stimulus controllers: import * as Turbo from "@hotwired/turbo"
import '@hotwired/turbo-rails'
```

Note that when you include turbo in js than default html response has double
request problem (search above for `double`)

WS websocket tab in Network tab is showing `Websocket` connection when you are
running `bin/webpack-dev-server` server which is sending updates using that
connnection.


# Devise fix

https://gorails.com/episodes/devise-hotwire-turbo
```
# app/controllers/turbo_controller.rb
class TurboController < ApplicationController # rubocop:todo Lint/ConstantDefinitionInBlock
  class Responder < ActionController::Responder
    def to_turbo_stream
      controller.render(options.merge(formats: :html))
    rescue ActionView::MissingTemplate => e
      raise e if get?

      if has_errors? && default_action
        render rendering_options.merge(formats: :html, status: :unprocessable_entity)
      else
        redirect_to navigation_location
      end
    end
  end

  self.responder = Responder
  respond_to :html, :turbo_stream
end

# config/initializers/devise.rb
# https://gorails.com/episodes/devise-hotwire-turbo
Rails.application.reloader.to_prepare do
  class TurboFailureApp < Devise::FailureApp # rubocop:todo Lint/ConstantDefinitionInBlock
    def respond
      if request_format == :turbo_stream
        redirect
      else
        super
      end
    end

    def skip_format?
      %w[html turbo_stream */*].include? request_format.to_s
    end
  end
end

Devise.setup do |config|

  # ==> Controller configuration
  # Configure the parent class to the devise controllers.
  config.parent_controller = 'TurboController'

  # ==> Warden configuration
  config.warden do |manager|
    manager.failure_app = TurboFailureApp
  end
end
```

# Sample apps

bootstrap modal https://github.com/dparfrey/turbo_modal_bootstrap
twitter https://github.com/gorails-screencasts/hotwire-twitter-clone
todoapp https://github.com/johnreitano/foo2


https://www.driftingruby.com/episodes/hotwire
https://www.youtube.com/watch?v=b7dx1Yt3FzU
todo https://www.ombulabs.com/blog/rails/hotwire/hotwire-demo.html
https://evilmartians.com/chronicles/hotwire-reactive-rails-with-no-javascript
https://blog.engineyard.com/the-ruby-unbundled-series-why-you-should-check-out-hotwire-now?reddit
