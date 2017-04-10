---
layout: post
title:  Rails through Vagrant to DigitalOcean
tags: ruby-on-rails vagrant digitalocean
---

When a lot of people are working on the same Rails application, than
*Vagrant* could help to set up environment quick and easy.  Even Vagrant
is not recommended for production, it is very usefull for testing
bootstrap scripts automatically. For production we can simply copy
bootstrap script and run commands manually.

# Vagrant

Default user/password is `vagrant/vagrant`

~~~
vagrant init
vagrant up
vagrant ssh
vagrant halt
vagrant destroy
~~~

Run command as `deployer` user. Usually need to wrap inside `bash` if you have
pipe `|`, for example `sudo -i -u deployer bash -c "ls | less -R"`

If you have multuple vagrant boxes and there is port collision, you can set auto
fix

~~~
config.vm.network :forwarded_port, guest: 8080, host: 80, auto_correct: true
~~~

# Deploy Ruby on Rails to VirtualBox


<script src="https://gist.github.com/duleorlovic/8815e439905dbd326a41.js"></script>

# Deploy Ruby on Rails on Nginx and Puma server

<script src="https://gist.github.com/duleorlovic/762c4ffdf43c8eb31aa7.js"></script>
