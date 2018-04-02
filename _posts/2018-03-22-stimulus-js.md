---
layout: post
---

# Install

~~~
git clone https://github.com/stimulusjs/stimulus-starter.git
cd stimulus-starter
yarn install
yarn start
gnome-open http://localhost:9000/
~~~

# Magic keywords

* `data-controller='hello'` controller that is defined in `hello_controller.js`.
  Each dash corresponts to underscore (or dash) in file name, two dashes
  corresponds to subfolder `users--list-item` -> `users/list_item_controller.js`
* `data-action='click->hello#greet'` event name is `click`, controller `hello`
  method `greet`. If element is `<a>`, `<button>`, `<input type="submit">` than
  you do not need to write `click`. `<input>`, `<select>` and `<textarea>` has
  `change` as default event. `<form>` has `submit` default event. If you want to
  `event.preventDefault()` (for example click on link) than pass parameter
  `greet(event) {}`
* `data-target='hello.name'` creates `nameTarget` property in a controller so we
  can use it to set value. We need to declare it also inside controller `static
  targets = [ 'name' ]`. Beside `this.nameTarget` you can check if there are
  more targets like this `this.nameTargets` or if exists at all
  `this.hasNameTarget` (return true or false)
* `data-hello-index='1'` used to pass data to controller which you can get on
  initialize instead of `this.element.getAttribute('data-hello-index'))` you can
  use stimulus shorthand `this.data.get('index')`. Also `this.data.has('index')`
  to check if data existis and `this.data.set('index', 2)` to set data so
  controller do not need to store any data, just use setter and getter to store
  data in DOM.

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
