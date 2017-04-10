---
layout: post
title: Capistrano
---

# Install

Capistrano is not server provisioning tool. You need to manually boot up server
and install ssh, apache, git... usually all tasks that requires sudo should be
done manually. Capistrano use single non priviliged user in non interactive ssh
session.

Capistrano version 3 is used here. Below is section for capistrano version 2.

~~~
cat >> Gemfile << HERE_DOC
# Use Capistrano for deployment
group :development do
  gem 'capistrano-puma', require: false
  gem 'capistrano-rails', require: false # this will load capistrano-bundler
  gem 'capistrano-rvm', require: false
end
HERE_DOC
bundle update capistrano # use latest version 3.8
bundle exec cap install
# this will generate  Capifile, config/deploy.rb and config/deploy/staging.rb...
cat > Capfile << HERE_DOC
# Load DSL and set up stages
require "capistrano/setup"
require "capistrano/console"

# Include default deployment tasks
require "capistrano/deploy"

# Include tasks from other gems included in your Gemfile
require 'capistrano/rails'
require "capistrano/bundler"
require "capistrano/rvm"
require "capistrano/puma"

# Load custom tasks from `lib/capistrano/tasks` if you have any defined
Dir.glob("lib/capistrano/tasks/*.rake").each { |r| import r }
HERE_DOC
~~~

# Configuration variables

[configuration](http://capistranorb.com/documentation/getting-started/configuration/)
files can set variables `set :my_var_name, "value"` and fetch values `fetch
:my_var_name`. There are some variables that are used by default:

* `:application` name of application
* `:deploy_to` path on the remove server where the app should be deployed,
initially is `-> { "/var/www/#{fetch(:application)}" }` but I like home folder.
Inside that folder there are:
  * `current` symlink to some release `/var/www/my_app/releases/20170101010101`
  * `releases/` contains timestamped subfolder
  * `repo/` hold git repository
  * `revisions.log` timestaped log for all deploy or rollback actions
  * `shared/` contains `linked_files` and `linked_dirs` that persists across
  releases (db configuration, user storage)
* `:scm` by default is `:git`
* `:repo_url` is url for git. It should be accessible from remote server. For
local `set :repo_url, 'ssh://orlovic@192.168.2.4/my_project`
* `:linked_files` is symlinked files from shared folder
* `:default_env` for specific env variables

~~~
# config/deploy.rb
lock "3.8.0"

set :application, "myapp"
set :repo_url, "orlovic@192.168.2.4:rails/temp/myapp"
set :deploy_to, "/home/deploy/#{fetch(:application)}"

# rails
set :rails_env, 'production'
~~~

~~~
# config/deploy/staging.rb
# if you use NAT network
server "127.0.0.1", user: "deploy", roles: %w{app db web}, port: 2222
# or if you use private network
server "192.168.3.2", user: "deploy", roles: %w{app db web}
~~~

# Preparing the app

[preparing](http://capistranorb.com/documentation/getting-started/preparing-your-application/)
is with `cap install`. First you need to set the server ip address. Server also
have roles `:web, :app, :db`. You can define in two ways. Properties will be
merged.

~~~
# using simple syntax
role :web, %w{hello@world.com example.com:1234}

# using extended syntax (which is equivalent)
server 'world.com', roles: [:web], user: 'hello'
server 'example.com', roles: [:web], port: 1234
~~~

For local vagrant (see below) you can use

~~~
# config/deploy.rb
set :repo_url, "orlovic@192.168.2.4:rails/temp/myapp"

# config/deploy/staging.rb
server "127.0.0.1", user: "vagrant", roles: %w{app db web}, port: 2222
~~~

If you are using private github repo, than permission problems for git will
occurs, so you can use your local keys on server with thise option:
`set :ssh_options, forward_agent: true`

You can check before deploying with `cap staging git:check` or `cap staging
deploy:check`


# Tasks

Nice [railscasts video](https://www.youtube.com/watch?v=UQj_01dnEiw) but for old
capistrano.

~~~
# lib/capistrano/tasks/notify.rake
namespace :my_tasks do
  desc "My first task"
  task :my_first_task do
    on roles(:all) do
      puts "Start my first task"
    end
    invoke 'my_tasks:notify'
  end

  task :notify do
    on roles(:all) do |host|
      puts "notify #{host}"
    end
  end
end
~~~

To run in specific env, run with: `cap staging my_tasks:my_first_task`

`desc` and `task` are rake methods. Others are taken from
[sshkit](https://github.com/capistrano/sshkit/blob/master/EXAMPLES.md):

* `on roles(:all) do |host|` will iterate to eah server host, You can run
locally with `run_locally do`. `roles(:all)` return list of all hosts
* `as`
* `capture`
* `test`
* `info`
* `error`
* `with` set ENV variable. only 
* `within` use folder. should be inside `on`
* `invoke` run other rake task
* `execute` used to run command: `execute :rake, 'assets:precompile', env: {
rails_env: fetch(:rails_env) }`


You can execute task in before or after hooks.
[flow](http://capistranorb.com/documentation/getting-started/flow/) has several
points that you can access.

~~~
before :finishing, :notify do
end
~~~

# Upgrading 2 to 3

[documentation](http://capistranorb.com/documentation/upgrading/)

* `repository` is renamed to `repo_url`

# Sample app

~~~
rails new myapp
cd myapp
git init . && git add . && git commit -m "rails new myapp"
sed -i Gemfile -e '/capistrano/c \
gem "capistrano-rails", group: :development'
# capistrano-rails will also install capistrano, capistrano-bundler
# rvm puma ?
cap install STAGES=virtual
echo "require 'capistrano/rails'" >> Capfile
~~~

On AWS create new instance and download new key for example `aws_test.pem`.

~~~
chmod 400 aws_test.pem
ssh -i "aws_test.pem" ubuntu@ec2-34-204-86-131.compute-1.amazonaws.com
~~~

You can install server localy using [Vagrant scripts]({{ site.baseurl }}
{% post_url 2015-02-16-rails-through-vagrant-to-digitalocean %}).
You can use default NAT address (access virtual box at ssh port 2222) and port
forwarding (`config.vm.network :forwarded_port, guest: 80, host: 8080`). In
order to access host machine from virtual box you need to know host IP address.
Other solution is to use Private IP (`private_network`) and host is easilly
determined from that.
You need to add public key on host after provision is done so virtual can access
host without password (maybe to try with
[vagrant-triggers](https://github.com/emyl/vagrant-triggers))
`ssh deploy@192.168.3.2 cat .ssh/id_rsa.pub >> ~/.ssh/authorized_keys`

~~~
vagrant init
# edit Vagrantfile
cat > Vagrantfile << HERE_DOC
Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox" do |vb, vb_config|
    # list of all machines can be found https://atlas.hashicorp.com/boxes/search
    vb_config.vm.box = "ubuntu/trusty64"
    vb.gui = true
    vb.memory = "2048"
    vb_config.vm.provision :shell, inline: $script, keep_color: true
    vb_config.vm.network :private_network, ip: "192.168.3.2"
  end
end
$script << SCRIPT
#!/bin/bash
set -e # Any commands which fail will cause the shell script to exit immediately
set -x # show command being executed
O
echo "STEP: update"
apt-get -y update # > /dev/null # update is needed to set proper apt sources
if [ "`id -u deploy`" = "" ]; then
  echo "STEP: creating user deploy"
  useradd deploy -md /home/deploy --shell /bin/bash
  echo deploy:deploy | chpasswd # change password to 'deploy'
  # gpasswd -a deploy sudo # add to sudo group
  # echo "deploy ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/deploy # don't ask for password when using sudo
  # if [ ! "`id -u vagrant`" = "" ]; then
  #   usermod -a -G vagrant deploy # adding to vagrant group if vagrant exists
  # fi
else
  echo "STEP: user deploy already exists"
fi

if [ "`which git`" = "" ]; then
  echo "STEP: install development tools: git node ..."
  apt-get -y install build-essential curl git nodejs
  apt-get -y install libgmp-dev # this is needed for json
  apt-get -y install libxslt-dev libxml2-dev # for nokogiri http://stackoverflow.com/questions/6277456/nokogiri-installation-fails-libxml2-is-missing
else
  echo "STEP: development tools already installed"
fi

export DATABASE_URL=postgresql://deploy@localhost/rails_db
if [ `echo $DATABASE_URL | cut -f 1 -d ':'` = "postgresql" ]; then
  if [ "`which psql`" = "" ]; then
    echo "STEP: installing postgres"
    apt-get -y install postgresql postgresql-contrib libpq-dev
  else
    echo STEP: postgres already installed
  fi
  if [ "`sudo -u postgres psql postgres -tAc "SELECT 1 FROM pg_roles WHERE rolname='deploy'"`" = "1" ]; then
    echo STEP: postgres user 'deploy' already exists
  else
    echo STEP: create postgresql database user deploy
    find /etc/postgresql -name pg_hba.conf -exec sed -i '/^local\s*all\s*all\s*peer/i # allow deploy to access without password - trust method\
  host all deploy 127.0.0.1/32 trust' {} \;
    # change md5 with trust, don't use this if you set up password for database user
    /etc/init.d/postgresql restart
    sudo -u postgres createuser --superuser --createdb deploy
  fi
else
  echo STEP: create mysql2 database user deploy
  if [[ `echo "SELECT user FROM mysql.user WHERE user = 'deploy'" | mysql` = "" ]]; then
    echo CREATE USER 'deploy'@'localhost' | mysql --user=root
    echo GRANT ALL PRIVILEGES ON * . * TO 'deploy'@'localhost' | mysql --user=root
    #FLUSH PRIVILEGES;
  else
    echo STEP: mysql user 'deploy' already exists
  fi
fi

SCRIPT
HERE_DOC
~~~

~~~
vagrant destroy -f # destroy without confirmation
vagrant up --provision # to provision again
vagrant ssh
# or directly
ssh -p 2222 vagrant@127.0.0.1
# username/password is deploy/deploy
ssh-keygen -f "/home/orlovic/.ssh/known_hosts" -R [127.0.0.1]:2222
yes | ssh-copy-id -p 2222 deploy@127.0.0.1
ssh -p 2222 deploy@127.0.0.1
~~~

# Logs

See multiple logs in one terminal window

~~~
vagrant ssh
sudo apt-get install multitail
mulitail /home/deploy/myapp/current/log/*
~~~

# Rubber

[railscast](http://railscasts.com/episodes/347-rubber-and-amazon-ec2?autoplay=true)

~~~
sed -i Gemfile -e '/^group :development do/a \
  # deploy and provision tool\
  gem "rubber"'
bundle exec rubber vulcanize complete_passenger_postgresql
~~~

vulcanize will copy
[templates](https://github.com/rubber/rubber/blob/master/templates/complete_passenger/templates.yml)
to `config/rubber` which you can customize, usually only `yml` files. Capistrano
`.rb` files usually do not need to change, and also configuration files
`role/*.conf` are uptodate and configurable. You can start from `rubber.yml` to
add AWS root security credentials and EC2 keypairs (which you download to home
`~/.ec2` folder, rename without `pem` and create public version `ssh-keygen -y
-f ~/.ec2/aws_test > ~/.ec2/aws_test.pub`).

Than you can provision instance with

~~~
cap rubber:create_staging
# Hit enter at prompts to accept the defaults
~~~

You can remove with

~~~
cap rubber:destroy
# type production.foo.com
~~~

You can run in vagrant

~~~
~~~


Some links
[How to deploy RubyonRails project to AWS EC2 using capistrano](https://www.youtube.com/watch?v=imdrYD4ooIk)

