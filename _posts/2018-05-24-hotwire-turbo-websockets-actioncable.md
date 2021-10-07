---
layout: post
---


# Hotwire

Note that Action Cable is deprecated in favor of hotwire. Here is old example
https://github.com/duleorlovic/premesti.se/commit/ad7cefe192cbbb6b97c13860eb6410d1b02cb5fc

Example of local hotwire folders
~/rails/tmp/turbo_modal_boostrap
*** ~/rails/tmp/hotwire ***
~/rails/tmp/rails_forms
~/rails/nested-habtm-forms-for-associations-in-rails


https://hotwire.dev/

GET are HTML and PATCH/POST are TURBO_STREAM requests for ALL forms on the page.
You should be able to edit inline twice double (at this stage, without importing
turbo in js... if you import than add turbo_stream.replace response)
There is an error `Error: Form responses must redirect to another location` if
we submit a form without `turbo_stream_tag`
https://github.com/hotwired/turbo-rails/issues/12#issuecomment-749857225


To run javascript on when turbo frame is loaded you can listen to events
https://turbo.hotwire.dev/reference/events
or you can use stimulus controller connect event.

Note that you do not have access to request when we broadcast over websocket so
to differentiate between current_user you need to use data attributes or meta
tag with `current_user.id` and add class in js if matches `author.id`
https://discuss.hotwire.dev/t/how-to-pass-current-user-id-to-a-controller/287/4
https://discuss.hotwire.dev/t/building-a-real-time-chat-system-having-trouble-with-chat-bubble-styling/2534/6

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

# Turbo drive

Once installed it is enabled for all links. To disable for particular link you
can use
https://turbo.hotwired.dev/handbook/drive#disabling-turbo-drive-on-specific-links-or-forms
```
<a href="/" data-turbo="false">Disabled</a>
```

Group disable does not work in Firefox (probably since bubling is not the same)
so better is to disable each link

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
