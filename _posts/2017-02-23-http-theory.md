---
layout: post
title: HTTP theory
---

[HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP) is appliation layer
protocol often based on TCP/IP layer (reliable) and follows classical client
server model.

Requests are sent by ony entity, the user-agent. Usually it is Web browser but
an be a robot that crawls the web to populate and maintain a search engine
index. Server is usually not a single machine, but a collection of servers
sharing the load (load balancing), cache, database server... Between server and
web browser there could be other proxies (logging, authentication, filtering).

Http is stateless, there is no link between two requests, but using Http Headers
we can create Http Cookies allowing session creation to share some context.

Http/1.0 open a TCP connection for each request/response exchange, so it is not
so efficient. Http/2 use multiplexing messages over a single connection, keep
the connection warm and more efficient.

~~~
GET / HTTP/1.1
Host: developer.mozzila.org

body line
~~~

Request consists of start line, headers, empty line and a body. Start line
defines:

* Method: GET,
[POST](https://developer.mozilla.org/en-US/docs/Web/HTTP) HEAD OPTIONS
* Path usually without protocol, domain, port, only absolute path. But it can
vary depending on method (for example `CONNECT asd.asd:80 HTTP/1.1` `OPTIONS *
HTTP/1.1`)
* Protocol Version (HTTP/1.1)

~~~
HTTP/1.1 200 OK
Date: 1 Jan 2017
Server: Apache
~~~

Response consists of status line, headers, empty line and a body. Status line
defines:

* Protocol Version (HTTP/1.1)
* Status code (200)
* Status message (non authoritative short description of status code, purely
informational)


> Using Http headers (not available in Http/0.9) we can set varius properties of
> request and response.

Header syntax is: case insensitive string followed by colon and value on single
line. They can be:

* general headers: like `via`, `connection`, `upgrade-insecure-requests`
* request headers: like `host`, `user-agent`, `accept`, `accept-type`, `accept-language`
* response headers: like `accept-ranges`, `vary`, `access-control-allow-origin`,
`etag`, `server`, `set-cookie`
* entity headers: like `content-type` `content-length`, `content-encoding`,
`last-modified`
  * `content-type` is it used to define body (usually POST requests) which can
  be single resource or multiple resource body (mime is multipart/form-data).
  Single resource usually has `content-lenght` but can have unknown length in
  case of `transfer-encoding`


# Identifying resources on the web

[link](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Identifying_resources_on_the_Web)
target of the Http request is "resourse" which is identified with Uniform
Resource Identifier URI.
URL Uniform Resource Locator (is kind of URI) is most
common form of URI and is known as web address (location)
`http://www.example.com:80/path/to/file.html?key1=value1?key2=value2#SomewhereInDocument`
It consists scheme or protocol (http, data, file, ftp, mailto, ssh, tel, urn,
view-source, ws/wss). Than follows domain name or ip address, port, path
(handled by web server), query (key value pairs separated with &) and fragment
(hash, anchor is never sent to the server).
URN is URI that identifies resource by name in namespace: `urn:ietf:rfc:7230`
[Data
URI](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Identifying_resources_on_the_Web)
can be used to embed small files inline in documents: `data:'image/jpeg',<data>`
or `data:text/plain,<script>alert('hi');</script>`. To encode data into base64
format you can use `uuencode` command line tool. Do not forget to add `,` before
data segment. 

# Mime types

[mime
type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_Types)
has format `type/subtype` (lowercase without space). File extension is not used
in browser, only mime type. [list of all mime
types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types).
Most common type(subtype) is one of the discrete:

* text (plain, html, css, javascript)
  * for css files we have to use mime type `text/css` not (`text/plain`)
  * `text/csv` is used for csv (chrome does not open, but chromium opens libre
  office). Usually do not need to specify since browser can detect that based on
  request (you can simple `render text: 'col1,col2'`)
* image (gif, png, jpeg, bmp, webp)
* audio (midi, mpeg, webm, ogg, wav)
* video (webm, ogg)
* application (xml, pdf, octet-stream)
  * `application/octet-stream` is unknown binary file, same as if
  `Content-Disposition: attachment` header is set, so browser will "Save as"
  * `content-type: application/x-www-form-urlencoded` and body
  `name=Joe%20User&request=Send%20me%20one%20of%20your%20catalogue` is another
  example of request.

or multipart:

* `multipart/form-data` is used for html forms and consists multiple parts
  separated with boundary string that starts with `--` and each part has its own
  mime type (inside Content-Type header) and Content-Disposition, for example

  ~~~
  Content-Type: multipart/form-data; boundary=aBoundaryString
  (other headers associated with the multipart document as a whole)

  --aBoundaryString
  Content-Disposition: form-data; name="myFile"; filename="img.jpg"
  Content-Type: image/jpeg

  (data)
  --aBoundaryString
  Content-Disposition: form-data; name="myField"

  (data)
  --aBoundaryString--
  ~~~

* `multipart/byteranges` is used to send partial content of the files

Mime sniffing is used when client believes that mime type is incorrect, so they
try to guess the correct value by looking at the resource. Server can block
sniffing by sending `X-Content-Type-Options`.

You can use mime url data inside inline html and css, for example:

~~~
<a href="data:text/plain;base64,SGVsbG8sIFdvcmxkIQ%3D%3D" download>Click here to
download file</a>

<img alt="Embedded Image" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIA..." />

div.image {
  width:100px;
  height:100px;
  background-image:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIA...);
}
~~~

If you chose that apex domain is your canonical location, than you need to add
http 301 redirection for `www.example.com` to `example.com`. Or you can use
special link element in head part of the page: `<link
href="http://example.cin/whaddup" rel="canonical">`

Http/1.1 enable warm connections so it does not close after first request is
completed `Connection: keep-alive` header. Drawback is server need to consume
resources to keep open connections (DOS attack).
Pipeling ie second request is send before answer for the first is received. Only
idempotent methods (GET, HEAD, PUT, DELETE) can be pipelined. It is difficult to
implement and browser do not use it (anyway Http/2 multiplexing solve this
problem). Browser simply opens 6 different connections with a server and use
them in parallel.
Cache control mechanism is introduced, `HOST` header enable multiple domains at
the same ip address.

SSL (and later [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security))
protocol is used to secure communication between server and web browser (https):

* connection is private (secure) by using symmetic cryptography
* identity (typically server) can be authenticated using publi-key crypthography
* connection ensures integrity

REST representational state transfer) pattern was introduced in 2000 for complex
applications (API defined using specific URI with basic Http/1.1 methods).

In 2005 extension like `server-sent events` and `websockets`.
In 2010 Google implements protocol SPDY which increase in responsiveness and
solving a problem of data transmitted duplication (inspiration for Http/2)
In 2015 Http/2 standardized with a several differences from Http/1.1
[link](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP)

* it is binary protocol rather than text, so it can no longer be read and
created manually
* it is multiplexed so parallel requests can be send on one connection
* compress headers
* allows server to populate data in client cache

http/2 is transparent to web developers, it is just additional step between
http/1.1 and transport protocol. When it is available on server and browser it
is automatically switched on and used.

In 2016 some important extensions are:

* `Alt-Svc` header for location of the resource
* `Client-Hints` information about client or web browser

# Cache

# Session

Using Http cookies you can link requests to some state of the server (even http
is state-less protocol).

# CORS

https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)
Web browsers enforce strict separation between web sites. Only resources from
*Same origin* can be accessed. But there are exceptions. Cross origin resource
sharing [CORS](https://developer.mozilla.org/en-US/docs/Glossary/CORS)

Allow cors on server so options request are resolved

~~~
echo "gem 'rack-cors', require: 'rack/cors'" >> Gemfile
bundle

sed -i config/application.rb -e '/^  end/i \
\
    config.middleware.insert_before 0, "Rack::Cors" do\
      allow do\
        origins "*"\
        resource(\
          "*",\
          headers: :any,\
          methods: :any,\
          expose: [\
            "access-token",\
            "expiry",\
            "token-type",\
            "uid",\
            "client",\
            "Content-Range", # clean_pagination\
            "Accept-Ranges", # clean_pagination\
          ],\
        )\
      end\
    end'
~~~

You can enable CORS only for specific actions. For example if you are loading
form on other sites to create and update, you can enable cors for that actions.
When you are loading javascript from other domains like
~~~
<script type="text/javascript" src="//localhost:3002/widget/1"></script>
~~~

than on your server you will get error

~~~
ActionController::InvalidCrossOriginRequest (Security warning: an embedded <script> tag on another site requested protected JavaScript. If you know what you're doing, go ahead and disable forgery protection on this action to permit cross-origin JavaScript embedding.):
~~~

To solve that add

~~~
  skip_before_action :verify_authenticity_token, only: [:widget]
~~~

When you want to submit form from another domain you get this error

~~~
ActionController::InvalidAuthenticityToken (ActionController::InvalidAuthenticityToken):
~~~
So I solve that by adding

~~~
  skip_before_action :verify_authenticity_token, only: [:widget, :create, :update]
~~~

This is when your controller is protected from CSRF attack (it contains
protect_from_forgery) .

To see response from submitted form request, you see in javascript console this
warning and $.ajax success callback is never triggered

~~~
Failed to load http://localhost:3002/days/1: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost' is therefore not allowed access.
~~~

To solve that add

~~~
  after_action :set_access_control_headers, :only => [:create, :update]

  def set_access_control_headers
    headers['Access-Control-Allow-Origin'] = '*'
  end
~~~

# CSP

Content security policy
[CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

# Authentication

You can use `WWW-Authenticate` header or setting sessiong key using Http cookie.

