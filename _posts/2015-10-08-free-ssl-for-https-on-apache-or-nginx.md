---
layout: post
title:  Free SSL for https on apache or nginx
tags: https ssl apache nginx
---

Following [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-with-a-free-signed-ssl-certificate-on-a-vps) here are some screenshots from Startssl how I registered [www.kontakt.in.rs](http://www.kontakt.in.rs).

Go to the [http://www.startssl.com/](http://www.startssl.com/) and on left menu find: *StartSSL Products* > *StartSSL Free* . Before you can get the keys, you need to authenticate yourself. Go to [Certificate Control Panel](https://www.startssl.com/?app=12). There are links for login using existing keys, but we will use signup.

# Signup and install browser certificate 

![Sign up form]({{ site.baseurl }}/assets/post_free_ssl/1_signup_form.jpg)

than you will receive a code in email, and it should be pasted on

![Somplete registration]({{ site.baseurl }}/assets/post_free_ssl/2_complete_registration.jpg)

Wait until they authorize your email account

![Notice for review]({{ site.baseurl }}/assets/post_free_ssl/3_notice_for_review.jpg)

Once your account is approved and you receive email, click on link provided and copy verification code

![Verify approved request]({{ site.baseurl }}/assets/post_free_ssl/4_verify_approved_request.jpg)
 
Generate private key
 
![Generate private key]({{ site.baseurl }}/assets/post_free_ssl/5_generate_private_key.jpg)
 
Install certificate
 
![Install certificate]({{ site.baseurl }}/assets/post_free_ssl/6_install_certificate.jpg)
 
Congratulations for browser certificate
 
![Congratulations for browser certificate]({{ site.baseurl }}/assets/post_free_ssl/7_congratulations_for_browser_certificate.jpg)


# Validation wizard for domain

Pick web ssl

![Chose web ssl]({{ site.baseurl }}/assets/post_free_ssl/validation_wizard1_chose_web_ssl.jpg)

Enter domain name

![Enter domain name]({{ site.baseurl }}/assets/post_free_ssl/validation_wizard2_enter_domain_name.jpg)

Choose email

![Choose email]({{ site.baseurl }}/assets/post_free_ssl/validation_wizard3_choose_email.jpg)

Verify email

![Verify email]({{ site.baseurl }}/assets/post_free_ssl/validation_wizard4_verify_email.jpg)

Success

![Success]({{ site.baseurl }}/assets/post_free_ssl/validation_wizard5_success.jpg)

# Certificates wizard

Pick web ssl

![chose web ssl]({{ site.baseurl }}/assets/post_free_ssl/cert_wizard1_chose_web_ssl.jpg)

Enter password

![Enter password]({{ site.baseurl }}/assets/post_free_ssl/cert_wizard2_enter_password.jpg)

Save **ssl.key**

![save ssl.key]({{ site.baseurl }}/assets/post_free_ssl/cert_wizard3_save_ssl_key.jpg)

Pick domain

![Select domain]({{ site.baseurl }}/assets/post_free_ssl/cert_wizard4_select_domain.jpg)

Add subdomain www

![Add subdomain www]({{ site.baseurl }}/assets/post_free_ssl/cert_wizard5_add_subdomain_www.jpg)

Processing

![Processing]({{ site.baseurl }}/assets/post_free_ssl/cert_wizard6_processing.jpg)

Additinal check

![Additional check]({{ site.baseurl }}/assets/post_free_ssl/cert_wizard7_additional_check.jpg)


# Final cert files

After you got email that certificate is ready, go to toolbox and download it

![Download certificate]({{ site.baseurl }}/assets/post_free_ssl/cert_wizard7_download_certificate.jpg )

We need two additional files:

* StartCom Root CA (ca.pem)
* StartSSL's Class 1 Intermediate Server CA (sub.class1.server.ca)

which you can find in Toolbox section.

Since server needs unencrypted version we need to create that (so it won't ask for password)

~~~
openssl rsa -in ssl.key -out private.key 
~~~

Copy those four files to server

~~~
scp {ca.pem,private.key,sub.class1.server.ca.pem,ssl.crt} YOURSERVER:~ 
~~~

# Local test

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

Temporary change `www.kontakt.in.rs` to point to localhost `127.0.0.1` and go to
[http://www.kontakt.in.rs:8080](http://www.kontakt.in.rs:8080) 
[https://www.kontakt.in.rs:8081](https://www.kontakt.in.rs:8081)

~~~
echo -e "127.0.0.1\twww.kontakt.in.rs" | sudo tee -a /etc/hosts
~~~


You should see certificate on apache


![final apache cert]({{ site.baseurl }}/assets/post_free_ssl/final_apache_cert.jpg)


and green lock on ngxin

![final nginx cert]({{ site.baseurl }}/assets/post_free_ssl/final_nginx_cert.jpg)



Don't forget to remove temporary dns in hosts

~~~
sudo sed -i '/www.kontakt.in.rs/c #127.0.0.1\twww.kontakt.in.rs' /etc/hosts
~~~

On ubuntu, you can rename `ssl.crt` to `ssl.crt.p12` (or ssl.crt.key) and double click will show you
details of certificate.

For heroku you can follow <https://gist.github.com/meskyanichi/3354578>
