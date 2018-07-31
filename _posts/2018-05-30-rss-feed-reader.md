---
layout: post
---

# Feedjira

http://feedjira.com/

~~~
feed = Feedjira::Feed.fetch_and_parse url
~~~

Opensource sites that uses feedjira:

* https://github.com/swanson/stringer Nice Sinatra ActiveRecord Backbone app. It
  uses hourly rake task (to create delayed job for each feed to fetch new items)
* https://github.com/amatriain/feedbunch
* https://github.com/feedbin/feedbin
