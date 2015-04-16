
For $20/month you can use heroku plugin [SSL Endpoint](https://devcenter.heroku.com/articles/ssl-endpoint) and follow simple steps:

~~~
# generate private key
openssl genrsa -des3 -out server.pass.key 2048
# strip password from it
openssl rsa -in server.pass.key -out server.key
# generate CSR
openssl req -nodes -new -key server.key -out server.csr
# use *.domainname.com to match all subdomains
~~~


But if you want to byepass heroku, you can use AWS CloudFront. Source is Kein blog post](https://ksylvest.com/posts/2014-05-06/setup-free-ish-ssl-tls-on-heroku-for-ruby-on-rails-or-any-other-framework)
