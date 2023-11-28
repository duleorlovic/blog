---
layout: post
tags: asd
---

# Validations

Some usefull validations, like validate email regexp

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

<%= f.select :job_type_id, Job.all.map { |job| [job.name, job.id] }, {
prompt: true }, class: 'my-class' %>
~~~

NOTE that in Rails 5.2 we do not need to add presence validations but if you
want to disable you can do with `belongs_to :job_type, optional: true`

You should avoid saving without validation `save(validate: false)` or
`update_attribute :name, 'my name'`. It is risky to save without validations.
Other methods also do not check validation
http://www.davidverhasselt.com/set-attributes-in-activerecord/
`update_column`, `@users.update_all`.

Conditional validations can be used with proc new like `validate :a, if: ->(o) {
}` but with parameters. Also `if: lambda {|a| }` (difference in required params
to block)

~~~
validates :password, confirmation: true, if: Proc.new { |a|
a.password.present?}
~~~

Validate occurs `:on` `:save` by default (but you can not use `validate :d, on:
:save`).  http://guides.rubyonrails.org/active_record_validations.html#on you
can split and specify to run on `on: :create` or `on: :update`. To run
validation on destroy you need hook `before_destroy` where you can `errors.add`
and `return false` so hook reverts.

DO NOT USE VALIDATION FOR BUSINESS LOGIC (except validation for columns), FOR
EXAMPLE DO NOT CREATE VALIDATION THAT AT LEAST ONE LOCATION USER SHOULD BE
ADMIN. This is bad, since you need to follow that validation in all tests (and
you can not destroy current location user, but you need to create another admin
location user). BETTER IS TO USE SERVICES IN CONTROLLER ON UPDATE AND DESTROY
result = UpdateOrDestroyLocationUser.new(current_user, current_location).update params

# Hooks

**NOTE THAT YOU SHOULD NOT USE HOOKS... Better is to just call a method where
needed, because you will not needed it always (for example in tests you want
different value) If you really need (default value that you really want) than
please use ||= conditional assignment**
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
`default_values_on_update` (which could get `nil` on update some fields)

~~~
# do not use after initialize since it will be run on every load
# after_initialize :default_values_on_initialize
before_validation :_default_values_on_create, on: :create
before_validation :_default_values_on_update, on: :update
before_save :_clear_unchecked_values

private

def _default_values_on_create
  self.logo ||= Rails.application.secrets.default_restaurant_logo
  # do not use self.some_true_value ||= true since that will override if
  # some_true_value = false
  self.some_true_value = true if some_true_value.nil?
  # http://guides.rubyonrails.org/active_record_callbacks.html#halting-execution
  # return value should be true or nil
  true
end
~~~

TO calculate some files you can use `after_save :update_total`. Do not use
`after_update :update_total` since you can not update in that method.

Custom validations `validate :my_method` or `validate { |customer|
customer.check_permissions }` should add `errors` to the object (return value is
not important and could be false).

~~~
class Customer
  def my_method
    errors.add(:name) if name != 'Duke'
  end
  def check_permissions
    errors.add(:name) if name != 'Duke'
  end
end
~~~

# Default Order

Do not use `default_scope`, since you need to use `.reoder` instead of `.order`.
If you really want to use, here is example:

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
  scope :by_status_param, -> (status_argument) { where status: status_argument }
  scope :for_user, (lambda do |user|
    where user: user
  end)
~~~

than query `User.first.comments.trashed` will return all trashed comments from
ALL users! `unscoped` will remove even association scopes, thanks Joseph
Ndungu on
[comment](http://www.victorareba.com/tutorials/creating-a-trashing-concern-in-rails-5).

Uncope can receive params what to unscope, example `User.unscope(:where)`. If
you use this in `belongs_to :location, -> { unscope :where }` than it will also
remove association belongs to condition, so you have to unscoped only columns
from default_scope `belongs_to :location, -> { unscope where: :operator_id }`
[excellent link how to remove
scope](https://singlebrook.com/2015/12/18/how-to-carefully-remove-a-default-scope-in-rails/)
I usually create additional classes with `self.default_scopes = []` so it does
not use default scope. Also set table name so I can use sql.
foreign_key is `"#{model}_id"`, class_name is `"#{model.capitalize}"` and it
should be used on both sides (belongs_to and has_many)

~~~
class UnscopedCustomer < Customer
  self.default_scopes = []
  self.table_name = "customers"
  belongs_to :unscoped_company, foreign_key: :company_id
end

class UnscopedCompany < Company
  self.default_scopes = []
  self.table_name = "companies"
  has_many :customers, foreign_key: :company_id
end
~~~

# Format date

Write datetime in specific my_time format
https://apidock.com/ruby/DateTime/strftime
https://www.foragoodstrftime.com
```
Time.zone.now.strftime '%F %T'
```

You can see default date formats but they are used only when locales are used
(for example byebug in a view)
```
(byebug) I18n.translate('date.formats.default')
"%m/%d/%Y"
(byebug) Date.today.to_s :default
"2018-11-05"
(byebug) l Date.today, format: :default
"11/05/2018"
```

So I override default to_s (output). For parsing (input) best way is to use
month name in select date `2/Nov/2001` so it can parse collectly. For inputs
with two numbers it can not guess the month or date `2/3/2000` so you need to
use `Date.strptime '2/3/2000' , '%m/%d/%y'` to return 3 Februar.

~~~
# config/initializers/date_time_formats.rb
# https://api.rubyonrails.org/v6.1.4/classes/DateTime.html#method-i-to_formatted_s
# https://apidock.com/ruby/DateTime/strftime
# all 3 classes
# puts user.updated_at.to_s :myapp_time
# puts Time.now.to_s :myapp_time
# puts Date.today.to_time :myapp_time # Date need to be type casted to Time
# puts Time.now.to_date :myapp_date # Time object to Date if we want myapp_date

Time::DATE_FORMATS[:default] = "%d-%b-%Y %I:%M %p" # this is same as for
# datepicker format = 'DD-MMM-YYYY h:mm A'
Time::DATE_FORMATS[:at_time] = ->(time) { time.strftime("%b %e, %Y @ %l:%M %p") }

Date::DATE_FORMATS[:myapp_date] = ->(date) { date.strftime("%b %e, %Y") }
# 9th November 1988
Date::DATE_FORMATS[:myapp_date_ordinalize] = ->(date) { date.strftime("#{date.day.ordinalize} %b %Y") }
~~~

`Time.now` and `Date.today` are using system time. If you are using
`browser-timezone-rails` than timezone will be set for each request.
In Rails 6 you do not need a gem, just include one js file and use around action
https://github.com/kbaum/browser-timezone-rails/issues/45#issuecomment-915124466

But if you
need something from rails console, you need to set timezone manually.
`Time.zone = 'Belgrade'` (list all in `rake time:zones:all`  or
`ActiveSupport::TimeZone.all`).
Problem is that the name is different
```
Time.zone.to_s # "(GMT+01:00) Europe/Belgrade"
Time.zone.name  # "Europe/Belgrade"

ActiveSupport::TimeZone.all.map &:to_s # [ ..."(GMT+01:00) Belgrade"
ActiveSupport::TimeZone.all.map &:name # [ ..."Belgrade"

# so we need to use utc_offset or time_zone.tzinfo.name
ActiveSupport::TimeZone.all.select { |timezone| timezone.utc_offset == Time.zone.utc_offset }
ActiveSupport::TimeZone.all.select { |timezone| Time.zone.name.include? timezone.tzinfo.name }.first

# group by utc_offset
``` 
It is good to always use `Time.zone.now` and
`Time.zone.today`. Rails helpres use zone (`1.day.from_now`)
Change system timezone (which is used by browsers) with `sudo dpkg-reconfigure
tzdata`. Note that `rails c` uses UTC `Time.zone # => "UTC"`, but byebug in
rails s returns default time zone `Time.zone # CEST Europe`.
`Time.zone.now.utc_offset` will return offset to UTC in seconds. I do not know
why `Time.zone.utc_offset` returns different results (3600 instead of 7200).

When parsing user input you should use `.zone` in methods like
```
Time.zone.parse '2010-01-02 09:00:00'
Date.zone.parse '2010-01-02 09:00:00'
```

* prefer using `Time.current` over `Time.now`, and `Date.current` over
`Date.today`
* to get weekday from `ActiveSupport::TimeWithZone` use
  [link](http://api.rubyonrails.org/classes/ActiveSupport/TimeWithZone.html)
  `weekday = t.to_a[6]`
* to get a weekday from a date use
<https://ruby-doc.org/stdlib-2.4.2/libdoc/date/rdoc/Date.html#method-i-cwday>
  * `d.wday` (0-6, Sunday is zero)
  * `d.cwday` (1-7. Sunday is 7, Moday is 1)
* to get first calendar date `Time.zone.today.beginning_of_month` or
`Time.zone.today.end_of_month`
  * get interval for previous month: `'Last Month':
  [Time.zone.today.prev_month.at_beginning_of_month,
  Time.zone.today.prev_month.at_end_of_month],`
* to get month date use `d.day`


# Multiline render js response

Write long string in multiple lines with `%()`, for example:

~~~
format.js do
  render js: %(
    $('##{key}').replaceWith('#{view_context.j view_context.render partial: 'product_table', locals: { products: Product.send( key).all, product_type_string: key.to_s}} ');
    jQuery(".best_in_place").best_in_place();
    )
end

format.js { render js: "windnw.location.assign('#{user_path}');" }
~~~

If you receive a lot of errors `An ActionView::MissingTemplate occurred in` and
`Missing template customer/sessions/new, customer_application/new,
application/new with {:locale=>[:in, :en], :formats=>["Application/*"],
:variants=>[], :handlers=>[:erb, :builder, :raw, :ruby, :coffee, :jbuilder]}.
Searched in:` on some landing or login pages than simply add:

~~~
# app/controllers/home_controller.rb
class HomeController < ApplicationController
  respond_to :html

  def index
    respond_with do |format|
      format.html { render :index }
      format.any { render :index }
    end
  end
end
~~~

Another solution to respond to all formats is to add `formats` argument

~~~
# app/controllers/home_controller.rb
class HomeController < ApplicationController
  def index
    render :index, formats: [:html]
  end
end
~~~

So this will render index for any type of requests

~~~
curl http://loc:3001
curl http://loc:3001 -H "Accept: application/json"
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

Show all databases in postgresql, first change user to postgres

~~~
sudo su -l postgres
psql

# or in one command
sudo -u postgres psql
~~~

~~~
\list
\connect database_name
\dt

CREATE USER orlovic WITH CREATEDB PASSWORD '<password>';
~~~

In rails you can access database using 'rails db' and see table definition

```
rails db
\d users
```

## Active record

arel https://blog.saeloun.com/2021/10/19/rails-arel-primer
https://gist.github.com/ProGM/c6df08da14708dcc28b5ca325df37ceb
* one issue is when you use different table name `self.table_name =
  'something_other'` than relations could not be picked correctly
* union is not supported [issue 929](https://github.com/rails/rails/issues/939)
  but you can try with raw sql and `from` statement. Union should have same
  number and type of columns. Also use `.from([Arel.sql('...')])`

  ~~~
    sql = <<~SQL
    ( ( SELECT id, location_id, package_saleid, subscriber_id, subscriber_name, location_package_id, sale_date, -total_amount_cents as total_amount_cents, total_amount_currency, subscriber_invoice_id, (bytes_uploaded_total + bytes_downloaded_total) as total_data_used
        FROM location_package_sales
      ) UNION (
        SELECT id, location_id, balance_refillid as package_saleid, NULL as subscriber_id, 'refill' as subscriber_name, NULL as location_package_id, created_at as sale_date, refill_amount_cents as total_amount_cents, refill_amount_currency as total_amount_currency, NULL as subscriber_invoice_id, NULL as total_data_used
        FROM location_balance_refills
      )
    ) AS location_package_sales_or_location_balance_refills
    SQL
    @all_location_package_sales_or_location_balance_refills = LocationPackageSaleOrLocationBalanceRefill
      .includes(:location, :customer, :subscriber_invoice, :location_package)
      .from([Arel.sql(sql)])
      .where(location_id: current_location)
  ~~~

  and model
  ```
  # this model is based on database view location_package_sale_or_location_balance_refills
  class LocationPackageSaleOrLocationBalanceRefill < ApplicationRecord
    belongs_to :location
    belongs_to :subscriber
    belongs_to :location_package
    belongs_to :subscriber_invoice

    include Monetable
    monetize_columns :total_amount, :price_to_location_with_tax

    def refill?
      subscriber_name == 'refill'
    end
  end
  ```

* execute sql from rails console. When you run sql result is iteratable (array
 of arrays of selected fields).

```
s = 'SELECT * FROM users'
r = ActiveRecord::Base.connection.execute s
r.each { |l| puts l }
```

* to find table name `ApplicationRecord.connection.table_exists? :user_sessions`
  or to see which database is used
```
rails runner "puts ActiveRecord::Base.configurations['development']"
rails runner "puts ActiveRecord::Base.configurations[Rails.env]"
# new version
bundle exec rails runner "puts ActiveRecord::Base.configurations.configs_for(env_name: Rails.env).to_s"
# or
ActiveRecord::Base.configurations.configs_for(env_name: Rails.env).last.adapter
```

* for single item, always use `@post.destroy` instead of `@post.delete` because
it propagates to all `has_many :comments, dependent: :destroy` (also need to
define this dependent param). `destroy` could be slow if there are a lot of
dependent items, so in that case you can use `dependent: :delete_all` since it
runs sql delete statement instead of loading each active record and call destroy
on it.
https://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html
If you use delete or use destroy without
`has_many dependent: destroy` than `ActiveRecord::InvalidForeignKey:
SQLite3::ConstraintException: FOREIGN KEY constraint failed` for sqlite db...
For mysql and postgres, error will be triggered if there are foreign keys
defined.
* for multiple items `Post.destroy_all` will run one by one (triggering
callbacks) so it is probably slow, so it is better to use `Post.delete_all`
which will remove in single sql query. To prevent foreign keys errors, you
should first `Comment.where(post: Post.all).delete_all` and than call
`Post.delete_all`. Watch out if you call delete_all on collection proxy
`post.comments.delete_all`
<https://stackoverflow.com/questions/23879841/activerecord-delete-all-method-updating-instead-of-deleting>
Default strategy is `:nullify` ie keep row, but set `post_id = nil`. So always
use `has_many :association, dependent: :destroy` so it will not use `nullify`
but `delete_all` strategy
[link](http://api.rubyonrails.org/classes/ActiveRecord/Associations/CollectionProxy.html#method-i-delete_all)

## Add json and hstore

Note that you can use simple text column (without default value) and add
`serialize :preferences, Hash` in your model, so `user.prefereces # => {}` is
defined for initial empty value.
You can not search serialized columns (you are limited to reading and writing
data) `User.where(params: { a: 1 })` gives error. You can only use raw sql, for
example `User.where('params = ?', { a: 1}.to_yaml)`.

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

* log levels in Rails https://guides.rubyonrails.org/debugging_rails_applications.html#log-levels
  ```
  # :debug, :info, :warn, :error, :fatal, :unknown are levels from 0 to 5
  config.log_level = :warn # 2
  ```
* To disable mute sql logs in rails logger put this in initializer file

  ~~~
  # config/initializers/silent_sql_log.rb
  ActiveRecord::Base.logger.level = Logger::INFO
  ~~~
  to enable sql logs in rails console
  ```
  ActiveRecord::Base.logger = Logger.new(STDOUT)
  ```
  To show file location where query is triggered
  ```
  ActiveRecord::Base.verbose_query_logs = true
  ```
* to disable assets logs use this

  ~~~
  # config/environments/development.rb
  config.assets.quiet = true
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
  same as in console `Author.select(%(authors.*, (SELECT COUNT(books.id) FROM
  books WHERE author_id = authors.id) AS books_count))`

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

  but remember, using `includes` have a problem with custom name calculated
  columns, so better is to use left join.

## Migrations:

* `rails g model user email_address` will generate migration and model. It is
good to edit `last_migration` and add `null: false` to not null fields
(particularly for foreign keys and enums) also usefull since in sql `where col
!= NULL` will not return any value (also `where NULL = NULL`), you need to use
`where col IS NOT NULL` (which is default for active record `.where.not(col:
nil)`). So better is to use `false` since `where col != false` will return some
results.  Also for string fields `if user.status != 'active'` will be true is
status is `nil` or `inactive`.
Add index `add_index :users, :email_address, unique: true`...
if you want to add index in different step (not in `t.references :c, index:
false` because name is too long (error like `Index name 'index_table_column' on
table 'table' is too long; the limit is 64 characters`) than you can use
different name in separate add_index command (can not rename in same command).
NOTE that you need to use exact column name (with `_id`)
`add_index :users, :company_id, name: 'index company on users'`
change_index does not exists, we need to remove_index and add_index
https://github.com/gregnavis/active_record_doctor to help you find columns
without index
* In migration Product.save will probably break in the future, because *Product*
will be validated for something that we did not know on that time.
We can call `Product.update_all fuzz: 'fuzzy'` or `Product.update_all 'new_col =
old_col'` to update all records in single sql query, without triggering
callbacks or validations.
You can also replace substring for example: `User.update_all('email =
REPLACE(email, "a", "x")')`
Better is to create local class and call reset column information. Also if we
are adding not null column we need to do it in two steps to populate existing
records.

~~~
# db/migrate/20161010121212_update_fuzz.rb
class UpdateFuzz < ActiveRecord::Migration
  # this is local Product class used only inside this migration
  class Product < ActiveRecord::Base
  end
  def change
    add_column :products, :fuzz, :string
    # reset schema infor  is needed since new or renamed columns will be ignored
    # on "save" or 'update_all'
    # Product.reset_column_information
    # check with Product.columns or Product.column_names
    Product.update_all fuzz: 'fuzzy'
    Product.find_each { |product| product.save! }
    change_column :products, :fuzz, :string, null: false
  end
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
* when you restore database you can see that list of performed migrations `rake
db:migrate:status` does not show that any migration was perfomed. You can
manually perform run specific single or all migrations
~~~
# status
rake db:migrate:status

# all including until this version
rake db:migrate VERSION=20161114162031`

# only this migration file
rake db:migrate:up VERSION=20161114162031`

# reverse if it is reversible
rake db:migrate:down VERSION=20161114162031`

# to remove migration so you can redo
mysql> DELETE FROM schema_migrations WHERE version = 20210316092150;
20161114162031;
# or insert if table is already there
mysql> insert into schema_migrations (version) values ('20200915080301');
~~~

if you want to redo migration (revert and migrate again) you can use `redo` 
To drop database in console you can `ActiveRecord::Migration.drop_table(:users)`
* you can use `rake db:migrate:redo STEP=2` to redo last two migrations, or you
can run all migrations to certain point with `rake db:migrate
VERSION=20161114162031` (note that `db:migrate:up` `:down` only run one
migration)
* `rake db:migrate` will also invoke `db:schema:dump` task
  [link](http://edgeguides.rubyonrails.org/active_record_migrations.html#running-migrations)
* to check current metaga data, for example schema enviroment you can run
  ```
  rails runner "ActiveRecord::Base.connection.execute('select * from ar_internal_metadata').each {|r|puts r}"

  # or for test database
  rails runner "ActiveRecord::Base.connection.execute('select * from ar_internal_metadata').each {|r|puts r}" -e test

  ```
* to change column null false to true (to add NOT NULL constraint) for example
  adding new not null column to existing table
  ```
    # add column without constrain
    add_column :families, :kind, :string
    # populate
    Family.all.find_each do |family|
      family.registered!
    end
    # add not null constain, third param is whether the value can be NULL
    # add a contrain
    change_column_null :families, :kind, false
    # drop a contrain (allows to be null)
    change_column_null :families, :kind, true
  ```

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
But rubocop suggest to use separate model for habtm relation (in that case I use
singular first model).

~~~
class CreateCampaignsTemplates < ActiveRecord::Migration
  def change
    create_table :campaign_templates, id: false do |t|
      t.references :campaign, foreign_key: true, null: false
      t.references :template, foreign_key: true, null: false

      t.timestamps
    end
  end
end
~~~

and adding `has_and_belongs_to_many :templates` in those classes.

You can force uniq with `add_index :campaign_templates, [:campaign_id,
:template_id], unique: true` and check in rails with

~~~
# add if not exists
campaign.templates << template unless campaign.templates.include? template

# remove
campaign.templates.delete template
~~~

If you have different name of the table: `t.references :donor, foreign_key:
true, null: false` but donor is actually in users table, than you need to change
foreign key (note that `add_foreign_key` params are in plurals!)

~~~
# to_table is in rails5 table_name, in model use class_name: 'User'
t.references :donor, foreign_key: { to_table: :users }, null: false

# in rails4 use references keyword also as parameter, not that foreight key
# constrains need to be added in separate command. note suffix "_id"
t.references :donor, foreign_key: false, null: false, references: :users
add_foreign_key :table, :users, column: :donor_id
~~~

or manually write relation

~~~
create_table :donations
  t.belongs_to :donor, index: true, null: false
end
add_foregn_key :donations, :users, column: :donor_id
~~~

Also in model you need to write: `has_many :donations, foreign_key: :donor_id`
and `belongs_to :donor, class_name: "User"`. If you need to specify class name
in has_many through association, use option `source: :user`

Note that `t.references` should be used with `foreign_key: true, null: false`
since foreign_key is not automatically used (`t.references` automatically add
`index`, so use `index: false` if you going to create composite index).
Note that `t.belongs_to` by default use `index: false`, so you need to use
`index: true` (or `add_index` later) (`add_column :users, :token, :string,
index: true` will not create index, you need to use `add_index` separatelly).
But usually you use `add_foreign_key` that
will also add index (name will be like: `fk_rails_123123`) if it is not added by
belongs_to, so you can safelly use `t.belongs_to ... index: false` if you are
using `add_foregn_key`.

For MySql I have to use `unsigned: true` in `t.belongs_to :user, null: false,
unsigned: true` or for `t.references :user, unsigned: true`
You can add column at specific location `add_column :feeds, :last_modified,
:datetime, after: :url` (after column does not work in psql since you need to
change table) https://stackoverflow.com/questions/27214251/postgresql-add-column-after-option-usage-in-rails-migration

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
For example if you find using specific column `User.find_by! column: 'val'`. Use
bang version instead of `@user = User.find_by name: 'Duke'` since that will
return `nil` instead of `raise ActiveRecord::RecordNotFound unless @user`
Also we can add index to `:updated_at` column since we sometimes order by
that column.

All foreign keys need to have index.
You can use `add_reference` (adding column and index) note that second param is
in singular.
If you want to add or remove reference in migration `rails g migration
add_user_to_leads user:references` (note plurar here) which will generate

~~~
class AddUserToLeads < ActiveRecord::Migration
  def change
    # reference will automatically include index on that column
    add_reference :leads, :contact, foreign_key: true
  end
end
~~~

For polymorphic associations `projectable_id` and `projectable_type` you can
create in migration `rails g model project projectable:references{polymorphic}`

~~~
class Organization < ActiveRecord::Base
  has_many :projects, :as => :projectable
end

class User < ActiveRecord::Base
  has_many :projects, :as => :projectable
end

class Project < ActiveRecord::Base
  belongs_to :projectable, :polymorphic => true
end
~~~

You can eager load polymorphic association without where conditions on other
tables
https://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html#module-ActiveRecord::Associations::ClassMethods-label-Eager+loading+of+associations
> Since only one table is loaded at a time, conditions or orders cannot reference tables other than the main one. If this is the case, Active Record falls back to the previously used LEFT OUTER JOIN based strategy. For example:

if you use conditions on polymorphic table than
ActiveRecord::EagerLoadPolymorphicError is raised.
since it can not join table name based on name stored in column
but you can specify relations
https://stackoverflow.com/questions/16123492/eager-load-polymorphic
For using in where you need to define specific associations (note: you need to
add `optional: true`)
```
class Notification < ApplicationRecord
  belongs_to :notifiable, polymorphic: true
  # notifiable: Comment, Task, Project need to implement subscribed_users since
  # we send body text to them

  belongs_to :comment, -> { where notifications: { notifiable_type: 'Comment' } }, foreign_key: :notifiable_id, optional: true
  belongs_to :task, -> { where notifications: { notifiable_type: 'Task' } }, foreign_key: :notifiable_id, optional: true

  scope :for_comments_on_tasks, (lambda do
    joins(:comment)
      .where(comments: { commentable_type: 'Task' })
  end)
```

When you create you can use `references` and `polymorphic: true`

~~~
  t.references :featureable, polymorphic: true, null: false
~~~

If you add one by one column than you need to add double index:

~~~
t.string :projectable_type
t.integer :projectable_id

# Bad: This will not improve the lookup speed
add_index :projects, :projectable_id
add_index :projects, :projectable_type

# Good: This will create the proper index
add_index :projects, [:projectable_type, :projectable_id]
~~~

You can add unique index on some existing columns (to see
validation error instead of database esception, should also be in rails
`validates :email, uniqueness: { scope: :user_id }`)

~~~
class AddUniqContacts < ActiveRecord::Migration
  def change
    add_index :contacts, [:email, :user_id], unique: true

    # partial index
    add_index :contacts, [:email, :user_id], unique: true, where: ""
  end
end
~~~

When you have composite index than you do not need to create index for first
column (ie you can perform fast query on first column or on first and second
column, but not on second column), but still need index for second column (or
add index in reverse direction)
https://github.com/gregnavis/active_record_doctor#removing-extraneous-indexes

You can also create ordered index (for example listing all posts and users
sorted alphabetically).
```
add_index :users_posts, [:user_id, :post_id], order: { user_id: :desc }
```

Do not add index for tables that has a lot or removing, since perfomance will be
bad. Also huge tables need huge indexes, so pay attention on size.

Remove index and foreign key if you want to remove column

~~~
  remove_foreign_key :online_payment_responses, :location_packages
  remove_column :online_payment_responses, :location_package_id
  if index_exists?(:online_payment_responses, name: 'fk_rails_b36eeaa704')
    remove_index :online_payment_responses, name: 'fk_rails_b36eeaa704'
  end
~~~

In mysql sometimes `LIMIT 10` is slower than `LIMIT 100` since it won't use
index for small stuff, but if table is huge than it's much slower without index.
To force index use something like `User.from("'users' FORCE INDEX
(my_user_index")`
[link](http://fuzzyblog.io/blog/rails/2017/02/24/understanding-low-level-index-issues-in-mysql.html)
If you receive error `undefined method map for "'users' FORCE INDEX
(my_user_index)":Arel::Nodes::SqlLiteral` than you can try to replace
`includes/references` with `joins`.

## Deadlocks

<https://dev.mysql.com/doc/refman/5.7/en/innodb-deadlock-example.html>
<http://api.rubyonrails.org/v5.1/classes/ActiveRecord/Locking/Pessimistic.html>
<http://www.chriscalender.com/advanced-innodb-deadlock-troubleshooting-what-show-innodb-status-doesnt-tell-you-and-what-diagnostics-you-should-be-looking-at/>

<https://vimeo.com/12941188>

For errors
~~~
Mysql2::Error: Lock wait timeout exceeded; try restarting transaction
Mysql2::Error: Deadlock found when trying to get lock; try restarting transaction: UPDATE
~~~
you can see examples on https://github.com/duleorlovic/deadlock_mysql
To get latest deadlock in mysql run `msql> SHOW ENGINE INNODB STATUS`

`Farmer.lock.find` will wait untill it is free to lock this farmer.

~~~
alert = nil
ActiveRecord::Base.transaction do
  if something_wrong?
    alert = 'Can not ...'
    raise ActiveRecord::Rollback
  end
end
if alert
~~~

Race conditions

When you have a lot of users and long db update commands than you probably have
exceptions for deadlocks and duplicate entries. You can retry
https://speakerdeck.com/rstankov/zero-exceptions-in-production?slide=71

~~~
# app/controllers/users_controller.rb
  def update
    retries ||= 2
    # perform some long task and in another browser and thread try to lock it
  rescue ActiveRecord::RecordNotUnique
    retries -= 1
    raise unless retries.nonzero?
    retry
  end
~~~


# MySql

* if you have
  `/home/orlovic/.rvm/gems/ruby-2.2.4/gems/mysql2-0.4.4/lib/mysql2.rb:31:in
  require: libmysqlclient.so.18: cannot open shared object file: No such file
  or directory -
  /home/orlovic/.rvm/gems/ruby-2.2.4/gems/mysql2-0.4.4/lib/mysql2/mysql2.so
  (LoadError)` than try `gem uninstall mysql2;gem install mysql2`
* on mac os if you have `Mysql2::Error: Can't connect to local MySQL server
  through socket '/var/run/mysqld/mysqld.sock' (2)` than create a link to socket
  file whish is located using `mysql_config --socket` or `mysqladmin variables`

  ~~~
  sudo mkdir /var/run/mysqld
  sudo ln -s /tmp/mysql.sock /var/run/mysqld/mysqld.sock
  ~~~

* create mysql user

  ~~~
  mysql -u root -p
  CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'asdf';

  # list all
  SELECT User FROM mysql.user;
  # change password
  SET PASSWORD FOR 'user-name-here'@'hostname-name-here' = PASSWORD('new-password-here');
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

`$(function)` is shorthand for `$(document).ready(function(){})` (and is not the
same as `$(document).on('ready', function(){})`) and it is when DOM is ready. If
you need to know when all images and iframes are loaded than use
`$(window).on('load', function() {})`.
Turbolinks is installed with adding `gem 'turbolinks'` and include in
application.js `//= require turbolinks`.

> Turbolinks intercepts all clicks on <a href> links to the same domain. When
> you click an eligible link, Turbolinks prevents the browser from following it.
> Instead, Turbolinks changes the browser’s URL using the History API, requests
> the new page using XMLHttpRequest, and then renders the HTML response.

> During rendering, Turbolinks replaces the current <body> element outright and
> merges the contents of the <head> element. The JavaScript window and document
> objects, and the HTML <html> element, persist from one rendering to the next.

With turbolinks rails acts as single page application. It will intercept `<a>`
links, but if you want to redirect using javascript `window.location` than you
can use `Turbolinks.visit link`, for example

~~~
$(document).on 'click', '[data-click]', (e) ->
  Turbolinks.visit $(this).data('click')
~~~

Each visit can be `advance` or `replace`, than can be set with
`data-turbolinks-action="replace"` or `Turbolinks.visit link, { action:
'replace' })`.
`restore` visit is when we click `Back` and can not be canceled.
You can disable turbolink on specific link with `data-turbolinks="false"`

Since `window` and `document` remains, all objects remain in memory.

So if you want to do something on every page and on first load than you need to
bind on `turbolinks:load`
Without turbolinks it would be on ready_page_load.coffee `$(document).on 'ready
page:load', ->`
This file is used when you need to iterate some elements `.each`, to activate
some plugins or when you need to bind click event listener and prevent it in
another data event click

~~~
// app/assets/javascripts/turbolinks_load.coffee
$(document).on('turbolinks:load', ->
  # in previous rails we used to catch up on 'ready page:load'
  console.log "document on turbolinks:load"

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

  # for infinite pagination use two divs, second need to have pagination links
  # <div data-scroll='true'>
  #   <div data-scroll-content='true'>
  #     <% @items.each do |item| %>
  #     <% end %>
  #     <%= will_paginate, class: 'hide-not-important' %>
  #
  # when you do not want auto trigger you can use
  # <div data-scroll='true' data-scroll-auto-trigger='false'>
  #   <div data-scroll-content='true'>
  #     ...
  #     <%= will_paginate @items, next_label: 'Show more...', class: 'show-more-pagination', page_links: false %>
  $('[data-scroll]').each ->
    if $(this).data('scrollAutoTrigger') == undefined
      autoTrigger = true
    else
      autoTrigger = $(this).data('scrollAutoTrigger')
    $(this).jscroll(
      nextSelector: 'a.next_page'
      autoTrigger: autoTrigger
      contentSelector: '[data-scroll-content]'
      loadingHtml: "<div class='col-xs-12 text-center mtb20 widget-loader'><img src='https://d1bnwaoqvo9ynq.cloudfront.net/assets/images/loader_widget_sb.gif' alt='loading'><p>Loading more...</p></div>"
      callback: ->
        console.log 'jscroll callback'
    )

  # we can not use $(document).on 'click', '[data-enable-if-valid]', (e) ->
  # since we can not prevent already bubbled click event in case of invalid input
  $('[data-enable-if-valid]').on 'click', (e) ->
    $button = $(this)
    # http://jqueryvalidation.org/Validator.element/
    # https://johnnycode.com/2014/03/27/using-jquery-validate-plugin-html5-data-attribute-rules/
    validator = $button.parents('form').validate()
    $inputs = $($button.data().enableIfValid).find('input')
    all_valid = true
    $inputs.each ->
      unless validator.element $(this)
        all_valid = false
    if all_valid
      console.log $button.data().enableIfValid + " is valid"
    else
      # this hack will prevent other data- events, like data-active-next
      $button.prop('disabled', 'disabled')
      setTimeout(
        ->
          $button.prop('disabled', false)
        200
      )
      console.log $button.data().enableIfValid + " is not valid"
~~~

When you are using `data-remote-true` for show edit form than request is JS and
any javascript inside `js.erb` or inside partial `$('#id').replaceWith('<%= j
render 'partial' %>');` is executed every time. Problem is with bootstrap
modals where request is HTML. remote modal content is deprecated in
[v4](https://getbootstrap.com/docs/3.3/javascript/#modals-options) so it's
better to use custom implementation

~~~
# app/assets/javascripts/document_on.coffee
$(document).on 'click', '[data-js-modal]', (e) ->
  arget = this.dataset.jsModal
  remote_link = this.href || this.dataset.jsModalHref
  $(target).modal('show')
  # use $.ajax and replaceWith since $.load use .html which discard scripts
  $.ajax(
    url: remote_link
  ).done (responseText) ->
    $(target + " .modal-content").replaceWith responseText
  e.preventDefault()

$(document).on 'click change', '[data-toggle-active]', (e) ->
  target = $(this).data().toggleActive
  if $(this).is(':checkbox')
    # if we click directly on checkbox, it will receive also the click event,
    # but usually that is fine (also when you are using bootstrap toggle)
    to_show = e.currentTarget.checked
  else
    to_show = !$(target).hasClass('active')
  if to_show
    $(target).addClass 'active'
  else
    $(target).removeClass 'active'
  console.log "data-toggle-active #{target}"
~~~


and trigger turbolinks load so it can catch new items. Instead of triggering,
you can call `initializeDatepicker()`. That is also needed below the
forms since ajax request in remote-true forms also need to run the initializaion
scripts.

~~~
# app/assets/javascripts/initialize_datepicker.coffee
window.initializeDatepicker = ->
  $date_elements = $('.date')
  return unless $date_elements
  ...
~~~

If you want to perform something on click for existing and new elements, but
only on particular page, you can bind bind click on document on that page inside
body. But when you navigate 10 times, it will trigger 10 times. So you can
unbind on `page:before-change`. Note that you should write that after function
declaration because when you click on page again, it will assign "on" on
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

* disable turbolinks `document.body.setAttribute('data-no-turbolink','true')`
* turbolinks are good if some of page content is changed using ajax.
  Browser back button when turbolink is enabled shows last content
  (not first that was fatched). When turbolinks is disabled, you can
  force refreshing the page with set_cache_buster before filter.
  Note that any input field stays populated (also hidden input field
  which you eventually populated in javascript):

  ~~~
  def set_cache_buster
    response.headers["Cache-Control"] = "no-cache, no-store, max-age=0, must-revalidate"
    response.headers["Pragma"] = "no-cache"
    response.headers["Expires"] = "Fri, 01 Jan 1990 00:00:00 GMT"
  end
  ~~~

# Seed

I prefer to use fixtures

```
# db/seeds.rb
Rails.logger = Logger.new(STDOUT)
Rake::Task["db:fixtures:load"].invoke
```

This task `db:fixtures:load` is defined
https://github.com/rails/rails/blob/master/activerecord/lib/active_record/railties/databases.rake#L415
for rspec you need
```
ENV["FIXTURES_PATH"] = "spec/fixtures"
Rake::Task["db:fixtures:load"].invoke
```

More advance seeding from fixture data is on
[minitests rails](_posts/2018-01-22-minitests-rails.markdown)

Old way is to use db/seeds.rb.
If you want indempotent seeds data you should have some identifier (for example
`id`) for wich you can run `where(id: id).first_or_create! do...end`.
User should use `first_or_initialize` since we can't create without
password, but we don't have password field. `do ... end` block is used only if
that object is not found.
`slice` is used for uniq fields (not generated by faker), but all other fields
should be populated in block. Slice is nice method to get params hash from
current active record object `user.slice :email, :name # { email: '', name: ''
}`

Use `faker` gem to generate example strings:

* `Faker::Internet.email`
* `Faker::Name.name` `Faker::PhoneNumber.cell_phone` `Faker::Avatar.image("my-own-slug", "50x50")`
* `Faker::Company.name` `Faker::Company.logo` **real logo**
* `Faker::Address.street_address` `Faker::Address.city`
* `Faker::Lorem.sentence` `Faker::ChuckNorris.fact`

~~~
# db/seeds.rb
# we keep all variables (defined as property var: :name) in global b hash
# so you an access them later as `b[:name]`
b = {}

# rubocop:disable Rails/Output
# JobType
[
  { var: :my_job_type, name: 'Admin/Office' }
].each do |doc|
  r = JobType.where(doc.except(:var)).first_or_create! do |job_type|
    # you do not need to call save! here
    # put common stuff here
    job_type.domain = 'my_domain'
    puts "JobType #{doc[:name]}"
  end
  b[doc[:var]] = r if doc[:var]
end

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
# never rescue from Exception, but use Standard error (this is default, when we
# rescue e) raise "Oh" is RuntimeError, which is descendant of StandardError
# if you can, be specific about errors for example Net::HTTPBadResponse
# for StandardError (like calling a method on nil value) let them to propagate
# do not: begin nil.asd rescue 1 end
rescue StandardError => e
  puts e.backtrace.reverse
  puts e.class, e.message
end
~~~

I do not know how to catch all exceptions without wrapping (maybe to use rack
app as the exception_notification does).  I will try to create new logger, it
is used for whole Rails application.
<http://stackoverflow.com/questions/6407141/how-can-i-have-ruby-logger-log-output-to-stdout-as-well-as-file>


# Mustache

[gem mustache](https://github.com/mustache/mustache) is nice to render user
templates `Hi {{name}}`, where `name` is placeholder which is inside handlebars

It is usefull to delegate fields in contacts model to some belongs_to
association, like `delegate User::FIELD_NAMES, to: :user`.


~~~
# app/services/template_render_service.rb
class TemplateRenderService
  attr_reader :template, :data

  def initialize(template, data)
    @template = template
    @data = data
  end

  def render
    m = Mustache.new
    m.raise_on_context_miss = true
    rendered = m.render(template, data)
    Result.new 'OK', rendered: rendered
  rescue Mustache::ContextMiss, Mustache::Parser::SyntaxError => e
    Error.new e.message
  end
end
~~~

usage

```
result = TemplateRenderService.new(email_body, name: '', role_name: '')
return result.message if result.success?

# in view use <%= simple_format result.message %> to convert \n to <br>
```

If handlebars contains dots than you need to nest data, like for `{{user.name}}`
data needs to be `user: { name: 'dusan' }`

# Rake tasks

You can write tasks with arguments [rails
rake](http://guides.rubyonrails.org/command_line.html#custom-rake-tasks)
~~~
# lib/tasks/db.rake
namespace :db do
  desc <<~HERE_DOC
    This task does nothing

    Example: rake db:nothing
  HERE_DOC
  task nothing: :environment do
    # environment is needed to load rails
  end
end
~~~

To read multiline description you should use `-D`
```
rake -T
rake -D
```

Instead of `:env` or `:environment` you can use other tasks on which it depends.
Also you can receive parameters into `args`

~~~
# lib/tasks/update_subdomain.rake
# instead of puts, we can use Rails.logger.info
Rails.logger = Logger.new(STDOUT)
namespace :update_subdomain do
  desc "update subdomains for my-user. default value is 'my_subdomain'"
  task :my_user, [:subdomain] => :environment do |task, args|
    # args[:subdomain] is holding your argument
    args.with_defaults subdomain: 'my_subdomain'
    user = User.find_by name: 'my-user'
    fail "Can't find user 'my-user'" unless user
    user.subdomain = args.subdomain
    user.save!
    Rails.logger.info "Updated #{user.subdomain}" || next if return_from_rake_task_now?
    Rails.logger.info 'Finished'
  end
end

# run with
rake update_subdomain:my_user
rake update_subdomain:my_user[]
rake update_subdomain:my_user[new_subdomain]
~~~

Another way of passing the arguments to rake task is using ARGV and exit
```
namespace :upload do
  desc <<~HERE_DOC
    Upload files

    cap production upload:files README.md
    cap production ROLES=db upload:files README.md
  HERE_DOC
  task :files do
    on roles(:all) do
      index = ARGV.index 'upload:files'
      ARGV[index + 1..-1].each do |file_name|
        puts "Uploading #{file_name} to #{current_path}/#{file_name}"
        upload! file_name, "#{current_path}/#{file_name}"
      end
      exit # so we do no proccess filename arguments as cap tasks
    end
  end

```

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
Rake::Task['seed:bid_types'].invoke

# call task from another task
Rake::Task['translate:copy'].invoke Rails.env
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

If you use
[sharing-cookies-across-subdomains-with-rails-3](http://makandracards.com/makandra/31381-sharing-cookies-across-subdomains-with-rails-3)
or
[what-does-rails-3-session-store-domain-all-really-do](http://stackoverflow.com/questions/4060333/what-does-rails-3-session-store-domain-all-really-do)

~~~
# config/initializers/session_store.rb
Rails.application.config.session_store :cookie_store, key: '_myapp_session',
domain: :all, expire_after: 60.minutes
~~~

If you do not use `expired_after` than it will be `-1` ie in Dev Tools ->
Application -> Cookies it will be `1969-12-31T23:59:59.000Z` 1970 - 1 second, ie
cookie will be discarded when browser is closed (on mobile when phone restarts,
since closing browser keeps current session).

This works fine for top level domains and subdomains for them.

For example if you have `domain: :all` and you sign in at `asd.local`, than you
will be signed in also on `asd.asd.local` and `asd.asd.asd.local` and so on...
Cookie will be with domain `.asd.local` for all those sites.

You need to know that some domains like `herokuapp.com` belongs to [Public
Suffix List](https://devcenter.heroku.com/articles/cookies-and-herokuapp-com) so
browser will prevent setting `domain: :all` cookie for them (because someone can
use attacker.herokuapp.com to read cookies from myapp.herokuapp.com).

Note you can not login at `in.rs` (PSL and browser will prevent cookie).

Rails cookie store knows for public suffix list, so for `trk.in.rs` it will use
`.trk.in.rs` and all others like `asd.in.local` it will use `.in.local`.

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

# you can specify different tld length
request.domain(2) = 'asd.zxc.com'
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

* labels on buttons, titles,...  should be upper case with no period: `This Job
Is Paused`
* sentences on error messages, flash messages...  should have period at the end:
`Job has been copied.`
* titleize should skip `from, and, to, your`, for example `Guide to Honeymooning
  with a Post Dated Passport`. Here is manual implementation
  ```
  def titleize(name)
    lowercase_words = %w{a an the and but or for nor of from your}
    name.split.each_with_index.map{|x, index| lowercase_words.include?(x) && index > 0 ? x : x.capitalize }.join(" ")
  end
  ```
  You can include the gem `gem 'titleize'` and than it will override
  String#titleize but we need to override how default label is generated with
  `humanize`
  https://github.com/rails/rails/blob/main/actionview/lib/action_view/helpers/tags/label.rb#L24
  `t('helpers.label.my_label')` is used as default value
  humanize is also used for `.human_attribute_name`
  https://github.com/rails/rails/blob/main/activemodel/lib/active_model/translation.rb#L64

  For forms we can override builder so required fields add star on labels
  https://en.wikipedia.org/wiki/Builder_pattern
  ```
  # config/initializers/label_builder.rb
module ActionView
  module Helpers
    module Tags # :nodoc:
      class Label < Base # :nodoc:
        class LabelBuilder # :nodoc:
          attr_reader :object

          def initialize(template_object, object_name, method_name, object, tag_value)
            @template_object = template_object
            @object_name = object_name
            @method_name = method_name
            @object = object
            @tag_value = tag_value
          end

          def translation
            # This is called only when we have not provided label attribute
            # Override defaults https://github.com/rails/rails/blob/main/actionview/lib/action_view/helpers/tags/label.rb#24
            content ||= @method_name.titleize

            # Add * for fields with presence validations
            content += ' *' if @object.class.validators_on(@method_name).any? { |v| v.kind == :presence }

            content
          end

          def to_s
            translation
          end
        end
      end
    end
  end
end

  ```

Use "Register" or "Sign up" for registrations and "Log in" and "Log out" for
sessions (since it is easy to differentiate from "sign up").

# 7 patterns

## Form Objects

form objects (query objects) for multiple in multiple out data. Even for single
table, main purpose is to add validation errors to the form. For complex
building object use service object.
More info on RegistrationForm
https://github.com/duleorlovic/rails_helpers_and_const/blob/main/app/forms/registration_form.rb

To define url for ActiveModel you need to overwrite some instance vars
https://stackoverflow.com/questions/3736759/ruby-on-rails-singular-resource-and-form-for
so better is to define url param `form_for @form, url: users_path`

## Query objects

For complex query so we can unit test them
https://thoughtbot.com/blog/a-case-for-query-objects-in-rails#a-better-query-object


## Decorators

Decorators can be used instead of callbacks. Differs from service object because
it just decorate existing model (it can be used instead of that model, usefull
for forms). Form object is like a decorator, but for multiple models.
So instead using `method_missing` you can use standard ruby `SimpleDelegator`
which delegates any method to object that was passed in initialization

~~~
# app/decorators/message_decorator.rb
class MessageDecorator < SimpleDelegator
  def message
    __getobj__
  end

  def save_and_send_notifications
    message.save && _send_notifications
  end

  def _send_notifications
    message.chat.moves.each do |move|
      next if move.user == message.user
      UserMailer.new_message(move.id, message.id).deliver_later
    end
    true
  end
end
~~~

Use in controller

~~~
  def show
    @message_decorator = MessageDecorator.new Message.new
  end

  def create_message
    @message_decorator = MessageDecorator.new Message.new _message_params
    if @message_decorator.save_and_send_notifications
      redirect_to chat_path(@chat), notice: t_crud('success_create', Message)
    else
      render :show
    end
  end

~~~

## Concerns

[DHH](https://signalvnoise.com/posts/3372-put-chubby-models-on-a-diet-with-concerns) and interesting is
[trashable](https://github.com/discourse/discourse/blob/master/app/models/concerns/trashable.rb)
[video](https://youtu.be/bHpVdOzrvkE?t=1640)
[blog](http://www.monkeyandcrow.com/blog/reading_rails_concern/)
example usage https://github.com/rails/rails/blob/87b0de450761d4404deb615c9b83307316ddb050/activesupport/lib/active_support/rescuable.rb#L11

~~~
module Someable
  extend ActiveSupport::Concern

  included do
    # this block is executed when module is included in some class
    # define methods validations, callbacks and associations here (before_ has_
    # macros) access module variables @@my_module_variable or class variable
    # self.class.class_variable_get :@@someable_value
    has_many :something_else, as: :someable
    class_attribute :tag_limit
    def increase_age(n)
      self.age + n
    end
  end

  # instance methods are defined here

  class_methods do
    # methods defined here are going to be on the class, not the instance of it
    # do not use "self." in method definition
    # you can set "@@my_module_variable" here
    # better is to set class_variable:  self.ancestors.first.class_variable_set :@@someable_value, 'some value'
    # also you can include other concerns here
    def tag_limit(value)
      klass = self.ancestors.first
      previous_columns = klass.class_variable_get(:@@monetized_columns) if klass.class_variables.include? :@@monetized_columns
      previous_columns ||= []
      klass.class_variable_set :@@monetized_columns, previous_columns + columns
      self.tag_limit_value = value
    end
  end
end
~~~

My implementation of friendly_id gem
<script src="https://gist.github.com/duleorlovic/724b8ab1eb44d7f847ee.js"></script>

# Run rails in production mode

You probably need to set up database user for production env.

Also set `config.force_ssl = false` in *config/environments/production.rb*

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

# Money

Dealing with money with <https://github.com/RubyMoney/money-rails>

Add money column is using `add_money` instead `add_column`. You can define where
to put cents and currency column
```
  add_money :invoices, :round_off, amount: { after: :round_off_applicable }, currency: { after: :round_off_cents }

  t.monetize :invoices, :round_off
```

in model
```
  monetize :round_off
```

# Carrierwave for uploading

## Store on server

~~~
cat >> Gemfile << HERE_DOC
gem 'carrierwave'
HERE_DOC
bundle
rails generate uploader Document
# we need just one field type string to store file url
rails g migration add_document_to_companies document:string
rake db:migrate
sed -i app/models/company.rb -e '/class Company/a \
  mount_uploader :document, DocumentUploader'
git add . && git commit -m "Adding carrierwave gem document uploader"
~~~

Replace `f.text_field :document` with `f.file_field :document` in your form. In
view you can use `company.document.url`.

~~~
<%# app/views/companies/_form.html.erb %>
  <%  if @company.document.present?  %>
    <%= image_tag @company.document, class: 'image-small'%>
    <%= f.check_box :remove_document %>
  <% end %>
  <%= f.file_field :document %>
  <%# keep the file on reload in case of other validation errors %>
  <%= f.hidden_field :document_cache %>

# app/controllers/companies_controller.rb
  def company_params
    params.require(:company).permit(:document, :remove_document)
  end
~~~

It is straightforward to use uploader in multiple fields. Also you can use
single table field for multiple files (field type json) but than you need
postgres database.

When you rendering json, than
[carrierwave will add nested
url](http://stackoverflow.com/questions/28184975/carrierwave-causing-json-output-to-become-nested-on-photo-key).
Solution is render json manually with `json.document_url company.document.url`
or to override uploader serilization with

~~~
# app/uploaders/document_uploader.rb
  def serializable_hash
    url
  end
~~~

Resizing is by adding mini magick and configure uploader. It works on Heroku
too. You can process files,
create new versions based on
[condition](https://github.com/carrierwaveuploader/carrierwave#conditional-versions) or process based on [condition](http://stackoverflow.com/questions/11778464/conditional-versions-process-with-carrierwave)

~~~
echo "gem 'mini_magick'" >> Gemfile
bundle

# app/uploaders/document_uploader.rb
  include CarrierWave::MiniMagick
  process resize_to_limit: [200, 300]
  process resize_to_limit: [300, 300], if: :logo?
  def logo?(picture)
    # check if we mount_uploader :logo_url or something else
    picture.file.headers.match(/logo_url/)
  end
  version :thumb do
    process resize_to_fill: [200, 300]
  end
~~~

## Store on AWS S3

For [Amazon
S3](https://github.com/carrierwaveuploader/carrierwave#using-amazon-s3) you need
to set up your AWS keys in *~/.bashrc* `export AWS_ACCESS_KEY_ID=123123` and
`export AWS_SECRET_ACCESS_KEY=123123`. Bucket should be created as standard USA
bucket.

~~~
cat >> Gemfile << HERE_DOC
gem 'fog'
HERE_DOC
cat > config/initializers/carrierwave.rb << 'HERE_DOC'
# https://github.com/jnicklas/carrierwave#using-amazon-s3
CarrierWave.configure do |config|
  config.fog_credentials = {
    :provider               => 'AWS',
    :aws_access_key_id      => Rails.application.secrets.aws_access_key_id,
    :aws_secret_access_key  => Rails.application.secrets.aws_secret_access_key,
    :region                 => Rails.application.secrets.aws_region # us-east-1
  }
  config.fog_directory  = Rails.application.secrets.aws_bucket_name
end
HERE_DOC

sed -i config/secrets.yml -e '/^test:/i \
  # aws s3\
  aws_bucket_name: <%= ENV["AWS_BUCKET_NAME"] %>\
  aws_access_key_id: <%= ENV["AWS_ACCESS_KEY_ID"] %>\
  aws_secret_access_key: <%= ENV["AWS_SECRET_ACCESS_KEY"] %>\
  # region is important for all non us-east-1 regions\
  aws_region: <%= ENV["AWS_REGION"] || "us-east-1" %>\
'

sed -i app/uploaders/document_uploader.rb -e '/storage :file/r \
  # storage :file\
  storage :fog'

git add . && git commit -m "Configure AWS S3"
~~~

## Store directly on AWS S3 and upload the key to the server

You can put the `direct_upload_form_for` on any page, let's use show:

~~~
cat >> Gemfile << HERE_DOC
# direct upload to S3
gem 'carrierwave_direct'
HERE_DOC
bundle

sed -i app/uploaders/document_uploader.rb -e '/DocumentUploader/a \
  include CarrierWaveDirect::Uploader'

sed -i app/uploaders/document_uploader.rb -e '/store_dir/c \
  # we do not use store_dir because of dirrect carrierwave\
  def store_dir_origin'

cat >> app/views/companies/show.html.erb << 'HERE_DOC'
<%= direct_upload_form_for @uploader do |f| %>
  <%= f.file_field :document %>
  <%= f.submit %>
<% end %>
HERE_DOC

sed -i app/controllers/companies_controller.rb -e '/def show/a \
   # @uploader = @company.document # do not use old since key will remain\
   @uploader = DocumentUploader.new\
   # default key is /uploads/<unique_guid>/foo.png\
   # you can change, but use ONLY ONE folder ie "1/2/a.txt" -> "2/a.txt"\
   # it always adds prefix "uploads" so it does not need to be written\
   @uploader.key = "uploads/#{@company.id}-#{request.ip}/${filename}"\
   @uploader.success_action_redirect = company_url(@company)\
   if params[:key]\
     @company.document.key = params[:key]\
     @company.save!\
     # we need to reload since old key is there\
     @company = Company.find(@company.id)\
     # or to redirect\
     redirect_to company_path(@company)\
   end\
   # you can call @company.remove_document! to remove from aws, but please\
   # reload after that with @company = Company.find(@company.id)'

sed -i config/initializers/carrierwave.rb -e '/^end/i \
  # max_file_size is not originally on carrierwave, but is added on CWDirect\
  # if file is greater than allowed than error is from Amazon EntityTooLarge\
  config.max_file_size = 20.megabytes  # defaults to 5.megabytes'
~~~

## Carrier wave in seed

You can open file and use it in seed `user.image url =
File.open(File.join(Rails.root, 'public/my_image.png'))`. But if your storage is
`fox` than you can use `user.remote_image_url_url =
'http://www.gstatic.com/webp/gallery/1.jpg'`. Note that file will be downloaded
and uploaded to your aws bucket so better is to set `storage
Rails.env.development? ? :file : :fog` and use first file.open method so it does
not need to download file.

# PDF

## Wicked PDF is using wkhtmltopdf

<https://github.com/mileszs/wicked_pdf>

Generate pdf from html files.
Just put in gemfile

~~~
cat >> Gemfile << HERE_DOC
# pdf generation from html
gem 'wicked_pdf'
gem 'wkhtmltopdf-binary'
HERE_DOC
~~~

And in your controller put the name of the downloaded file

~~~
class ThingsController < ApplicationController
  def show
    respond_to do |format|
      format.html
      format.pdf do
        render pdf: "thing-#{params[:id]}"   # Excluding ".pdf" extension.
      end
    end
  end
end
~~~

You can also set template to another file, layout to pdf and header

~~~
render template: 'periods/visits', pdf: "visits-#{@period.end_date}", layout: 'pdf', header: { right: '[page] of [topage]' }
~~~

Template

~~~
# app/views/things/show.pdf.erb
<h1>Hi</h1>
<div class="alwaysbreak"></div>
<h1>Second page</h1>
~~~

Layout

~~~
<%# app/views/layouts/pdf.pdf.erb
<!doctype html>
<html>
  <head>
    <meta charset='utf-8' />
    <%= stylesheet_link_tag wicked_pdf_asset_base64("pdf") %>
  </head>
  <body>
    <div id="content">
      <%= yield %>
    </div>
  </body>
</html>
~~~

Styles

~~~
// app/assets/stylesheets/pdf.scss
table, th, td {
  border: 1px solid black;
  border-collapse: collapse;
  padding: 5px;
  text-align: center;
}
* {
  font-size: 12px;
}
div.alwaysbreak { page-break-before: always; }
~~~

Since we did not include this style in asset pipeline, we need to precompile it:

~~~
# config/initializers/assets.rb
Rails.application.config.assets.precompile += ['pdf.css']
~~~

If you want to use existing html template than use param `render pdf: 'name',
template: 'show.html'` and create `app/views/layouts/pdf.html.erb` and use
helper classes to show hide content:

~~~
.pdf-hidden {
  display: none;
}
~~~

~~~
.html-hidden {
  display: none;
}
~~~

Spacing issue on old version of wkhtmltopdf and only Arial font
https://github.com/wkhtmltopdf/wkhtmltopdf/issues/3147

PDFTK pdf toolkit can be used to merge two pdf https://rubygems.org/gems/pdf-toolkit/versions/1.1.0 `gem 'pdf-toolkit'`

`gem 'pdf-reader'` can be used to read metadata of pdf

https://github.com/boazsegev/combine_pdf can be used to merge pdfs in ruby

PDFTRON is commercial tool with a lot of examples https://www.pdftron.com/documentation/samples/rb/PDFRedactTest


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

* when you need a image logo in pdf than you can use `fit` option https://www.rubydoc.info/gems/prawn/0.12.0/Prawn/Images

  ~~~
    require 'open-uri'
    uri_escaped = URI.escape "http:#{tax_holder.logo_url}"
    image open(uri_escaped), fit: [column_2_width, 89], position: :right
  ~~~

## PDF reader

https://github.com/yob/pdf-reader

```
reader = PDF::Reader.new("somefile.pdf")
# this will work only for computer generated pdf (not for scanned documents)
reader.pages.first.text
reader.pages.first.fonts.first
```
For images https://github.com/yob/pdf-reader/blob/master/examples/extract_images.rb

This is used for test pdf rendering. There is also
https://github.com/prawnpdf/pdf-inspector but I do not see any advantage of pdf
inspector (which uses pdf reader).

```
subscriber_invoice = create :subscriber_invoice, invoice_header_text: 'My Header'
pdf = SubscriberInvoicePdf.new subscriber_invoice
reader = PDF::Reader.new(StringIO.new(pdf.render))
expect(reader.pages.first.text).to include 'My Header'
```

## Docx MS word

https://github.com/urvin-compliance/caracal

## Gems

* https://awesome-ruby.com/
* recurring events https://github.com/rossta/montrose
  ```
  Montrose.daily(total: 10, starts: today, until: ends, between: starts..ends,
                 month: :january, interval: 2)
  Montrolse.weekly(on: [:monday, :friday])
  Montrose.every(:week, until:ends)
  Montrose.every(2.weeks, on: [:monday, :friday])

  # enumerator
  r.events.take(10)
  ```

  tickle recurring events https://github.com/yb66/tickle


* rubyzip to create zip files https://github.com/rubyzip/rubyzip
  ```
  gem 'rubyzip', require: 'zip'
  ```
  In script
  ```
  require 'zip'
  file_name = '2019-12-24 12:49:14 +0000_Generic-Generic-Name_Switch_Letter.pdf'
  system "echo 123 > #{file_name}"
  Zip::File.open('a.zip', Zip::File::CREATE) { |zipfile| zipfile.add file_name, './' + file_name }

  ```
  Test like in https://github.com/rubyzip/rubyzip/blob/master/test/file_test.rb
  ```
    zf_read = ::Zip::File.new result.data[:zipfile_name]
    assert_equal 2, zf_read.entries.length
  ```
* ms word documents create from scratch
  https://github.com/urvin-compliance/caracal or from html
  https://github.com/karnov/htmltoword

# Style guide


* do not reference model class directly from a view (how to render counts than
?)
* do not use SQL outside of models
* use `touch: true` on `belongs_to` associations
* use `ENV.fetch` instead of `ENV[]` so it raises exception when env variable
does not exists

My style in rails models use following order to best practice

1. `include` and `extend` other modules, `devise`, `has_paper_trail`
1. `FIELDS = %i[name].freeze` and other constants
1. `attr_accessor` or `serialize :col, Hash`
1. `belongs_to :workflow` associations with plugins `acts_as_list scope:
1  [:workflow_id]`
1.  `has_many :users` associations
1. enums `enum status: %i[draft accepted]`
1. validations `validates :name, presence: true`
1. validate declarations `validate :_check_nested_resource`
1. callbacks declarations `before_validation :_default_values_on_create, on: :create`
1. scopes `scope :by_status_param, ->(status_argument) { where status: status_argument }`
1. class methods `def self.find_first_unpublished`
1. instance methods `def full_name`
1. validate definitions `def _check_nested_resource`
1. callbacks definitions `def _default_values_on_create`

# Rubocop

My preferred configuration is
https://github.com/duleorlovic/config/blob/master/.rubocop.yml

In old project you can generate `.rubocop.yml` file that will ignore all
offenses, than you can gradually fix one by one.

~~~
rubocop --auto-gen-config
# this will generate .rubocop_todo.yml which you can include
cat >> .rubocop.yml << HERE_DOC
inherit_from: .rubocop_todo.yml
HERE_DOC
~~~

To disable specific folder you can use Exclude
```
# config, tasks and test for setup data could be very long
Metrics/BlockLength:
  Exclude:
    - 'lib/**/*'
    - 'test/**/*'
    - 'config/**/*'
```

To auto fix use `--auto-correct` or shorthand `-a`
```
rubocop -a
# to run only specific cop
rubocop -a --only 'Rails/HttpPositionalArguments'
```

# Geocoder

You can use plain postgresql function
```
Location.order(Arel.sql "POINT(latitude, longitude) <-> POINT(#{current_location.latitude},#{current_location.longitude})")
```
or you can use gem geocoder.

If you need to search with association, for example [User has many
locations](https://stackoverflow.com/questions/14188568/using-geocoder-on-a-child-association-how-to-find-all-parents-in-a-given-locati)
you can with joins to geocoded model on which `near` is defined.

~~~
near = Location.near('Paris, France')
users = User.joins(:locations).merge(near)
~~~

also if you need to get associated objects you can (location has many views)

~~~
near = Location.near([latitude, longitude], MAX_USER_DISTANCE_MILES)
views.
  joins(:location).
  select("views.*"). # somehow we need this so we got views, instead of users
  merge(near).
  includes(:sport). # so we can get sport.name without N+1
  where(sport: sport) # conditional on view
~~~


# Autoloading

<https://www.bigbinary.com/videos/learn-ruby-on-rails/how-autoloading-works-in-rails>
In development rails uses Constant Missing hooks to auto load new files from
`app/controllers helpers mailers and models`
If you need to load files from lib, you can `config.autoload_paths +=
%W(#{config.root}/lib)`. Rails look for file name that is snake case of constant
name, for example `SeniorDeveloper` should be defined in `senior_developer.rb`.
You can `require_dependency 'not_conventional_file_name'`

There is ruby `autoload :Jeep, 'Jeep'` which is usefull since it will not
`require 'jeep'` if `Jeep` is not used. If we use `Jeep` in a file, than it will
required.


# Generators

Overriding scaffold https://guides.rubyonrails.org/generators.html Rails
original templates are here
https://github.com/rails/rails/tree/master/railties/lib/rails/generators/erb/scaffold/templates
so to override you need to create `lib/templates/erb/scaffold/index.html.erb.tt`
 ```
 mkdir -p lib/templates/erb/scaffold
 vi lib/templates/erb/scaffold/index.html.erb.tt
 # use <%%= when you need to output <%=
 spring stop
 rails g scaffold post title
 ```

To overwrite all generators you can search for generator name. Since some
generators can be from different gems, you should download whole rails repo
```
git clone git@github.com:rails/rails.git
cd rails
find . -name active_record.rb
find . -name test_unit.rb
find . -name scaffold_controller
find . -name scaffold_generator.rb

# mkdir path (gem_name)/lib/rails/generators/THIS_PATH/templates
mkdir lib/templates/THIS_PATH -p

# for example
lib/templates/active_record/model/model.rb.tt
lib/templates/erb/scaffold/index.html.erb.tt
lib/templates/erb/scaffold/show.html.erb.tt
lib/templates/rails/scaffold_controller/controller.rb.tt
```

If you need to overwrite ruby code (not template) you need to copy whole file
https://stackoverflow.com/questions/32384713/override-rails-scaffold-generator
```
mkdir -p lib/rails/generators/erb/scaffold/

# lib/rails/generators/erb/scaffold/scaffold_generator.rb
# this is a copy from https://github.com/rails/rails/blob/master/railties/lib/rails/generators/erb/scaffold/scaffold_generator.rb
```

You can repeat process with
```
git clean . -f && rails g scaffold posts name
```

In templates I can use:
```
index_helper # posts
edit_<%= singular_route_name %>_path # edit_post_path
attributes # object for 'name', 'id', 'email'
attribute.field_type # :text_field
attribute.column_name # :name
model_resource_name # post
```

Use minus to remove new line (do not generate empty line) and also start loops
on beggning of the line (do not generate spaces before the loop)
```
<% default_method = attributes_names.include?('name') ? 'name' : attributes_names.first -%>
<% attributes.each do |attritube| -%>
<% end -%>
```

https://rdoc.info/github/erikhuda/thor/master/Thor/Actions.html

We can manually create generator
```
# lib/generators/initializer_generator.rb
class InitializerGenerator < Rails::Generators::Base
  def create_initializer_file
    create_file 'config/initializers/initializer.rb', '# AA'
  end
end
```
or we can generate generator `rails g generator initializer` which will depend
on `Rails::Generators::NamedBase` so it expects parameter, like model name.

* `file_name` (post), `singular_name` (post), `plural_name` (posts), `class_name` (Post), `table_name` (posts), `singular_table_name` (post)
* arguments using
 https://www.rubydoc.info/github/erikhuda/thor/master/Thor/Base/ClassMethods#class_option-instance_method
 `class_option :scope, type: :string, default: 'read_products'`
* `hook_for :test_framework, as: :scaffold` will use scaffold test framework
* `source_paths [__dir__]` is used so you can use relative path to templates
* `template 'my.rb', "destination/#{file_name}.rb"`

# Thor

https://github.com/erikhuda/thor/wiki

```
# cli.rb
#!/usr/bin/env ruby
require 'thor'
class MyCLI < Thor
  include Thor::Actions

  def self.source_root
    File.dirname(__FILE__)
  end

  desc 'cao NAME', 'primer NAME'
  option :from, required: true
  option :yell, type: :boolean
  def cao(name)
    output = []
    output << "from: #{options[:from]}" if options[:from]
    output << "Cao #{name}"
    output = output.join("\n")
    if options[:yell]
      puts output.upcase
    else
      puts output
    end
  end

  desc 'g', 'generisi a'
  def g
    template 'a'
  end
end

MyCLI.start(ARGV)
```
You can run with `ruby ./cli.rb cao Dule --from Mile --yell`

# Chamber

You can use `chamber` gem
https://github.com/thekompanee/chamber/wiki/Basic-Usage for secure keys, after
adding to Gemfile and `chamber init`
https://github.com/thekompanee/chamber/wiki/Installation

```
You can find more information about namespace keys here:
  * https://github.com/thekompanee/chamber/wiki/Namespaced-Key-Pairs
--------------------------------------------------------------------------------
                                Your Encrypted Keys
You can send your team members any of the file(s) located at:
  * .chamber.enc
and not have to worry about sending them via a secure medium, however do
not send the passphrase along with it.  Give it to your team members in
person.
You can learn more about encrypted keys here:
  * https://github.com/thekompanee/chamber/wiki/Keypair-Encryption#the-encrypted-private-key
--------------------------------------------------------------------------------
                                Your Key Passphrases
The passphrases for your encrypted private key(s) are stored in the
following locations:
  * .chamber.enc.pass
In order for them to decrypt it (for use with Chamber), they can use something
like the following (swapping out the actual key filenames if necessary):

$ cp .chamber.enc .chamber.pem
$ ssh-keygen -p -f .chamber.pem
```

Those files are generated (only `.chamber.pub.pem` is not git ignored)
```
.chamber.enc  .chamber.enc.pass  .chamber.pem  .chamber.pub.pem
```

Inside `config/settings.yml` you can use plain yml, but if you prepend with
`_secure` that value will be encrypted.

```
# settings.yml
_secure_key:                development_value
```

than when you run `chamber secure` it will replace all `_secure_` values with
```
_secure_key:                ASDWQWEASD...
```
To show all keys you can use `chamber show`.
In console
```
require 'chamber'
Chamber.env.key
```

When deploy, you need to set `export RAILS_ENV=production` and provide
`.chamber.pem` file.

If the output of `chamber show` is `{}` maybe you need `bin/chamber` so it loads
that instead gems version.


# Tips

* I got an error `ActionDispatch::Cookies::CookieOverflow
  (ActionDispatch::Cookies::CookieOverflow):` when there is `flash[:notice]`
  that is greater than certain value for example: `flash.now[:notice] =
  '1'*1972`, `flash.now[:alert] = '1'*1_974` and `flash[:notice] = '1'*1980`,
  `flash[:alert] = '1'*1981`
* There is a new line in `Base64.encode64 string` so use `Base64.strict_encode64
  string` https://stackoverflow.com/questions/2620975/strange-n-in-base64-encoded-string-in-ruby
* parse url to get where user come from `URI.parse(request.referrer).host` or
  just `URI(request.referrer).host`. You can parse uri query with CGI (values
  are arrays) and Rack utils parse query (values are strings)

  ~~~
  (byebug) CGI::parse uri.query
  {"dst"=>["http://www.trk.in.rs/?a=1"]}
  (byebug) Rack::Utils.parse_query uri.query
  {"dst"=>"http://www.trk.in.rs/?a=1"}
  ~~~

* `spring stop` in many cases:
  * when you export some ENV and use them in `config/secrets.yml` but can't see
    in `rails c`
  * when you put `byebug` in some of the gem source
* load databe from `db/schema.rb` (instead of `rake db:migrate`) is with `rake
  db:schema:load`.
* run at port 80 `sudo service apache2 stop` and `rvmsudo rails s -p 80`. Note
  that db user will to *root* instead of *orlovic* so you need to hardcode it in
  *config/database.yml*. Rember to `-b 0.0.0.0` if you access from outside.
  You can make default option to bind on localhost 0.0.0.0
  ~~~
  # config/boot.rb
  # Set up gems listed in the Gemfile.
  ENV['BUNDLE_GEMFILE'] ||= File.expand_path('../../Gemfile', __FILE__)

  require 'bundler/setup' if File.exists?(ENV['BUNDLE_GEMFILE'])

  # https://fullstacknotes.com/make-rails-4-2-listen-to-all-interface/
  require 'rubygems'
  require 'rails/commands/server'

  module Rails
    class Server
      alias :default_options_bk :default_options
      def default_options
        default_options_bk.merge!(Host: '0.0.0.0')
      end
    end
  end
  ~~~
  When you are using `local.trk.in.rs` you can set local ip *192.168.0.4* as
  Redirection (301 or 302) (but not as Forwarding since that does not work for
  some api requests from android).
* `Rails.logger` is stdout and every controller, model and view has `logger`
  method which you can use like `logger.debug my_var`
* if you want to join some fields with `,` but do not know if they all exists,
  you can `[phone1, phone2].keep_if(&:present?).join(', ')`
* use include for n+1 query (bullet) and joins when you don't need to reference
  associated models. Both are using INNER JOIN
  * `Comment.all(include: :user, conditions: { users: { admin: true}})` will
    load also the user model
* [N+1](http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations)
  problem is solved with `includes(:associated_table)` This sometimes generate separate query and sometimes it is using left joins
  https://makandracards.com/makandra/1148-why-preloading-associations-randomly-uses-joined-tables-or-multiple-queries
  `joins(:associated_table)` is INNER JOIN and do not load the
  models. Works also for nested joins `includes(jobs: :user)` or for multiple
  you can use array `includes(jobs: [:user, :company])`.
  * Inner `joins` differs if you have has_many or belongs_to association. For
  `has_many :associated_table` `joins` will return multiple values
  because of multi values of :associated_table per item, or none is
  :associated_table does not exists for current item. For `belongs_to :a` it
  will return single if data exists, or nothing will be returned, if belongs_to
  is empty, so better is to use left join. (expecially when you search by
  description OR teacher name, you should not exclude books without teacher)
  * custom joins if you want to use left inner join
    https://thoughtbot.com/upcase/videos/advanced-querying-custom-joins
    ```
    Person.joins(<<-SQL).
      LEFT JOIN people managers
        ON managers.id = people.manager_id
    SQL
    where(managers: { id: Person.find_by!(name: 'Eve') })
    ```
    custom left joins are better than includes because you are explicit about
    joining and you do not eager load (do not create AR objects) if not need.
    In Rails 5 there is a method `left_outer_joins`
    https://edgeguides.rubyonrails.org/active_record_querying.html#left-outer-joins
    when you want to perform custom sql you can use
    ```
    task.projects
        .select('projects.*')
         .select(<<-SQL)
           activities.name AS last_task_activity_name,
           tasks.updated_at AS last_task_updated_at
         SQL
         .joins(<<-SQL)
           INNER JOIN tasks ON tasks.project_id = projects.id
           INNER JOIN activities
           INNER JOIN (
             SELECT project_id, MAX(id) as max_id
             FROM tasks
             GROUP BY project_id
           ) last_task
           ON last_task.project_id = projects.id AND
             last_task.max_id = tasks.id AND
             tasks.activity_id = activities.id
         SQL
    ```
  * `includes` will return the same number of items, with association object
  loaded in memory. eager load `includes` can be defined in association
  definition `has_many :comments, -> { includes :author }` but this is bad since
  it will **always** load two tables.  Better is to make a query
  `Post.includes(:comments).all` or for instance
  `@post.comments.includes(:author)`. You need to eager load before calling
  `.all` or `.each`. Do not need eager load belongs_to association.
You can use procs to further filter relations. Note that in case of
`Customer.joins(:radaccts)` customer argument is not a Customer but a relation
so this filter only works for `customer.radaccts`
~~~
has_many   :radaccts, -> (customer)  { where('radacct.location_id = ?', customer.location_id) if customer.class == Customer }, primary_key: :username, foreign_key: :username, inverse_of: :customer
~~~

You can assign attributes with `user.assign_attributes user_params`.
In rails `belongs_to .. foreign_key` https://guides.rubyonrails.org/v5.2/association_basics.html#options-for-belongs-to-foreign-key
is used is you have different name of the column than name_of_association_id
Sometime you extend model `class ResellerPackage < Package` se we need to
explicitly define column names.
`has_many .. inverse_of` is used when you `accepts_nested_attributes_for` and
you want to access them in `user.reseller_packages`
Of if you have two relation to the same class
```
# Profile
belongs_to :user
belongs_to :created_by, class_name: 'User'

# User
has_one :profile

user = User.first
user.profile.created_by # it could be User.second, but
user.profile.created_by.profile # will return profile of the first
```

You can specify conditions in relation
```
class Company < ApplicationRecord
  has_many :company_users, dependent: :destroy
  # has_many :users, through: :company_users # do not use this since company
  # users can be disabled, so better is to use active_users
  has_many :active_users, -> { where(company_users: { status: :active }) },
           through: :company_users, source: :user
```
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
since it remove my custom selected data like calculated columns, so instead
`references` I use `joins('LEFT OUTER JOIN sports ON sports.id =
users.sport_id')` and `distinct`.  Distinct is needed to remove duplicated since
inner join with has_many relation (also habtm) returns multiple results (single
is only for belongs_to relation).
Here is example of how includes/references is similar to left join but includes
is using all `:projects` columns
~~~
all = Model
  .select(%(
    users.id,
    COUNT(projects.id) as projects_count
  ))
  .includes(:projects)
  .references(:projects)
  .group('users.id')
all.last.projects_count # 123

all = Model
  .select(%(
    users.id,
    COUNT(projects.id) as projects_count
  ))
  .left_outer_joins(:posts)
  .group('users.id')
  all.last.projects_count # here it works
~~~
You need to use `group` and then there is a problem if you want to use
`all.count` on grouped relation:
```
ActiveRecord::StatementInvalid (PG::SyntaxError: ERROR:  syntax error at or near "AS")
LINE 1: SELECT COUNT(users.*, COUNT(posts.id) AS posts_count) AS cou...
```
which is solved using `all_grouped.returns_count_sum.count`
```
User.select('users.*, COUNT(posts.id) AS posts_count').left_outer_joins(:posts).group('users.id').returns_count_sum.count
```
So instead of COUNT JOIN GROUP you can use subquery (double select wrapped
select)
```
User.select('users.*, (SELECT COUNT(*) FROM posts WHERE users.id = posts.user_id) AS posts_count')
```

When you need to apply filtering WHERE for calculated columns, since in where
you can not use aliases nor aggregate functions, you can use three approaches:
* use database view
* if you are using join and group than you can apply `HAVING` with repeated cond
```
User.select('users.*, COUNT(posts.id) AS posts_count').left_outer_joins(:posts).group('users.id').having('COUNT(posts.id) > 1')
```
* if you use GROUP_CONCAT `posts.body` (postgres is string_agg) (instead of
  COUNT) than you can still use WHERE for column `posts.body` (it is existing
  column, not new custom calculated column) so it works out of the box
* if you are using subquery than you can repeat it in where condition (note that
  we are not using aggregate in where condition, we are using subquery)
```
User.select('users.*, (SELECT COUNT(*) FROM posts WHERE users.id = posts.user_id) AS posts_count').where('(SELECT COUNT(*) FROM posts WHERE users.id = posts.user_id) > 1')
# you can also use subquery in join group relation
User.select('users.*, COUNT(posts.id) AS posts_count').left_outer_joins(:posts).group('users.id').where('(SELECT COUNT(*) FROM posts WHERE users.id = posts.user_id) > 1')
```

To select all that does not have nested objects
```
leagues.left_outer_joins(:matches).where.not("matches.id": nil).group("leagues.id")
```

Rails 5 has method
[left_outer_joins](http://edgeguides.rubyonrails.org/active_record_querying.html#left-outer-joins)
* when you need to eager load for a single object (show action) than you can
simply repeat rails default before action `set_post` with: `@post =
Post.includes(:comments).find params[:id]`


* `some_2d_array.each_with_index do |(col1, col2),index|` when you need
  [decomposition
  array](http://docs.ruby-lang.org/en/2.1.0/syntax/assignment_rdoc.html#label-Array+Decomposition)
  to some variables, you can use parenthesis.
* long output of a command (for example segmentation fault) can be catched with
  `rails s 2>&1 | less -R`
* use different layout and template based on different params is easy using
locales [look for adminlte example]( {{ site.baseurl }}
{% post_url 2016-10-28-adminlte-free-template-on-rails %})

* title and meta tags for the template can be added using helper functions

~~~
# app/helpers/page_helper.rb
module PagesHelper
  def login_layout(login_title = nil)
    @login_title = login_title
    @login_layout = true
  end

  def login_layout?
    @login_layout
  end

  def login_title
    @login_title
  end

  def title(name)
    @title = name
  end

  def fetch_title
    @title || fetch_breadcrumb_list.keys.last
  end

  def page_description(description)
    content_for(:page_description) { description.html_safe }
  end

  def breadcrumb(list)
    @breadcrumb = list
  end

  def fetch_breadcrumb_list
    @breadcrumb || {}
  end
end

# app/views/layouts/_breadcrumb.html.erb
<nav aria-label="breadcrumb">
  <ol class="breadcrumb">
    <% if fetch_breadcrumb_list.blank? %>
      <li class='breadcrumb-item active' aria-current='page'>
        <i class="fa fa-dashboard"></i>
        <%= t('dashboard') %>
      </li>
    <% else %>
      <li class='breadcrumb-item'>
        <%= link_to dashboard_path do %>
          <i class="fa fa-dashboard"></i>
          <%= t('dashboard') %>
        <% end %>
      </li>
    <% end %>
    <% fetch_breadcrumb_list.each_with_index do |(text, link), i| %>
      <% last_item = i == fetch_breadcrumb_list.length - 1 %>
      <li class="breadcrumb-item <%= 'active' if last_item %>" <%= 'aria-current="page"' if last_item %>>
        <% if link.present? %>
          <%= link_to link do %>
            <%= text %>
          <% end %>
        <% else %>
          <%= text %>
        <% end %>
      </li>
    <% end %>
  </ol>
</nav>

# app/views/customers/index.html
<%
  page_title "Customers List"
  case params[:non_table_filter]
  when "registered_today"
    page_description "Registered Today"
  when "registered_this_month"
    page_description "Registered This Month"
  when "renewed_in_advance"
    page_description "Renewed In Advance"
  end
  breadcrumb "Customers": nil
%>
~~~

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
  returns hash with string keys so if you merge something you should use string
  `chat.as_json(except: [:id]).merge("sport_id" => view.sport.id)`. `to_json`
  returns string, so `as_json` it much better.

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
  for `User.all`. Notice that if you use
  `User.where(params.slice(:mobile)).first_or_initialize` than it is a problem
  when `params[:mobile]` does not exists, and first User fill be used (not new
  user)
* for `gon` gem you should `include_gon` before other javascript files. Note
  that `if gon` will still raise error if `gon` is not called in controller (so
  `gon` is undefined)

* access helpers outside of a view
  * in rails console you can use `helper.number_to_currency 10`
  * if it is standard base helper, than just call it from ActionController,
    ActionView instance or Rails application routes.

  ~~~
  ActionController::Base.helpers.pluralize(count, 'mystring')
  ActionController::Base.helpers.strip_tags request.body # html to text
  ActionController::Base.helpers.link_to 'name', link
  ActionController::Base.helpers.j "can't be blank" # can\'t be blank
  ActionController::Base.helpers.humanized_money_with_symbol amount
  ActionController::Base.helpers.time_ago_in_words Time.zone.now
  ActionController::Base.helpers.distance_of_time_in_words 1.day
  ActionController::Base.helpers.distance_of_time_in_words i.completed_at -
  i.started_at, include_seconds: true
  ActionController::Base.helpers.number_with_delimiter 1234

  # no need to use ActionView::Base.new but just for reference
  # ActionView::Base.new.number_to_human 123123 # 123 Thousand
  # ActionView::Base.new.number_to_human_size 123123 # 120 KB

  # route helper url helper
  Rails.application.routes.url_helpers.jobs_url
  # note that it is different than `jobs_url` in view since in view it uses
  # request to determine host, so maybe it is better to use

  ActionController::Base.helpers.asset_path('icon.png')

  ERB::Util.html_escape t('errors.messages.blank') # can&#39;t be blank

  "longs string".truncate 20
  ~~~

  * if it your custom helper you can call from *ApplicationController*

  ~~~
  ApplicationController.helpers.my_custom_helper
  ~~~

  If you want to use it inside controller or any other class you can include

  ~~~
  include MyHelper
  ~~~

  But best options in controllers in Rails 5 is to use `helpers`

  ~~~
  class MyController < ApplicationController
    def index
      alert helpers.titleize_message
    end
  end
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
    delegate :form_tag, :text_field_tag, to: 'ActionController::Base.helpers'
  ~~~

* do not use `form_tag path` without block, since chrome will autoclose tags,
but IE10 wont.
* do not name your model with `Template` since there is module `Template`
  somewhere.
* request object contains a lot of data:
  * `request.xhr?` is it ajax
  * `request.ip` `request.referrer` `request.remote_ip`
  * `request.user_agent` `env["HTTP_USER_AGENT"]`

* if you want to add flash_alert for all `flash.now[:alert]='message'` you can
  use

  ~~~
  # app/controllers/application_controller.rb
  after_action :check_flash_message

  # rubocop:disable Metrics/AbcSize
  def check_flash_message
    return unless request.xhr? && request.format.js?
    response.body += "flash_alert('#{view_context.j flash.now[:alert]}');" if flash.now[:alert].present?
    response.body += "flash_notice('#{view_context.j flash.now[:notice]}');" if flash.now[:notice].present?
  end
  # rubocop:enable Metrics/AbcSize
  ~~~

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

  * you can catch all error using routes https://coderwall.com/p/w3ghqq/rails-3-2-error-handling-with-exceptions_app We can rescue in application controller
  but not for `ActionController::RoutingError` since that is done before rails
  so you need to use `config.exceptions_app`, for example:

  ~~~
  # app/controllers/application_controller.rb
    rescue_from ActiveRecord::RecordNotFound, ActionController::UnknownController, ::AbstractController::ActionNotFound, ActionController::RoutingError do |exception|
      redirect_to error_path title: 'Record Not Found', description: 'Could not find resource: ' + request.url
    end

  # config/application.rb
  class Application < Rails::Application
    config.exceptions_app = self.routes
  end

  # config/routes.rb
  MyApp::Application.routes.draw do
    get 'error', controller: 'home'
    get '/404', controller: 'home', action: 'routing_error'
    get '/500', controller: 'home', action: 'internal_server_error'
  end

  # app/controllers/home_controller.rb
  class HomeController < ApplicationController
    def error
      @title = params[:title]
      @description = params[:description]
      render :error, formats: [:html]
    end

    def routing_error
      @title = 'Page Not Found'
      @description = 'Cound not find this page'
      render :error, formats: [:html]
    end

    def internal_server_error
      @title = 'Internal Server Error'
      @description = 'There was an error and administrators were notified'
      render :error, formats: [:html]
    end
  end
  ~~~

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
  add a line in model
  ~~~
  acts_as_list scope: :post, column: :priority
  default_scope { order('position ASC') }
  ~~~
  use that priority in associations `has_many :comments, -> { order priority:
  :asc }, dependent: :destroy`
  Note that this association with order will prevent
  accepts_nested_attributes_for to work properly (error is post must exists)
  Position is starting from 1, 2...
  To perform bulk update for ordering, use form to pass ids

  ~~~
  def sort
    @job.questions.each do |question|
      # check if this question is in params, since there could be several request of sorting when we save them all
      if params['question'].include? question.id.to_s
        question.position = params['question'].index(question.id.to_s) + 1
        question.save!
      end
    end
  end
  ~~~

* `enum status: [:paused]` should be type integer, or it will return nil. Use
synonim for new, like unproccessed, draft. You can access values with symbols
`:draft` and outside of class with `Class.statuses`. It is better to use string
column instead of integer since it is more readable in db, and you can change
the order. In this case define enum as a hash
  ```
  enum status: {
    paused: 'paused',
  }
  # or if you are using const
  enum kind: Constant.SMS_TEMPLATES.transform_keys(&:downcase).transform_values(&:to_s)
  # or if you are using array of symbols
  enum status: %i[active admin inactive].each_with_object({}) { |k, o| o[k] = k.to_s }
  # or simpler using helper convert_to_hash defined on ApplicationRecord
  enum status: convert_to_hash(%w[active admin inactive])


  # app/models/application_record.rb
  # enum status: convert_to_hash(%w[active admin inactive])
  def self.convert_to_hash(array)
    array.each_with_object({}) { |k, o| o[k.to_s] = k.to_s }
  end
  ```

* params usually need to be striped, so you can use this code to get rid of all
  unnecessary spaces (not that we create new object that is returned, params
  stays unschanged)

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

  also if you want to replace '\n' new lines in text area (`f.text_area`) with
  <br> so it looks the same when displaying it . Example is with nested associated elements:

  ~~~
  params[:contact] = params[:contact].each_with_object({}) { |(k,v),o| o[k] = v.each_with_object({}) { |(k1,v1),o1| o1[k1] = (v1.class == String ? v1.gsub(/\n/,'<br>'): v1) } }
  ~~~
  Alternativelly you can keep `\n` and use `<%= simple_format @contact.text %>`
  It will replace `\n` with `<br>` and two or more `\n\n` with `<p>`... also you
  can insert any html tags (if sanitize is true if will convert `<form>` and
  `<script>` tag into regular text and disable `on=` and `href="javascript`
  attributes.
  https://apidock.com/rails/ActionView/Helpers/TextHelper/simple_format

  If you want to strip params globaly (for every non get request) you can use:

  ~~~
  before_action :strip_params

  def strip_params
    return if request.get?

    params
      .values
      .select { |v| [ActionController::Parameters, ActiveSupport::HashWithIndifferentAccess].include? v.class }
      .each do |item_parameters|
      item_parameters.each do |k, v|
        next unless v.is_a? String

        item_parameters[k] = v.split.join(' ')
      end
    end
  end
  ~~~

* if you notice in logs double requests (messages are double rendered) it is
  probably that you use `gem 'rails_12factor'` which should be only on
  production `group: :production`
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
  so check if return value could be `false` and make sure before filters (like
  before_create) always return not false value (you can return `true` or `nil`).
  I noticed that if you raise validation exception in `before_` callbacks than
  exception will be ignored, but `before_` block will return false
  In Rails 5 return value is not important, and if we want to halt before block
  we need to call `throw(:abort)`
  https://blog.bigbinary.com/2016/02/13/rails-5-does-not-halt-callback-chain-when-false-is-returned.html
* callbacks are defined executed in the order they are defined, or you can use
  `:prepend` option
* in callbacks should only be some value format correction or other simple task.
In callback should not be Mailer to send email or other external task since it
will be triggered even in you test when you prepare some data, or when creating
seed data
* in action pack (action controller) if you `render` something in before_filter
  than execution will be halted (return value is not important for
  ActionController callbacks)
* subclasses can use `skip_callback` if you want to skip callback
* do not use callbacks, expecially after_callbacks since it ties with save, for
  example, you should be able to repeat some actions (sending email, charging)
  without saving. Use delegator instead

  ~~~
  class FacebookNotifyComment < SimpleDelegator
    def initialize(comment)
      super(comment)
    end

    def save
      if __getobj__save
        post_to_wall
      end
    end

    private

    def post_to_wall
      Facebook.post(title: title)
    end
  end

  class CommentsController < ApplicationController
    def create
      @comment = FacebookNotifyComment.new(Comment.new(comments_params))
      if @comment.save
        redirect_to blog_path, notice: "Comment was posted'
      else
       render 'new'
      end
    end
  end
  ~~~


* to start new project from specific rails, run `rails _4.2.7.1_ new myapp` .
  `gem list | grep rails` can show you installed versions
* `"data-disable-with": "<i class='fa fa-spinner fa-spin'></i> #{name}"` works
on any element except `f.submit` since it is `input` element and you will see
`<i>` tags... better is to use `f.button` but than you lose `params[:commit]`
which is included in `f.submit`. So solution is to include `hidden_field_tag
:commit, "Some label"`

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
    @item = Item.new example_params if Rails.env.development? && params[:example]
    end
  end

  private

  def example_params
    i = 1
    while Item.find_by name: "Example Name #{i}"
      i += 1
    end
    self.name = "Example Name #{i}"
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

* you can iterate in groups batches `<% company.jobs.group_by(&:user).each do
  |user, jobs| %>`
* `Person.find_in_batches do |people|` will iterate but load only 1000 per run.
  Or shorter is `Person.find_each do |person|`
* to count by grouping you can group_by specific column
`User.group(:company_id).count.values.max`
if you want to order in sql, than you need to name the count and order by that
name (note that we need to use string not symbol for `'count_id'`:
`User.group(:company_id).order('count_id desc').count(:id)`.
you can group by joined table, for example  https://thoughtbot.com/upcase/videos/advanced-querying-aggregations
```
Person.joins(:employees).group('people.name').count('employees_people.id')
```
* `User.all(joins: :comments, select: "users.*, count(comments.id) as
comments_count", group: "users.id")` you can provide string to select and output
just specific value like comments_count.
* you can use `.having` with rails just write before `.count`, like
`c=Item.reorder("").group([:url, :title, :feed_id]).having('COUNT(*) >
1').count(:id)`


Maybe find can help
~~~
Mail.find(
    :all, 
    :select => 'count(*) count, country', 
    :group => 'country', 
    :conditions => ['validated = ?', 't' ], 
    :order => 'count DESC',
    :limit => 5)
~~~

* to run ruby script with rails you can include

~~~
require File.expand_path('/home/orlovic/rails/myApp/config/environment', __FILE__)
puts User.all
~~~

* even when you are using rails data attritubes you have to use `to_json` (as
you need in html).
And if you use jQuery data method than you do not need to use JSON.parse.

  ~~~
  <%= f.text_field :name, "data-predefined-range": {a: 3} %>
  <input data-predefined-rage="<%= {a: 3}.to_json %>">
  <%= f.select :current_city, options_from_collection_for_select(City.all, :uuid, :name), {}, 'data-select-target': '#current-location', 'data-select-options': Location.all_by_city_uuid.to_json %>

  <script>
  range = JSON.parse(input.dataset.predefinedRange);
  range = $(input).data("predefinedRage");

  <model>
  # return hash {city_uuid: [{id: l1.id, name: l1.name}, ...], ...}
  def self.all_by_city_uuid
    Location.all.each_with_object({}) do |location, result|
      if result[location.city_id].present?
        result[location.city_id].append(id: location.id, name: location.name)
      else
        result[location.city_id] = [{ id: location.id, name: location.name }]
      end
    end
  end
  ~~~

* security tips
<https://github.com/brunofacca/zen-rails-security-checklist>

* if you want to use `key.to_sym` for all keys you can use `hash.symbolize_keys`
but if you want that recursively for nested hashes as well you can use
`params[:some_pararam].deep_symbolize_keys`

* memoization is nice if you have network calls or sql calls or db query

~~~
def MyClass
  def self.issues
    @issues ||= Brand.find(1).issues.inject({}) do |hash, issue|
      hash[issue.title] = issue
      hash
    end
  end
end
~~~

If the value is falsy `nil` or `false` than you should not use `||=` since it
will have affect as `=`. You should check if instance variable is defined. You
can also use `begin` `end` for multiline definition

~~~
class User < ActiveRecord::Base
def main_address
   return @main_address if defined? @main_address
    @main_address = begin
      main_address = home_address if prefers_home_address?
      main_address ||= work_address
      main_address ||= addresses.first # some semi-sensible default
    end
  end
end
~~~

* read raw column value before typecast
`task.read_attribute_before_type_cast('completed_on')`

* chechbox change can activate ajax call, just define requect src and method
Usefull inside form if you want to show validation errors before actual submit,
or outside of foem if you want to submit change without form (no need to click
on submit button). I do not know how to send value...

~~~
  <%= f.check_box :renew, data: { url: customer_path(@customer), remote: true, method: :patch } %>
  <%= check_box_tag name, value, checked, data: { url: toggle_todo_path(todo), remote: true } %>
~~~

* `printf '.'` if you need to print single dot
* turn off sql debug log messages in console `ActiveRecord::Base.logger = nil`
* to show full backtrace and lines from gems you can call
  `Rails.backtrace_cleaner.remove_silencers!`
* put `gem 'irbtools', require: 'irbtools/binding'` in Gemfile so you can load
scripts in rails console: `vi 'tmp/my_script.rb'` and later you can use just
`vi`. Do not exit with `<leader>q`. but use `:wq`.
* single file rails application in one file
<https://christoph.luppri.ch/articles/2017/06/26/single-file-rails-applications-for-fun-and-bug-reporting/>
<https://gist.github.com/kidlab/72fff6e239b0af1dd3e5> for something that need
test or we just want to showcase in one file onepage rails
* use `.blank?` instead of `.empty?` since it will work for `nil` and `""`
* reject empty associated params `params.require(:visit).permit(:post_id).reject
{ |_, v| v.blank? }`
* debug network request <https://github.com/aderyabin/sniffer>
* use <https://github.com/burke/zeus> gem to speed up preload rails like spring

~~~
zeus init
zeus start
touch config/boot
~~~

* [simple captcha 2](https://github.com/kikyous/simple-captcha2) is
[generating](https://github.com/kikyous/simple-captcha2/blob/master/lib/simple_captcha/view.rb#L119)
a key (session id) and value (6 chars) when `view.show_simple_captcha` is
called.
* instead of guard clauses `errors.add :name, 'is invalid' and return false
unless item.save` you can move important things in front `item.save ||
(errors.add :name, 'is invalid' and return false)`
* gem [countries](https://github.com/hexorx/countries) uses
  [money](https://github.com/RubyMoney/money) gem.
  Also
  ```
  # Gemfile
  # country select box and store value as ISO code
  gem 'country_select'
  ```

  to use with bootstrap_form
  ```
        <%= f.form_group :country_iso_code, label: { text: 'Country' } do %>
          <%= f.country_select :country_iso_code, { iso_codes: true, include_blank: 'Select Country' }, class: 'form-control' %>
        <% end %>
  ```
  To select currency

  ~~~
  <%= f.form_group :country, label: { text: 'Country' } do %>
    <%= f.country_select :country, { iso_codes: true, include_blank: 'Select Country'}, class: 'form-control', 'data-select-currency-based-on-country': '#currency-select', 'data-countries-and-currency-codes': ISO3166::Country.all.inject({}) {|a,c| a[c.alpha2]=c.currency&.code;a }.to_json %>
  <% end %>
  <%= f.select :currency, [['Currency based on country', 0]] + Money::Currency.table.values.map {|currency| [currency[:name] + ' (' + currency[:iso_code] + ')', currency[:iso_code]] }, { disabled: 0, selected: (f.object.currency.present? ? f.object.currency : 0) }, id: 'currency-select' %>
  <script>
    $('[data-select-currency-based-on-country]').on('change', function() {
      var $target = $($(this).data('selectCurrencyBasedOnCountry'));
      var options = $(this).data('countriesAndCurrencyCodes');
      $target.val(options[$(this).val()]);
    });
  </script>
  ~~~
* single line one liner rails app for active record can be found in
[contributibuting](https://github.com/rails/rails/blob/master/CONTRIBUTING.md)
* http client https://docs.ruby-lang.org/en/2.0.0/Net/HTTP.html

~~~
require 'net/http'
require 'uri'

def fetch(uri_str, limit = 10)
  # You should choose better exception.
  raise ArgumentError, 'HTTP redirect too deep' if limit == 0

  url = URI.parse(uri_str)
  req = Net::HTTP::Get.new(url.path, { 'User-Agent' => 'Mozilla/5.0 (etc...)' })
  response = Net::HTTP.start(url.host, url.port) { |http| http.request(req) }
  case response
  when Net::HTTPSuccess     then response
  when Net::HTTPRedirection then fetch(response['location'], limit - 1)
  else
    response.error!
  end
end

print fetch('http://www.ruby-lang.org/')
~~~

* you can use `URI::regexp` to get elements from urls, for example
  ~~~
  'https://www.premesti.se/my-profile?open-moda=true'.match URI.regexp
=> #<MatchData
 "https://www.premesti.se/my-profile?open-moda=true"
 1:"https"
 2:nil
 3:nil
 4:"www.premesti.se"
 5:nil
 6:nil
 7:"/my-profile"
 8:"open-moda=true"
 9:nil>
 # to see positions of specific parts just look at
 URI.regexp
  ~~~

* validate url with custom validation
  ```
  # https://coderwall.com/p/ztig5g/validate-urls-in-rails
  # use in your models like:
  # validates :video_url, url: true, allow_blank: true
  class UrlValidator < ActiveModel::EachValidator
    def validate_each(record, attribute, value)
      record.errors[attribute] << (options[:message] || "must be a valid URL") unless url_valid?(value)
    end

    # a URL may be technically well-formed but may
    # not actually be valid, so this checks for both.
    def url_valid?(url)
      url = URI.parse(url) rescue false
      url.kind_of?(URI::HTTP) || url.kind_of?(URI::HTTPS)
    end
  end
  ```
* use bang methods to raise exception `User.create! email: 'invalid'`. If you
  use it inside callbacks than no exception will be raise, but rollback will be
  performed...
* sometime tests uses precompiled assets from `/public/assets/` so make sure to
  clean clear them. Also `/log/test.log` became huge if you do not limit it. You
  can limit test log with

  ~~~
  # config/environments/test.rb
  # limit log file to 50MB, when it reaches that it will start from empty file
  config.logger = ActiveSupport::Logger.new(config.paths['log'].first, 1, 50.megabytes)
  ~~~

* colorize matching string in console

~~~
# config/initializers/colorize.rb
class String
  COLOR = 31 # red

  def red
    "\e[#{COLOR}m#{self}\e[0m"
  end

  def colorize(string, return_nil: true, only_matched_lines: false)
    last_index = 0
    res = ''
    while (new_index = self[last_index..-1].index(string))
      if last_index + new_index - 1 > -1
        res += self[last_index..last_index + new_index - 1]
      end
      res += string.red
      last_index = last_index + new_index + string.length
    end
    res += self[last_index..-1]
    if only_matched_lines
      res = res.split("\n").select { |l| l.index string }.join("\n")
    end
    # rubocop:disable Rails/Output
    puts res
    # rubocop:enable Rails/Output
    return_nil ? nil : res
  end
end

# run tests with a command
# rails test config/initializers/colorize.rb
# require 'minitest/autorun'
# class Test < Minitest::Test
#   def test_one_substring
#     s = 'My name is John.'
#     assert_equal "My name is \e[31mJohn\e[0m.", s.colorize('John', return_nil: false)
#   end
#
#   def test_two_substrings
#     s = 'John is my name, John.'
#     r = "\e[31mJohn\e[0m is my name, \e[31mJohn\e[0m."
#     assert_equal r, s.colorize('John', return_nil: false)
#   end
#
#   def test_no_found
#     s = 'My name is John.'
#     assert_equal "My name is John.", s.colorize('Mike', return_nil: false)
#   end
#
#   def test_whole
#     s = 'My name is John.'
#     assert_equal "\e[31mMy name is John.\e[0m", s.colorize(s, return_nil: false)
#   end
#
#   def test_return
#     s = 'My name is John.'
#     assert_nil s.colorize('John')
#   end
#
#   def test_only_matched_lines
#     s = "first_line\nsecond_line John\nthird_line\nJohn fourth_line"
#     assert_equal "second_line #{'John'.red}\n#{'John'.red} fourth_line", s.colorize('John', only_matched_lines: true, return_nil: false)
#   end
# end
~~~

* if you ever need to write form tag in pure html for rails than you need to add
  authenticity token
  ~~~
  <form action='<%= my_url %>' method='post'>
    <%= hidden_field_tag :authenticity_token, form_authenticity_token %>
    <input type='text' name='name'>
  </form>
  ~~~

* gem `will_paginate` can use raw sql for selecting and for counting
 https://www.rubydoc.info/github/mislav/will_paginate/WillPaginate%2FActiveRecord%2FBaseMethods:paginate_by_sql

  if you need rails model without table https://gist.github.com/dalibor/228654
  it's different for Rails 4 and Rails 5
  https://stackoverflow.com/questions/41494951/how-to-create-activerecord-tableless-model-in-rails-5

* constants
```
# config/initializers/const.rb
# rubocop:disable Naming/MethodName
class Const
  def self.common
    hash_or_error_if_key_does_not_exists(
      site_name: 'My CRM',
      domain: 'www.trk.in.rs',
    )
  end

  def self.COLLAPSE_KEYS
    hash_or_error_if_key_does_not_exists(
      new_message: 'new_message',
    )
  end

  def self.PORT
    # Rack::Server.new.options[:Port] should be called only once (otherwise it returns 9292)
    @port ||= Rack::Server.new.options[:Port]
  end

  def self.hash_or_error_if_key_does_not_exists(hash)
    # https://stackoverflow.com/questions/30528699/why-isnt-an-exception-thrown-when-the-hash-value-doesnt-exist
    # raise if key does not exists hash[:non_exists] or hash.values_at[:non_exists]
    hash.default_proc = ->(_h, k) { raise KeyError, "#{k} not found!" }
    # raise when value not exists hash.key 'non_exists'
    def hash.key(value)
      k = super
      raise KeyError, "#{value} not found!" unless k

      k
    end
    hash
  end
end
# rubocop:enable Naming/MethodName
```

here is also a rspec test

```
# spec/models/constant_spec.rb
RSpec.describe Constant do
  it 'store string values' do
    expect(Constant.SUBSCRIBER_EVENTS[:EDIT]).to eq 'edit'
    expect(Constant.SUBSCRIBER_EVENTS.key 'edit').to eq :EDIT
  end

  it 'raise error if key does not exists' do
    expect do
      Constant.SUBSCRIBER_EVENTS[:not_exists]
    end.to raise_error KeyError

    expect do
      Constant.SUBSCRIBER_EVENTS.values_at 'not_exists'
    end.to raise_error KeyError

    expect do
      Constant.SUBSCRIBER_EVENTS.key 'not_exists'
    end.to raise_error KeyError
  end
end
```

* rails routes instead of grep you can search by `rails routes -g user` but only
  for rails > 5.2.1
* contribute to rails guide https://edgeguides.rubyonrails.org/contributing_to_ruby_on_rails.html#contributing-to-the-rails-documentation
  There you can find commands how to run tests.
  Clone the form of the https://github.com/rails/rails and update
  `guides/source/*.md` files.

  ```
  cd guides
  bundle install
  bundle exec rake guides:generate
  ```
  To run tests for activerecord you can try with
  ```
  cd activerecord
  bundle install
  bundle exec rake
  # it runs for 10 minutes

  # if it complains about non existins database
  sudo su - postgres
  createdb activerecord_unittest1
  createdb activerecord_unittest2
  ```
  Run specific test and include byebug
  ```
  bundle exec ruby -w -Iactiverecord/test activerecord/test/cases/enum_test.rb
  # include byebug
  bundle exec ruby -w -Iactionview/test -rbyebug actionview/test/template/form_options_helper_test.rb -n test_select_with_include_blank_false_and_required
  ```
  To test local changes you can point gem to your folder
  ```
  # Gemfile
  gem 'actionview'
  ```
  Commit on your fork and some branch_name. You can rebase to master
  ```
  cd ~/rails/rails
  git checkout master
  # pull from rails repository
  git pull rails master
  # push to my repository
  git push
  # rebase
  git checkout my_branch
  git rebase -i master
  # select squash so there is only one commit
  git push origin my_branch
  ```
  On web there is a button 'Create pull request'.
  In description of pull requests you should write some code example.

* Rails 6 uses Utf8mb4 instead Utf8 so you can store emojis 😀 everywhere, I’m
  💯% sure. Here is plain ascii (￣︶￣). 🙌
* Rails credentials are available on rails 5.2. You need to export
RAILS_MASTER_KEY=`cat config/master.key` and you can use inside rails and config
  ```
  # config/database.yml

  ```
* when bcrypt is updated you need to update secrets credentials with 
```
rails credentials:edit
```
When you do not have `config/master.key` or `config/credentials/development.key` error is
```
.rvm/gems/ruby-2.5.3/gems/activesupport-6.0.0.rc1/lib/active_support/message_encryptor.rb:206:in `rescue in _decrypt': ActiveSupport::MessageEncryptor::InvalidMessage (ActiveSupport::MessageEncryptor::InvalidMessage)
```

  Access with
  ```
  Rails.application.credentials[:secret_key_base]
  # or
  Rails.application.credentials.secret_key_base
  ```
  you can create development credentials (config/credentials/development.key and
  config/credentials/development.yml.enc) which will be loaded in development
  environment and take precedence over master credentials (if there is a
  development.key, can not use ENV for key for environment credentials)
  you can commit that keys in repository.
  ```
  rails credentials:edit -e development
  git add config/credentials/development.yml.env
  git add config/credentials/development.key -f

  # also for test
  rails credentials:edit -e test
  git add config/credentials/test.yml.env
  git add config/credentials/test.key -f
  ```
  Env RAILS_MASTER_KEY is needed to read credentials.
* dhh videos On Writing Software Well
  https://www.youtube.com/watch?v=wXaC0YvDgIo&list=PL9wALaIpe0Py6E_oHCgTrD6FvFETwJLlx
  ```
  ```
* actiontext is included in Rails 6 but you need to install
  ```
  rails action_text:install
  ```
  add stylesheets is not needed in app/assets/stylesheets/application.sass `//= require actiontext`

  and you can use
  ```
  # app/models/event.rb
    has_rich_text :content

  # app/views/events/_form.html.erb
    f.rich_text_area :content

  # app/views/events/show.html.erb
    <%= @event.content %>
  ```

* on heroku default timeout is 30s and if requests takes more that that it will
  timeout and no exception notification will be sent. You can use
  https://github.com/sharpstone/rack-timeout to raise exception on 30s
  ```
  # config/initializers/rack_timeout.rb
  # Heroku timeout is 30s
  Rails.application.config.middleware.insert_before Rack::Runtime, Rack::Timeout, service_timeout: 30

  # do not use same log file as Rails
  Rack::Timeout::Logger.logger = Logger.new('log/timeout.log')
  Rack::Timeout::Logger.logger.level = Logger::ERROR
  ```

  ```
  # Gemfile
  # raise timeout error before server timeouts
  gem 'rack-timeout', require: 'rack/timeout/base'
  ```

* `A copy of xxx has been removed from the module tree but is still active!`
  it could be that spring does not load it since it happens in tests also
  You can see all auto loaded paths
  ```
  bin/rails r 'puts ActiveSupport::Dependencies.autoload_paths'
  ```
  My solution is to copy all methods to the strategy so it does not use those
  classes.

* `rake tmp:cache:clear` for rails clear cache
* check gemes dependencies
  ```
  gem install bundler-audit
  bundle audit check --update
  ```
* camel case to underscore snake_case is with `'HiBye'.underscore` but in
  ActiveRecord it is better to use `LocationUser.model_name.collection`
* csrf When you want to submit form from another domain you get this error

  ~~~
  ActionController::InvalidAuthenticityToken (ActionController::InvalidAuthenticityToken):
  ~~~

  This is when your controller is protected from CSRF attack (it contains
  protect_from_forgery) .
  You can mute it instead of raising exception:

  ~~~
  sed -i app/controllers/application_controller.rb -e '/protect_from_forgery/c \
    # protect_from_forgery with: :exception\
    protect_from_forgery with: :null_session'
  ~~~

  You can disable CSRF Token authenticity for specific actions

  ~~~
  skip_before_action :verify_authenticity_token, only: [:my_insecure_post_action]
  ~~~


  Another way to bypass cors check is to override `verified_request?`
  ```
  # app/controllers/application_controller.rb
    def verified_request?
      super || (session['warden.user.user.key'] && request.format.json?)
    end
  ```

  In my feature tests (for example bank payment redirection back with POST than I
  do not have authenticity error `verified_request?` returns true, but for
  development `verified_request?` returns false
* `ActiveSupport::SafeBuffer` is needed when you whant to render html tags on
  page or email. You need to start with `t='text'.html_safe` and than add to it
  some other string `t+='<script>`. If you use interpolation `"#{t}"` or string
  + safejoin `'a' + t`, or translation `I18n.t('name', name: t)`, or when you
  save and load to database than you need to call `.html_safe` on a result
  (which is a string).
  If you have array than you can use
  https://apidock.com/rails/ActionView/Helpers/OutputSafetyHelper/safe_join
  ```
  @name_with_arrows = ActionController::Base.helpers.safe_join(array, " #{Constant::ARROW_CHAR} ")
  ```
* `form_with` is used instead of `form_for @model` and `form_tag url`. Here is
  example
  ```
  # nothing special here
  form_with url: users_path do |f|
  # all forms are remote: true by default so we need to use `local: true`
  form_with url: users_path, local: true do |f|
  # for model use model:
  form_with model: @user do |f|
  ```

  To determine if it is a POST or a PATCH, form check `@user.persisted?` and in
  case it is persisted it will add hidden input field (method will remain post)
  ```
  <form action="/books/1" accept-charset="UTF-8" method="post">
    <input type="hidden" name="_method" value="patch">
    <input type="hidden" name="authenticity_token" value="asd">
  </form>
  ```
* custom exception class
  ```
  module TrkDatatables
    class TrkError < StandardError
      def message
        "TrkDatatables: #{super}"
      end
    end
  end
  ```
  than if there is some posibility to catch error (for example TypeError for dig
  method) you can write
  ```
    def search_all
      @params.dig(:search, :value) || ''
    rescue TypeError => e
      raise TrkError, e.message + '. Global search is in a format: { "search": { "value": "ABC" } }'
    end
  ```

  You can rescue in controller with
  ```
  # app/controllers/application_controller.rb
  rescue_from TrkDatatables::TrkError do |exception|
    respond_to do |format|
      format.html { redirect_to root_path, alert: exception.message }
      format.json { render json: { error_message: exception.message, error_status: :bad_request }, status: :bad_request }
    end
  end
  ```

  When we need to permit hash or hash of hashes you simply permit `{}`
  ```
  # [jv_percentages][operator_percentages][123] = 1
  # [jv_percentages][operator_location_percentages][123][456] = 1
  def jv_percentages_params
    params.require(:jv_percentages).permit(
      operator_percentages: {},
      operator_location_percentages: {},
    )
  end

  jv_percentages_params # => { operator_percentages: { '123': '1' },
  operator_location_percentages: { '123': { '456': '1' } } }
  ```

  Params can be different format, so event this code can raise exception
  ```
  params.require(:subscriber).permit(:name)
  ```
  when params is an array `[]` and `A NoMethodError occurred in ` undefined
  method `permit` for []:Array.
  When it is a hash than it's class is `ActionController::Parameters` but when
  it is array than it is really array of `ActionController::Parameters`.
  You can simulate with
  ```
  curl -X POST -g "localhost:3001/subscribers" -d '{"subscriber":[]}'
  ```
* you can not use `serialize :custom_sign_up_values,
  ActionController::Parameters` since when using
  ```
  def sign_up_params
    params.require(:subscriber).permit(
      custom_sign_up_values: current_location.custom_sign_up_labels},
      )
    end
  end
  ```
  it will be converted to HashWithIndifferentAccess and error will be raised
  ```
    ActiveRecord::SerializationTypeMismatch:
       can't serialize `custom_sign_up_values`: was supposed to be a ActionController::Parameters, but was a ActiveSupport::HashWithIndifferentAccess. -- {"Room#"=>"room_number", "Passport ID"=>"passport_id"}
  ```
  so for existing records you should update https://stackoverflow.com/questions/48773769/existing-data-serialized-as-hash-produces-error-when-upgrading-to-rails-5
  ```
  # app/models/model_hack.rb
  # Call with: ModelHack.where.not(custom_sign_up_values: nil).find_each {|m| m.update_custom_sign_up_values! }
  class ModelHack < ApplicationRecord
    self.table_name = 'subscribers'
    serialize :custom_sign_up_values

    def update_custom_sign_up_values!
      return unless custom_sign_up_values.present?

      h = custom_sign_up_values
      return if h.is_a? Hash

      self.custom_sign_up_values = h.to_unsafe_h.to_h
      save!
    end
  end
  ```
* for rails helpers, when you want to use block syntax for your helper than you
  should use *capture* https://stackoverflow.com/questions/32985379/appending-to-yield-in-content-tag because what is inside block will be rendered two times on the page (capture is not required if *yield* is inside *content_tag*)
  ```
  # app/helpers/text_helper.rb
  module TextHelper
    def detail_view_one(title, text = nil, label_class: 'col-sm-2 dt__long--min-width', text_class: 'col', &block)
      <<~HTML
        <dt class='#{label_class}'>#{title}</dt>
        <dd class='#{text_class}'>#{block_given? ? capture(&block) : text}</dd>
        <dt class='w-100'></dt>
      HTML
        .html_safe
    end

    def detail_view_list(item = nil, *fields, label_class: 'col-sm-2 dt__long--min-width', skip_blank: [])
      content_tag 'dl', class: 'row' do
        if block_given?
          yield
        else
          detail_view item, *fields, label_class: label_class, skip_blank: skip_blank
        end
      end
    end
  end

  # app/views/posts/show.html.erb
        <%= detail_view_list do %>
          <%= detail_view_one Happening.human_attribute_name(:recurrence) do %>
            <%= link_to 'A', 'b' %>
            OK
          <% end %>
        <% end %>
  ```

* routes
  ```
  resources :post do
    resources :comments, shallow: true
  end
  ```
* skip showing password on view
  ```
  # app/helpers/text_helper.rb
  # when editing use value '', and placeholder hidden with stars
  # <% if f.object.password.present? %>
  #   <%= f.text_field :password, placeholder: hidden_with_stars(f.object.password), value: '' %>
  # <% else %>
  #   <%= f.text_field :password, placeholder: 'Your password' %>
  # <% end %>
  #
  # on controller side restore original password if new is not provided
  # if @isp.password.present? && !isp_params[:password].present?
  #   # keep old password
  #   params[:isp][:password] = @isp.password
  # end
  def hidden_with_stars(text)
    return '' unless text.present?
    if text.length > 2
      text.first + '*' * (text.length - 2) + text.last
    else
      '*' * text.length
    end
  end
  ```
* when generating model under admin namespace you need to use foreign_key since
  model name is without `Admin` but tables are with `admin_`
  ```
  # app/models/admin/page.rb
  class Admin::Page < ApplicationRecord
    belongs_to :run, foreign_key: :admin_run_id
  end
  # also for run
  class Admin::Run < ApplicationRecord
    has_many :pages, dependent: :destroy, foreign_key: :admin_run_id
  end
  ```
  and always use `admin_` for variable names, for example `admin_run.pages.each
  {|admin_page| }`.

* find column type using `User.column_for_attribute('status')` or
  `User.columns_hash['status']`
* find all association names for model
  ```
  User.reflect_on_all_associations.map &:name
  # => [:roles, :user_activities]
  ```

* Rails 5 mailers uses with and params
  ```
  UserMailer.with(user: @user).welcome_email.deliver_later
  ```
* upgrading from 4.2 to 5.0 https://www.fastruby.io/blog/rails/upgrades/upgrade-rails-from-4-2-to-5-0#config-files
  * look at differences http://railsdiff.org/4.2.10/5.0.7.2
  * change `gem 'rails'. '~> 5.0.0'` and run `bundle update rails` to see which
    gems should also be upgraded
* debug find where it is redirected use overwritted redirect_to
  ```
  # app/controllers/application_controller.rb
  def redirect_to(options = {}, response_status = {})
    ::Rails.logger.error("Redirected by #{caller(1).first rescue "unknown"}")
    super(options, response_status)
  end
  ```
* Rack middleware https://medium.com/@shashwat12june/rack-and-rack-middleware-f93513ac92a6
  You can skip Rails in rack middleware if you return like this example
  ```
  class FilterLocalHost
    def initialize(app)
      @app = app
    enddef call(env)
      req = Rack::Request.new(env)
      if req.ip == "127.0.0.1" || req.ip == "::1"
        [403, {}, ["forbidden"]]
      else
        @app.call(env)
      end
    end
  end
  ```

* extract link from text, for example extract url from email
  ```
  text = all_emails.last.body.to_s
  END_CHARS = %{.,'?!:;}
  links = URI.extract(text, ['http']).collect { |u| END_CHARS.index(u[-1]) ? u.chop : u }
  ```
* result model
  ```
  # confir/initializers/result.rb
  class Result
    attr_accessor :message, :data

    # Example of returning results, for example in service:
    #   return Result.new 'Next task created', next_task: next_task
    # and use in controller:
    #   if result.success? && result.data[:next_task] == task
    def initialize(message, data = {})
      @message = message
      @data = OpenStruct.new data
    end

    def success?
      true
    end
  end

  class Error < Result
    def success?
      false
    end
  end
  ```

* template runner https://guides.rubyonrails.org/rails_application_templates.html
  ```
  # on existing app
  rails app:template LOCATION=~/template.rb
  ```
* before Rails 5 we used `attribute_was` to get original value of some field.
That previous value can be read in any hook, for example `after_validation` you
can read for `operator_id && operator_id_was`.
  but in Rails 6 we use `.previous_changes` which list all columns that was
  changed https://api.rubyonrails.org/classes/ActiveModel/Dirty.html or
  https://api.rubyonrails.org/classes/ActiveRecord/AttributeMethods/Dirty.html
  to list what is goinog to be changed `.changes_to_save`
  ```
  @member_profile.previous_changes
  {"zip"=>["07068", "07065"], "updated_at"=>[Tue, 22 Jun 2021 08:41:40.987696000 UTC +00:00, Tue, 22 Jun 2021 08:41:41.454602000 UTC +00:00]}
  ```
* add delay in response
  ```
  # before_action :_sleep_some_time

  def _sleep_some_time
    sleep 2
  end
  ```
* to add message without column use `errors.add :base, 'msg'` so it will show
  something like `errors.full_messages # 'msg'` (no mention of the column)
* for string columns we should have validation for length since there are errors
  like `ActiveRecord::ValueTooLong (PG::StringDataRightTruncation: ERROR:  value
  too long for type character varying(100)` so
* error is raise when using a string for unsafe funtions so just wrap `Arel.sql`
  https://api.rubyonrails.org/v5.2/classes/ActiveRecord/UnknownAttributeReference.html
  ```
  .order(Arel.sql('COALESCE(last_message_created_at, created_at) desc'))
  ```
* you can use images from `app/javascript/images/name_of_image.jpg` after you
  import to pack
  ```
  require.context('../images', true)
  ```
  and you can use in rails like
  ```
  <%= image_pack_tag 'media/images/name_of_image.jpg' %>
  <img src="<%= asset_pack_path 'media/images/name_of_image.jpg' %>" width="100%" height="100%" alt=" demo instructions"></img>
  ```
* common errors in rails
  * use `enum status: %i[initialized]` and add a check in controller
    ```
    if @payment.initialized?
    ```
    and later add another state that is similar to initialized but you forgot to
    update all places where we need to check status
    ```
    enum status: %i[initialized draft]
    if @payment.initialized? || @payment.draft?
    ```
* InvalidAuthenticityToken https://stackoverflow.com/a/60394170/287166
 and rails https://github.com/rails/rails/issues/21948
 I reproduce by opening two tabs and on second log in and log out, session is
 invalid so when I log in on first tab, I got an error.
 Two solutions:
 * extend session store to cookie
 * refresh if no token.
 ```
  # find the name in Storage in developer tools
  # config/initializers/session_store.rb
  Rails.application.config.session_store :cookie_store, key: '_myapp_session', expire_after: 2.weeks
  ```
  but that will keep user log in after closing the browser (like Remember me
  button).

  Second solution is to add no-store no-cache
  https://github.com/rails/rails/issues/21948#issuecomment-205371135
  ```
* dotenv nice gem to load env to ruby. If you want to load to shell you can
  https://gist.github.com/mihow/9c7f559807069a03e302605691f85572?permalink_comment_id=3923897#gistcomment-3923897
  set -o allexport; source .env; set +o allexport
  ```
* fake phone us mobile +12025550123
* show page inside iframe
  ```
  <iframe width='100%' height="500px" srcdoc="
    <%= @run.response.gsub '"', "'" %>
   '></iframe>
  ```
  on w3school we can see the page
  https://www.w3schools.com/tags/tryit.asp?filename=tryhtml_iframe
  so we can try similar locally using caddy ssl server

* show json pretty_print with `JSON.pretty_generate hash`
* change database with `rails db:system:change --to=postgresql`

* gem sequel you can create schema with
  ```
  DB = Sequel.connect(...)
  DB.extension :schema_dumper
  puts DB.dump_schema_migration
  File.write 'tmp/db.schema', DB.dump_schema_migration
  ```
* when loading some files that do not follow naming convention
  ```
  2022-09-16T11:09:38.133430+00:00 app[web.1]: /app/vendor/bundle/ruby/3.1.0/gems/zeitwerk-2.6.0/lib/zeitwerk/loader/helpers.rb:127:in `const_get': uninitialized constant Overrides (NameError)

  raise Zeitwerk::NameError.new("expected file #{file} to define constant #{cpath}, but didn't", cref.last)
  ```
* adding wild card domain to hosts to allow any subdomain you do not need to use
star, just use dot like `.mydomain.com`. For multiple level subdomains you need
to use regexp
```
# config/application.rb
    config.hosts << /.*gitpod.io/
```
* to chech if variable exists in template you can use
  ```
  # app/views/users/show.html.erb
  if local_assigns[:user].present?
    <%= user.email %>
  end
  ```
* rails one time command `rails runner -e staging "puts "
* reduce memory with env variable: `MALLOC_ARENA_MAX=2` TODO: track memory usage
with aws
* kaminari first page is page 1
* copy files from ruby
```
def self.initialize_fixture_blob
  return if File.exist? "#{Rails.root}/tmp/storage/or/9s/or9sbwfely5gby30qdvtoa1cu09a"

  FileUtils.mkdir_p "#{Rails.root}/tmp/storage/or/9s"
  FileUtils.cp(
    "#{Rails.root}/test/fixtures/files/computer_text.png",
    "#{Rails.root}/tmp/storage/or/9s/or9sbwfely5gby30qdvtoa1cu09a"
  )
end
```
* contact form should use recaptcha but also check for the content
  ```
  asd
  ```
