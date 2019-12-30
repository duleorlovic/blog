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
