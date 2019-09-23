---
layout: post
---

https://code.tutsplus.com/articles/a-beginners-guide-to-design-patterns--net-12752

Three basic kind of design pattern:
* structural
* creational
* behavioral


https://www.youtube.com/watch?v=3SGRjUWScRw&feature=youtu.be

Instead of

~~~
class Member < ActiveRecord::Base
  has_many :email_addresses
  has_one :primary_email_address, conditions: { primary: true }
end
~~~

Use belongs_to on members

~~~
class Member < ActiveRecord::Base
  has_many :email_addresses
  belongs_to :primary_email_address, class_name: "EmailAddress", foreign_key: :email_address_id
end
~~~

https://thoughtbot.com/blog/handling-associations-on-null-objects
