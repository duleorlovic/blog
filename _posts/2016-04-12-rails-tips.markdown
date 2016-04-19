---
layout: post
title: Rails tips
---

# Nested forms

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
  when we use fields_for :answer, than when we use ajax twice we got twice
  question[answers_attributes][0][]
  question[answers_attributes][0][]
  and only latest will be considered
  probably because uniq number is reset for each fields_for (id is passed with hidden field)
%>
<%= question_form.fields_for "answers_attributes[]", answer do |ff| %>
  <div class="field">
    <%= ff.hidden_field :id %>
    <%= ff.text_field :content, placeholder: "Answer" %>
    <%= ff.number_field :score, placeholder: 'Score' %>
    <%= ff.number_field :position, placeholder: 'Position' %>
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
~~~

# Validations

Some usefull validations

~~~
validates :email, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i }
validates :email, format: { with: User.email_regexp, allow_blank: true }

validates :email, uniqueness: { scope: :user_id }

~~~

# Tips

* [sprockets](https://github.com/rails/sprockets) are using for compiling assets
  (`//= require_tree .`)

* default value for column could be before save so it is callend only on
  updating record. Another solution is `after_initialize :default_values` but
  that is also called when you read object. Another approach is to put in rails
  migration but than you need another migration if you want to change value. So
  my suggestion is:

  ~~~
  before_save :default_values
  private
  def default_values
    self.logo ||= Rails.application.secrets.default_restaurant_logo
  end
  ~~~

* if you want indempotent seed data you should have some identifier for wich you
  can run `where(id: id).first_or_create do...end` (for user
  `first_or_initialize` since we can't create without password, but we don't
  store password). `do ... end` is perfoment only if item is not found.

  ~~~
  # db/seeds.rb
  # deterministic data
  [
    {:name => "Admin/Office"}
  ].each do |doc|
    job_type = JobType.where(doc).first_or_create do
      puts "JobType #{doc[:name]}" 
    end

  # deterministic and random data
  NUMBER_OF_FAKE_USERS = 5
  (
    [
      { email: 'asd@asd.asd', password: 'asdasd', role: User.roles[:manager] },
    ] +
    Array.new(NUMBER_OF_FAKE_USERS).map do
      { email: Faker::Internet.email, password: Faker::Internet.password }
    end
  ).each do |doc|
    User.where(doc.slice(:email)).first_or_initialize do |user|
      user.password = doc[:password]
      user.save!
      puts "User #{user.email}"
    end
  end
  ~~~

* if you want to add or remove reference in migration `rails g migration
  add_user_to_leads user:references` than you can write

  ~~~
  class AddUserToLeads < ActiveRecord::Migration
    def change
      add_reference :leads, :contact, index: true, foreign_key: true
    end
  end
  ~~~

* if you want to add unique index on some columns (to see validation error
  instead of database esception, should also be in rails `validates :email,
  uniqueness: { scope: :user_id }`

  ~~~
  class AddUniqContacts < ActiveRecord::Migration
    def change
      add_index :contacts, [:email, :user_id], unique: true
    end
  end
  ~~~

# Format date

Write datetime in specific format

~~~
# config/initializers/mytime_formats.rb
# puts user.updated_at.to_s :myapp_time
# puts Time.now.to_s :myapp_time
# puts Date.today.to_time :myapp_time # Date object need to be type casted to Time
# puts Time.now.to_date :myapp_date # Time object to Date if we want myapp_date
Time::DATE_FORMATS[:myapp_time] = lambda { |time| time.strftime("%b %e, %Y @ %l:%M %p") }
Date::DATE_FORMATS[:myapp_date] = lambda { |date| date.strftime("%b %e, %Y") }
Date::DATE_FORMATS[:myapp_date_ordinalize] = lambda { |date| date.strftime("#{date.day.ordinalize} %b %Y") }
~~~

# Multiline render js response

Write long string in multiple lines with `%()`, for example:

~~~
format.js do
  render js: %(
    $('##{key}').replaceWith('#{view_context.j view_context.render partial: 'product_table', locals: { products: Product.send( key).all, product_type_string: key.to_s}} ');
    jQuery(".best_in_place").best_in_place();
    )
end
~~~

# Autosize

Textarea should be [autosized](http://www.jacklmoore.com/autosize/) (download
`autosize.js` from `dist/` folder). Put below your textarea for ajax responses
and put in your main.js file on jQuery load.

~~~
<%# app/views/forms/_form.html.erb %>
<%= f.text_area :title %>
<%= javascript_tag "autosize($('textarea'))" if request.xhr? %>

// app/assets/javascripts/main.js.erb
$(function() {
  autosize($('textarea'));
});
~~~

* parse url to get where user come from `URI.parse(request.referrer).host`
