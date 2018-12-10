---
layout: post
---

Common problem is to enable search on the site, but with differt kind of results
(posts, comments, books). So we need to combine different models in one action
and still enable pagination and sorting (so merge arrays is not considered
here).

Also, you might need to show all instances for
searched used user (`belongs_to :created_by`, or habtm `book_users`).

Solution is to create new model without table (tabless active record). For rails
4 https://gist.github.com/dalibor/228654 but for Rails 5 we need to override `load_schema!`
https://stackoverflow.com/questions/41494951/how-to-create-activerecord-tableless-model-in-rails-5

```
# app/models/abstract_tableless_model.rb
# https://stackoverflow.com/a/45743718/287166
# to define you tableless model you can
class AbstractTablelessModel < ApplicationRecord
  self.abstract_class = true

  def self.attribute_names
    @attribute_names ||= attribute_types.keys
  end

  def self.load_schema!
    @columns_hash ||= Hash.new

    # From active_record/attributes.rb
    attributes_to_define_after_schema_loads.each do |name, (type, options)|
      if type.is_a?(Symbol)
        type = ActiveRecord::Type.lookup(type, **options.except(:default))
      end

      define_attribute(name, type, **options.slice(:default))

      # Improve Model#inspect output
      @columns_hash[name.to_s] = ActiveRecord::ConnectionAdapters::Column.new(name.to_s, options[:default])
    end

    # Apply serialize decorators
    attribute_types.each do |name, type|
      decorated_type = attribute_type_decorations.apply(name, type)
      define_attribute(name, decorated_type)
    end
  end

  def persisted?
    false
  end
end
```

We use `left_outer_joins` on searchable for which there is only one instance.
But if you need to search for some nested has_many relation (for example
`book_users`) so there could be multiple instances than we need to use
`distinct`
If you want to split on table to two models based on one fields
```
class Book < ApplicationRecord
end

class ShortBook < Book
end

class LongBook < Book
end
```
than you shoud know that `where(long_book: { user: 'me' })` produces the same
result as `where(short_book: { user: 'me' })`.

One solution could be to use database views
```
CREATE VIEW search AS
  SELECT ...
  UNION ALL
  SELECT ...
```
But for updating database view we need migration, so I prefer to have that SQL
in the code.

Similar to but without database query
https://blog.bigbinary.com/2016/05/30/rails-5-adds-or-support-in-active-record.html

For Postgresql there could be improvements using `pg_search`
https://github.com/Casecommons/pg_search
