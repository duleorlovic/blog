---
layout: post
tags: https ssl apache nginx
---

# SSL using lets encrypt

https://certbot.eff.org/docs/what.html
> Digital certificate uses a public key and a private key to enable secure
> communication between a client program (web browser, email client, etc.) and a
> server over an encrypted SSL (secure socket layer) or TLS (transport layer
> security) connection.

To install you can check instructions on https://certbot.eff.org or use
`certbot-auto` command. It is usually like
```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update

sudo apt-get install certbot python-certbot-apache
```

It will write to `/etc/letsencrypt`, `/var/log/letsencrypt` and
`/var/lib/letsencrypt`.

To just get certificate you can run `certbot certonly`.
To select server which you are running add option like `certbot --apache` or
`certbot --nginx`. It will find your domains from server configuration.
To obtain certificate using 'stangalone' webserver `certbot --standalone` which
will bind to port 80, you need to stop current apache server.

https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-18-04
This super script is so good that you need just one command:

```
sudo certbot --nginx  -d premesti-se.trk.in.rs -d en-premesti-se.trk.in.rs -d sr-latin-premesti-se.trk.in.rs -d premesti.se -d www.premesti.se
```

Check if nginx is listening on port 443 for ssl https

```
sudo netstat -lptun | grep nginx
```

to check if it is accepting connections

```
nmap localhost
# vi /etc/hosts to add mydomain.com
curl https://mydomain.com
```

You can not `curl https://192.168.1.3` or `curl https://localhost` since
certificate is not valid.

To see current obtained certificates
https://certbot.eff.org/docs/using.html#managing-certificates
```
certbot certificates
```

If you have forced ssl on rails server than to obtain certificate, but if server
is already running it will redirect to https://new-domain and reject since it is
not yet configured.
Also `curl 123.123.123.123` or `curl trk.in.rs` will not give any response since
it returns http 301 Moved permanently to https, which you can check with `curl
-I trk.in.rs`, and you can see response on `curl https://123.123.123.123`.
Also redirection could be on cloudflare so you need to disable Crypto->Always
use https. Disabling proxy (use only dns = grayed cloud) did not help for root
domain (only subdomain stopped redirection to https when Crypto-Always use https
is enabled) so you can leave Proxy On.
Certbot authenticator is checking subdomain and also root domain, so you need to
have working dns for root domain.

```
sudo certbot certonly --nginx -d main.trk.in.rs
```

It will create 4 files and you can read cert.pem
```
sudo ls /etc/letsencrypt/live/main.trk.in.rs
cert.pem  chain.pem  fullchain.pem  privkey.pem

sudo keytool -printcert -file /etc/letsencrypt/live/main.trk.in.rs/cert.pem | grep trk
```

https://letsencrypt.org/docs/rate-limits/
Rate limits for failed attempt is 5 per hostname per hour.
Limit for renewal (or duplicate) is 5 per week per combination of hostnames in
certificate.
Main limit is per registered domain (50 per week). You can include up to 100
names per certificate, so it is possible that you generate certificates for 5000
unique subdomains per week.

# Lets encrypt for heroku

https://letsencrypt.org/
Is free and Heroku supports it using
[ACM](https://devcenter.heroku.com/articles/automated-certificate-management)
but only for paid dynos.
Use cloudflare.com for free ssl, and we point directly to https herokuapp (no
need to setup dns on heroku). For example type=CNAME name=www
content=move-index.herokuapp.com
On Crypto tab on Cloud Flare select FULL (not Flexible) SSL.
You can check the Always use HTTPS (`Redirect all requests with scheme “http” to
“https”. This applies to all http requests to the zone`) or create Page rules
that redirects from http to https.
There could be a problem when we using http on Rails and submitting the form on
https, heroku logs will give

```
HTTP Origin header (https://www.premesti.se) didn't match request.base_url (http://www.premesti.se)
Completed 422 Unprocessable Entity in 6ms
ActionController::InvalidAuthenticityToken (ActionController::InvalidAuthenticityToken):
```

On Page rules add redirect non www to www.

~~~
https://premesti.se/* -> https://www.premesti.se/$1

# Also we you did not use  `Always use HTTPS` you can add rules that redirects
http://premesti.se/* -> https://www.premesti.se/$1
http://www.premesti.se/* -> https://www.premesti.se/$1
~~~

~~~
dig premesti.se
# should have
;; ANSWER SECTION:
premesti.se.		78	IN	A	104.28.14.137
premesti.se.		78	IN	A	104.28.15.137

curl premesti.se -I
# should have
HTTP/1.1 301 Moved Permanently
Location: https://www.premesti.se/

curl https://premesti.se -I
# should have
HTTP/1.1 301 Moved Permanently
Location: https://www.premesti.se/
~~~


# Manual install and local test

You can test localy in virtual box, we will use ports 8080 and 8081

~~~
vagrant init
sed -i '/base/c \  config.vm.box = "ubuntu/trusty64"\
  config.vm.network "forwarded_port", guest: 80, host: 8080\
  config.vm.network "forwarded_port", guest: 443, host: 8081\
' Vagrantfile
vagrant up
vagrant ssh
~~~

## Install ssl on apache2

~~~
sudo apt-get update # get right sources
sudo apt-get -y install apache2
sudo a2enmod ssl
sudo mkdir /etc/apache2/ssl
cd /vagrant
sudo cp {ca.pem,private.key,sub.class1.server.ca.pem,ssl.crt} /etc/apache2/ssl
# copy configuration for http
cat /etc/apache2/sites-enabled/000-default.conf | sudo tee -a /etc/apache2/sites-enabled/000-default.conf
# add ssl configuration
sudo sed -i '1c <VirtualHost *:443>' /etc/apache2/sites-enabled/000-default.conf
sudo sed -i '/VirtualHost...443/a SSLEngine on\
  SSLProtocol all -SSLv2\
  SSLCipherSuite ALL:!ADH:!EXPORT:!SSLv2:RC4+RSA:+HIGH:+MEDIUM\
  SSLCertificateFile /etc/apache2/ssl/ssl.crt\
  SSLCertificateKeyFile /etc/apache2/ssl/private.key\
  SSLCertificateChainFile /etc/apache2/ssl/sub.class1.server.ca.pem \
' /etc/apache2/sites-enabled/000-default.conf
sudo service apache2 restart
sudo tail -f /var/log/apache2/error.log
~~~

## Install on ngxin

~~~
sudo apt-get update # get right sources
sudo apt-get -y install nginx
sudo mkdir /etc/nginx/ssl
sudo sed -i '/listen.80.default_server/a \\tlisten 443 ssl;\
\tssl_certificate /etc/nginx/ssl/ssl.crt;\
\tssl_certificate_key /etc/nginx/ssl/private.key;\
' /etc/nginx/sites-enabled/default
sudo cp /vagrant/private.key /vagrant/ssl.crt /etc/nginx/ssl
sudo service nginx restart
sudo tail -f /var/log/nginx/error.log
~~~
