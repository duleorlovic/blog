---
layout: post
title:  Rails through Vagrant to DigitalOcean
categories: ruby-on-rails vagrant digitalocean
---

When a lot of people is going to take a part in development, than Vagrant can help to set up quickly and easy any Rails application.

After installing virtualbox and vagrant, you should creat Vagrantfile with command `vagrant init`

Here are my overrides of default configuration

~~~
# define some inputs... could grep from config
DATABASE_NAME="database_development" 
TEST_DATABASE_NAME="database_test" 
DATABASE_USER="vagrant"
#TODO pg_hba.conf
ADDITIONAL_PACKAGES="libmagickwand-dev "
TARGET_RUBY_VERSION="2.2" # its better to specify since RUBY_VERSION is taken from vagrant


Vagrant.configure(2) do |config|
  config.vm.network "forwarded_port", guest: 3000, host: 3000
  config.vm.provider "virtualbox" do |vb|
    #vb.memory = "2048"
    vb.customize ["modifyvm", :id, "--memory", "2048"]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end

  config.vm.provision "shell", inline: <<-SHELL
    # Needed for docs generation and database default
    update-locale LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8 LC_ALL=en_US.UTF-8
    apt-get -y update
    # install developlment tools
    apt-get -y install build-essential curl git nodejs #{ADDITIONAL_PACKAGES}
    # postresql
    apt-get -y install postgresql postgresql-contrib libpq-dev

    # check if database is not already created
    if [ `su postgres -c 'psql -l' | grep #{DATABASE_NAME} | wc -l` -eq 0 ]
      then
        echo "creating databases: '#{DATABASE_NAME}'"
        # if non vagrant user want to be able to create database without password, we need to change
        # md5 to trust in /etc/postgresql/9.1/main/pg_hba.conf
        sed -i 's/md5/trust/' /etc/postgresql/9.3/main/pg_hba.conf
        /etc/init.d/postgresql restart
        su postgres -c "createuser --superuser --createdb #{DATABASE_USER}"
        su postgres -c "createdb -O #{DATABASE_USER} #{DATABASE_NAME}"
        su postgres -c "createdb -O #{DATABASE_USER} #{TEST_DATABASE_NAME}"
    else
      echo "database '#{DATABASE_NAME}' already created"
    fi

    # check if rvm is installed
    if [ "`su -l vagrant -c 'type -t rvm'`" != "function" ]; then
      echo install rvm, not so usefull in VMbut usefull if you want to set up locally and follow this steps
      # fix issue with signature https://github.com/wayneeseguin/rvm/issues/3110
      su -l vagrant -c 'gpg --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3'
      su -l vagrant -c 'curl -sSL https://get.rvm.io | bash -s stable'
      # it is sucessfully installed in bash_... so no need to source
      su -l vagrant -c 'rvm use --install --default #{TARGET_RUBY_VERSION}'
      su -l vagrant -c 'cd /vagrant && gem install bundler'
    else
      echo "rvm already installed" 
    fi

    echo run bundle
    su -l vagrant -c 'cd /vagrant && bundle'
    su -l vagrant -c 'cd /vagrant && rake db:create'
    su -l vagrant -c 'cd /vagrant && rake db:migrate'
    su -l vagrant -c 'cd /vagrant && rake db:seed'
    # port need to be changed since rails 4.2 http://stackoverflow.com/questions/26570609/rails-4-2-0-beta2-cant-connect-to-localhost
    su -l vagrant -c 'cd /vagrant && rails s -d -b 0.0.0.0'
    echo -e "Rails server started$!.\nYou can kill the server with this command: 'kill -9 \$(cat /vagrant/tmp/pids/server.pid)'"
  SHELL

end
~~~


If you want to deploy to DO you can using [vagrant-digitalocean](https://github.com/smdahlen/vagrant-digitalocean):

~~~
vagrant plugin install vagrant-digitalocean
vagrant box add digital_ocean https://github.com/smdahlen/vagrant-digitalocean/raw/master/box/digital_ocean.box
~~~

and in Vagrantfile, add the following:

~~~
  config.vm.provider :digital_ocean do |provider,override|
  override.ssh.private_key_path = "~/.ssh/id_rsa"
    override.vm.box = 'digital_ocean'
    override.vm.box_url = "https://github.com/smdahlen/vagrant-digitalocean/raw/master/box/digital_ocean.box"

    provider.token = "#{ENV['DO_TOKEN']}"
    provider.image = "ubuntu-14-04-x64"
  end

  # add in sheel script
  useradd -m -d /home/vagrant -s /bin/bash -U vagrant
  adduser vagrant sudo
  # %sudo       ALL=(ALL:ALL) NOPASSWD:ALL
~~~
