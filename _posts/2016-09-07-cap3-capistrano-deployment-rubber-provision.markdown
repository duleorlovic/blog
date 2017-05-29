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
is with `cap install`.

Tasks are usually only for specific roles so you server needs to belongs to that
role if you want task to be executed. Three main roles

* `:web` role is nginx/apache server
* `:app` role is for rails app
* `:db` role is for mysql/postgresql database (requires `primary: true`)

You can define in two ways. Properties will be merged.

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

## Task Syntax

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

You can use `puts` and `run` to run command in shell. If you use sudo, than you
should enable `default_run_options[:pty] = true` so when he ask for password it
prompts in current shell. Also helpfull is `ssh_options[:forward_agent] = true`
which uses you keys on remote server to download private repositories.. Also
helpfull is `ssh_options[:forward_agent] = true` which uses you keys on remote
server to download private repositories.

~~~
task :hello do
  puts "Here are all files"
  run "ls"
  run "#{sudo} ls /etc"
end
~~~

# Upgrading capistrano 2 to 3

[documentation](http://capistranorb.com/documentation/upgrading/)
here are differences between cap 2 and cap 3

* `repository` is renamed to `repo_url`
* environment is set instead `RAILS_ENV=vagrant cap -T` to second param `cap
vagrant -T`
* instead of positional argument for role `server "123.123.123.123", :web` use
hash `roles` argument `server "123.123.123.123", roles: [:web]`

# Sample app on EC2

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

# Capistrano with custom Vagrant script

You can install server localy using [Vagrant scripts]({{ site.baseurl }}
{% post_url 2015-02-16-rails-through-vagrant-to-digitalocean %}).
You can use default NAT address (access virtual box at ssh port 2222) and port
forwarding (`config.vm.network :forwarded_port, guest: 80, host: 8080`). In
order to access host machine from virtual box you need to know host IP address.
So my solution is to use Private IP (`private_network`) and host is easilly
determined from that.
You need to add public key on host after provision is done so virtual can access
host without password (maybe to try with
[vagrant-triggers](https://github.com/emyl/vagrant-triggers)), so two commands
are needed

~~~
ssh-keygen -f "/home/orlovic/.ssh/known_hosts" -R 192.168.3.2
ssh deploy@192.168.3.2 cat .ssh/id_rsa.pub >> ~/.ssh/authorized_keys
~~~

Here are scripts

~~~
vagrant init
# edit Vagrantfile
cat >> Vagrantfile << HERE_DOC
Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox" do |vb, vb_config|
    # list of all machines can be found https://atlas.hashicorp.com/boxes/search
    vb_config.vm.box = "ubuntu/trusty64"
    vb.memory = "2048"
    vb_config.vm.provision :shell, inline: $script, keep_color: true
    vb_config.vm.network :private_network, ip: $server_ip
  end
end

# $ruby_version = `grep "^ruby" Gemfile | awk "{print $2}"`
$ruby_version = `ruby --version | awk "{print $2}"`.split("p").first # 2.3.1p112
$public_key = `cat ~/.ssh/id_rsa.pub`.strip
$server_ip = "192.168.3.2"

$nginx_config = <<-NGINX_CONFIG
# https://www.digitalocean.com/community/tutorials/deploying-a-rails-app-on-ubuntu-14-04-with-capistrano-nginx-and-puma
upstream app {
    # Path to Unicorn SOCK file, as defined previously
    server unix:/home/deploy/myapp/shared/tmp/sockets/puma.sock;
}

server {
    listen 80 default_server deferred;
    server_name myapp.local;
    root /home/deploy/myapp/current/public;
    access_log /home/deploy/myapp/current/log/nginx.access.log;
    error_log /home/deploy/myapp/current/log/nginx.error.log;


    location ^~ /assets/ {
      gzip_static on;
      expires max;
      add_header Cache-Control public;
    }

    try_files $uri/index.html $uri @app;
    location @app {
        proxy_pass http://app;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        access_log /home/deploy/myapp/current/log/nginx.rails.access.log;
    }

    error_page 500 502 503 504 /500.html;
    client_max_body_size 10M;
    keepalive_timeout 10;

    if ($request_method !~ ^(GET|HEAD|PUT|PATCH|POST|DELETE|OPTIONS)$ ){
      return 405;
    }

    if (-f $document_root/system/maintenance.html) {
      return 503;
    }
}
NGINX_CONFIG

$script = <<-SCRIPT
set -e # Any commands which fail will cause the shell script to exit immediately
set -x # show command being executed
L=en_US.UTF-8
update-locale LANG=$L LANGUAGE=$L LC_ALL=$L # needed for database default enc
# or install missing locale with, for example sr_RS
# locale-gen sr_RS

echo "STEP: update"
# apt-get -y update > /dev/null # update is needed to set proper apt sources
if [ "`id -u deploy`" = "" ]; then
  echo "STEP: creating user deploy (without sudo access)"
  useradd deploy -md /home/deploy --shell /bin/bash
  echo deploy:deploy | chpasswd # change password to 'deploy'
  echo STEP: generate keys and adding host public key to vb authorized keys
  sudo -i -u deploy /bin/bash -c "yes '' | ssh-keygen -N ''"
  sudo -i -u deploy /bin/bash -c "echo #{$public_key} >> ~/.ssh/authorized_keys"
  # TODO: cap staging git:check will add to known hosts
  sudo -i -u deploy /bin/bash -c "ssh-keyscan #{$server_ip[0..-2]+"1"} >> ~/.ssh/known_hosts"

  # gpasswd -a deploy sudo # add to sudo group
  # echo "deploy ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/deploy # don't ask for password when using sudo
  # if [ ! "`id -u vagrant`" = "" ]; then
  #   usermod -a -G vagrant deploy # adding to vagrant group if vagrant exists
  # fi
else
  echo "STEP: user deploy already exists"
fi

if [ "`which git`" = "" ]; then
  echo "STEP: install development tools: git nodejs ..."
  apt-get -y install build-essential curl git nodejs multitail
else
  echo "STEP: development tools already installed"
fi

if [ "`which nginx`" = "" ]; then
  echo "STEP: install ngix server"
  apt-get -y install nginx
  cat << 'NGINX_CONFIG' | sudo tee /etc/nginx/sites-available/default
  #{$nginx_config}
NGINX_CONFIG
  # TODO: it seems we need to restart nginx later, probable at this moment puma
  # sockets do not exists yet
  sudo service nginx restart
else
  echo "STEP: nginx already installer"
fi

export DATABASE_URL=postgresql://deploy:deploy@localhost/myapp_production
DATABASE_NAME=${DATABASE_URL##*/}
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
    sudo -u postgres createuser --superuser --createdb deploy
    sudo -u postgres psql -U postgres -d postgres -c "alter user deploy with password 'deploy';"
  fi
  if sudo -u postgres psql -lqt | cut -d \\\| -f 1 | grep -wq $DATABASE_NAME; then
    echo STEP: $DATABASE_NAME already exists
  else
    echo STEP: creating $DATABASE_NAME
    sudo -u deploy createdb $DATABASE_NAME
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

if [ "`sudo -i -u deploy which bundle`" = "" ]; then
#if [ ! -f /usr/local/rvm/scripts/rvm ]; then
  echo "STEP: installing rvm for system so it can download sudo requirements"
  gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
  # multi-user install to /usr/local/rvm, we use vagrant since it is in sudoers list
  sudo -i -u vagrant /bin/bash -c "curl -sSL https://get.rvm.io | sudo -i bash -s stable --ruby"
  usermod -a -G rvm deploy # adding to rvm group so it can access /usr/local/rvm
  usermod -a -G rvm vagrant # adding to rvm group so it can access /usr/local/rvm

  if [ ! "#{$ruby_version}" = "" ]; then
    echo "STEP: installing ruby version #{$ruby_version}"
    # sometime we need to remove old cache
    # rm -rf $rvm_path/archives/rubygems-* $rvm_path/user/{md5,sha512}
    sudo -i -u vagrant /bin/bash -c "rvm install #{$ruby_version}"
  fi

  echo "STEP: install bundle for deploy"
  sudo -i -u deploy /bin/bash -c "rvm install 2.4"
  sudo -i -u deploy /bin/bash -c "gem install bundler"
else
  echo "STEP: rvm and bundler already installed"
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
multitail /var/log/nginx/* /home/deploy/myapp/current/log/*
~~~

# Rubber

This gem is used just as wrapper for common stack, like passenger_postgresql,
targeting EC2 instances and do all provision stepts.
It depends on capistrano 2.
[railscast](http://railscasts.com/episodes/347-rubber-and-amazon-ec2?autoplay=true)

~~~
cat >> Gemfile << HERE_DOC
  # deploy and provision tool
  gem "rubber"
HERE_DOC
bundle exec rubber vulcanize complete_passenger_postgresql
# bundle exec rubber vulcanize complete_passenger_mysql
~~~

Vulcanize will copy
[templates](https://github.com/rubber/rubber/blob/master/templates/complete_passenger/templates.yml)
to `config/rubber` which you can customize, usually only `yml` files. Capistrano
`.rb` files usually do not need to change, and also configuration files
`role/*.conf` are uptodate and configurable.
You can start configuring `config/deploy.rb` and `config/rubber/rubber.yml` to
add AWS root security credentials and EC2 keypairs (which you download to home
`~/.ec2` folder, rename without `pem` and create public version `ssh-keygen -y
-f ~/.ec2/aws_test > ~/.ec2/aws_test.pub`).

Than you can provision instance with

~~~
cap rubber:create_staging
# this is the same as manually
# cap rubber:create rubber:bootstrap deploy
# Hit enter at prompts to accept the default value for params
# or you can set params in env var like
# ALIAS=web001 ROLE=app cap rubber:create
~~~

You can remove (terminate) all EC2 instances with

~~~
cap rubber:destroy
# type production.foo.com
~~~

You can create additional instances:

~~~
ALIAS=web01 ROLES=app cap rubber:create
ALIAS=web01 ROLES=app cap rubber:bootstrap # this is idempotent
cap deploy:cold
~~~

and you can see current in `config/rubber/instance-vagrant.yml`

Roles should be defined in that instance file. Note that for first time we are
defining roles using env ROLES on rubber create task.

Note that if [current instance- file
does not exists](https://github.com/rubber/rubber/blob/master/lib/rubber/recipes/rubber/utils.rb#L20)
`rubber:create_staging` will create new instance using `staging_roles`. If that
instance file exists, than roles from it will be used.

## Rubber Vagrant rubber provision

It works only for old version of vagrant 1.7.4 (just remove and install that
[version .deb file](https://releases.hashicorp.com/vagrant/1.7.4/) using
software app)

~~~
vagrant -v # should return 1.7.4
vagrant plugin install rubber
~~~

It depends on older bundler so run

~~~
gem uninstall bunder
gem install bundler -v 1.10.5
rm Gemfile.lock
bundle install
~~~

Add vagrant env to secrets, database and env

~~~
echo "vagrant: *default" >> config/secrets.yml
cat >> config/database.yml << HERE_DOC
vagrant:
  <<: *default
HERE_DOC
cp config/environments/production.rb config/environments/vagrant.rb
~~~

To be able to ssh into vagrant you need to use your keys or manually copy
later.

You need to define your ruby version and roles that you need. Note that current
ruby version need to be same, so check that `rvm list` returns current same as
your from Vagrantfile.

~~~
cat >> Vagrantfile << HERE_DOC
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.box_check_update = false
  config.vm.network :private_network, ip: "192.168.70.10"

  # do not generate new key, copy my key and use it
  config.ssh.insert_key = false
  config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/authorized_keys"
  config.ssh.private_key_path = ["~/.ssh/id_rsa", "~/.vagrant.d/insecure_private_key"]
  # ruby build fails on 512MB so we increase
  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
  end

  config.vm.provision :rubber do |rubber|
    rubber.rubber_env = 'vagrant'

    # If you remove this line, staging_roles from config/rubber/rubber.yml is used
    rubber.roles = 'web,app,apache,hadproxy,monit,passenger,postgresql_master,db:primary=true'

    # Only necessary if you use RVM locally.
    rubber.rvm_ruby_version = '2.3.1'
  end
end
HERE_DOC
~~~

To create vagrant machine you can run vagrant commands (we use tee to save log)

~~~
vagrant up
# when machine already created
vagrant provision
# or to save a log
vagrant destroy -f && vagrant up 2>&1 | tee tmp/log.log
~~~

Note that if you make changes to `config/rubber/rubber.yml` you need to remove
`config/rubber/instance-vagrant.yml`.

I have error

~~
 ** [out :: default.foo.com] The following packages have unmet dependencies:
 ** [out :: default.foo.com] libapache2-mod-passenger : Depends: passenger (= 1:5.0.8-1~trusty1) but it is not going to be installed
 ** [out :: default.foo.com] Unable to correct problems, you have held broken packages.
~~

so I need to remove version parameter from `config/rubber/rubber-passenger.yml`

Another error

~~~
 ** ERROR:  Error installing rubber:
 ** xmlrpc requires Ruby version >= 2.3.
 ** /tmp/gem_helper:32:in `block in <main>'
 ** Unable to install versioned gem rubber:3.2.2
 ** RuntimeError
~~~

I updated ruby version in `config/rubber/rubber-ruby.yml` to match 3.2.1
Also if [ruby-build is failed](https://github.com/rbenv/ruby-build/issues/721)
you can increase memory or add option to `config/rubber/deploy-setup.rb`

Also I need to comment out `rsudo "service vboxadd setup"` from
`config/rubber/deploy-setup.rb`.
And for rails asset pipeline you need to add `nodejs` package to
`config/rubber/rubber-ruby.yml` to `packages` list.

If you change hostname or key, maybe there is connection error like

~~~
. ** timeout in initial connect, retrying
Trying to enable root login
  * executing `rubber:_ensure_key_file_present'
  * executing `rubber:_allow_root_ssh'
  * executing "sudo -p 'sudo password: '  bash -l -c 'mkdir -p /root/.ssh && cp /home/vagrant/.ssh/authorized_keys /root/.ssh/'"
    servers: ["192.168.70.10"]

 ** Failed to connect to 192.168.70.10, retrying
  * executing `rubber:_ensure_key_file_present'
  * executing `rubber:_allow_root_ssh'
  * executing "sudo -p 'sudo password: '  bash -l -c 'mkdir -p /root/.ssh && cp /home/vagrant/.ssh/authorized_keys /root/.ssh/'"
    servers: ["192.168.70.10"]
~~~

when you try `ssh root@192.168.70.10` you see

~~~
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:ur+QgdymSTNZYqImXqmRgw3cZonpVjZjd7gtNyxyR
Please contact your system administrator.
Add correct host key in /home/orlovic/.ssh/known_hosts to get rid of this message.
Offending RSA key in /home/orlovic/.ssh/known_hosts:3
  remove with:
  ssh-keygen -f "/home/orlovic/.ssh/known_hosts" -R 192.168.70.10
RSA host key for 192.168.70.10 has changed and you have requested strict checking.
Host key verification failed.
~~~

Than just remove the key from known_hosts and it will resume without errors.

To add secrets you can write them in a `keys.sh` file and source from
`~/.bashrc` before return if not interactivelly.

~~~
# keys.sh
export SECRET_KEY_BASE=52b57...
~~~

Rails is running on port 7000 inside virtualbox. hadproxy routes standard http
80 port to rails 7000, so you can access site on <http://default.foo.com/>

If you see only dots, it is probably stuck on passenger

~~~
 * executing `rubber:passenger:serial_add_to_pool_reload_default'
 ** Waiting for passenger to startup
  * executing "sudo -p 'sudo password: '  bash -l -c 'while ! curl -s -f http://localhost:$CAPISTRANO:VAR$/ &> /dev/null; do echo .; done'"
    servers: ["default.foo.com"]
    [default.foo.com] executing command
 ** [out :: default.foo.com] .
 ** [out :: default.foo.com] .
~~~

Please check rails logs (passenger logs is in apache log) to see if there is any
error on index page.

To see rubber setup logs, you can ssh and run

~~~
ssh root@192.168.70.10
tail -f /var/log/apache2/* /mnt/your_app_name-vagrant/current/log/*
~~~

or you can use existing task:

~~~
RAILS_ENV=vagrant cap rubber:tail_logs
~~~

To use different domain you need to update `domain: my-domain.vagrant` inside
`config/rubber/rubber.yml`.
To use different subdomain you need to set up in Vagrant file using `define`.

~~~
Vagrant.configure("2") do |config|
  config.vm.define "mysubdomain" do |m|
  end
end
~~~

I tried with `v.name = 'mysubdomain'` inside `Vagrantfile` but that just rename
virtual machine name
([machine.name](https://github.com/rubber/rubber/blob/master/lib/rubber/vagrant/provisioner.rb#L39)]
inside rubber plugin do not take that name). All that plugin does is a call to
script command

~~~
RUN_FROM_VAGRANT=true RUBBER_ENV=vagrant ALIAS=mysubdomain ROLES='web,app' EXTERNAL_IP=192.168.70.10 INTERNAL_IP=192.168.70.10 RUBBER_SSH_KEY=/home/orlovic/.ssh/id_rsa,/home/orlovic/.vagrant.d/insecure_private_key ruby -e "require 'capistrano/cli'; Capistrano::CLI.execute" rubber:create -S initial_ssh_user=vagrant
# only rubber:create uses ENV['ROLES'], other task need RUBBER_ENV (this is
copied from RAILS_ENV)
# RUBBER_SSH_KEY and FILTER (not req). and create, refresh, destroy use ALIAS
... cap rubber:refresh -S initial_ssh_user=vagrant
... cap rubber:bootstrap
... cap deploy:migrations
~~~

For existing machines you can update `name: mysubdomain'` in
`config/rubber/instance-vagrant.yml`.

# Debug capistrano tasks

You can just pust `require "byebug"` at the top of you .rb file and use `byebug`
command inside the tasks, or you can require using `ruby -rbyebug` (no need to
add require byebug). This does not work in .conf files

~~~
RAILS_ENV=vagrant bundle exec ruby -rbyebug $(which cap) rubber:util:backup
~~~

You can debug rubber rb files from you gem. If you want to debug rubber plugin
commands than you need to download source, create gem and copy files
[source](https://github.com/duleorlovic/rubber-vagrant-plugin/commit/409801992b9ee87c09e97b198efa85c5e4176322)

To list all tasks your can run `cap -T`. To find all tasks you need to set up
env and use option `-v`

~~~
RAILS_ENV=vagrant cap -vT
~~~

Some tasks could be only for specific roles or providers

~~~
  after "rubber:bootstrap", "rubber:base:reinstall_virtualbox_additions"
  task :reinstall_virtualbox_additions, only: { provider: 'vagrant' } do
    rsudo "service vboxadd setup"
  end

  before "rubber:install_packages", "rubber:base:prepare_passenger_install"
  task :prepare_passenger_install, roles: :passenger do
    ...
  end
~~~

Rubber reads all [deploy-*
files](https://github.com/rubber/rubber/blob/master/templates/base/config/deploy.rb#L79)
(you can find that line in `config/deploy.rb`).
In `rubber.yml` you can overwrite some settings for specific roles, enviroments
and hosts.

# Videos

[Ruby on Rails - Railscasts PRO #337 Capistrano Recipes (pro)](https://www.youtube.com/watch?v=uXla2yyzH_8)
[How to deploy RubyonRails project to AWS EC2 using capistrano](https://www.youtube.com/watch?v=imdrYD4ooIk)

