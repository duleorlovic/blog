---
layout: post
---

# Net::HTTP

Http client https://docs.ruby-lang.org/en/2.0.0/Net/HTTP.html
Set uri

~~~
uri = URI('http://www.example.com/search.cgi')

# add some params to uri
params = { :limit => 10, :page => 3 }
uri.query = URI.encode_www_form(params)
~~~

Simple one liner `Net::HTTP.get`

~~~
# you can get body (without status)
response_body = Net::HTTP.get uri
# but I prefer to fetch also the status
response = Net::HTTP.get_response uri
~~~

Simple one liner `Net::HTTP.post_form`

~~~
response = Net::HTTP.post_form(uri, 'q' => ['ruby', 'perl'], 'max' => '50')
~~~

Longer format is with `HTTP.new` or `HTTP.start` block syntax (better since you
do not need to call `http.start` and it automatically close connection). Longer
means than you create separate `request` object and use with http connection to
make request

~~~
http = Net::HTTP.new(uri.host, uri.port)
request = Net::HTTP::Get.new '/'
response = http.request request
~~~

For https you need to enable ssl. If url is https but you do not use ssl than
you will get following error `EOFError: end of file reached`

~~~
# if you are using https
http.use_ssl = true
# or the best is to use
http.use_ssl = (uri.scheme == "https")
# you can also disable verify ssl with
http.verify_mode = OpenSSL::SSL::VERIFY_NONE

# or in block mode

res = Net::HTTP.start(url.host, use_ssl: true, verify_mode: OpenSSL::SSL::VERIFY_NONE) {|http| http.request(req) }
~~~

You can define timeouts

~~~
http.open_timeout = 1
# when response takes to much time
http.read_timeout = 3
# when you POST a lot of data
http.write_timeout = 10
~~~

On request object you can set headers

~~~
header = {'Content-Type': 'application/json'}
request = Net::HTTP::Post.new(uri, header)
# or
request['Content-Type'] = 'text/json'
~~~

On response you can read headers

~~~
response.each_key {|k| res[k]}
{"server"=>["nginx/1.6.2"], "date"=>["Wed, 17 Oct 2018 22:22:51 GMT"], "content-type"=>["text/html"], "content-length"=>["184"], "connection"=>["close"], "location"=>["https://paytm.com/business/payments"]}
~~~

Set Form data

~~~
request.set_form_data(user: { name: 'Duke' })
# or
request.body = { user: { name: 'Duke' } }.to_json
# or plain text when request['Content-Type'] = 'application/x-www-form-urlencoded'
# ie in Postman there is no json curly braces
request.body = 'grant_type=client_credentials'
~~~

Basic authentication header

~~~
request.basic_auth username, password
~~~

# Snippets

Handle redirection

~~~
require 'net/http'
require 'uri'

def fetch(uri_str, limit = 10)
  # You should choose better exception.
  raise ArgumentError, 'HTTP redirect too deep' if limit == 0

  url = URI.parse(uri_str)
  request = Net::HTTP::Get.new(url.path, { 'User-Agent' => 'Mozilla/5.0 (etc...)' })
  response = Net::HTTP.start(url.host, url.port) { |http| http.request(request) }
  case response
  when Net::HTTPSuccess     then response
  when Net::HTTPRedirection then fetch(response['location'], limit - 1)
  else
    response.error!
  end
end

print fetch('http://www.ruby-lang.org/')
~~~

# HTTParty

This gem is a little bit simpler than plain Net::HTTP.
It does not support multipart requests (needed for uploading a file)

# RestClient

~~~
require 'rest-client'
RestClient.post '/profile', file: File.new('photo.jpg', 'rb')
~~~

# Faraday

Faraday allows you to choose any implemetnation (net/http, typhoeus)
~~~
require 'faraday'
conn =
Faraday.new do |f|
  f.request :multipart
  f.request :url_encoded
  f.adapter :net_http
end
file_io = Faraday::UploadIO.new('photo.jpg', 'image/jpeg')
conn.post('http://example.com/profile', file: file_io)
~~~

# Debug requests

Ruby one liner http server

~~~
ruby -rsocket -e "trap('SIGINT') { exit }; Socket.tcp_server_loop(8080) { |s,_| puts s.readpartial(1024); puts; s.puts 'HTTP/1.1 200'; s.close }"
~~~

And open <http://localhost:8080/> to see all headers and body

~~~
GET / HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.32 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9,sr;q=0.8,hr;q=0.7,bs;q=0.6,da;q=0.5,pt;q=0.4,bg;q=0.3
Cookie: _session_id=...
~~~
