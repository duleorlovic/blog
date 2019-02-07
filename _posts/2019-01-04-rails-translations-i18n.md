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
# pluralize
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
You can also see some default en translations for errors.
To see Rails default datetime formats go to
https://github.com/svenfuchs/rails-i18n/blob/master/rails/locale/en.yml
to see current translation you can use
```
I18n.translate 'date.formats.default`
=> "%Y-%m-%d"
```

And you can change attribute name `activerecord.attributes.user.email: имејл`
To translate also plurals you can use `User.model_name.human(count: 2)`. For
attributes you can use `User.human_attribute_name("email")`
[link](http://guides.rubyonrails.org/i18n.html#translations-for-active-record-models)

~~~
en:
  activerecord:
    models:
      user:
        zero: No dudes
        one: Dude
        other: Dudes
~~~

Separate translations into different files (for example
`activerecord_models.sr.yml`) and folders for `/completed/activerecord.sr.yml`
include them with:

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

For form objects `include ActiveModel::Model` you should translate
`activemodel`. For `ApplicationRecord` translate `activerecord`.
~~~
# config/locales/activemodel.sr.yml
sr:
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
    models:
      user:
        one: корисник
        other: корисници
        accusative: корисника
        some_customer_message: Моја порука
~~~

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
require "i18n/backend/pluralization"
I18n::Backend::Simple.send(:include, I18n::Backend::Pluralization)
~~~

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


For google translate look for two scripts, one for vim and one for whole yml.
https://github.com/duleorlovic/config/tree/master/ruby
