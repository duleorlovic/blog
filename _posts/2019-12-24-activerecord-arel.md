---
layout: post
---

# Arel

https://github.com/rails/arel/tree/7-1-stable
https://github.com/rails/rails/blob/master/activerecord/lib/arel/predications.rb
https://www.youtube.com/watch?v=ShPAxNcLm3o&feature=youtu.be

`left_outer_joins` is available in [recent
rails](https://guides.rubyonrails.org/active_record_querying.html#left-outer-joins)

Online SQL to Arel [www.scuttle.io](http://www.scuttle.io) EXCELLENT
Playground https://jpospisil.com/2014/06/16/the-definitive-guide-to-arel-the-sql-manager-for-ruby.html
```
bundle console
```

cheatsheet https://devhints.io/arel
```
arel_table = User.arel_table
arel_table = Arel::Table.new(:users)
arel_col = arel_table[:email]

# select fields
# NOTE that AS should use string instead of symbols
arel_table.project(arel_col, arel_col.as('custom_name')).to_sql
# aggregates
arel_table.project arel_table[:age].sum
arel_table.project arel_table[:age].count.as('user_count')

# limit offset
arel_table.take(3).skip(2)

# order
arel_table.order(arel_col, arel_col.desc)

# where restrictions
arel_table.where arel_col.eq 'my@email.com'
arel_col.eq 'string'
arel_col.gteq Date.today.beginning_of_month
arel_col.matches "%#{query}%"  # generate ILIKE

# AR use joins(:photos) but Arel use join
photos = Photo.arel_table
arel_table.join(photos, Arel::Nodes::OuterJoin).on(arel_col.eq photos[:user_id]).project(photos[:name].as('photo_name'))

# functions
SEPARATOR = Arel::Nodes.build_quoted(' ')

Arel::Nodes::NamedFunction.new(
  'concat',
  [arel_table[:first_name], SEPARATOR, arel_table[:last_name]]
)
```

True relation example is (found using online arel editor http://www.scuttle.io )
```
Arel::Nodes::SqlLiteral.new('1').eq(1)

# Arel::Nodes::SqlLiteral.new('true') is also true relation, but can not be
# first, ie Arel::Nodes::SqlLiteral.new('true').and raises error
# NoMethodError (undefined method `and' for "true":Arel::Nodes::SqlLiteral):
```

To execute you can use
```
User.select(Arel.star).to_sql # SELECT * FROM users
# note that if you use joins it will select all columns (id of joined table will
# override id of User) so better is to use with table
User.select(User.arel_table[Arel.star]).to_sql # SELECT users.* FROM users
User.select(User.arel_table[:email].as('m')).first.m

User.where(User.arel_table[:email].eq('Hi')) # it the same as User.where(email: 'Hi')
User.where(User.areal_table[:log_count].gt(200)) # SELECT users.* FORM users WHERE (users.log_count > 200)
User.where(
  User.arel_table[:email].eq('hi').and(
    User.arel_table[:id].eq(22).or(
      User.arel_table[:id].in(23,34)
    )
  )
)
```

# Tips

* in one request there should be only one query per table
* instead of `count` use `.size` (which will use ruby `.length` if loaded). To
  force loading (for example to show sieze before `.each` is performed) you can
  use `users.load.size`. In this case there is no edditional query for
  `users.each`. Only way when `.count` is used is when you actually do not load
  all records, for example showing some link for all items. Similarly do not use
  `empty?`, `any?` without `load`. Do not use `present?` or `blank?` if you do
  not need to load all. `exists?` will always executes sql query.
* do not write
  [QueryMethods](https://api.rubyonrails.org/classes/ActiveRecord/QueryMethods.html),
  [FinderMethods](https://api.rubyonrails.org/classes/ActiveRecord/FinderMethods.html),
  and [Calculations](https://api.rubyonrails.org/classes/ActiveRecord/Calculations.html)
  inside methods on ActiveRecord objects since someone will eventually use then
  in index action and N+1 will be triggered. Use scopes instead if you need all
  nested recources (you can have several associations based on one table)
  ```
  class Post
    has_many :comments
    has_many :active_comments, -> { active }, class_name: "Comment"
  end

  class Comment
    belongs_to :post
    scope :active, -> { where(soft_deleted: false) }
  end

  class PostsController
    def index
      @posts = Post.includes(:active_comments)
    end
  end
  ```

* to fetch all indexes for given table name use
  ```
  @connection ||= ActiveRecord::Base.connection
  @tables ||=
    if ActiveRecord::VERSION::MAJOR == 5
      connection.data_sources
    else
      connection.tables
    end
  @models = ActiveRecord::Base.descendants
  # reject table_name == 'schema_migrations'
  @connection.indexes(@model.table_name)
  ```
* to find belongs_to associations
  ```
  model.reflects_on_all_associations(:belongs_to).
  ```
