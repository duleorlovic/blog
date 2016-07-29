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
  when we use fields_for :answer, than when we use ajax `new` twice we got two
  question[answers_attributes][0][]
  question[answers_attributes][0][]
  and only latest will be considered
  probably because uniq number is reset for each fields_for
  (here id is passed with hidden field)
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

# Hooks

Default value for column could be in migration but than you need another
migration if you want to change value. If we put on `after_initialize
:default_values` than it is called also when you read object. If you have
`validates :logo` than you can not put on `before_save :default_logo` since
validation will fail before that. We need before validation:

~~~
before_validation :default_values, on: :create

private

def default_values
  self.logo ||= Rails.application.secrets.default_restaurant_logo
end
~~~

# Order

~~~
default_scope { order('created_at DESC') }
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

# Upload file

Since default file button does not look nice we just hide it (android has some
probles when it is hidden, so we just move it), and use label to trigger it and
`on change` we submit the form. Note that we use `id` because we could have more
than one form on a page.

~~~
<%# _suppression_list.html.erb %>
  <div class="pull-right">
    <%= form_tag import_suppression_path(suppression_list), multipart:true, class: 'pull-right' do %>
      <span data-chosen-filename></span> &nbsp;
      <%= label_tag :file, 'Upload CSV', for: "upload-file-#{suppression_list.id}", style: 'display:inline' %>
      <%= file_field_tag :file, id: "upload-file-#{suppression_list.id}", 'data-upload-file': true, style: 'position:absolute; top: -1000px' %>
    <% end %>
  </div>
~~~

~~~
# javascripts/leads.coffee
$(document).on 'change', '[data-upload-file]', (e) ->
  console.log("upload csv")
  e.preventDefault()
  file = this.files[0]
  allowedExtensions = /^(text\/csv)$/
  $(this).closest('form').find("[data-chosen-filename]").text this.value
  if (this.files.length == 0 )
    alert("Please chose one csv file")
    return
  if(! allowedExtensions.test(file.type))
    alert("File type is not allowed")
    return
  $(this).closest('form').submit()
~~~

# Postgres

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

* adding array of any type and hstore is easy, just add default value `[]` and
`''` (breaks for `{}`). You can create hstore extension in migration. If you
need arrays of hstore than you need to go for json or json array.

  ~~~
  class CreatePhones < ActiveRecord::Migration
    def change
      execute "create extension hstore"
      create_table :phones do |t|
        t.string :my_array, array: true, default: []
        t.hstore :my_hash, default: ''
        t.json :my_json, default: []
        t.json :my_array_of_json, array: true, default: []
      end
    end
  end
  ~~~

  You can enable hstore [link](https://gist.github.com/terryjray/3296171) `sudo
  psql -d Scuddle_app_development -U orlovic` and `CREATE EXTENSION hstore;` or
  in one command: `sudo su postgres -c "psql Scuddle_app_test -c 'CREATE
  EXTENSION hstore;'"`. You should do this also for *test* and *development*
  database. If you don't know database name you can use [rails
  runner](http://guides.rubyonrails.org/command_line.html#rails-runner)

  ~~~
  sudo su postgres -c "psql `rails runner 'puts ActiveRecord::Base.configurations["production"]["database"]'` -c 'CREATE EXTENSION hstore;'"
  ~~~

  If you need to run `rake db:migrate:reset` to rerun all migration to check if
db/schema is in sync, than the best way is to alter user to have superuser
privil `sudo su postgres -c "psql -d postgres -c 'ALTER USER orlovic WITH
SUPERUSER;'"` where orlovic is database username (you can see those usernames -
roles using pqadmin visual program)
[link](https://github.com/diogob/activerecord-postgres-hstore/issues/99).

* dump database from production for local inspection, you can download from
heroku dump file and import in database

  ~~~
  export DUMP_FILE=a.dump
  export DATABASE_NAME=$(rails runner 'puts ActiveRecord::Base.configurations["development"]["database"]')
  chmod a+r $DUMP_FILE
  sudo su postgres -c 'pg_restore -d $DATABASE_NAME --clean --no-acl --no-owner -h localhost $DUMP_FILE'
  ~~~

* you should explicitly add timestamps with `t.timestamps`
* upgrade from 9.3 to 9.4
  [link](https://gist.github.com/dideler/60c9ce184198666e5ab4)

  ~~~
  sudo pg_dropcluster 9.4 main --stop
  sudo pg_upgradecluster 9.3 main
  sudo pg_dropcluster 9.3 main
  ~~~

# Mailer

Usefull is to have internal notification

~~~
# config/secrets.yml
development: &default
  default_mailer_sender: <%= ENV["DEFAULT_MAILER_SENDER"] || "support@example.com" %>
  internal_notification_email: <%= ENV["INTERNAL_NOTIFICATION_EMAIL"] || "internal@example.com" %>
~~~

~~~
# mailers/application_mailer.rb

~~~

~~~
ApplicationMailer.internal_notification('SmsService', send: args)
                 .deliver_now
~~~

# Turbolinks

With turbolinks rails acts as single page application. So if you want to do
something on every page and on first load than you need to bind on two events:

~~~
$(document).on('ready page:load', function(){
  console.log("document on ready page:load");
  /* Activating Best In Place */
  jQuery(".best_in_place").best_in_place();
  autosize($('textarea'));
});
~~~

The best approach is unobtrusive javascript

~~~
// Used to toggle class 'active' to selector
// example <button data-toggle-active=".popup"></button>
$(document).on('click','[data-toggle-active]', function(e) {
  $( $(this).data('toggle-active')).toggleClass('active');
  e.preventDefault();
  console.log("click [data-toggle-active]="+$(this).data('toggle-active'));
});
~~~

# Timezones


Change system timezone (which is used by browsers) with `sudo dpkg-reconfigure
tzdata`. Note that `rails c` uses UTC `Time.zone # => "UTC"`, but byebug in
rails s returns default time zone `Time.zone # CEST Europe`. You can use users
timezone with
[browser-timezone-rails](https://github.com/kbaum/browser-timezone-rails).
`Time.zone.now.utc_offset` will return offset to UTC in seconds. I do not know
why `Time.zone.utc_offset` returns different results (3600 instead of 7200).

# Seeds

If you want indempotent seeds data you should have some identifier (for example
`id`) for wich you can run `where(id: id).first_or_create! do...end`.
Devise user should use `first_or_initialize` since we can't create without
password, but we don't have password field. `do ... end` block is used only if
that object is not found.
`slice` is used for uniq fields (not generated by faker), but all other fields
should be populated in block.

Use `faker` gem to generate example strings:

* `Faker::Internet.email`
* `Faker::Name.name` `Faker::PhoneNumber.cell_phone` `Faker::Avatar.image("my-own-slug", "50x50")`
* `Faker::Company.name` `Faker::Company.logo` **real logo**
* `Faker::Address.street_address` `Faker::Address.city`
* `Faker::Lorem.sentence` `Faker::ChuckNorris.fact`

~~~
# db/seeds.rb
# deterministic data
[
  {:name => "Admin/Office"}
].each do |doc|
  job_type = JobType.where(doc).first_or_create! do
    # you do not need to call save! here
    puts "JobType #{doc[:name]}" 
  end

# deterministic and random data
NUMBER_OF_FAKE_USERS = 5
(
  [
    { email: 'asd@asd.asd', password: 'asdasd', role: User.roles[:manager],
    remote_avatar_url: 'https://placehold.it/350x150' },
  ] +
  Array.new(NUMBER_OF_FAKE_USERS).map do
    { email: Faker::Internet.email, password: Faker::Internet.password }
  end
).each do |doc|
  User.where(doc.except(:password, :remote_avatar_url)).first_or_create! do |user|
    user.password = doc[:password]
    user.remote_avatar_url = doc[:remote_avatar_url]
    user.confirm! # do not skip_confirmation!, we need to have confirmed so 
    # we can log in as this user from admin panel. sign_in will fail for
    # autogenerated users who did not confirm or skip confirmation
    # note that admin can NOT sign in as any not confirmed user
    puts "User #{user.email}"
  end
end

asd_user = User.find_by! email: 'asd@asd.asd'
~~~

# Custom logger

You can add tags (methods for request object), for example:

~~~
# config/application.rb
config.log_tags = [:remote_ip]
~~~

But if you want custom logger, than you can override existing

~~~
# config/initializers/logger_formatter.rb
class ActiveSupport::Logger::SimpleFormatter
  # from activesupport/lib/active_support/core_ext/logger.rb
  def call(severity, time, progname, msg)
    "#{severity_color severity} #{String === msg ? msg : msg.inspect}\n"
  end

private

  def severity_color(severity)
    case severity
    when "DEBUG"
      "\033[0;34;40m[DEBUG]\033[0m" # blue
    when "INFO"
      "\033[1;37;40m[INFO]\033[0m" # bold white
    when "WARN"
      "\033[1;33;40m[WARNING]\033[0m" # bold yellow
    when "ERROR"
      "\033[1;31;40m[ERROR]\033[0m" # bold red
    when "FATAL"
      "\033[7;31;40m[FATAL]\033[0m" # bold black, red bg
    else
      "[#{severity}]" # none
    end
  end
end
~~~

I need to put backtrace in reverse order, so I wrap that code (for example whole
seeds.rb).

~~~
begin
# code
rescue Exception => e
  puts e.backtrace.reverse
  puts e.class, e.message
end
~~~

I do not know how to catch all exceptions without wrapping (maybe to use rack
app as the exception_notification does).  I will try to create new logger, it
is used for whole Rails application.
<http://stackoverflow.com/questions/6407141/how-can-i-have-ruby-logger-log-output-to-stdout-as-well-as-file>

http://stackoverflow.com/questions/6407141/how-can-i-have-ruby-logger-log-output-to-stdout-as-well-as-file

# Mustache

[mustache](https://github.com/mustache/mustache) is nice to render user
templates

Usage is simple as 

~~~
data_for_body == current_user.contacts.first
@template_body = TemplateRenderService.new(@template.body, data_for_body).render
# in view use <%= simple_format @template_body %> to convert \n to <br>
~~~

It is usefull to delegate fields in contacts model to some belongs_to
association, like `delegate User::FIELD_NAMES, to: :user`.


~~~
# app/services/template_render_service.rb
class TemplateRenderService
  attr_reader :template_body, :data_for_body

  def initialize template_body, data_for_body
    @template_body = template_body
    @data_for_body = data_for_body
  end

  def render
    m = Mustache.new
    m.raise_on_context_miss = true
    m.render(template_body, data_for_body)
  rescue Mustache::ContextMiss, Mustache::Parser::SyntaxError => e
    e.message
  end
end
~~~

# Rake tasks

You can write tasks with arguments [rails
rake](http://guides.rubyonrails.org/command_line.html#custom-rake-tasks)
Instead of `:env` or `:environment` you can use other tasks on which it depents.

~~~
# lib/tasks/update_subdomain.rake
namespace :update_subdomain do
  desc "update subdomains for my-user. default value is 'my_subdomain'"
  task :my_user, [:subdomain] => :environment do |t, args|
    args.with_defaults subdomain: 'my_subdomain'
    user = User.find_by name: 'my-user'
    fail "Can't find user 'my-user'" unless user
    user.subdomain = args
    user.save!
    puts "Updated #{user.subdomain}"
  end
end

# run with
rake update_subdomain:my_user
rake update_subdomain:my_user[]
rake update_subdomain:my_user[new_subdomain]
~~~

# Share cookies on multiple subdomains

If you use [sharing-cookies-across-subdomains-with-rails-3](http://makandracards.com/makandra/31381-sharing-cookies-across-subdomains-with-rails-3) or [what-does-rails-3-session-store-domain-all-really-do](http://stackoverflow.com/questions/4060333/what-does-rails-3-session-store-domain-all-really-do)
you need to know that some domains like `herokuapp.com` belongs to [Public
Suffix List](https://devcenter.heroku.com/articles/cookies-and-herokuapp-com) so
browser will prevent setting cookie for them (because someone can use
attacker.herokuapp.com to read cookies from myapp.herokuapp.com).

# Text syntax on buttons and messages

* labels on buttons, titles,...
  * should be upper case with no period: `This Job Is Paused`
* sentences on error messages, flash messages...
  * should have period at the end: `Job has been copied.`

# 7 patterns

From
[7-ways-to-decompose-fat-activerecord-models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
suggestions for code organization I use mostly:

* service objects

  ~~~
  # app/services/match_leads.rb
  def MatchLeads
    def initialize(leads)
      @leads = leads
    end

    def perform
    end
  end
  ~~~

* form objects (query objects) for multiple in multiple out data

  ~~~
  class Search
    include ActiveModel::Validations
    include ActiveModel::Conversion
    extend ActiveModel::Naming

    # input
    attr_accessor :address, :food, :pure_vegetarian, :cuisines

    # output
    attr_accessor :target_location, :users, :menu_items, :tag_counts

    validates :address, presence: true

    def initialize(h)
      @address = h.try :[], :address
      @food = h[:food]
      @pure_vegetarian = h[:pure_vegetarian]
      @cuisines = h[:cuisines] || []
    end

    def perform
      filter_users_by_address
      filter_by_pure_vegetarian if pure_vegetarian.present?
      filter_by_any_cuisine if cuisines.present?
      filter_by_food if food.present?
      generate_tag_counts
    end

    private

    DISTANCE_IN_MILES = 10 # 50
    def filter_users_by_address
      self.target_location = Geocoder.search(address).first
      if target_location.present?
        self.users = User.active.near(
          [target_location.latitude, target_location.longitude],
          DISTANCE_IN_MILES
        )
      else
        self.users = User.active.near(address, DISTANCE_IN_MILES)
      end
    end

    def filter_by_pure_vegetarian
      self.users = users.where(pure_vegetarian: true)
    end

    def filter_by_any_cuisine
      # we use filter
      # http://stackoverflow.com/questions/17496629/fuzzy-tag-matching-with-acts-as-taggable
      self.users = users.tagged_with(cuisines, any: true)
      # http://stackoverflow.com/questions/4665979/how-to-find-number-of-tag-matches-in-acts-as-taggable-on
      # @users = @users.tagged_with(params[:cuisines], any: true).all.to_a
      # @users.sort_by! { |o| -(params[:cuisines] & o.cuisine_list).length }
    end

    def filter_by_food
      # TODO: this does not return relation
      # User need to have at least one MenuItem
      search_param = "%#{food}%"
      # we need those @menu_items results
      @menu_items = MenuItem.joins(:user)
                    .where(user: users.map(&:id))
                    .where("users.business_name ILIKE ? OR "\
                             "menu_items.name ILIKE ? " \
                             " OR menu_items.description ILIKE ?",
                           search_param, search_param, search_param)
                    .limit(5)
      self.users = users.where(id: menu_items.map(&:user).uniq)
    end

    def generate_tag_counts
      # https://github.com/mbleigh/acts-as-taggable-on/blob/master/lib/acts_as_taggable_on/taggable/collection.rb#L90
      # since we need relation and filter_by_food does not return relation
      # and there is PG::SyntaxError: ERROR:  subquery has too many columns
      # we will load users and use their ids
      users_for_tags = User.where(id: users.map(&:id))
      self.tag_counts = users_for_tags.tag_counts_on :cuisines
    end
  end
  ~~~

# Tips

* [sprockets](https://github.com/rails/sprockets) are using for compiling assets
  (`//= require_tree .`)
* parse url to get where user come from `URI.parse(request.referrer).host`
* prefer using `@post.destroy` instead of `@post.delete` because it is propagate
* `spring stop` in many cases:
  * when you export some ENV and use tham in `config/secrets.yml` but can't see in `rails c`
* render
  [json](http://api.rubyonrails.org/classes/ActiveModel/Serializers/JSON.html)
  can be customized with `render json: @phones.as_json(only: [:id], expect:
  [:created_at], include: :posts)`
* run rails in production mode:

  ~~~
  RAILS_ENV=production rake db:create
  rake assets:precompile RAILS_ENV=production
  export RAILS_SERVE_STATIC_FILES=true
  # export DO_NOT_USE_CANONICAL_HOST=true
  # export JAVASCRIPT_ERROR_RECIPIENTS=duleorlovic@gmx.com
  # remove eventual config.action_controller.asset_host = asdasd
  rails s -b 0.0.0.0 -e production
  ~~~

* load databe from `db/schema.rb` (instead of `rake db:migrate`) is with `rake
  db:schema:load`.
* run at port 80 `sudo service apache2 stop` and `rvmsudo rails s -p 80`. Note
  that db user will to *root* instead of *orlovic* so you need to hardcode it in
  *config/database.yml*. Rember to `-b 0.0.0.0` if you access from outside.
  When you are using `local.trk.in.rs` you can set local ip *192.168.0.4* as
  Redirection (301 or 302) (but not as Forwarding since that does not work for
  some api requests from android).
* `Rails.logger` is stdout and every controller, model and view has `logger`
  method which you can use like `logger.debug my_var`
* if you want to join some fields with `,` but do not know if they all exists,
  you can `[phone1, phone2].keep_if(&:present?).join(', ')`
* use include for n+1 query and joinswhen you don't need associated models. Both
  are using INNER JOIN
  * `Comment.all(include: :user, conditions: { users: { admin: true}})` will
    load also the user model
  * `User.all(joins: :comments, select: "users.*, count(comments.id) as
    comments_count", group: "users.id")` you can output just specific value
* `some_2d_array.each_with_index do |(col1, col2),index|` when you need
  [decomposition
  array](http://docs.ruby-lang.org/en/2.1.0/syntax/assignment_rdoc.html#label-Array+Decomposition)
  to some variables, you can use parenthesis.
* fake objects could be generated with `OpenStruct.new name: 'Dule'`
* long output of a command (for example segmentation fault) can be catched with
  `rails s 2>&1 | less -R`
* use different layout and template based on different params is easy using
  locales.

    layout proc { |controller| controller.new_layout? ? 'adminlte' : 'application' }
    before_filter :set_locale_for_layout
    def set_locale_for_layout
      if session[:layout] == true
        if params[:layout] == "false"
          # disable new layout
          session[:layout] = false
          I18n.locale = 'en'
          logger.debug "layout became false"
        else
          logger.debug "layout still true"
          I18n.locale = 'in'
        end
      else
        # session[:layout] == false
        if params[:layout] == "true"
          # enable new layout
          session[:layout] = true
          logger.debug "layout became true"
          I18n.locale = 'in'
        else
          I18n.locale = 'en'
          logger.debug "layout still false"
        end
      end
    end

    def new_layout?
      I18n.locale == :in
    end

  so if you add param `?layout=true` it will render `index.in.html.erb` variant
  using new layout and put that in session (so all form submittions also use new
  layout). You can disable new_layout with `?layout=false`. If you have locale
  files (probably `devise.en.yml`) than you need to copy to `:in`

  ~~~
  # config/locales/devise.en.yml
  en: &default
  #  here is original translations

  # copy all translations for new locale
  in:
    <<: *default
  ~~~

* title and meta tags for the template can be added using helper functions

  ~~~
  # app/views/pages/index.html
  <% page_title 'My Page' %>

  # app/helpers/application_helper.rb
    def page_title(title)
      content_for(:page_title) { title }
    end
  ~~~

* `inverse_of` is needed when you have validation errors for
  `accepts_nested_attributes_for`

* for clone with associations you can use dup or clone but easiest way to with
  new (build is deprecated) json except. If you use method `as_json.exept "id"`
  than pass string arguments (or splat array `(*%(id))`), if it is param
  `as_json except: [:id]` than you can use symbols. Please note that `as_json`
  returns has with string keys so if you merge something you should use string
  `chat.as_json(except: [:id]).merge("sport_id" => view.sport.id)`

  ~~~
  # http://stackoverflow.com/questions/5976684/cloning-a-record-in-rails-is-it-possible-to-clone-associations-and-deep-copy
  def clone_with_associations
    # except bookmarks since we do not need them
    # copy all fields and belongs_to associations
    new_job = self.user.jobs.new self.as_json except: * %(id created_at first_published_at)
    new_job.job_title = "(Copy) " + new_job.job_title
    # if status is active put paused
    new_job.status = :paused if new_job.active?
    # copy location, ie create new one
    if self.location
      new_job.location_name = self.location.address
    end
    # cloning images
    # todo with fog
    # cloning questions with answers
    self.questions.each do |question|
      new_question = new_job.questions.new question.as_json except: * %(id created_at)
      question.answers.each do |answer|
        new_answer = new_question.answers.new answer.as_json except: * %(id created_at)
      end
    end
    new_job
  end

  # job is not saved so you need to call
  # new_job = @job.clone_with_associations
  # new_job.save!
  ~~~

* if you put `byebug` inside `User.first_or_initialize` than you will see empty
  for `User.all`
* dump and restore postgresql database

  ~~~
  pg_dump `rails runner 'puts  ActiveRecord::Base.configurations["development"]["database"]'` > dump
  # pg_dump PlayCityServer_development > dump
  ~~~

  ~~~
  rake db:drop
  rake db:create
  psql `rails runner 'puts  ActiveRecord::Base.configurations["development"]["database"]'` < dump
  ~~~


* for `gon` gem you should `include_gon` before other javascript files. Note
  that `if gon` will still raise error if `gon` is not called in controller (so
  `gon` is undefined)
