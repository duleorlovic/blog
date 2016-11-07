---
layout: post
title: RoR polymorphic or sti
---

# Polymorphic

[Polymorphic](http://guides.rubyonrails.org/association_basics.html#polymorphic-associations)
associations are used to add refence to something that is shared in different
models, for example locations and banners has many clicks so clicks belongs to
both locations and banners, so let's call them `clickable`.

~~~
# db/migrate/123_add_polymorphic.rb
  def change
    add_reference :clicks, :clickable, polymorphic: true, index: true
  end

# app/models/click.rb
class Click < ActiveRecord::Base
  belongs_to :clickable, polymorphic: true
end

# app/models/location.rb
class Location < ActiveRecord::Base
  has_many :clicks, as: :clickable
  validates_presence_of :name, :link_url
end

# app/models/banner.rb
class Banner < ActiveRecord::Base
  has_many :clicks, as: :clickable
  validates_presence_of :name, :link_url
end
~~~

Usage is the same as it is for regular one-to-many relations.

# Single table inheritance

[STI](http://eewang.github.io/blog/2013/03/12/how-and-when-to-use-single-table-inheritance-in-rails/)
http://enterpriserails.chak.org/full-text/chapter-10-multiple-table-inheritance
http://thibaultdenizet.com/tutorial/single-table-inheritance-with-rails-4-part-1/
https://www.youtube.com/watch?v=t8I4_8HcMPo

STI maps all fields into a single table
http://martinfowler.com/eaaCatalog/classTableInheritance.html CTI maps each
class
into its own table http://martinfowler.com/eaaCatalog/classTableInheritance.html
