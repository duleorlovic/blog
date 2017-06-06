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

You can try
[cocoon](https://github.com/nathanvda/cocoon) gem and use
`link_to_add_association` [tutorial
post](https://www.sitepoint.com/better-nested-attributes-in-rails-with-the-cocoon-gem)

# Validations

Some usefull validations

~~~
validates :email, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i }
validates :email, format: { with: User.email_regexp, allow_blank: true }
validates :email, uniqueness: { scope: :user_id }
~~~

When validating associations, always is `_id` since we use that in select input
inside form (don't validate object `validates :job_type, presence: true` since
error will not be shown on `:job_type_id`).

~~~
validates :job_type_id, presence: true

<%= f.select :job_type_id, ... %>
~~~

[form
select](https://apidock.com/rails/ActionView/Helpers/FormOptionsHelper/select)
can accept a lot of options as 3th param:

* `select: value` if you need different than `job.job_type_id`
* `disabled: [values]` to disable some options
* `label: 'My label'`
* `prompt: 'Please select'` this is shown only if not already have some value
* `include_blank: 'Please select'` this is shown always (even already have
value)


# Hooks

Default value for column could be in migration but than you need another
migration if you want to change value. If we put on `after_initialize
:default_values_on_initialize` than it is called also when you read object (that
is required if you want to change default values for existing objects) If you
have `validates :logo` than you can not put on `before_save :default_logo` since
validation will fail before that. We need `before_validation` which occurs only
on update and create.

Note that business logic could be that some fields could be cleared. For example
some logo is default value, but someone could completely remove logo.
So you need to separate `default_values_on_create` and
`default_values_on_updarte` (which could get `nil` on update some fields)

~~~
after_initialize :default_values_on_initialize
before_validation :default_values_on_create, on: :create
before_validation :default_values_on_updarte, on: :update

private

def default_values_on_initialize
  self.some_no_nil_number ||= 1
  self.some_positive_number = 1 unless some_positive_number.to_i > 0
  self.some_false_value ||= false
  # do not use self.some_true_value ||= true since that will override if
  # some_true_value = false
  self.some_true_value = true if some_true_value.nil?
  # http://guides.rubyonrails.org/active_record_callbacks.html#halting-execution
  # return value should be true or nil
  true
end

def default_values_on_create
  self.logo ||= Rails.application.secrets.default_restaurant_logo
  # return value should be true or nil
  # http://guides.rubyonrails.org/active_record_callbacks.html#halting-execution
  true
end
~~~

# Default Order

~~~
default_scope { order('created_at DESC') }
~~~

Usually default scope is not good practice (except for order). Even to trashable
concern is bad usage, for example if you use trashed on user and comments model

~~~
rails g migration add_trashed_to_users trashed:boolean
rails g migration add_trashed_to_comments trashed:boolean

# user.rb & comment.rb
  default_scope { where(trashed: nil) }
  scope :trashed, -> { unscoped.where(trashed: true) }
~~~

than query `User.first.comments.trashed` will return all trashed comments from
ALL users! `unscoped` will remove even association scopes, thanks Joseph
Ndungu on
[comment](http://www.victorareba.com/tutorials/creating-a-trashing-concern-in-rails-5).

Uncope can receive params what to unscope, example `User.unscope(:where)`

# Format date

Write datetime in specific my_time format

~~~
# config/initializers/mytime_formats.rb
# puts user.updated_at.to_s :myapp_time
# puts Time.now.to_s :myapp_time
# puts Date.today.to_time :myapp_time # Date object need to be type casted to Time
# puts Time.now.to_date :myapp_date # Time object to Date if we want myapp_date
Time::DATE_FORMATS[:at_time] = lambda { |time| time.strftime("%b %e, %Y @ %l:%M %p") }
Date::DATE_FORMATS[:myapp_date] = lambda { |date| date.strftime("%b %e, %Y") }
Date::DATE_FORMATS[:myapp_date_ordinalize] = lambda { |date| date.strftime("#{date.day.ordinalize} %b %Y") }
~~~

`Time.now` and `Date.today` are using system time. If you are using
`browser-timezone-rails` than timezone will be set for each request. But if you
need something from rails console, you need to set timezone manually.
`Time.zone = 'Belgrade'` (list all in `rake time:zones:all`). It is good to
always use `Time.zone.now` and `Time.zone.today`. Rails helpres use zone
(`1.day.from_now`)

* to get weekday from `ActiveSupport::TimeWithZone` use
  [link](http://api.rubyonrails.org/classes/ActiveSupport/TimeWithZone.html)
  `weekday = t.to_a[6]`
* to get first calendar date `Time.zone.today.beginning_of_month` or
`Time.zone.today.end_of_month`

## Timezones

Change system timezone (which is used by browsers) with `sudo dpkg-reconfigure
tzdata`. Note that `rails c` uses UTC `Time.zone # => "UTC"`, but byebug in
rails s returns default time zone `Time.zone # CEST Europe`. You can use users
timezone with
[browser-timezone-rails](https://github.com/kbaum/browser-timezone-rails).
`Time.zone.now.utc_offset` will return offset to UTC in seconds. I do not know
why `Time.zone.utc_offset` returns different results (3600 instead of 7200).


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

# Database

## Active record

* union is not supported [issue 929](https://github.com/rails/rails/issues/939)


## Add json and hstore

Note that you can use simple text column (without default value) and add
`serialize :preferences, Hash` in your model, so `user.prefereces # => {}` is
defined for initial empty value.

Adding array of any type and hstore is easy, just add default value `[]` and
`''` (breaks for `{}`). You can create hstore extension in migration. If you
need arrays of hstore than you need to go for json or jsonb (better optimization
only pq 9.4) or json/jsonb array.

~~~
class CreatePhones < ActiveRecord::Migration
  def change
    execute "create extension hstore"
    create_table :phones do |t|
      t.string :my_array, array: true, default: []
      t.hstore :my_hash, default: ''
      t.jsonb :my_json, default: []
      t.jsonb :my_array_of_json, array: true, default: []
    end
  end
end
~~~

DB user need to be superuser to be able to create extension. If you need to
run `rake db:migrate:reset` to rerun all migration to check if db/schema is in
sync, than the best way is to alter user to have superuser privil `sudo su
postgres -c "psql -d postgres -c 'ALTER USER orlovic WITH SUPERUSER;'"` where
orlovic is database username (you can see those usernames - roles using
pqadmin visual program)
[link](https://github.com/diogob/activerecord-postgres-hstore/issues/99).

You can enable hstore manually
[link](https://gist.github.com/terryjray/3296171) `sudo psql -d
Scuddle_app_development -U orlovic` and `CREATE EXTENSION hstore;` or in one
command: `sudo su postgres -c "psql Scuddle_app_test -c 'CREATE EXTENSION
hstore;'"`. You should do this also for *test* and *development* database. If
you don't know database name you can use [rails
runner](http://guides.rubyonrails.org/command_line.html#rails-runner)

~~~
sudo su postgres -c "psql `rails runner 'puts ActiveRecord::Base.configurations["production"]["database"]'` -c 'CREATE EXTENSION hstore;'"
~~~

## Dump database

Dump database from production for local inspection, you can download from
heroku to dump file `tmp/b001.dump` and import in database

~~~
export DUMP_FILE=tmp/b001.dump
export DATABASE_NAME=$(rails runner 'puts ActiveRecord::Base.configurations["development"]["database"]')
chmod a+r $DUMP_FILE

rake db:drop db:create

sudo su postgres -c "pg_restore -d $DATABASE_NAME --clean --no-acl --no-owner -h localhost $DUMP_FILE"
~~~

Or you can use [my
helper](https://github.com/duleorlovic/config/blob/master/bashrc/rails.sh#L39)
`load_dump`

You can dump local database with

~~~
pg_dump `rails runner 'puts  ActiveRecord::Base.configurations["development"]["database"]'` > dump
# pg_dump PlayCityServer_development > dump
~~~

Restore from local dump

~~~
rake db:drop
rake db:create
psql `rails runner 'puts  ActiveRecord::Base.configurations["development"]["database"]'` < dump
~~~

To restore on heroku you need to push the file somewhere on internet, for
example AWS S3 and than run in console

~~~
heroku pg:backups restore --confirm playcityapi https://s3.amazonaws.com/duleorlovic-test-us-east-1/b001.dump DATABASE_URL
~~~

## Postgres tips

* you should explicitly add timestamps with `t.timestamps`
* upgrade postgres from 9.3 to 9.4
  [link](https://gist.github.com/dideler/60c9ce184198666e5ab4)

  ~~~
  wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
  sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main" >> /etc/apt/sources.list.d/postgresql.list'

  # Also probably optional but I like to update sources and upgrade
  sudo apt-get update
  sudo apt-get upgrade
  sudo apt-get install postgresql-9.4
  sudo pg_dropcluster 9.4 main --stop
  sudo pg_upgradecluster 9.3 main
  sudo pg_dropcluster 9.3 main
  ~~~

* To disable sql log in rails logger put this in initializer file:

  ~~~
  # config/initializers/silent_sql_log.rb
  ActiveRecord::Base.logger.level = Logger::INFO
  ~~~

* to allow any user to log in to database, ie any user in config/database.yml
  even without password, you need to change postgres config:

  ~~~
  sudo vi /etc/postgresql/9.1/main/pg_hba.conf
  # change all this words [md5, ident, peer] to trust
  sudo /etc/init.d/postgresql restart
  ~~~

  create one user:

  ~~~
  sudo su - postgres
  psql postgres
  CREATE ROLE "orlovic" WITH CREATEDB LOGIN;
  \q

  # you can remove with DROP ROLE orlovic;
  # or you can change anything on role
  # https://www.postgresql.org/docs/9.3/static/sql-alterrole.html
  # if you need uppercased (it was created `orlovic` instead of `Orlovic`)
  ALTER ROLE orlovic RENAME TO "Orlovic";
  # or
  pgadmin3 # login as postgres without password and chage it to Orlovic

  # create database is with linux command
  createdb # this will create db with same name as current user
  createdb orlovic
  ~~~

* you can use production database locally with url from heroku config env
  variable `HEROKU_POSTGRESQL_CRIMSON_URL:
  postgres://flvmstxfdk:3fo9O62B-Q4BZ7EP8N4YU@ec2-107-20-191-205.compute-1.amazonaws.com:5432/d5ttgvqpjs1ftb`
  and put it inside `config/database.yml` under `development` under `url:
  postgres://asd.asd@asd.com:123/asd` item
* you can use [rails counter
  cache](http://guides.rubyonrails.org/association_basics.html#options-for-belongs-to-counter-cache)
  with `belongs_to :author, counter_cache: true` but than you need to have
  `authors.books_count` column. It is easier to have [sql for counter
  cache](https://medium.com/@eric.programmer/the-sql-alternative-to-counter-caches-59e2098b7d7#.cmuuhtayh)

  ~~~
  # app/models/author.rb
  class Author < ApplicationRecord
    scope :with_counts, -> {
      select <<~SQL
        authors.*,
        (
          SELECT COUNT(books.id) FROM books
          WHERE author_id = authors.id
        ) AS books_count
      SQL
    }
  end

  # app/controllers/authors_controller.rb

  @authors = Author.with_counts.all
  @author = Author.with_counts.find params[:id]

  # app/views/authors/index.html.erb

  <% @authors.each do |author| %>
    <% autor.books_count %>
  <% end %>
  <%= @author.books_count %>
  ~~~


## Migrations:

* `rails g model user email_address` will generate migration and model. It is
good to edit `last_migration` and add `null: false` to not null fields
(particularly for foreign keys) and
`add_index :users, :email_address, unique: true`
* if we call `Products.update_all fuzz: 'fuzzy'` in migration, it will
  probably break in the future, because *Products* will be validated for
  something that we did not know on that time. Better is to create local class
  and call reset column information:

  ~~~
  # db/migrate/20161010121212_update_fuzz.rb
  class UpdateFuzz < ACtiveRecord::Migration
    # this is local Product class used only inside this migration
    class Product < ActiveRecord::Base;
    end
    # Product.reset_column_information not sure if we need this
    Product.update_all fuzz: 'fuzzy'
  end
  ~~~

* to change type from string to integer (using cast)

  ~~~
  class ChangeScoreTypeInFilledAnswers < ActiveRecord::Migration 
    def up
      change_column :filled_answers, :score, 'integer USING CAST(score AS integer)'
    end
    def down
      change_column :filled_answers, :score, :string
    end
  end
  ~~~

* use *db/seed.rb* to add some working data (users, products) that should not go
  to production. add data to migration file if something needs to be in db
  (select box, customer plans)
* when you restora database `rake db:migrate:status` does not know that any
  migration was perfomed. You can manually perform specific migration `rake
  db:migrate:up VERSION=20161114162031`
* `rake db:migrate` will also invoke `db:schema:dump` task
  [link](http://edgeguides.rubyonrails.org/active_record_migrations.html#running-migrations)


## Has_many through

`has_many :roles; has_many :projects, through: roles` can be used for many to
many associations. You can automatically add new role, no need to `user.save`.

~~~
user.roles.create project: project
# or
user.projects << project
~~~

[has_and_belongs_to_many](http://guides.rubyonrails.org/association_basics.html#the-has-and-belongs-to-many-association)
(habtm) can be generated with `rails g migration create_campaigns_templates
campaign:references template:references` and add `id: false` as
[suggested](http://guides.rubyonrails.org/association_basics.html#updating-the-schema)
Note that both model names are plural.

~~~
class CreateCampaignsTemplates < ActiveRecord::Migration
  def change
    create_table :campaigns_templates, id: false do |t|
      t.references :campaign, foreign_key: true, null: false
      t.references :template, foreign_key: true, null: false
    end
  end
end
~~~

and adding `has_and_belongs_to_many :templates` in those classes.

If you have different name of the table: `t.references :donor, foreign_key:
true, null: false` but donor is actually in users table, than you need to change
foreign key

~~~
# to_table is in rails 5
t.references :donor, foreign_key: { to_table: :users }, null: false
~~~

or manually write relation

~~~
create_table :donations
  t.belongs_to :donor, index: true, null: false
end
add_foregn_key :donations, :users, column: :donor_id
~~~

Also in model you need to write: `has_many :donations, foreign_key: :donor_id`
and `belongs_to :donor, class_name: "User"`

Note that `t.belongs_to` by default use `index: false`, so you need to use
`index: true`. But usually you use `add_foreign_key` that will also add index
(name will be like: `fk_rails_123123`) if it is not added by belongs_to, so you
can safelly use `t.belongs_to ... index: false` if you are using
`add_foregn_key`.

For MySql I have to use `unsigned: true` in `t.belongs_to :user, null: false,
unsigned: true`

If you need to update specific fields of association, add validation or cdd
ustom order then you should create the model and use `has_many :templates,
through: campaign_templates, order: 'campaign_templates.created_at ASC'`
Note that here we user singular first part so model name looks better

~~~
class CampaignTemplate < ActiveRecord::Base
  belongs_to :campaign
  belongs_to :template
end
~~~

If you accidentaly use `belongs_to :templates` (pluralized) than error is
`undefined method relation_delegate_class' for Templates:Module`

## Database indexes

Indexes should be on all columns that are references in `WHERE, HAVING, ORDER
BY` parts of sql.
For example if you find using specific column `User.find_by column: 'val'`.
Also we can add index to `:updated_at` column since we sometimes order by
that column.

All foreign keys need to have index.
You can use `add_reference` (adding column and index). if you want to add or
remove reference in migration `rails g migration add_user_to_leads
user:references` which will generate

~~~
class AddUserToLeads < ActiveRecord::Migration
  def change
    # reference will automatically include index on that column
    add_reference :leads, :contact, foreign_key: true
  end
end
~~~

For polymorphic associations `owner_id` and `owner_type`

~~~
class Organization < ActiveRecord::Base
  has_many :projects, :as => :owner
end

class User < ActiveRecord::Base
  has_many :projects, :as => :owner
end

class Project < ActiveRecord::Base
  belongs_to :owner, :polymorphic => true
end
~~~

You need to add double index:

~~~
# Bad: This will not improve the lookup speed
add_index :projects, :owner_id
add_index :projects, :owner_type

# Good: This will create the proper index
add_index :projects, [:owner_type, :owner_id]
~~~

You can add unique index on some existing columns (to see
validation error instead of database esception, should also be in rails
`validates :email, uniqueness: { scope: :user_id }`)

~~~
class AddUniqContacts < ActiveRecord::Migration
  def change
    add_index :contacts, [:email, :user_id], unique: true
  end
end
~~~

Do not add index for tables that has a lot or removing, since perfomance will be
bad. Also huge tables need huge indexes, so pay attention on size.

In mysql sometimes `LIMIT 10` is slower than `LIMIT 100` since it won't use
index for small stuff, but if table is huge than it's much slower without index.
To force index use something like `User.from("'users' FORCE INDEX
(my_user_index")`
[link](http://fuzzyblog.io/blog/rails/2017/02/24/understanding-low-level-index-issues-in-mysql.html)
If you receive error `undefined method map for "'users' FORCE INDEX
(my_user_index)":Arel::Nodes::SqlLiteral` than you can try to replace
`includes/references` with `joins`.


# MySql

* if you have
  `/home/orlovic/.rvm/gems/ruby-2.2.4/gems/mysql2-0.4.4/lib/mysql2.rb:31:in
  require: libmysqlclient.so.18: cannot open shared object file: No such file
  or directory -
  /home/orlovic/.rvm/gems/ruby-2.2.4/gems/mysql2-0.4.4/lib/mysql2/mysql2.so
  (LoadError)` than try `gem uninstall mysql2;gem install mysql2`
* create mysql user

  ~~~
  mysql -u root -p
  CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'asdf';
  ~~~

* restore mysql sql dump

  ~~~
  mysql -u myuser -pMYPASSWORD my_database_name < tmp/web001db.sql
  ~~~

* create database with specific character set

  ~~~
  CREATE DATABASE my_app DEFAULT CHARACTER SET 'ascii';
  ~~~

  or in `config/database.yml` with

  ~~~
  development:
    adapter: mysql2
    encoding: ascii
  ~~~

On Heroku it is better to use *JawsDB MySQL* than *ClearDB MySQL* since it has
more MB in for free usage.

* to chech if mysql database exists use mysqlshow

  ~~~
  exists = capture("echo $(mysqlshow --user=#{env.db_user} --password=#{env.db_pass} #{env.db_name} | grep -V Wildcard | grep -o #{env.db_name})")
  if exists.strip.size == 0
    # does not exists
  ~~~

# Turbolinks

With turbolinks rails acts as single page application. So if you want to do
something on every page and on first load than you need to bind on two events
`ready page:load`:

~~~
// app/assets/javascripts/ready_page_load.coffee
$(document).on('ready page:load', ->
  console.log "document on ready page:load"
  # Activating Best In Place 
  jQuery(".best_in_place").best_in_place()
  autosize $('textarea')

  $('[data-disable-button-if-empty]').on('change paste keyup input', ->
    disabled = this.value.length == 0
    button = $(this).parents('form').first().find('[type=submit]')
    button.prop('disabled', disabled)

  # initial status
  $.each $('[data-disable-button-if-empty]'), (index, el) ->
    $(el).trigger('change')

  $('[data-on-change-submit]').on 'change', ->
    $(this).parents('form').first().submit()
~~~

If you want to perform something on click for existing and new elements, but
only on particular page, you can bind bind click on document on that page inside
body. But when you navigate 10 times, it will trigger 10 times. So you can
unbind on `page:before-change`. Note that you should write that after function
declaration because when you click on paga again, it will assign "on" on
previous version, and unassign latest version of your_function

~~~
<script>
function your_function() {
  LOG && console.log("your_function call");
}
$(document).on('click', '.class', your_function);
$(document).one('page:before-change',function() {
  LOG && console.log("unassign your_function");
  $(document).off('click', your_function);
});
</script>
~~~

# Seed

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
  { name: "Admin/Office" }
].each do |doc|
  job_type = JobType.where(doc).first_or_create! do |job_type|
    # you do not need to call save! here
    # put common stuff here
    job_type.domain = 'my_domain'
    puts "JobType #{doc[:name]}"
  end
end if Rails.env.development?

# deterministic and random data
1.upto(10).map do |i|
{ email: "user#{i}@asd.asd", password: Faker::Internet.password }
end.each do |doc|
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

Another use of rake is to seed

~~~
# lib/tasks/seed.rake
namespace :seed do
  task :bid_types => :environment do
    ["live", "silent", "teacup"].each do |name|
      BidType.find_or_create_by! name: name
    end
  end
end

# db/seed.rb
Rake::Task["seed:bid_types"].invoke
~~~

so you can create with `rake db:seed` or `rake seed:bid_types`

If you want to know inside some code whether you run from rake or from rails
(for example you do not want to send emails for seed users), you can use

~~~
if File.basename($0) == "rake"
  # I'm from rake
else
  # I'm from rails
end
~~~

# Sessions and share cookies on multiple subdomains

If you use [sharing-cookies-across-subdomains-with-rails-3](http://makandracards.com/makandra/31381-sharing-cookies-across-subdomains-with-rails-3) or [what-does-rails-3-session-store-domain-all-really-do](http://stackoverflow.com/questions/4060333/what-does-rails-3-session-store-domain-all-really-do)

~~~
# config/initializers/session_store.rb
Rails.application.config.session_store :cookie_store, key: '_myapp_session',
domain: :all, expire_after: 60.minutes
~~~

This works fine for top level domains and subdomains for them.

For example if you have `domain: :all` and you sign in at `asd.dev`, than you
will be signed in also on `asd.asd.dev` and `asd.asd.asd.dev` and so on...
Cookie will be with domain `.asd.dev` for all those sites.

You need to know that some domains like `herokuapp.com` belongs to [Public
Suffix List](https://devcenter.heroku.com/articles/cookies-and-herokuapp-com) so
browser will prevent setting `domain: :all` cookie for them (because someone can
use attacker.herokuapp.com to read cookies from myapp.herokuapp.com).

Note you can not login at `in.rs` (PSL and browser will prevent cookie).

Rails cookie store knows for public suffix list, so for `trk.in.rs` it will use
`.trk.in.rs` and all others like `asd.in.dev` it will use `.in.dev`.

You can login at `trk.in.rs` but it wont be shared to `asd.in.rs`. Cookie will
be with domain `.trk.in.rs`, so you will be signed in also on `1.trk.in.rs`
`2.1.trk.in.rs` ...

Cookie are send on each request. Cookies usually does not expire, but you can
set `expire_after: 60.minutes`. They are send again when you refresh close &
open window.

Note that flash messages are also for each domain. So if you redirect from one
domain to another, you can not use flash messages. Old flash message will be
shown when user come back to previous domain (on which request flash was set).

# Subdomains and long domains

Rails has method [extract
domains](http://api.rubyonrails.org/classes/ActionDispatch/Http/URL.html#method-c-extract_domain)
but it requires second param which determine if it is top level and second level
domain.

~~~
ActionDispatch::Http::URL.extract_domain 'dule.asd.zxc.com', 1 # 'zxc.com'
ActionDispatch::Http::URL.extract_domain 'dule.asd.zxc.com', 2 # 'asd.zxc.com'
~~~

You could try to guess based on next to last string and determine if it
is shorter or equal than 3 chars.
Rails domain is always last two strings.

~~~
# dule.asd.zxc.com
request.host = 'dule.asd.zxc.com'
request.domain = 'zxc.com'
request.subdomain = 'dule.asd'
~~~

When you use url for, you can pass `host` parameter.
But if you pass also `subdomain` than host param will be trimmed to only last
two strings [issue](https://github.com/rails/rails/issues/2025)

~~~
url_for
ActionDispatch::Http::URL.url_for host: 'dule.asd.zxc.com'
# http://dule.asd.zxc.com
ActionDispatch::Http::URL.url_for host: 'dule.asd.zxc.com', subdomain: 'sub'
# http://sub.zxc.com
# note that we are missing 'asd' in domain name
~~~

# Text syntax on buttons and messages

* labels on buttons, titles,...
  * should be upper case with no period: `This Job Is Paused`
* sentences on error messages, flash messages...
  * should have period at the end: `Job has been copied.`

# 7 patterns

From
[7-ways-to-decompose-fat-activerecord-models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
suggestions for code organization I use mostly:

You can check also <http://trailblazer.to/>
You can organize domain (controller, model and view) into separate folders
[drawers](https://github.com/NullVoxPopuli/drawers)

## Service objects

~~~
# app/services/match_posts.rb
def MatchPosts
  def initialize(posts)
    @posts = posts
  end

  def perform
  end
end
~~~

or more complex with exception rescue.

~~~
# app/services/my_service.rb
class MyService
  class Result
    attr_accessor :success, :message
    def initialize(success, message)
      @success = success
      @message = message
    end

    def success?
      success
    end
  end

  # Some custom exception if needed
  class ProcessException < Exception
  end

  def initialize(h)
    @user = h[:user]
  end

  def process(posts)
    success_message = do_something posts
    Result.new true, success_message
  rescue ProcessException => msg
    Result.new false, msg
  end

  private

  def do_something(posts)
    raise ProcessException, "Error: empty posts" unless posts
    "Done with do_something"
  end
end
~~~

~~~
# main.rb
require './my_service.rb'

my_service = MyService.new user: 'me'
puts my_service.process(1).success? # true
puts my_service.process(1).message # Done with do_something
puts my_service.process(false).success? # false
puts my_service.process(false).message # empty posts
~~~

## Form Objects

form objects (query objects) for multiple in multiple out data

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

## Concerns

[DHH](https://signalvnoise.com/posts/3372-put-chubby-models-on-a-diet-with-concerns) and interesting is
[trashable](https://github.com/discourse/discourse/blob/master/app/models/concerns/trashable.rb)

~~~
module Someable
  extend ActiveSupport::Concern

  included do
    has_many :something_else, as: :someable
    class_attribute :tag_limit
  end

  # methods defined here are going to extend the class, not the instance of it
  module ClassMethods
    def tag_limit(value)
      self.tag_limit_value = value
    end
  end
end
~~~

## Observable

We an use observable objects to send notifications [implementation in 3
languages](http://www.diatomenterprises.com/observer-pattern-in-3-languages-ruby-c-and-elixir/)
It could look like [action as a
distance](https://en.wikipedia.org/wiki/Action_at_a_distance_(computer_programming))
antipattern but if we explicitly add than is it fine.

~~~
module ObservableImplementation
  def observers
    @observers ||= []
  end

  def notify_observers(*args)
    observers.each do |observer|
      observer.update(*args)
    end
  end

  def add_observer(object)
    observers << object
  end
end

class Task
  include ObservableImplementation
  attr_accessor :counter
  def initialize
    self.counter = 0
  end

  def tick
    self.counter += 1
    notify_observers(counter)
  end
end

class PutsObserver
  def initialize(observable)
    observable.add_observer self
  end

  def update(counter)
    puts "Count has increased by #{counter}"
  end
end

class DotsObserver
  def initialize(observable)
    observable.add_observer self
  end

  def update(counter)
    puts "." * counter
  end
end

task = Task.new
task.tick
DotsObserver.new(task)
task.tick # ..
PutsObserver.new(task)
task.tick # ...
# Count has increased by 3
~~~

# Unsubscribe links

~~~
# app/views/layouts/_email_footer.html.erb
<p>
  <%= link_to "Unsubscribe", link_for_unsubscribe(user, controller.mapping_to_unsubscribe_group(controller.action_name)) %> from this type of email.
</p>
<p>
  <%= link_to "Manage", settings_user_url(user) %> which emails you receive.
</p>

# app/controllers/application_mail.rb
  def mapping_to_unsubscribe_group method_name
    group = ApplicationHelper::UNSUBSCRIBE_MAPPING_METHOD_TO_GROUP[method_name.to_s]
    if group.nil?
      puts "!!!!! No group found for method_name=#{method_name.to_s}"
      ExceptionNotifier.notify_exception(Exception.new("just to notify that there is no group for method_name #{method_name}") )
    end
    group
  end

# app/helpers/application_helper.rb
  UNSUBSCRIBE_MAPPING_METHOD_TO_GROUP = {
    "first_application_instructions" => "tips_and_help_emails_jobseeker",
    "application_in_review_instructions" => "tips_and_help_emails_jobseeker",
    "new_candidate_email" => "new_candidate_or_applicant",
    }
  def link_for_unsubscribe user, unsubscribe_group
    referral_token_and_unsubscribe_group = user.referral_token + unsubscribe_group.to_s
    unsubscribe_url key: Base64.encode64(referral_token_and_unsubscribe_group)
  end

# config/routes.rb
  get 'unsubscribe', to: 'welcome#unsubscribe'

# app/controllers/welome_controller.rb
  # GET /unsubscribe
  def unsubscribe
    referral_token_and_unsubscribe_group = Base64.decode64 params[:key] 
    referral_token = referral_token_and_unsubscribe_group[0..User::REFERRAL_TOKEN_LENGTH-1]
    @unsubscribe_group = referral_token_and_unsubscribe_group[User::REFERRAL_TOKEN_LENGTH..-1]
    if current_user
      if current_user.referral_token == referral_token
        @user = current_user
        @user.unsubscribe[ @unsubscribe_group] = "true"
        @user.save!
      else
        redirect_to root_path, alert: "This key is for different user, please log out"
      end
    else
      if @user = User.find_by( referral_token: referral_token)
        # we found the user
        @user.unsubscribe[ @unsubscribe_group] = "true"
        @user.save!
      else
        flash[:alert] = "Can't find user by this key. You need to signin in order to unsubscribe"
        redirect_to new_user_session_path and return
      end
    end
  end

# db/migrate/_add_unsubscribe_to_user.rb
class AddUnsubscribeToUser < ActiveRecord::Migration
  def change
    add_column :users, :unsubscribe, :hstore, default: {}
  end
end
~~~

# Run rails in production mode

You probably need to set up database user for production env.

Also set `config.force_ssl = false` in *config/environments/production.rb*

If you use canonical host you need to disable it `export
DO_NOT_USE_CANONICAL_HOST=true` and remove eventual
`config.action_controller.asset_host = asdasd`

~~~
RAILS_ENV=production rake db:create
RAILS_ENV=production rake assets:precompile

export RAILS_SERVE_STATIC_FILES=true
rails s -b 0.0.0.0 -e production
~~~

# Get template name

You can try directly in controller to `byebug` and try something like

~~~
view_context.view_renderer.instance_variable_get('@lookup_context').instance_variable_get('@view_paths').instance_variable_get('@paths').first.instance_variable_get('@cache').instance_variable_get('@data').instance_variable_get('@backend').values.first.instance_variable_get('@backend').values.first.instance_variable_get('@backend').values.first.instance_variable_get('@backend').values.first.values
~~~

Another is with patch
[ActionView::TemplateRenderer](http://stackoverflow.com/questions/4973699/rails-3-find-current-view-while-in-the-layout/8310881#8310881)
and this is only accessible in view.

~~~
# getting current template is not possible in rails
# https://github.com/rails/rails/issues/4876
# patch taken from
# http://stackoverflow.com/questions/4973699/rails-3-find-current-view-while-in-the-layout/8310881#8310881
# note 3th comment (Lukas) about exception notification issue
class ActionController::Base
  attr_accessor :active_template

  def active_template_virtual_path
    self.active_template.virtual_path if self.active_template
  end
end

class ActionView::TemplateRenderer
  alias_method :_render_template_original_, :render_template

  def render_template(template, layout_name = nil, locals = {})
    if @view.controller && @view.controller.respond_to?('active_template=')
      @view.controller.active_template = template
      result = _render_template_original_( template, layout_name, locals)
      @view.controller.active_template = nil
    else
      result = _render_template_original_( template, layout_name, locals)
    end
    result
  end
end
~~~

# Email Style

<https://github.com/Mange/roadie>

# Money

Dealing with money with <https://github.com/RubyMoney/money-rails>

# Tips

* parse url to get where user come from `URI.parse(request.referrer).host`
* always use `@post.destroy` instead of `@post.delete` because it propagates
  to all `has_many :comments, dependent: :destroy` (also need to define this
  dependent param)
* `spring stop` in many cases:
  * when you export some ENV and use them in `config/secrets.yml` but can't see
    in `rails c`
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
* use include for n+1 query (bullet) and joins when you don't need associated
  models. Both are using INNER JOIN
  * `Comment.all(include: :user, conditions: { users: { admin: true}})` will
    load also the user model
  * `User.all(joins: :comments, select: "users.*, count(comments.id) as
    comments_count", group: "users.id")` you can output just specific value
* [N+1](http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations)
  problem is solved with `includes(:associated_table)` wich is actually LEFT
  OUTER JOINS, but `joins(:associated_table)` is INNER JOIN. Works also for
  nested joins `includes(jobs: [:user])`.
  * If model has_many :associated_table then `joins` will return multiple values
  because of multi values of :associated_table per item, or none is
  :associated_table does not exists for current item.
  * `includes` will return the same number of items, with association object
  loaded in memory.  eager load `includes` can be defined in association
  definition `has_many :comments, -> { includes :author }` but this is bad since
  it will **always** load two tables.  Better is to make a query
  `Post.includes(:comments).all` or for instance
  `@post.comments.includes(:author)`. You need to eager load before calling
  `.all` or `.each`
* If you want to filter with raw SQL like: `.where('jobs.title ILIKE "%duke%"')`
you need to `reference` them (see below) or use join (filtering using hash works
without referencing since rails knows which table to look `.where(jobs: { title:
}, user: company.users)`). Along with
[includes](http://apidock.com/rails/ActiveRecord/QueryMethods/includes)
`includes(user: :user_profile).where("user_profiles.name = 'dule'")`
you need to explicitly reference them `includes(user:
:user_profile).where("user_profiles.name = 'dule'").references(user:
:user_profile)`
* note that you should not use `joins` and `includes` in the same time, for the
same columns (you can use it for different columns). Joins
could be replaced with `references`. But I have some problems with `references`
since it remove my custom selected data, so instead `references` I use
`joins('LEFT OUTER JOIN sports ON sports.id = users.sport_id')` and `distinct`.
Rails 5 has method
[left_outer_joins](http://edgeguides.rubyonrails.org/active_record_querying.html#left-outer-joins)
* when you need to eager load for a single object (show action) than you can
simply repeat rails default before action `set_post` with: `@post =
Post.includes(:comments).find params[:id]`


* `some_2d_array.each_with_index do |(col1, col2),index|` when you need
  [decomposition
  array](http://docs.ruby-lang.org/en/2.1.0/syntax/assignment_rdoc.html#label-Array+Decomposition)
  to some variables, you can use parenthesis.
* fake objects could be generated with `OpenStruct.new name: 'Dule'`
* long output of a command (for example segmentation fault) can be catched with
  `rails s 2>&1 | less -R`
* use different layout and template based on different params is easy using
locales [look for adminlte example]( {{ site.baseurl }}
{% post_url 2016-10-28-adminlte-free-template-on-rails %})

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
* to change class **fields_with_error** you can override
  ActionView::Base.field_error_proc in any controller

  ~~~
  ActionView::Base.field_error_proc = Proc.new { |html_tag, instance|
    #html = %(<div class="field_with_errors">#{html_tag}</div>).html_safe
    # add nokogiri gem to Gemfile
    elements = Nokogiri::HTML::DocumentFragment.parse(html_tag).css "label, input"
    elements.each do |e|
      if e.node_name.eql? 'input'
        e['class'] ||= ''
        e['class'] = e['class'] << " error"
        html_tag = "#{e}".html_safe
      end
    end
    html_tag
  }
  ~~~


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
* for `gon` gem you should `include_gon` before other javascript files. Note
  that `if gon` will still raise error if `gon` is not called in controller (so
  `gon` is undefined)

* access helpers outside of a view
  * if it is standard base helper, than just call it from ActionController,
    ActionView instance or Rails application routes.

  ~~~
  ActionController::Base.helpers.pluralize(count, 'mystring')
  ActionView::Base.new.number_to_human 123123
  Rails.application.routes.url_helpers.jobs_url
  ~~~

  * if it your custom helper you can call from *ApplicationController*

  ~~~
  ApplicationController.helpers.my_custom_helper
  ~~~

  * you can include all

  ~~~
  include ActionView::Helpers::TextHelper
  include Rails.application.routes.url_helpers
  ~~~

  * or you can delegate it

  ~~~
    delegate :url_helpers, to: 'Rails.application.routes'

    # call with `url_helpers.jobs_url` in you class
  ~~~


* do not name your model with `Template` since there is module `Template`
  somewhere.
* request object contains a lot of data:
  * `request.xhr?` is it ajax
  * `request.ip` `request.referrer` `request.remote_ip`
  * `request.env["HTTP_USER_AGENT"]`

* Heroku problems:
* `undefined method url_options' for #<Module:` maybe problem with
  [puma](https://gist.github.com/IAMRYO/e8bee5a8e6710ad4b970)

* rails router
  * you can mount namespace under different path

  ~~~
  namespace :admin, path: 'aadmin' do
    get '/', to: 'admin#index'
  end
  ~~~

  * you can get dynamic path resoulution with `link_to "Dynamic #{e}",
  controller: :pages, action: e, id: @user.id`

* export csv

  ~~~
  # app/models/user.rb
  def self.to_csv
    CSV.generate do |csv|
      csv << %w(Name Email Sports)
      all.each do |user|
        csv << [user.name, user.email, user.sports.map(&:name).join(', ')]
      end
    end
  end

  # app/controllers/users_controller.rb
    respond_to do |format|
      format.html
      format.csv { send_data @users.to_csv, filename: 'users.csv', type:
      'text/csv', disposition: 'inline' }
    end

  # app/views/index.html.erb
  <%= link_to "Export csv", users_path(sport: params[:sport], format: :csv) %>
  ~~~

* [acts_as_lists](https://github.com/swanandp/acts_as_list) is nice gem, you
  just need to `rails g migration add_priority_to_comment priority:integer`,
  add a line in model `acts_as_list scope: :post, column: :priority` and to use
  that priority in associations `has_many :comments, -> { order priority: :asc
  }, dependent: :destroy`
* `enum status: [:paused]` should be type integer, or it will return nil

* params usually need to be striped, so you can use this code to get rid of all
  unnecessary spaces

  ~~~
  params_contact_striped = params[:contact].each_with_object({}) { |(k,v),o| o[k] = v.split.join(" ") } # strip spaces
  ~~~

  or you can do in before save callback

  ~~~
  # app/models/contact.rb
    before_save :strip_fields

    def strip_fields
      self.email.strip! if email.present?
      self.first_name.strip! if first_name.present?
    end
  ~~~

* also if you want to replace '\n' new lines in text area (`f.text_area`) with
<br> so it looks the same when displaying it (alternativelly you can use `<%=
simple_format @contact.text %>`).  Example is with nested associated elements:

  ~~~
  params[:contact] = params[:contact].each_with_object({}) { |(k,v),o| o[k] = v.each_with_object({}) { |(k1,v1),o1| o1[k1] = (v1.class == String ? v1.gsub(/\n/,'<br>'): v1) } }
  ~~~

* if you notice in logs double requests (messages are double rendered) it is
  probably that you use `gem 'rails_12factor'` which should be only on
  production `group: :production`
* use cancancan and define all your actions in app/models/ability. If you want
  to use
  [load_resource](https://github.com/ryanb/cancan/wiki/authorizing-controller-actions#load_resource),
  you should separately define: index (with hash), show/edit/update/destroy
  (with block or hash), new/create (with hash). For nested resources just write
  parent association
* when we use CDN for assets, in root folder it should contain crossdomain.xml
  file so flash recorder works nice.

  ~~~
  <?xml version="1.0"?>
  <!-- http://www.osmf.org/crossdomain.xml -->
  <!DOCTYPE cross-domain-policy SYSTEM "http://www.adobe.com/xml/dtds/cross-domain-policy.dtd">
  <cross-domain-policy>
      <allow-access-from domain="*" />
      <site-control permitted-cross-domain-policies="all"/>
  </cross-domain-policy>
  ~~~

* very nasty issue is when your before action returns
  false [halting
  execution](http://guides.rubyonrails.org/active_record_callbacks.html#halting-execution)
  so check if return value could be false and make sure before filters (like
  before_create) always return not false value (you can return true or nil).

# PDF

## Wicked PDF

<https://github.com/mileszs/wicked_pdf>

## Prawn

* if you need custom symbols in prawn than you need to use ttf fonts. Not all
  contains every symbol. I downloaded from
  [dejavu-fonts.org](http://dejavu-fonts.org/wiki/Download)

  ~~~
  # app/pdfs/common_pdf.rb
  module CommonPdf
    def h(amount)
      ActionController::Base.helpers.humanized_money_with_symbol amount
    end

    def set_up_common_font
      # http://dejavu-fonts.org/wiki/Download
      font_families.update(
        "DejaVu Sans" => {
          normal: "#{Rails.root}/lib/DejaVuSans.ttf",
          bold: "#{Rails.root}/lib/DejaVuSans-Bold.ttf",
        }
      )
      font "DejaVu Sans"
    end
  end

  # app/pdfs/my.pdf.rb
  class MyPdf < Prawn::Document
    include CommonPdf
    set_up_common_font
    text h 10
  end
  ~~~

* to start new project from specific rails, run `rails _4.2.7.1_ new myapp` .
  `gem list | grep rails` can show you installed versions
* for multiple form submit buttons you can use rails builder

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
* when you call render partial with current object than first param is string
(not hash `partial: `) `<%= render 'layouts/audit_log', current_object: @user
%>`
* you can use config inside your initializers so you do not need to write
`Rails.application.` again, but does not work for `cache_store`

  ~~~
  # config/initializers/dalli.rb
  Rails.application.configure do
    secrets.internal_notification_email
    config.,,,
  end
  ~~~

* for complex forms, it is advised to be able to easily populate all required
fields, so I populate data in controller or in model. For unique fields you need
to iterate...

  ~~~
  <%# app/views/items/_form.html.erb
  <% if Rails.env.development? %>
    <%= link_to "Example", new_item_path(example: true) %>
  <% end %>

  # app/constrollers/items_controller.rb
  def new
    @item = Item.new
    if Rails.env.development? && params[:example] == "true"
      @item.assign_attributes(
        name: 'Example Name',
        phone: '123',
      )
    end
  end

  # or
  # app/controllers/items_controller.rb
  def new
    @item = Item.new
    @item.example if Rails.env.development? && params[:example] == "true"
  end

  # app/models/item.rb
  def example
    i = 1
    while self.class.find_by name: "Example Name #{i}"
      i += 1
    end
    self.name = "Example Name #{i}"
  end
  ~~~

* to assign multiple attributes to active record object you can use `slice`
(`pluck` is for database query) and `assign_attributes`

  ~~~
  user.assign_attributes other_user.slice :email, :phone
  ~~~

* to count by grouping you can group_by specific column
`User.group(:company_id).count.values.max`

* to run ruby script with rails you can include

~~~
require File.expand_path('/home/orlovic/rails/myApp/config/environment', __FILE__)
puts User.all
~~~

* if you are using rails data attritubes than you to not need to use `to_json`.
And if you use jQuery data method than you do not need to use JSON.parse.

  ~~~
  <%= f.text_field :name, "data-predefined-range": {a: 3} %>
  <input data-predefined-rage="<%= {a: 3}.to_json %>">

  <script>
  range = JSON.parse(input.dataset.predefinedRange);
  range = $(input).data("predefinedRage");
  ~~~

* security tips
<https://github.com/brunofacca/zen-rails-security-checklist>

* if you want to use `key.to_sym` for all keys you can use `hash.symbolize_keys`
but if you want that recursively for nested hashes as well you can use
`params[:some_pararam].deep_symbolize_keys`
