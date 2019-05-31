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

* `data-controller='hello'` controller that is defined in `hello_controller.js`.
  Each dash corresponds to underscore (or dash) in file name, two dashes
  corresponds to subfolder `users--list-item` -> `users/list_item_controller.js`
* `data-action='click->hello#greet'` event name is `click`, controller `hello`
  method `greet`.
  If element is `<a>`, `<button>`, `<input type="submit">` than you do not need
  to write `click->`.
  `<input>`, `<select>` and `<textarea>` has `change` as default event.
  `<form>` has `submit` default event.
  If you want to `event.preventDefault()` (for example click on link) than pass
  parameter `greet(event) {}`
* `data-target='hello.name'` creates `nameTarget` property in a controller so we
  can use it to access element and set value. We need to declare it also inside
  controller `static targets = [ 'name' ]`. Beside `this.nameTarget` you can
  check if there are more targets like this `this.nameTargets` or if exists at
  all `this.hasNameTarget` (return true or false)
  to access current element getElementById you can use `this.element`
* `data-hello-index='1'` used to pass data to controller which you can get on
  initialize instead of `this.element.getAttribute('data-hello-index'))` you can
  use stimulus shorthand `this.data.get('index')`. Also `this.data.has('index')`
  to check if data existis and `this.data.set('index', 2)` to set data so
  controller do not need to store any data, just use setter and getter to store
  data in DOM.

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
