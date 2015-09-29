---
layout: post
title:  Rails through Vagrant to DigitalOcean
categories: ruby-on-rails vagrant digitalocean
---

When a lot of people are working on the same Rails application, than *Vagrant* 
could help to set up environment quick and easy.
Even Vagrant is not recommended for production, it is very usefull for testing
bootstrap scripts automatically. For production we can simply copy bootstrap
script and run manually.
Testing can be done localy using VirtualBox, or remotely on Digital Ocean
using their API key.

In your Rails project, you can create *Vagrantfile* with command `vagrant init`
There you define common configuration and overrides for each provider. 
Let's start virt VirtualBox provider

~~~
# Vagrantfile
  config.vm.hostname = "my_app.example.com"

  config.vm.provider :virtualbox do |vb, override|
    # list of all machines can be found https://atlas.hashicorp.com/boxes/search
    override.vm.box = "ubuntu/trusty64"
    vb.customize ["modifyvm", :id, "--memory", "2048"]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    override.vm.network "forwarded_port", guest: 80, host: 3000
    override.vm.provision :file, source: '~/.secrets_staging.env', destination: '/vagrant/.secrets.env'
    override.vm.provision :shell, path: 'vagrant/boostrap.sh', keep_color: true
  end
~~~

We will use `~/.secrets_staging.env` file to define all secrets and variables
like `export RAILS_ENV=production` 
Virtualbox and Digital ocean provision scripts are running as root user 
(`vagrant ssh` virtual box use `vagrant` user). Root access is not recomended 
on production, so its better to create another user `deployer` which will be
 used for deploying and ssh.
[Source link](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)
To ssh using deployer user on VirtalBox run `ssh -p 2222 deployer@127.0.0.1`

To boot machine on digital ocean, you need to register *DIGITAL_OCEAN_TOKEN*
API key. If you already added ssh key to your account https://cloud.digitalocean.com/settings/security than you can rename it to *Vagrant* or put it's *name* as `provider.ssh_key_name = name` param.

~~~
# Vagrantfile
  config.vm.provider :digital_ocean do |provider, override|
    override.ssh.private_key_path = '~/.ssh/id_rsa'
    override.vm.box = 'digital_ocean'
    override.vm.box_url = "https://github.com/smdahlen/vagrant-digitalocean/raw/master/box/digital_ocean.box"

    override.vm.provision :file, source: '~/.my_app_staging.env', destination: '/vagrant/.secrets.env'
    override.vm.provision :shell, path: 'vagrant/bootstrap_ruby_on_rails_image.sh', keep_color: true
    override.vm.synced_folder ".", "/vagrant", type: "rsync",
      rsync__exclude: [".git/", "tmp/", "log/", "lib/", "docs/", "public/"]

    provider.token = ENV["DIGITAL_OCEAN_TOKEN"]
    provider.image = 'ruby-on-rails'
    provider.region = 'nyc2'
    provider.size = '1gb'
  end
~~~

Before provisioning it, you need to install plugin `vagrant plugin install [vagrant-digitalocean](https://github.com/smdahlen/vagrant-digitalocean)`. To see all images (regions, sizes) available: `vagrant digitalocean-list images $DIGITAL_OCEAN_TOKEN`. At the end run: `vagrant up --provider=digital_ocean`. If you want to resize its memory, you can simply change size and `vagrant rebuild`

We can use [ruby-on-rails](https://www.digitalocean.com/community/tutorials/how-to-use-the-ruby-on-rails-one-click-application-on-digitalocean) image, but its better to start from scratch, plan Ubuntu, and follow best practices.

