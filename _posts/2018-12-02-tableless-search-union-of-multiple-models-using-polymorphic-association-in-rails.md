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

Another solution is to use gem https://github.com/pboling/activerecord-tablefree
but we will not use it here.


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

# Postgres full text search

For Postgresql there could be improvements using `pg_search`
https://github.com/Casecommons/pg_search
https://www.postgresql.org/docs/current/textsearch.html
It supports preprocessing and indexing so searching for `satisfy` will result in
`satisfies` (ilike does not include derived words), automatically add index, and
results are ranked (so you do not get thousend of results, but 5 most relevant
for example two words are close to each other *proximity ranking*).
Preprocessing includes:
* parsing documents into tokens using *parser* (numbers, words, complex words)
* converting tokens into lexemes using *dictionaries*. Lexeme is a string but
  it includes all variant (uppercase, suffix). We exclude stop words (is, a, it)
  Map synonyms to a single word using *Ispell*, map phrases to a single word
  using *thesaurus*, map variations of word to canonical form using *Ispell
  dictionary*, map different variation of a word to a canonical form using
  *Snowball stemmer rules*.


Text search types https://www.postgresql.org/docs/current/datatype-textsearch.html
```
SELECT 'The Fast Orlovics'::tsvector; # 'Fast' 'Orlovics' 'The'
# to perform normalisation you can use to_tsvector
SELECT to_tsvector('english', 'The Fast Orlovics') # 'fast':2 'orlov':3
# you can add weight to lexemes A, B, C, D (default) so title words are more
# relevant than body words
SELECT 'a:1A fat:2B,4C cat:5D'::tsvector;
```

Queries
* `&` and, `|` or, `!` not, `<->` phrase or `<1>` phrase with distance
```
SELECT 'the & (orlovic | fast)'::tsquery; # 'the' & ( 'orlovic' | 'fast' )
# perform normalisation of query strings: ignore the, strip suffix
SELECT to_tsquery('the & (orlovics | fast)'); # 'orlov' | 'fast'
```

Peform matching
```
SELECT to_tsvector( 'postgraduate' ) @@ to_tsquery( 'postgres:*' ); # t
```
In addition to `to_tsquery` there is `plainto_tsquery` and `plainto_tsquery`
You can also use implicit conversion `text @@ text` which is equivalent to
`to_tsvector(x) @@ plainto_tsquery(y)`.

todo 12.2.2
https://www.postgresql.org/docs/current/textsearch-tables.html
