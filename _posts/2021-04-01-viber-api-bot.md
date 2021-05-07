---
layout: post
---

# Viberroo

https://github.com/vikdotdev/viberroo

Add to Gemfile
```
# Gemfile
gem 'viberroo'
```

Create token on https://partners.viber.com/account/5558103922505666680/info and
add to rails

```
# config/initializers/viberroo.rb

Viberroo.configure do |config|
  config.auth_token = '<your_viber_bot_api_token>'
end
```

Use rake task to register webhook
```
# lib/tasks/viber.rake
namespace :viber do
  desc "TODO"
  task set_webhook: :environment do
    Viberroo::Bot.new(response: Viberroo::Response.new({})).set_webhook(
      url: 'https://baa8eb8b4f1f.ngrok.io/viber',
      event_types: %w[conversation_started subscribed unsubscribed],
      send_name: true,
      send_photo: true
    )
  end

  task remove_webhook: :environment do
    Viberroo::Bot.new.remove_webhook
  end
end
```

Use controller to receive events
```
    @response = Viberroo::Response.new(params.permit!)
    @bot = Viberroo::Bot.new(response: @response)
```

`@response.params.event` could be https://github.com/vikdotdev/viberroo/blob/master/lib/viberroo/response.rb#L40
`conversation_started`, `subscribed`, `unsubscribed`, `delivered`, `seen`,
`failed`, `message`

Bot always work as 1-to-1 communication. Use community to create a 1-to-many
communication.

Viber example keyboards
https://developers.viber.com/docs/tools/keyboard-examples/
