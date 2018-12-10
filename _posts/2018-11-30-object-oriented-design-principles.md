---
layout: post
---

https://thoughtbot.com/upcase/intermediate-ruby-on-rails

[dhh tips for rails](https://www.youtube.com/watch?v=D7zUOtlpUPw)
* epipsode 1: do not use comments but method names or constants... follow
  table of content: method definition should be in same order as they are used
  in the class (in before callbacks)
  administered concern define methods on associations. Also use `or` `|` to join
  two arrays

~~~
module Account::Administered
  extend ActiveSupport::Concern

  included do
    has_many :administratorships, dependent: :delete_all do
      def grant(person)
        create_or_find person: person
      end

      def revoke(person)
        where(person_id: person.id).destroy_all
      end
    end

    has_many :administrators, through: :administratorships, source: :person
  end

  def all_administrators
    administrators | all_owners
  end

  def administrator_candidates
    people.users.
      where.not(id: administratorships.pluck(:person_id)).
      where.now(id: ownerships.pluck(:person_id)).
      where.not(id: owner_person.id)
  end
end
~~~
* episode 2: use callbacks to initiate background jobs
* episode 3: use globals in request/response cycle. For background jobs you need
  to pass them as params.
* episode 4

