---
layout: post
---

# Localisation i18n translations

Tips <https://devhints.io/rails-i18n>
Translate models using `activerecord`
https://guides.rubyonrails.org/i18n.html#translations-for-active-record-models
so you can use
```
User.model_name.human
# 'asd'.pluralize # asds so for english that is ok but if you use i18n, than use
# note that you have to write translations in yml.file if you use human(count)
User.model_name.human(count: 2)
# attribute
User.human_attribute_name(:email)
```

To translate active record messages for specific attributes, you can overwrite
messages for specific model and attributes (default ActiveRecord messages taken)
<https://github.com/rails/rails/blob/master/activerecord/lib/active_record/locale/en.yml#L23>
<https://apidock.com/rails/v4.2.7/ActiveModel/Errors/generate_message>

Also you can change format `errors.format: Polje "%{attribute}" %{message}`
https://github.com/rails/rails/blob/master/activemodel/lib/active_model/locale/en.yml#L4

And you can change attribute name `activerecord.attributes.user.email: имејл`
To translate also plurals you can use `User.model_name.human(count: 2)`. For
attributes you can use `User.human_attribute_name("email")`
[link](http://guides.rubyonrails.org/i18n.html#translations-for-active-record-models)
For `ApplicationRecord` translate `activerecord`.
For form objects `include ActiveModel::Model` you should translate
`activemodel`. There you can use `t('successfully')` instead
`I18n.t('successfully')` if you `include AbstractController::Translation`

~~~
# config/locales/activerecord_activemodels.en.yml
en:
  activerecord:
    models:
      user:
        zero: No dudes
        one: Dude
        other: Dudes
      customer:
        one: корисник
        other: корисници
        accusative: корисника
        some_customer_message: Моја порука
  activemodel:
    attributes:
      landing_signup:
        current_city: Који је твој град ?
    errors:
      messages:
        group_not_exists_for_age: Не постоји група (%{age}год) на овој локацији
      models:
        landing_signup:
          attributes:
            current_city:
              blank: Не може бити празно ?
~~~

Separate translations into different files (for example
`activerecord_activemodels.sr.yml`) include them with:

~~~
# config/application.rb
config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]

# Whitelist locales available for the application
I18n.available_locales = [:en, :pt]

# Set default locale to something other than :en
I18n.default_locale = :pt
~~~

To raise error when translation is missing `.translation_missing` class
```
# config/application.rb
config.action_view.raise_on_missing_translations = true
```

This way you can change default_locale and run system tests and you will get
exception if some locale is missing.

No need to write quotes in yml unless you have:
* colon `:` , than simply escape it's meaning and use quotes `name: "name: Ime"`
* start with `%{}` or `,` or `.`
* have `answer: Yes` or `answer: No`, is somehow casts to `true` or `false`
* have double quotes inside `processing: <i class="fa fa-spinner fa-spin
 datatable-spinner"></i>Обрада...`. Than you need to switch to single quotes.
 Note that it does not help to wrap double quotes with single quote, this won't
 work: `processing: '<i class="c"></i>'` since you have double quote inside.

When debugging `SyntaxError: [stdin]:60:33: unexpected identifier` you should
run `rake tmp:clear` so all yml end .erb files are compiled again.

If you want to reuse same translation you can use alias, but only inside same
file, so in case of form object, you can add to activerecord_models.yml

```
sr:
  activerecord:
    attributes:
      subscriber: &subscriber_attributes
        name: Назив
      project: &project_attributes
        name: Назив
  activemodel:
    attributes:
      project_task_notification:
        <<: *project_attributes
        <<: *subscriber_attributes
        project_name: Назив
```


For custom errors can be different for each attribute or same. Can also accept
param, for example

~~~
    errors.add :from_group_age, :group_not_exists_for_age, age: age
~~~

https://stackoverflow.com/questions/6166064/i18n-pluralization
For serbian you can provide pluralization

~~~
# config/locales/plurals.rb
# https://github.com/svenfuchs/i18n/blob/master/test/test_data/locales/plurals.rb
serbian = {
  i18n: {
    plural: {
      keys: %i[one few many other],
      rule: lambda { |n|
        if n % 10 == 1 && n % 100 != 11
          :one
        elsif [2, 3, 4].include?(n % 10) && ![12, 13, 14].include?(n % 100)
          :few
        # elsif (n % 10).zero? || [5, 6, 7, 8, 9].include?(n % 10) || [11, 12, 13, 14].include?(n % 100)
        #   :many
        # there are no other integers, use :many if you need to differentiate
        # with floats
        else
          :other
        end
      }
    }
  }
}
{
  sr: serbian,
  'sr-latin': serbian,
}
~~~

~~~
# config/initializers/pluralization.rb
require 'i18n/backend/pluralization'
I18n::Backend::Simple.send(:include, I18n::Backend::Pluralization)
~~~

If you send `count` parameter `t('sent_messages', count: 2)` it will try to pick
specific item based on number
~~~
# config/locales/sr.yml
sr:
  sent_messages:
    # 1, 21, 31 ...
    one: %{count} порука је послата
    # 2, 3, 4, 22, 23, 24, 32, 33, 34 ...
    few: %{count} поруке су послате
    # all other integers: 5, 6, ... 9, 10, 11, 12, 13, 14 ... 20, 25, ...
    many: %{count} порука је послато
~~~

Note that you have to provide `few` translation for all words, since it could
happend that count is 2 and translation is missing.

~~~
I18n.t 'sent_messages', count: 15
# or if you want to translate model
"#{chat.moves.size} #{Move.model_name.human count: chat.moves.size}"
~~~

You can translate to any language with

~~~
I18n.t 'sent_messages', locale: :sr
~~~

Example for Serbian localizations translations:

~~~
# config/locales/sr.yml
sr:
  # https://github.com/rails/rails/blob/master/activemodel/lib/active_model/locale/en.yml#L8
  errors:
    format: Поље "%{attribute}" %{message}
    messages:
      blank: не сме бити празно
      invalid: није исправно
  neo4j:
    errors:
      messages:
        required: мора постојати
        taken: је већ заузет
    models:
      user: корисник
      location:
        one: локација
        other: локације
    attributes:
      user:
        email: Имејл
        password: Лозинка
        password_confirmation: Потврда лозинке
        remember_me: Запамти ме
~~~

When you use `.capitalize`, `.titleize` or `.upcase` than you need first to call
`.mb_chars`. For example
~~~
'ž'.upcase
=> "ž"

'ž'.mb_chars.upcase.to_s
 => "Ž"
~~~
Some common words translations can be found
<https://github.com/svenfuchs/rails-i18n/blob/master/rails/locale/en.yml>

To translate with accusative you need to joins strings or use param in
translation

~~~
module TranslateHelper
  # there are two ways of calling this helper:
  # t_crud 'are_you_sure_to_remove_item', item: @move
  # t_crud 'edit', User
  def t_crud(action, model_class)
    if model_class.class == Hash
      t(action, item: t("neo4j.models.#{model_class[:item].name.downcase}.accusative"))
    else
      "#{t(action)} #{t("neo4j.models.#{model_class.name.downcase}.accusative")}"
    end
  end
end
~~~

Translate latin to cyrilic with <https://github.com/dalibor/cyrillizer> You need
to set language in config

~~~
# Gemfile
# translate cyrillic
gem 'cyrillizer'
~~~

~~~
# config/initializers/cyrillizer.rb
Cyrillizer.language = :serbian
~~~

In console

~~~
'my string'.to_cyr
 => "мy стринг"
~~~

Note that some chars looks the same but are not when rendered on html page
~~~
 # for example first line is not correct link a href
 <a href='%{confirmation_url}'>Поново пошаљи упутство за потврду</а>"
 <a href='%{confirmation_url}'>Поново пошаљи упутство за потврду</a>"
~~~

# Locale

When changing locale `I18n.locale = :sr` in some methods, note that this is
global variable in thread, so when you have 5 puma threads than on GET requests
(simply refresh couple of times) you will get different locales.
Here is my Rails controbution to guide about it https://github.com/rails/rails/pull/34911
One way is to use `I18n.with_locale` for example

```
class UserMailer < ActionMailer::Base
  default from: 'noreply@translation.io'

  def invitation(user)
    I18n.with_locale(user.locale) do
      mail subject: t('invitation'), to: user.email
    end
  end
end
```

For controller, you need to use around filters

```
  around_action :set_locale_from_session

  def set_locale_from_session
    if session[:locale].present?
      I18n.with_locale session[:locale].to_sym do
        yield
      end
    else
      yield
    end
  end
```

# Translating user content

https://github.com/shioyama/mobility#quickstart

```
# Gemfile
# translation
gem 'mobility', '~> 0.8.6'

# this will generate config/initializers/mobility.rb
rails g mobility:install

# app/models/activity.rb
class Activity < ApplicationRecord
  extend Mobility
  translates :name
end

# in migration add default value
  create_table :activities, id: :uuid do |t|
    t.json :name, default: {}
```

For google translate look for two scripts, one for vim and one for whole yml.
https://github.com/duleorlovic/config/tree/master/ruby

fallbacks
```
club.name fallback: false
club.name fallback: [:en]
```

If fallback is not false than longer translate will fallback to short
automatically (`'sr-latin'` to `:sr`)

Note that passing `locale` options to reader or using locale accessors will
disable fallbacks
```
word.meaning(locale: :de)
#=> nil
word.meaning_de
#=> nil
Mobility.with_locale(:de) { word.meaning }
# if in model we have translate :meaning, fallbacks: { de: :ja}
#=> "(名詞):動きやすさ、可動性"
```

Global fallback
```
# config/initializers/mobility.rb
  config.default_options[:fallbacks] = { sr: :en, en: :sr }
```
Dynamic fallback https://github.com/shioyama/mobility/pull/328
https://github.com/shioyama/mobility/issues/314

You can search find_by using `@>` and passing json
```
Activity.where("name @> ?",{en: 'Climbing'}.to_json)
# or using i18n scope
Activity.i18n.where(name: 'Climing')
```

# Enums

```
# app/models/user.rb
class User < ApplicationRecord
  enum status: [:active, :pending, :archived]
end

# app/models/application_record.rb
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true

  def self.human_enum_name(enum_name, enum_value)
    I18n.t("activerecord.attributes.#{model_name.i18n_key}.#{enum_name.to_s.pluralize}.#{enum_value}")
  end
end


# config/locales/activerecord.en.yml
en:
  activerecord:
    attributes:
      user:
        statuses:
          active: "Active"
          pending: "Pending"
          archived: "Archived"

# usage:
User.human_enum_name(:status, :pending)
User.human_enum_name(:status, user.status)
```

# Tips

* if you want to see which translation is used (if there is parent default
  value) you can try https://github.com/fphilipe/i18n-debug
* for big content pages, you can translate using `page.sr.html.erb` instead of
  yml translations
* fallbacks, if can not find the key in curreny localy, it can search in
  fallback locale. In this case you can translate only specific keys
  ```
  # config/application.rb
  config.i18n.fallbacks = { en_GB: [:en] }

  # config/locales/en_GB.yml
  en_GB:
    soccer: Footbal

  en:
    soccer: Soccer
    words: Words

  # logs from i18n-debug
  en_GB.show.soccer => 'Footbal'
  en_GB.show.words => nil
  en.show.words => 'Words'
  ```

* To see Rails default datetime formats go to
  https://github.com/svenfuchs/rails-i18n/blob/master/rails/locale/en.yml
  to see current translation you can use
  ```
  I18n.translate 'date.formats.default`
  => "%Y-%m-%d"
  ```
  Use can check formats https://apidock.com/ruby/DateTime/strftime with
  ```
  Time.zone.now.strftime '%e %B'
  ```

* to localise date and timestamps you can use `I18n.l Time.now, format: :formal`

  ```
  en:
    time:
      formats:
      default: '%H:%M'
      formal: The time is %H:%M

  ```
* `helper.number_with_precision`, `distance_of_time_in_words_to_now`,
  `number_to_currency`, `number_to_human`, `number_to_human_size` are some
  helpers that uses translations
* use `_html` suffix when you use tags in translated content. For example
  `title_html: Hi <b>man</b>` and `<%= t('title_html') %>` will not escape and
  you do not need to use html_safe or raw
* to add route instead of subdomain, use https://stackoverflow.com/a/8237800
  ```
  # config/routes.rb
    devise_for :users, only: :omniauth_callbacks, controllers: {
      omniauth_callbacks: 'devise/my_omniauth_callbacks',
    }
    scope '(:locale)', locale: /#{I18n.available_locales.join("|")}/ do
      root 'pages#home'

      devise_for :users, skip: :omniauth_callbacks, controllers: {
        registrations: 'devise/my_registrations',
      }
    end

  # config/
  ```
