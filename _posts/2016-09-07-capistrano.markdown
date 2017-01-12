---
layout: post
title: Capistrano
---

# Install

Capistrano is not server provisioning tool. You need to manually boot up server
and install ssh, apache, git... usually all tasks that requires sudo should be
done manually. Capistrano use single non priviliged user in non interactive ssh
session.

~~~
gem install capistrano
echo 'gem "capistrano", group: :development' >> Gemfile
budle exec cap install
# this will generate config/deploy.rb and config/deploy/staging.rb
~~~

You can install server using [Vagrant scripts]({{ site.baseurl }} {% post_url 2015-02-16-rails-through-vagrant-to-digitalocean %})

~~~
vagrant init
# edit Vagrantfile
vagrant up --provision # to provision again
vagrant ssh
~~~

~~~
# Vagrantfile
config.vm.provider "virtualbox" do |v, override|
  v.gui = true
  override.vm.network "forwarded_port", guest: 3000, host: 3000
  # list of all machines can be found https://atlas.hashicorp.com/boxes/search
  override.vm.box = "ubuntu/trusty64"
  vb.customize ["modifyvm", :id, "--memory", "2048"]
  vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
end
~~~

# Tasks

Nice [railscasts video](https://www.youtube.com/watch?v=UQj_01dnEiw)
To get started, you can run with: `cap production my_task`

~~~
set :my_val, "My Value"
task :my_task do
  puts "My task my_val=#{fetch :my_val}"
end
task :goodbye do
  puts "goodbye"
end
after :my_task, :goodbye
~~~

# Options

Set the server ip address. Server also have roles `:web, :app, :db`
`set :ssh_options, forward_agent: true` to use all your local keys on
server so you can download private repository

Some links
[How to deploy RubyonRails project to AWS EC2 using capistrano](https://www.youtube.com/watch?v=imdrYD4ooIk)

