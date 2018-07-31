---
layout: post
---

# Net::HTTP

Http client https://docs.ruby-lang.org/en/2.0.0/Net/HTTP.html
Set uri

~~~
uri = URI('http://www.example.com/search.cgi')
~~~

Simple one liner POST

~~~
res = Net::HTTP.post_form(uri, 'q' => ['ruby', 'perl'], 'max' => '50')
~~~

Longer format is with `HTTP.new` or `HTTP.start` block syntax

~~~
http = Net::HTTP.new(uri.hostname, uri.port)
# if you are using https
http.use_ssl = true
res =  http.request(req)
~~~

Set headers

~~~
header = {'Content-Type': 'application/json'}
req = Net::HTTP::Post.new(uri, header)
# or
req['Content-Type'] = 'text/json'
~~~

Set Form data

~~~
req.set_form_data(user: { name: 'Duke' })
# or
req.body = {user: {name: 'Duke'}.to_json
~~~

Basic authentication header

~~~
req.basic_auth username, password
~~~

Handle redirection

~~~
require 'net/http'
require 'uri'

def fetch(uri_str, limit = 10)
  # You should choose better exception.
  raise ArgumentError, 'HTTP redirect too deep' if limit == 0

  url = URI.parse(uri_str)
  req = Net::HTTP::Get.new(url.path, { 'User-Agent' => 'Mozilla/5.0 (etc...)' })
  response = Net::HTTP.start(url.host, url.port) { |http| http.request(req) }
  case response
  when Net::HTTPSuccess     then response
  when Net::HTTPRedirection then fetch(response['location'], limit - 1)
  else
    response.error!
  end
end

print fetch('http://www.ruby-lang.org/')
~~~
~~~
