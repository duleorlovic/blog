---
layout: published_post
---

# HTML Form Input

Start from basic docs for html
[input](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input)
and [forms](https://www.w3.org/TR/html401/interact/forms.html)
https://html.spec.whatwg.org/multipage/forms.html
The most important attribute is
* `name` identify the input in data submitted with the form. If not provided,
  this input will not we included.
* `type` to identify a input type that input element represents
* `value` when specified, it is initial value. For unselected `checkbox` or
  `radio` it is not submitted. They have default value `on`

There is a lot of input types (like `color` which will popup
color picker) but here we will focus on:
* `text` single line text field (line breaks are automatically removed)
* `checkbox` allowing single value to be selected/deselected
* `hidden` it is not displayed, but value is submitted
* `radio` allowing a single value to be selected out of multiple choices
* `submit` ie `<input type='submit'` acts like a button that submits the form.
  You can use `<button>OK</button>` (default is `type='submit'`) instead (button
  that does not submit the form is `<button type='button'>`.


Now start with what Rails provides https://api.rubyonrails.org/v5.2.1/classes/ActionView/Helpers/FormHelper.html#method-i-fields_for






Similar to nested forms, you can have select tag with multiple options (habtm
relation) so we can update without ajax (it is created in one request).
Since in html forms we can only send value or array
* single value (html `name='name'`) In rails `text_field_tag :name` and
  we got `params[:name] # => 'Duke'`
* array (html `name='ids[]'`) In rails `text_field_tag 'ids[]'` or
  `f.text_field :ids, name: 'ids[]'` and we got `params[:ids] #=> [1,2]` Note
  that you need to permit array `params.permit(ids: [])`. When you remove all
  elements from DOM than nothing is send to server, so you need to add empty and
  reject empty values
  ```
  before_save :clear_unchecked_values
  def clear_unchecked_values
    self.custom_sign_up_labels = custom_sign_up_labels.reject(&:blank?)
    true
  end
  ```
* hash is when you nest inside brackets and define specific key, for example
  `name='user[name]`, in rails `f.text_field :name` (when f.object.class ==
  User).  Note that you need to permit each key.

  Hash values can also be string, array or another hash. For array (html
  `name='user[ids][]`) in rails you can use `f.text_field :ids, name:
  'user[ids][]` (or general `"#{f.object.class.name.underscore}[ids][]"` and we
  got `params[:user][:ids] #=> [1,2]`.  Hash with hash values rails use it for
  `fields_for & accepts_nested_attributes_for` html
  `name='user[posts_attributes][0][id]` and we got
  `params[:user][:posts_attributes]["0"][:id]`. Note that you need to permit
  each key `params.require(:user).permit(posts_attributes: [:id,:name])`

So for habtm or has_many through, we need to mark for destruction and than add
params

```
# modal
class Book < ApplicationRecord
  has_many :book_topics
  has_many :topics, through: :book_topics
  accepts_nested_attributes_for :book_topics
end
class BookTopic < ApplicationRecord
  belongs_to :book
  belongs_to :topic
end

# html
      <%= f.select :topic_ids, options_from_collection_for_select(Topic.all, :id, :topic_name, -> (topic) { @book.book_topics.map(&:topic).include? topic }), {}, multiple: true %>

# controller
    @book.book_topics.each &:mark_for_destruction
    book_topics_params = {
      book_topics_attributes: params[:book][:topic_ids].map {|id| { topic_id: id } }
    }
    if @book.update _book_params.merge book_topics_params

```


# Nested forms

<http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html>
If you want to use nested forms like question and answers, good approach is to
`create!` on new action and redirect to edit. That way you have `question_id`.
For new questions or delete questions, you can simply use ajax. So start with
`rails g scaffold questions title;rails g model answers question:references`.

~~~
# models/question.rb
class Question < ActiveRecord::Base
  has_many :answers, dependent: :destroy
  accepts_nested_attributes_for :answers, allow_destroy: true
end

# questions/_form.html.erb
  <div id="answers">
    <h2>Answers</h2>
    <% @question.answers.each do |answer| %>
      <%= render partial: 'answer', locals: { question_form: f, answer: answer } %>
    <% end %>
  </div>
  <%= link_to "Create new answers", create_answer_question_path, remote: true,
  method: :post %>

# questions/_answer.html.erb
<%#
  question_form - we need this because we don't want to generate <form> tags
                - we need just fields
  answer - target answer

  we hard code "answers_attributes[]" because
  when we use fields_for :answer, than when we use ajax `new` twice we got same
  name for different records
  question[answers_attributes][0][id] (value 111)
  question[answers_attributes][0][id] (value 222)
  and only latest will be considered
  it is because uniq number is reset for each fields_for
  this sequential "0", "1" is used so you can show `fields_for :answers` for
  existing and new answers (which does not have id) so they are all separated
  with hard coded `answers_attributes[]` it is
  question[answers_attributes][111][id] (value 111)
  question[answers_attributes][222][id] (value 222)
  but for unsaved objects it will be
  question[answers_attributes][][id] (value nil)
  so there are two solutions:
    * always create objects and than render form
    * add fake id (used for key), but not provide a hidden input field 'id'
%>
<%= question_form.fields_for "answers_attributes[]", answer do |ff| %>
  <div class="field">
    <%= ff.hidden_field :id %>
    <%= ff.text_field :content, placeholder: "Answer" %>
    <%= ff.number_field :score, placeholder: 'Score' %>
    <%= ff.number_field :position, placeholder: 'Position' %>
    <%= link_to "Destroy", destroy_answer_question_path(answer.question, answer_id: answer.id), remote: true, method: :delete %>
  </div>
<% end %>

# questions/create_answer.js.erb
<% output = nil %>
<% form_for(@question) do |f| %>
  <% output = j render partial: 'answer', locals: { question_form: f, answer:
  @answer } %>
  <% end %>
$('#answers').append('<%= output %>');

# config/routes.rb
  resources :questions do
    member do
      post :create_answer
      delete :destroy_answer
    end
  end

# controllers/questions_controller.js
  def create_answer
    @answer = @question.answers.create!
  end

  def destroy_answer
    @answer = @question.answers.find(params[:answer_id])
    @answer.destroy!
  end

    def question_params
      params.require(:question).permit(
        :title, :time_limit,
        answers_attributes: [:id, :score, :content, :_destroy]
      )
    end
~~~

You can try
[cocoon](https://github.com/nathanvda/cocoon) gem and use
`link_to_add_association` [tutorial
post](https://www.sitepoint.com/better-nested-attributes-in-rails-with-the-cocoon-gem)

`has_many :ip_addresses, inverse_of: :subscriber` is needed when you have
validation errors for `accepts_nested_attributes_for`
https://robots.thoughtbot.com/accepts-nested-attributes-for-with-has-many-through
Validation of uniqueness does not work for bulk update with `_attributes` since
it check only what is in db (not in params), so one solution is to add
`inverse_of` and to add validation
```
  # this works when we create one record (not for batch update with
  # ip_address_attributes)
  validates :fix_ip_address, presence: true, uniqueness: { scope: :parent_location_id }
  # so we need to validate that also
  validate :uniqueness_for_batch_update
  def uniqueness_for_batch_update
    ips = subscriber.ip_addresses.map(&:fix_ip_address)
    errors.add(:fix_ip_address, 'already exists') if ips.size != ips.uniq.size
  end
```

# Multiple form submit buttons for different actions

you can use rails builder

  ~~~
  <%= f.submit "Some label" %>
  <%= f.submit "Some other label" %>
  ~~~

  will generate

  ~~~
  <input type="submit" name="commit" value="Some label">
  <input type="submit" name="commit" value="Some other label">
  ~~~

  so you can check on server

  ~~~
  if params[:commit] == "Some label"
  ~~~

  When you are not using `f.submit` but plain `<button>Some label</button>` than
  you need to add `hidden_field_tag :commit, "Some label"`

  Sometimes there is a problem when automatic translator on the page change
  button labels and inputs so commit param is different...
  There are two solutions for that:
  Instead of `commit` you can use
  [formaction](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Input#attr-formaction)
  So you do not need to parse commits but you need different action methods to
  handle.

  ~~~
  <%= form_for('url') do |f| %>
      <%= f.submit 'Create' %>
      <%= f.submit 'Special Action', formaction: special_action_path %>
  <% end %>
  ~~~

  Another solution to translations of input submit is to use buttons with value
  as the same as the text inside button tag.
  ```
  <%= form_for('url') do |f| %>
    <%# instead of <input> we use <button> with value (which is the same as inner text) so automatic page translators do not change that value %>
    <%= f.button 'Create', value: 'Create' %>
    <%= f.button 'Special Action', value: 'Special Action' %>
  <% end %>
  ```
  this will generate
  ```
  <form action="url">
    <button name="button" type="submit" value="Create">Create</button>
    <button name="button" type="submit" value="Special Action">Special Action</button>
  </form>
  ```
  so you can grab `params[:button] == 'Create'`

  If you want to disable utf-8 and authenticity hidden fields, remove hidden
  input `name='commit'` for submit buttons `button_tag 'OK', name: nil` and use
  plain param name instead of in brackets `f.hidden_field :name` generate
  `name='[name]'` (but `f.text_field :name` generate `name='name'`), than use
  `hidden_field_tag :name`

  ~~~
    <%= bootstrap_form_tag url: @atom_payment.atom_server_link, layout: :horizontal, enforce_utf8: false, authenticity_token: false do |f| %>
      <% @atom_payment.atom_params.each do |atom_param| %>
        <%= hidden_field_tag atom_param[:name], atom_param[:value] %>
      <% end %>
      <%= button_tag "Pay Now", name: nil, class: 'btn btn-primary btn-block', 'data-disable-with': 'Processing...' %>
    <% end %>
  ~~~
* if you want to access hash keys by symbol or string you can instantiate with
`params = HashWithIndifferentAccess.new name: 'Duke'` so you can use
`params[:name]` or `params["name"]`.

Here is a rails app, and here is a gist

# Rails and Forms

https://api.rubyonrails.org/v5.2.1/classes/ActionView/Helpers/FormHelper.html

## Input outside of a form

`<input>` can be outside of a `<form>`, all it needs is `form='id_of_a_form'`

## Dialog element

There is native html tag for modals `<dialog>`. When you call
`dialogEl.showModal()` there will be backgrop and autofocus is triggered (if
there is `autofocus` attribute).
https://alligator.io/html/dialog-element/

## Fieldset & Legend

Use `<fieldset>` to group several input fields and set caption on it with
`<legend>`.
When it is disabled, all input fields won't be submitted.
You can set caption also on firgure
```
<figure>
<img src="/wp-content/uploads/flamingo.jpg" alt="flamingo">
<figcaption><i>fig. 1</i> A pink flamingo.</figcaption>
</figure>
```

# Input Attributes

## Autosuggestion

Pure html autoselect suggestions (but not required from list) can be done using
[datalist](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/datalist)
and `<input list='id_of_datalist'>` attribute.

## Autocomplete

Autocomplete can be enabled with specific value https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete
or disabled because off
[security](https://developer.mozilla.org/en-US/docs/Web/Security/Securing_your_site/Turning_off_form_autocompletion)
(even disabled, browser can ask for auto save password, it will populate them)

Firefox has soft refresh which persist input values and disabled attribute on
refresh the page https://stackoverflow.com/questions/5985839/bug-with-firefox-disabled-attribute-of-input-not-resetting-when-refreshing
This can be disabled by hard refresh or autocomplete `off`

## Autofocus

Auto focus input fields on page load (or dialog show). If you need to show focus
on input with existing value than you can use callback `onfocus`
https://stackoverflow.com/questions/511088/use-javascript-to-place-cursor-at-end-of-text-in-text-input-element/2345915#2345915
```
  <%= f.search_field :s, autofocus: true, onfocus: 'this.selectionStart = this.selectionEnd = this.value.length;' %>
```

## Disabled

Disabled inputs do not receive `click` event, and they are not submitted with
the form.
This can be used to preventDefault on click event for other `data-` event
listeners.

## Required

`required` is boolean attribute, when is present, user myst specify a value. On
all except (color, hidden, range, submit, image, reset, button). When it is on
`checkbox` than user have to select it before procceeding.
Required inputs has pseudoclass `:required`.

## Tabindex

`tabindex` should be `0` so it is reachable by sequential keyboard navigation.

## Placeholder

There are three ways of providing more info about form field, but the best way
is using `label` and to avoid placeholders.
* `label` element is outside of a input
* `placeholder` text that is shown when field is empty. So when is not empty, no
  show. Also browsers translators does not work on placeholders since it is
  attribute (not a value or a text object).
* adjacent elements (google sign in use this)
