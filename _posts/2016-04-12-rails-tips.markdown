---
layout: post
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

[form
select](https://apidock.com/rails/ActionView/Helpers/FormOptionsHelper/select)
can accept as 2th param (choices) two variant:

* flat collection `[['name', 123],...]`
* if you need manual tags `<select><option></option></select>` than you can use
`<%= select tag "statuses[]", options_for_select([[]], selected: 1) %>`
* nested collection `grouped_options_for_select()`

  Preselected value can be defined as second param in `options_for_select([],
  selected: f.object.user_id`, or as 4th param in
  `options_from_collection_for_select(User, :id, :email, selected:
  f.object.user_id)`. Note that it can be `value` or hash `selected: value`.

as 3th param (options):

Sometimes when options do not include value that you want to set and you
use prompt to be shown, please preform check `{ prompt: 'Select package'
}.merge( options.present? ? { selected: @customer.some_other_package.id
} : {} )`. If target selected value is not in options, that first option will be
used, or prompt is shown when options is empty.
* `disabled: [values]` to disable some options
* `label: 'My label'`
* `prompt: 'Please select'` this is shown only if not already have some value
* `include_blank: 'Please select'` this is shown always (even already have
value) I found usefull only with select2 where we use custom placeholder and
blank option is not selectable `<%= f.select :customer_name_and_username,
options_from_collection_for_select(current_location.customers, 'id',
'name_and_username'),  { include_blank: true, label: 'Customer' },
'data-select2': true, placeholder: 'Search by Customer Name or Username' %>`

as 4th params (html options)
* `multiple: true` so it is multi_select (instead of dropdown). Multi select
  will be shown with its own scrollbar (use `size: 5` to limit the size).
  In controller you need to allow arrays

  ~~~
  def event_params
    params.require(:event).permit(
      :title, skills: []
    )
  end
  ~~~

*  Form array can be set on any input, use square brackets `name='user_ids[]'`.
  In view you can set array of hidden fields
  ```
    <% @isp_push_packages.isp_package_ids.each do |isp_package_id| %>
      <%= f.hidden_field 'isp_package_ids[]', value: isp_package_id %>
    <% end %>
  ```
  or using select `multiple: true` (it will automatically add `[]` to the name)
  ```
  <%= f.select isp_package_ids, options, {}, multiple: true %>
  ```
  You can also nest in two dimensions ie it will be a hash with array values
  `name='user_ids[#{company.id}][]`. In this case you need to permit hash
  (Rails 5.2) or whitelist in before Rails 5.2

  ```
  params.require(:post).permit(reseller_location_ids: {}) # Rails 5.2
  permit(files: params[:post][:files].key # before Rails 5.2

  <%= select_tag 'reseller_location_ids[#{operator.id}]', options, multiple: true %>
  # do not know how to use f.select since can not have method with a name
  `name[name]`
  ```
  Note that if it is not required and nothing is selected than nothing will be
  sent, you should use hidden field or default value `{}`

  ```
  <%= hidden_field_tag 'reseller_location_ids[0][]' %>
  # or
  location_ids = (params[:reseller_location_ids] || {})[params[:reseller_operator_id]] || []
  ```

*  And in model you need to clean empty strings `[""]` when nothing is selected
  ~~~
  before_validation :clean_empty_skills
  def clean_empty_skills
    self.skills = skills.select &:present?
  end
  ~~~

You should avoid saving without validation `save(validate: false)` or
`update_attribute :name, 'my name'`. It is risky to save without validations.
Other methods also do not check validation
http://www.davidverhasselt.com/set-attributes-in-activerecord/
`update_column`, `@users.update_all`.

Conditional validations can be used with proc new like `if: -> { }` but with
parameters. Also `if: lambda {|a| }` (difference in required params to block)

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
month name in select date `2/Nov/2001` is it can parse collectly. For inputs
with two numbers it can not guess the month or date `2/3/2000` so you need to
use `Date.strptime '2/3/2000' , '%m/%d/%y'` to return 3 Februar.

~~~
# config/initializers/date_time_formats.rb
# all 3 classes
# puts user.updated_at.to_s :myapp_time
# puts Time.now.to_s :myapp_time
# puts Date.today.to_time :myapp_time # Date need to be type casted to Time
# puts Time.now.to_date :myapp_date # Time object to Date if we want myapp_date

Time::DATE_FORMATS[:default] = "%d-%b-%Y %I:%M %p" # this is same as for
# datepicker format = 'DD-MMM-YYYY h:mm A'
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

* one issue is when you use different table name `self.table_name =
  'something_other'` than relations could not be picked correctly
* union is not supported [issue 929](https://github.com/rails/rails/issues/939)
  but you can try with raw sql and `from` statement. Union should have same
  number and type of columns. Also use `.from([Arel.sql('...')])`

  ~~~
    sql = <<~SQL
    ( ( SELECT id, location_id, package_saleid, customer_id, customer_name, location_package_id, sale_date, -total_amount_cents as total_amount_cents, total_amount_currency, customer_invoice_id, (bytes_uploaded_total + bytes_downloaded_total) as total_data_used
        FROM location_package_sales
      ) UNION (
        SELECT id, location_id, balance_refillid as package_saleid, NULL as customer_id, 'refill' as customer_name, NULL as location_package_id, created_at as sale_date, refill_amount_cents as total_amount_cents, refill_amount_currency as total_amount_currency, NULL as customer_invoice_id, NULL as total_data_used
        FROM location_balance_refills
      )
    ) AS location_package_sales_or_location_balance_refills
    SQL
    @all_location_package_sales_or_location_balance_refills = LocationPackageSaleOrLocationBalanceRefill
      .includes(:location, :customer, :customer_invoice, :location_package)
      .from([Arel.sql(sql)])
      .where(location_id: current_location)
  ~~~

* execute sql from rails console. When you run sql result is iteratable (array
 of arrays of selected fields).

```
s = 'SELECT * FROM users'
r = ActiveRecord::Base.connection.execute s
r.each { |l| puts l }
```

* for single item, always use `@post.destroy` instead of `@post.delete` because
it propagates to all `has_many :comments, dependent: :destroy` (also need to
define this dependent param). If you use delete or use destroy without
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

## Dump database

Dump database from production for local inspection, you can download from
heroku manually or using commands:

~~~
heroku pg:backups:capture # it will create b002.dump
heroku pg:backups:download # it will download to latest.dump
~~~

Save it for example `tmp/b001.dump`.

## Restore database

You can dump LOCAL database with `pg_dump`. Note that this is plain sql, but
heroku dump is binary format (size is much smaller).

~~~
export DUMP_FILE=tmp/b001.dump
export DATABASE_NAME=$(rails runner 'puts ActiveRecord::Base.configurations["development"]["database"]')

# pg_dump $DATABASE_NAME > $DUMP_FILE
# we must use heroku style, replace: mypassword myuser and mydb
# PGPASSWORD=mypassword pg_dump -Fc --no-acl --no-owner -h localhost -U myuser $DATABASE_NAME > $DUMP_FILE
pg_dump -Fc --no-acl --no-owner $DATABASE_NAME > $DUMP_FILE
scp $DUMP_FILE 192.168.1.3:
ssh 192.168.1.3
sudo cp b001.dump /var/www/html/

heroku pg:backups restore --confirm move-index http://trkcam.duckdns.org/b001.dump DATABASE_URL
heroku pg:backups
~~~


Restore from local textual and binary dump

~~~

chmod a+r $DUMP_FILE

rake db:drop db:create

# textual dump
psql $DATABASE_NAME < $DUMP_FILE

# binary dump
sudo su postgres -c "pg_restore -d $DATABASE_NAME --clean --no-acl --no-owner -h localhost $DUMP_FILE"
~~~

Or you can use [duleorlovic's load_dump
helper](https://github.com/duleorlovic/config/blob/master/bashrc/rails.sh#L39)


To restore on heroku you need to dump with same flags (dump is binary) and push
the file somewhere on internet, for example AWS S3 and than run in console

~~~
heroku pg:backups restore --confirm playcityapi https://s3.amazonaws.com/duleorlovic-test-us-east-1/b001.dump DATABASE_URL
~~~

## Heroku upgrade database plan

Upgrade heroku *hobby-dev* na *hobby-basic* ($9/month max 10M rows).
All plans https://elements.heroku.com/addons/heroku-postgresql

~~~
heroku addons:create heroku-postgresql:hobby-basic
# Creating heroku-postgresql:hobby-basic on ⬢ myapp... $9/month
# Database has been created and is available
#  ! This database is empty. If upgrading, you can transfer
#  ! data from another database with pg:copy
# Created postgresql-defined-42601 as HEROKU_POSTGRESQL_CHARCOAL_URL
# Use heroku addons:docs heroku-postgresql to view documentation

heroku pg:copy DATABASE_URL HEROKU_POSTGRESQL_CHARCOAL_URL
# ▸    WARNING: Destructive action
#  ▸    This command will remove all data from CHARCOAL
#  ▸    Data from DATABASE will then be transferred to CHARCOAL
#  ▸    To proceed, type myapp or re-run this command with --confirm myapp
# 
# > myapp
# Starting copy of DATABASE to CHARCOAL... done
# Copying... done
~~~

Change DATABASE_URL

Upgrading from hobby-basic to standard-0

~~~
heroku pg:info
heroku addons:create heroku-postgresql:standard-0
# save the variable name HEROKU_POSTGRESQL_{some color}_URL
heroku maintenance:on
heroku pg:copy DATABASE_URL HEROKU_POSTGRESQL_color_URL
heroku pg:promote HEROKU_POSTGRESQL_color_URL
heroku maintenance:off
heroku config:set DATABASE_URL=....url from config
heroku addons:destroy HEROKU_POSTGRESQL_color_old_URL
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

* To disable sql logs in rails logger put this in initializer file

  ~~~
  # config/initializers/silent_sql_log.rb
  ActiveRecord::Base.logger.level = Logger::INFO
  ~~~
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
(particularly for foreign keys) also usefull since in sql `where col != NULL`
will not return any value (also `where NULL = NULL`), you need to use `where col
IS NOT NULL` (which is default for active record `.where.not(col: nil)`). So
better is to use `false` since `where col != false` will return some results.
Add index `add_index :users, :email_address, unique: true`...
if you want to add index in different step (not in `t.references :c, index:
false` because name is too long (error like `Index name 'index_table_column' on
table 'table' is too long; the limit is 64 characters`) than you can use
different name in separate add_index command (can not rename in same command).
NOTE that you need to use exact column name (with `_id`)
`add_index :users, :company_id, name: 'index company on users'`
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
    # this is needed since renamed columns will be ignored on "save"
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
mysql> DELETE FROM schema_migrations WHERE version = 20161114162031
~~~

if you want to redo migration (revert and migrate again) you can use `redo` 
To drop database in console you can `ActiveRecord::Migration.drop_table(:users)`
* you can use `rake db:migrate:redo STEP=2` to redo last two migrations, or you
can run all migrations to certain point with `rake db:migrate
VERSION=20161114162031` (note that `:up` `:down` only run one migration)
* `rake db:migrate` will also invoke `db:schema:dump` task
  [link](http://edgeguides.rubyonrails.org/active_record_migrations.html#running-migrations)
* to check current metaga data, for example schema enviroment you can run
  ```
  rails runner "ActiveRecord::Base.connection.execute('select * from ar_internal_metadata').each {|r|puts r}"

  # or for test database
  rails runner "ActiveRecord::Base.connection.execute('select * from ar_internal_metadata').each {|r|puts r}" -e test

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
    create_table :campaigns_templates, id: false do |t|
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
# to_table is in rails5
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
`index`).
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
add_user_to_leads user:references` which will generate

~~~
class AddUserToLeads < ActiveRecord::Migration
  def change
    # reference will automatically include index on that column
    add_reference :leads, :contact, foreign_key: true
  end
end
~~~

For polymorphic associations `projectable_id` and `projectable_type` you can create in
migration `rails g model project projectable:references{polymorphic}`

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

You can eager load polymorphic association https://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html#module-ActiveRecord::Associations::ClassMethods-label-Eager+loading+of+associations
but can not use it in where https://stackoverflow.com/questions/16123492/eager-load-polymorphic
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
  end
end
~~~

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
Rake::Task['db:fixtures:load'].invoke
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
# never rescue from Exception, but use Standard error
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

  def initialize(template_body, data_for_body)
    @template_body = template_body
    @data_for_body = data_for_body
  end

  def render
    m = Mustache.new
    m.raise_on_context_miss = true
    message = m.render(template_body, data_for_body)
    Result.new message
  rescue Mustache::ContextMiss, Mustache::Parser::SyntaxError => e
    Error.new e.message
  end
end
~~~

usage

```
result = TemplateRenderService.new(email_body, name: '', role_name: '')
return result.message if result.success?
```

If handlebars contains dots than you need to nest data, like for `{{user.name}}`
data needs to be `user: { name: 'dusan' }`

# Rake tasks

You can write tasks with arguments [rails
rake](http://guides.rubyonrails.org/command_line.html#custom-rake-tasks)
~~~
# lib/tasks/db.rake
namespace :db do
  desc 'This task does nothing'
  task nothing: :environment do
    # environment is needed to load rails
  end
end
~~~

Instead of `:env` or `:environment` you can use other tasks on which it depends.
Also you can receive parameters into `args`

~~~
# lib/tasks/update_subdomain.rake
# instead of puts, we can use Rails.logger.info
Rails.logger = Logger.new(STDOUT)
namespace :update_subdomain do
  desc "update subdomains for my-user. default value is 'my_subdomain'"
  task :my_user, [:subdomain] => :environment do |task, args|
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

Use "Register" or "Sign up" for registrations and "Log in" and "Log out" for
sessions (since it is easy to differentiate from "sign up").

# 7 patterns

From
[7-ways-to-decompose-fat-activerecord-models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
suggestions for code organization I use mostly:

You can check also <http://trailblazer.to/>
You can organize domain (controller, model and view) into separate folders
[drawers](https://github.com/NullVoxPopuli/drawers)

If there is a lot of business logic in view, I prefer to put that in controller
so it is more visible.


## Service objects

Always define only one public method: `call`, `perform` or `process`

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

You should return object that can be success and hold some data

```
# app/models/result.rb
class Result
  attr_accessor :message, :data

  # you can return in service like:
  #   return Result.new 'Next task created', next_task: next_task
  # and use in controller:
  #   if result.success? && result.data[:next_task] == task
  def initialize(message, data = {})
    @message = message
    @data = data
  end

  def success?
    true
  end
end

# app/models/error.rb
class Error < Result
  def success?
    false
  end
end
```

and you can rescue from exceptions:

~~~
# app/services/my_service.rb
class MyService
  # Some custom exception if needed
  class ProcessException < Exception
  end

  def initialize(h)
    @user = h[:user]
  end

  def process(posts)
    success_message = do_something posts
    Result.new success_message
  rescue ProcessException => e
    Error.new e.message
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

Service object is similar to Command pattern which is implemented in gem
<https://github.com/collectiveidea/interactor>

## Form Objects

form objects (query objects) for multiple in multiple out data

~~~
# app/form_objects/landing_signup.rb
class LandingSignup
  include ActiveModel::Model
  FIELDS = %i[ current_city current_location current_group prefered_group email].freeze
  attr_accessor(*FIELDS)

  validates :email, presence: true

  def save
    _create_users
    _create_roles
  end

  # no need for initialize since AR will pick from params
  # but if you need to set default values, here is a code
  def initialize(attributes)
    super(attributes)
    _after_initialize
  end

  def _after_initialize
    self.send_email_invitation = true if send_email_invitation.nil?
  end
end
~~~

Usage

~~~
# app/controllers/pages_controller.rb
class PagesController < ApplicationController
  def home
    @landing_signup = LandingSignup.new
    @landing_signup.current_city = City.first
  end

  def landing_signup
    @landing_signup = LandingSignup.new landing_signup_params
    @landing_signup.current_city = City.first
    if @landing_signup.save
      sign_in @landing_signup.user
      redirect_to dashboard_path, notice: @landing_signup.notice
    else
      flash.now[:alert] = @landing_signup.errors.full_messages.to_sentence
      render :home
    end
  end
end
~~~

To define url for ActiveModel you need to overwrite some instance vars
https://stackoverflow.com/questions/3736759/ruby-on-rails-singular-resource-and-form-for
so better is to define url param `form_for @form, url: users_path`

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

  # methods defined here are going to extend the class, not the instance of it
  # do not use "self." in method definition
  # you can set "@@my_module_variable" here
  # better is to set class_variable:  self.ancestors.first.class_variable_set :@@someable_value, 'some value'
  # also you can include other concerns here
  class_methods do
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

## Observable

We an use observable objects to send notifications [implementation in 3
languages](http://www.diatomenterprises.com/observer-pattern-in-3-languages-ruby-c-and-elixir/)
It could looks like [action as a
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

You can use
https://api.rubyonrails.org/v5.2.2.1/classes/ActiveSupport/MessageVerifier.html
to encode strings or id
```
  @unsubscribe = Rails.application.message_verifier(:unsubscribe).generate(@user.id)
```

Old example

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

```

in model
```
  monetize_columns :round_off
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

## Wicked PDF

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


# Style guide

[toughtbot style
guide](https://github.com/thoughtbot/guides/tree/master/best-practices)

* do not reference model class directly from a view (how to render counts than
?)
* do not use SQL outside of models
* use `touch: true` on `belongs_to` associations
* use `ENV.fetch` instead of `ENV[]` so it raises exception when env variable
does not exists

My style in rails models use following order:

1. `include` and `extend` other modules, `devise` or, `serialize :col, Hash`
1. `FIELDS = %i[name].freeze` and other constants
1. `attr_accessor`
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

# Geocoder

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

# Authorization policy

## Cancan

use cancancan and define all your actions in app/models/ability. If you want to
use
[load_resource](https://github.com/ryanb/cancan/wiki/authorizing-controller-actions#load_resource),
you should separately define: index (with hash), show/edit/update/destroy (with
block or hash), new/create (with hash). For nested resources just write parent
association

Note that following next `cannot` rule will override a previous `can` rule, so
it is enough to set `can :manage, :all` and than write what `cannot :destroy,
Project`

Define abilities
https://github.com/CanCanCommunity/cancancan/wiki/defining-abilities
```
can :read, Article # autorize! :read, Article will return true
can :crud, Article # authorize! :read/:create/:update/:destroy, Article will return true
# https://github.com/CanCanCommunity/cancancan/wiki/Action-Aliases
aliase_action :index, :show, to: :read | :new, to: :create | :edit. to: :update
can :manage, Article # authorize! :any_action, Article will return true
can [:update, :destroy], [Post, Comment] # array notation
```
Hash of conditions
```
# can read only posts that belongs to user
can :read, Post, user_id: user.id
```
Defining with a block is evaluated only when instance is passed (for example not
in index action when class is used, ie if you call `can? :update, Project` it
will return true)
```
can :update, Project do |project|
  false
end
```

Defining with a block without other arguments can be used for defining
Abilities and roles in database
https://github.com/CanCanCommunity/cancancan/wiki/Abilities-in-Database
```
can do |action, subject_class, subject|
end
```

Example that only admins can change, for example company_id, with
can_change_user_company_id
```
# in models/ability.rb
can :can_change_user_company_id, User if user.admin?

<!-- users/_form.html.erb -->
<% if can? :can_change_user_company_id, @user %>
  <!-- role checkboxes go here -->
<% end %>

# users_controller.rb
def update
  authorize! :can_change_user_company_id, @user if params[:user][:company_id]
  # ...
end
```


## Pundig

https://github.com/varvet/pundit

## Other authorization

* https://github.com/palkan/action_policy

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

# Action Cable

https://www.sitepoint.com/create-a-chat-app-with-rails-5-actioncable-and-devise/
example
https://github.com/duleorlovic/premesti.se/commit/ad7cefe192cbbb6b97c13860eb6410d1b02cb5fc

Consumer is a client of a web socket connection that can subscribe to one or
multiple channels. Each ActionCable server may handle multuple connections (it
is created per browser tab).

~~~
rails g channel chat
# app/assets/javascripts/channels/chat.coffee
# app/channels/chat_channel.rb
~~~

You can broadcast with

~~~
ActionCable.server.broadcast("chat:#{chat.id}", data: data)
ChatChannel.broadcast_to chat, data: data
~~~

or in javascript

~~~
App.chat = App.cable.subscriptions.create 'ChatChannel'
App.chat.send({data: data})
~~~

You can call server method with `@perform 'away', my_id:
$('main').data('my-id')`

You can not use devise `current_user` when rendering for actioncable, there will
be an error

~~~
ActionView::Template::Error (Devise could not find the `Warden::Proxy` instance on your request environment.
Make sure that your application is loading Devise and Warden as expected and that the `Warden::Manager` middleware is present in your middleware stack.
If you are seeing this on one of your tests, ensure that your tests are either executing the Rails middleware stack or that your tests are using the `Devise::Test::ControllerHelpers` module to inject the `request.env['warden']` object for you.):
~~~

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
* use include for n+1 query (bullet) and joins when you don't need associated
  models. Both are using INNER JOIN
  * `Comment.all(include: :user, conditions: { users: { admin: true}})` will
    load also the user model
* [N+1](http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations)
  problem is solved with `includes(:associated_table)` wich is actually LEFT
  OUTER JOINS, `joins(:associated_table)` is INNER JOIN and do not load the
  models. Works also for nested joins `includes(jobs: :user)` or for multuple
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

In rails belongs_to foreign_key https://guides.rubyonrails.org/v5.2/association_basics.html#options-for-belongs-to-foreign-key
is used is you have different name of the column than name_of_association_id

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
since it remove my custom selected data, so instead `references` I use
`joins('LEFT OUTER JOIN sports ON sports.id = users.sport_id')` and `distinct`.
Distinct is needed to remove duplicated since inner join with has_many relation
(also habtm) returns multiple results (single is only for belongs_to relation).
Here is example of how includes/references do not let me use counts as
calculated columns, but joins let works fine (even .to_sql is similar)
~~~
all = Model
  .select(%(
    users.id,
    COUNT(projects.id) as projects_count
  ))
  .includes(:projects)
  .references(:projects)
all.last.projects_count # does not work
all = Model
  .select(%(
    users.id,
    COUNT(projects.id) as projects_count
  ))
  .joins(%(
    LEFT JOIN projects ON user_id = users.id
  ))

  # do not use .joins(:projects) since it will inner join, ie you will get rows
  # for each user and each project.
  # It is fine if it is belongs_to relation (only one corresponding project)
  all.last.projects_count # here it works
~~~
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
  * if it is standard base helper, than just call it from ActionController,
    ActionView instance or Rails application routes.

  ~~~
  ActionController::Base.helpers.pluralize(count, 'mystring')
  ActionController::Base.helpers.strip_tags request.body # html to text
  ActionController::Base.helpers.link_to 'name', link
  ActionController::Base.helpers.j "can't be blank" # can\'t be blank

  ActionView::Base.new.number_to_human 123123

  Rails.application.routes.url_helpers.jobs_path

  ERB::Util.html_escape t('errors.messages.blank') # can&#39;t be blank
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
  * `request.env["HTTP_USER_AGENT"]`

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
  Position is starting from 1, 2...
  To perform bulk update, use form to pass ids

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
`:draft` and outside of class with `Class.statuses`.

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
  <br> so it looks the same when displaying it (alternativelly you can use `<%=
  simple_format @contact.text %>`).  Example is with nested associated elements:

  ~~~
  params[:contact] = params[:contact].each_with_object({}) { |(k,v),o| o[k] = v.each_with_object({}) { |(k1,v1),o1| o1[k1] = (v1.class == String ? v1.gsub(/\n/,'<br>'): v1) } }
  ~~~

  If you want to strip params globaly (for every non get request) you can use:

  ~~~
  before_action :strip_params
  def strip_params
    return if request.get?
    params
      .values
      .select {|v| [ActionController::Parameters, ActiveSupport::HashWithIndifferentAccess].include? v.class }
      .each do |item_parameters|
        item_parameters.each do |k,v|
          next unless v.class == String
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
* turn of sql debug log messages in console `ActiveRecord::Base.logger = nil`
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
* rails has `attribute_was` to get original value of some field. That previous
value can be read in any hook, for example `after_validation` you can read for
`operator_id && operator_id_was`.
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

## run tests with a command
## rails test config/initializers/colorize.rb
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
  def self.COMMON
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
  Run specific test and include byebug
  ```
  cd actionview
  bundle exec ruby -w -Itest -rbyebug actionview/test/template/form_options_helper_test.rb -n test_select_with_include_blank_false_and_required
  ```
  Commit on your fork and some branch_name. In description of pull requests you
  can include code example
  ```
  ```

* Rails 6 uses Utf8mb4 instead Utf8 so you can store emojis 😀 everywhere, I’m
  💯% sure. Here is plain ascii (￣︶￣). 🙌
* when bcrypt is updated you need to update secrets credentials with `rails
  credentials:edit`
```
.rvm/gems/ruby-2.5.3/gems/activesupport-6.0.0.rc1/lib/active_support/message_encryptor.rb:206:in `rescue in _decrypt': ActiveSupport::MessageEncryptor::InvalidMessage (ActiveSupport::MessageEncryptor::InvalidMessage)
```

  you can create development credentials (config/credentials/development.key and
  config/credentials/development.yml.enc) with
  ```

  rails credentials:edit -e development
  ```
* dhh videos On Writing Software Well
  https://www.youtube.com/watch?v=wXaC0YvDgIo&list=PL9wALaIpe0Py6E_oHCgTrD6FvFETwJLlx
  ```
  ```
* actiontext is included in Rails 6 but you need to add stylesheets
  ```
  # app/assets/stylesheets/application.sass
  //= require actiontext
  ```

  and you can use
  ```
  # app/models/event.rb
    has_rich_text :content

  # app/views/events/_form.html.erb
    f.rich_text_area :content

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
