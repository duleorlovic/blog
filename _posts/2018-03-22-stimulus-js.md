---
layout: post
---

# Install

To checkout demo you can

~~~
git clone https://github.com/stimulusjs/stimulus-starter.git
cd stimulus-starter
yarn install
yarn start
gnome-open http://localhost:9000/
~~~

To add to existing Rails app

```
rails webpacker:install:stimulus
```

# Magic keywords

cheatsheet https://gist.github.com/mrmartineau/a4b7dfc22dc8312f521b42bb3c9a7c1e

* `data-controller='hello-word'` controller that is defined in
  `hello_word_controller.js`.  Each dash corresponds to underscore (or dash) in
  file name, two dashes corresponds to subfolder `users--list-item` ->
  `users/list_item_controller.js`
* `data-action='click->hello-word#greet'` event name is `click`, controller
  `hello-word` dashed case (note that camelcase name helloWord or snake case
  hello_word is not used anywhere, only example is when you define method in
  cammes case `sayHello()`).
  If element is `<a>`, `<button>`, `<input type="submit">` than you do not need
  to write `click->`.
  `<input>`, `<select>` and `<textarea>` has `change` as default event.
  `<form>` has `submit` default event.
  If you want to `event.preventDefault()` (for example click on link) than pass
  parameter `greet(event) {}`. To see who invoke the click you can use
  `event.currentTarget`
* `data-hello-target='name'` creates `nameTarget` property in a controller so we
  can use it to access element and set value. We need to declare it also inside
  controller `static targets = [ 'name' ]`. Beside `this.nameTarget` you can
  check if there are more targets like this `this.nameTargets` or if exists at
  all `this.hasNameTarget` (return true or false)
  You can use any term, for example `static values = { authorId: String }` and
  `this.authorIdValue`
* to access current element on which whole controller is connected you can use
  `this.element`. To access element on which action is triggered you can pass
  the event to the action `hello(event) { event.currentTarget }`
* `data-hello-index='1'` used to pass data to CONTROLLER which you can get on
  initialize instead of `this.element.getAttribute('data-hello-index'))` you can
  use stimulus shorthand `this.data.get('index')`. Only for data on controller
  element. To access data on some input (not controller) you can attach target
  and use `this.nameTarget.getAttribute('data-hello-index')`.
  But since controller can be initialized on parent of the action element,
  better is to use `event.currentTarget.getAttribute('data-hello-index')`
  Also `this.data.has('index')` to check if data existis and
  `this.data.set('index', 2)` to set data so controller. Do not need to store
  any data in js, just use those setter and getter to store data in DOM.
  For complex json objects you can use `.to_json` and jQuery `.data()`
  ```
  <%= f.select :company, {}, 'data-my-controller-target': 'second', 'data-my-controller-groups': MyModel::MY_HASH.to_json %>
  let groups = $('[data-my-controller-target=second]').data('myControllerGroups')
  ```

* communicating between controllers is best to use event dispatch https://github.com/stimulusjs/stimulus/issues/200#issuecomment-434731830
  ```
  ```

  for addListener event listener you can use closures
  ```
  let controller = this
  some.addListener('some_event', function() {
    controller.do_it()
  }
  ```

  but better is to use events

  ```
window.dispatchWindowEvent = function(event_name, ...detail) {
  const event = new CustomEvent(event_name, { detail: detail })
  window.dispatchEvent(event)
}

    autocomplete.addListener('place_changed', function() {
      let place = autocomplete.getPlace();
      window.dispatchWindowEvent('google-maps-place-changed', marker, place)
      console.log('place_changed');
    });


           google-maps-place-changed@window->google-map#placeChanged

  placeChanged(event) {
    let marker, place
    [marker, place] = event.detail
  ```

  for google initMap callback you can dispatch event from window
```
window.dispatchMapEvent = function(...args) {
  const event = new CustomEvent('google-maps-callback', { args: args })
  window.dispatchEvent(event)
}
```

Controllers

~~~
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  // triggered when controller is connected to the document
  // you can add classes to element
  connect() {
    this.element.classList.add('controller-initialized')
  }
  // anytime when controller is disconnected from the dom
  disconnect() {}
  // initialize is once when controller is first instantiated
  initialize() {
    this.showCurrentSlide()
  }

  showCurrentSlide() {
    this.slideTargets.forEach((el, i) => {
      el.classList.toggle("slide--current", this.index == i)
    })
  }

  # getter
  get index() {
    return parseInt(this.data.get('index'))
  }

  # setter
  set index(value) {
    this.data.set('index', value)
  }
}
~~~

Example of adding Rxjs to stimulus so user on slow connections get latest
results, do not load if they clicked on the same link and show loader
https://www.mikewilson.dev/posts/stimulus-and-rxjs-for-a-spa-like-experience/

# Reflex

To install run
```
bundle add stimulus_reflex
rails stimulus_reflex:install
```

Invoke methods using `data-reflex` attributes
```
# app/views/books/_form.html.erb
<%= form.text_field :title, data: { reflex: 'change->ExampleReflex#form', current_count: 1 } %>
```

Pass parametars using data attributes and access them using `element.dataset`
note that you should use strings instead of simbols for multiwords, since `data:
{ current_count: 1 }` will translate underscore to minus
`element.dataset['current-count']`.
```
# app/reflex/counter_reflex.rb
class CounterReflex < StimulusReflex::Reflex
  def increment
    @current_count = element.dataset['current-count']
  end
end
```

To trigger from javascript you need to register and use stimulate. Passign
arguments in different way, now is method args instead of `element.dataset`
as with when we use data attributes.

```
# app/views/books/_form.html.erb
  <div data-controller='hello-word' data-hello-word-count='<%= @count %>'>
    count <%= @count %>
    <div data-target='hello-word.output'></div>
    <%= link_to 'Increase', '#', data: { action: 'hello-word#increment' } %>
  </div>

# app/javascript/controllers/hello_controller.js
import { Controller } from "stimulus"
import StimulusReflex from 'stimulus_reflex'

export default class extends Controller {
  static targets = [ "output" ]

  connect() {
    this.outputTarget.textContent = 'Hello, Stimulus!'
    StimulusReflex.register(this)
  }

  increment(event) {
    this.outputTarget.textContent = 'trying to increment'
    let count = this.data.get('count')
    this.stimulate('ExampleReflex#increase_from_js', count)
  }
}


# app/reflex/example_reflex.rb
class ExampleReflex < ApplicationReflex
  def increase_from_js(count = 0)
    @count = count.to_i + 1
  end
end
```

Inside reflex, you can access session which you can use in controller.
```
# app/reflex/example_reflex.rb
  def increase_from_js(count = 0)
    @count = count.to_i + 1
    session[:count] = @count
  end
    @count = session[:count]
  end

# app/controllers/books_controller.rb
  def index
    @count = session[:count]
  end
```

In controller you should use conditional assignment
```
# app/controllers/books_controller.rb
  def index
    @book ||= Book.new
  end
```

Use generator `rails generate stimulus_reflex user` to generate
`app/javascript/controllers/user_controller.js` and `app/reflexes/user_reflex.rb
`.

# Nested form

https://web-crunch.com/posts/ruby-on-rails-marketplace-stripe-connect
```
// app/javascript/controllers/nested_form_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = ["add_perk", "template"]

  add_association(event) {
    event.preventDefault()
    var content = this.templateTarget.innerHTML.replace(/TEMPLATE_RECORD/g, new Date().valueOf())
    this.add_perkTarget.insertAdjacentHTML('beforebegin', content)
  }

  remove_association(event) {
    event.preventDefault()
    let perk = event.target.closest(".nested-fields")
    perk.querySelector("input[name*='_destroy']").value = 1
    perk.style.display = 'none'
  }
}
```

In view https://web-crunch.com/posts/ruby-on-rails-marketplace-stripe-connect
```
# app/views/projects/_form.html.erb
  <div data-controller="nested-form">
    <template data-target='nested-form.template'>
      <%= form.fields_for :perks, Perk.new, child_index: 'TEMPLATE_RECORD' do |perk| %>
        <%= render 'perk_fields', form: perk %>
      <% end %>
    </template>

    <%= form.fields_for :perks do |perk| %>
      <%= render 'perk_fields', form: perk %>
    <% end %>

    <div data-target="nested-form.add_perk">
      <%= link_to "Add Perk", "#", data: { action: "nested-form#add_association" }, class: "btn btn-white" %>
    </div>
  </div>
```

* text area auto expand with stimulus plugin

```
yarn add stimulus-textarea-autogrow

// app/javascript/controllers/index.js
import TextareaAutogrow from "stimulus-textarea-autogrow"
application.register("textarea-autogrow", TextareaAutogrow)


// app/views/contancts/index.html.erb
<%= f.text_area :text, required: true, 'data-controller': 'textarea-autogrow' %>

# or better is to override form builder (look at bootstrap how to create custom
# form builder
class MyFormBuilder < BootstrapForm::FormBuilder
  def text_area(name, options = {})
    options.reverse_merge! 'data-controller': 'textarea-autogrow'
    super
  end
end
```
