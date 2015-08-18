---
layout: post
title:  Rails settings in cache and beta features
categories: ruby-on-rails settings cache beta-features
---

Did you ever wanted to use some configuration options, but you find very difficult to use secrets.yml file (*secrets.yml* file should be used for secrets, right ?), tired of using `heroku config` to often, or even worse, deploying each time you change some constant in your code ?

Here is a nice solution how to use *Rails.cache* to quickly change some params without need to restart rails application. Its Rails 3 ready. Some credits go to [catarse_settings](https://github.com/catarse/catarse_settings_db/blob/master/app/models/catarse_settings_db/setting.rb). So here is my beta features code that is accessible for *admin* user:

~~~
# app/models/my_setting.rb
class MySetting < ActiveRecord::Base
  validates_presence_of :name

  # Migration file:
  # class CreateMySettings < ActiveRecord::Migration
  #   def change
  #     create_table :my_settings do |t|
  #       t.string :name
  #       t.text :value
  #       t.text :description
  #     end
  #   end
  # end

  # Never update values directly, use MySetting[:foo]='one,two'
  # Return value is always string.
  # For unknown key, empty string is returned
  # Integers are OK in format "123" so .to_i can be safelly performed
  # (all money value should be in cents)

  class << self
    # This method returns the values of the config simulating a Hash, like:
    #   MySetting[:foo]
    # It can also bring Arrays of keys, like:
    #   MySetting[:foo, :bar]
    # ... so you can pass it to a method using *.
    # It is memoized, so it will be correctly cached.
    def [] *keys
      if keys.size == 1
        get keys.shift
      else
        keys.map{|key| get key }
      end
    end
    def []= key, value
      set key, value
    end

    def get_without_cache(key)
      # return nil in case of unknown key so we know that it is not set
      find_by_name(key).value rescue nil
    end

    private
    def get key
      Rails.cache.fetch("/configurations/#{key}") do
        get_without_cache(key)
      end
    end

    def set key, value
      find_or_create_by!(name: key).update(value: value)
      Rails.cache.write("/configurations/#{key}", value)
      value
    end
  end
  
end
~~~


In all places in my rails application I use this `beta` helper method that uses two MySetting variable. `live_features` are comma separated list of feature names that are live, and `beta_users` are emails for users that can see non live features (of course only when they are logged in).

~~~
# app/helpers/application_helper.rb

def beta(symbol_of_feature, content = nil)
  # example of usage:
  # <% if beta :name_of_feature %>
  #   <div>Something</div>
  # <% end %>
  # <%= beta :name_of_feature, render( "something") %>
  # some_code if beta(:name_of_feature)
  #
  # to make feature available to all, add it to MySetting[:live_features]
  # some users can see non live features if there are listed in MySettings[:beta_users]
  # for features on non logged in pages (for example GET homepage) you can override default with url param:
  # ?enable_feature=name_of_feature

  # to catch emails in this example "asd@asd.asd,\r\ndsa@dsa.dsa"
  r = Regexp.new(/\b[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,4}\b/)     

  if MySetting[:live_features].split(/[\s,]+/).include?(symbol_of_feature.to_s) ||
     (current_user && MySetting[:beta_users].scan(r).include?(current_user.email)) ||
     (![nil, "method"].include?(defined?( params)) && params[:enable_feature] == symbol_of_feature.to_s) # in mailer params are not available so check before use it

    if content.present?
      content
    else
      true
    end
  else
    false
  end
end
~~~


To keep track of features I usually write them in seed file

~~~
# db/seeds.rb
[
  {:name => 'signup_bonuse', :value => '2500', :description => 'This is bonuse for new customers. Set it to 0 if you do not want to give this bonuse'},
  {:name => 'beta_users', :value => 'asd@asd.asd, admin@asd.asd', :description => 'List of emails of users that can see non live features'},
  {:name => 'live_features', :value => 'signup_bonuse_feature', :description => 'List of features that all users can use. Note that beta_users can always see all features'},
].each do |doc| 
  next if MySetting.where( name: doc[:name]).present?
  MySetting[doc[:name]] = doc[:value]
  puts "MySetting #{doc[:name]} seeded."
end
~~~
