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
# this will add redis to Gemfile, stimulus to package.json, remove turbolinks
```

https://turbo.hotwire.dev/handbook/introduction

Turbo frames
Frames is used to scope navigation of that independent segments, keeping the
rest of the page inact. Note that links to other pages need to have
`data-turbo-frame': '_top'` same as with iframes
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

Turbo frame tag inline (lazily load) with `src: template_path` attribute
```
<%= turbo_frame_tag 'new_message', src: new_room_message_path(@room), target: '_top' %>
```

Turbo Stream with 5 actions: append, prepend, replace, update, remove

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

todo https://www.ombulabs.com/blog/rails/hotwire/hotwire-demo.html
https://evilmartians.com/chronicles/hotwire-reactive-rails-with-no-javascript
https://blog.engineyard.com/the-ruby-unbundled-series-why-you-should-check-out-hotwire-now?reddit
