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
# I have a problem with double request it seems that turbo frames does not work
# (it triggers only once) second click on edit does nothing
# one way to solve is to add response in controller
  format.turbo_stream { render turbo_stream: turbo_stream.replace("#{_params_step}-edit", template: "profile/#{_params_step}") }
```

GET are HTML and PATH/POST are TURBO_STREAM requests
You should be able to edit inline twice double.

https://turbo.hotwire.dev/handbook/introduction

Turbo frames
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

Turbo frame tag inline (lazily load) with `src: template_path` attribute (not
sure why we need to set up target: '_top' on frame tag)
```
<%= turbo_frame_tag 'new_message', src: new_room_message_path(@room), target: '_top' %>
```

To run javascript on when turbo frame is loaded you can listen to events
https://turbo.hotwire.dev/reference/events
or you can use stimulus controller connect event.
For example for modal we automatically open it, and we close on submit
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

Turbo Stream with 5 CRUD actions: append, prepend, replace, update, remove (it
is limited to DOM changes, for javascript invocation use stimulus)


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

In in view we use helper `turbo_stream.append 'id', partial: 'edit'` for example
```
# app/views/posts/create.turbo_stream.erb
<%= turbo_stream.append 'messages', @message %>
```
In controller

```
def create
  respond_to do |format|
    format.turbo_stream do
      render turbo_stream: turbo_stream.append 'id', partial: 'edit', locals: { post: @post }
    end
  end
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
