---
layout: post
---

# Complete provision

https://gorails.com/deploy/ubuntu/18.04
ubuntu 19 (eoan) does not include passenger yet https://github.com/phusion/passenger/issues/2104

On EC2 you already have non root user `ubuntu` so you can skip following steps
if `ssh -i $PEM_FILE ubuntu@$PRODUCTION_IP` works

To create non root user try:
```
ssh root@$PRODUCTION_IP
adduser deploy
# fill and enter
adduser deploy sudo
sudo vi /etc/ssh/sshd_config
# PasswordAuthentication yes
sudo systemctl restart ssh
```
make sure you can access as deploy

```
ssh deploy@$PRODUCTION_IP
ssh-copy-id deploy@$PRODUCTION_IP
ssh-copy-id -i ~/.ssh/specific_key.pub deploy@$PRODUCTION_IP
ssh deploy@$PRODUCTION_IP
```

Install node and yarn as deploy user
```
# sudo pwd
# curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo add-apt-repository ppa:chris-lea/redis-server
# click Enter
sudo apt-get update
sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev software-properties-common libffi-dev dirmngr gnupg apt-transport-https ca-certificates redis-server redis-tools nodejs yarn
# click Enter
```

Installing latest node can be done with `n` package
```
yarn global add n
yarn global bin
sudo /home/ubuntu/.yarn/bin/n latest
```

install rbenv and ruby as deploy user
```
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
git clone https://github.com/rbenv/rbenv-vars.git ~/.rbenv/plugins/rbenv-vars
exec $SHELL
rbenv install 2.6.3
rbenv global 2.6.3
ruby -v
gem install bundler
bundle -v
```
Add env variable for mastey key and database url
```
mkdir move_index
vi move_index/.rbenv-vars
# add lines
DATABASE_URL=postgres://deploy:1234@127.0.0.1/move_index_production
RAILS_MASTER_KEY=1234
RAILS_ENV=production
# check current value of max-old-space-size
# $HOME/.rbenv/bin/rbenv exec bundle exec node -e 'console.log(v8.getHeapStatistics().heap_size_limit/(1024*1024))'
NODE_OPTIONS=--max-old-space-size=460

echo '# this will load master key in env' >> ~/.bashrc
echo 'export $(cat ~/move_index/.rbenv-vars)' >> ~/.bashrc
```
or add this task to copy master key from shared to config folder. You need to
create shared file `vi /var/www/my_app/shared/master.key`.
```
# config/deploy.rb
namespace :config do
   task :symlink do
      on roles(:app) do
        execute :ln, "-s #{shared_path}/master.key #{release_path}/config/master.key"
      end
   end
end

after 'deploy:symlink:shared', 'config:symlink'
```

passenger and nginx
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger bionic main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update
sudo apt-get install -y nginx-extras libnginx-mod-http-passenger
sudo vim /etc/nginx/conf.d/mod-http-passenger.conf
# change the line for passenger_ruby
# passenger_ruby /home/deploy/.rbenv/shims/ruby;

sudo service nginx start
```
configure nginx (replace myapp with your app name, usually github folder name)

```
sudo vi /etc/nginx/sites-enabled/default

server {
  listen 80;
  listen [::]:80;

  server_name _;
  root /home/deploy/myapp/current/public;

  passenger_enabled on;
  passenger_app_env production;

  location /cable {
    passenger_app_group_name myapp_websocket;
    passenger_force_max_concurrent_requests_per_process 0;
  }

  # Allow uploads up to 100MB in size
  client_max_body_size 100m;

  location ~ ^/(assets|packs) {
    expires max;
    gzip_static on;
  }
}

sudo service nginx reload
```
install postgres
```
sudo apt-get -y install postgresql postgresql-contrib libpq-dev
sudo su - postgres

createuser --pwprompt deploy
# enter PASSWORD
createdb -O deploy move_index_production
exit
# test if you can access with:
psql -d move_index_production
# change deploy user as superuser so it can create extensions like pgcrypto uuid
sudo su postgres -c "psql -d postgres -c 'ALTER USER deploy WITH SUPERUSER;'"
```

dump postgresql with
```
pg_dump move_index_production > m.dump
scp m.dump IP:
ssh IP
psql move_index_production < ~/m.dump
```

# Install capistrano

Capistrano is not server provisioning tool. You need to manually boot up server
and install ssh, apache, git... usually all tasks that requires sudo should be
done manually. Capistrano use single non priviliged user in non interactive ssh
session. Rubber has script for provisioning a server `cap rubber:create` and
installing packages `cap rubber:bootstrap`

Capistrano version 3 is used here. Below is section for capistrano version 2.

~~~
cat >> Gemfile << HERE_DOC
# Use Capistrano for deployment
group :development do
  gem 'capistrano', '~> 3.14', require: false
  gem 'capistrano-rails', '~> 1.6', require: false

  gem 'capistrano-puma', require: false
  gem 'capistrano-rvm', require: false

  # use those if you do not use puma and rvm
  gem 'capistrano-passenger', '~> 0.2.0'
  gem 'capistrano-rbenv', '~> 2.2'

  # cap production rails:c
  gem 'capistrano-rails-console', require: false
  # cap production postgres:replicate
  gem 'capistrano3-postgres', require: false
end
HERE_DOC

bundle exec cap install
# this will generate  Capfile, config/deploy.rb and config/deploy/staging.rb...

cat > Capfile << HERE_DOC
# Load DSL and set up stages
require 'capistrano/setup'
require 'capistrano/deploy'
require 'capistrano/scm/git'
install_plugin Capistrano::SCM::Git

require 'capistrano/rails'
require 'capistrano/rails/console'

# Include tasks from other gems included in your Gemfile
require 'capistrano/bundler'
require 'capistrano/rvm'
require 'capistrano/puma'
# if you use passenger and rbenv
require 'capistrano/passenger'
require 'capistrano/rbenv'

require 'capistrano/rails/console'
require 'capistrano3/postgres'

# Load custom tasks from `lib/capistrano/tasks` if you have any defined
# and add to deploy.rb like `after "deploy:published", "translation:setup"`
Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }
HERE_DOC
~~~

To see all tasks you can run
```
cap -T
cap -T postgres
```
To see description rake task
```
cap -D my-task
```

Custom tasks have some additional format to plain rake tasks
```
# lib/capistrano/tasks/rake.cap.rb
desc 'Invoke a rake command on the remote server: cap qa "invoke[db:migrate]"'
task :invoke, [:command] => 'deploy:set_rails_env' do |task, args|
  on primary(:app) do
    within current_path do
      with :rails_env => fetch(:rails_env) do
        rake args[:command]
      end
    end
  end
end
```
You need to set credentials master key
```
# /home/deploy/myapp/.rbenv-vars
RAILS_MASTER_KEY=1234
DATABASE_URL=postgresql://deploy:PASSWORD@127.0.0.1/myapp_production
```

Configure server
```
# config/deploy.rb
lock '~> 3.16.0'

set :application, 'myapp'
set :repo_url, 'git@github.com:duleorlovic/myapp.git'

# Default branch is :master
# ask :branch, `git rev-parse --abbrev-ref HEAD`.chomp
set :branch, :main

# Default deploy_to directory is /var/www/my_app_name
# set :deploy_to, "/var/www/my_app_name"
set :deploy_to, "/home/ubuntu/#{fetch(:application)}"
```

```
# config/deploy/production.rb
server 'production-app.myapp.com',
       user: 'ubuntu',
       roles: %w[app db web],
       ssh_options: {
         keys: '~/.ec2/singapore-webapp01-keypair.pem'
       }
```

To deploy you can
```
cap production deploy
```

On digital ocean you can create snapshots and use it to create new droplets
(new droplet can not be less than current hdd size od snapshot).

For redis and sidekiq look below.

# Configuration variables

[configuration](http://capistranorb.com/documentation/getting-started/configuration/)
files can set variables `set :my_var_name, "value"` and fetch values `fetch
:my_var_name`. There are some variables that are used by default:

* `:application` name of application
* `:deploy_to` path on the remote server where the app should be deployed,
initially is `-> { "/var/www/#{fetch(:application)}" }` but I like home folder.
Inside that folder there are:
  * `current` symlink to some release `/var/www/my_app/releases/20170101010101`
  * `releases/` contains timestamped subfolder
  * `repo/` hold git repository
  * `revisions.log` timestaped log for all deploy or rollback actions. You can
    rollback with `cap production deploy:rollback`
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

append :linked_dirs, 'log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'vendor/bundle', '.bundle', 'public/system', 'public/uploads'

# deploy with: cap staging deploy BRANCH=branch, it does not support
# underscores like: my_branch_name
set :branch, ENV['BRANCH'] if ENV['BRANCH']
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

* `:web` role is nginx/apache server. When I look at
  https://github.com/capistrano/rails/blob/master/lib/capistrano/tasks/assets.rake#L136
  I see that `assets_roles` is by defailt `[:web]` so there we execute tasks
  like `deploy:asset`. Probably you need to include also `:worker` role
  ```
  # config/deploy.rb
    # we need assets on worker since we send emails that uses images (like logo)
    set :assets_roles, %i[web worker]
  ```
* `:app` role is for rails app. Capistrano's built-in tasks `deploy:check`,
  `deploy:published` or `deploy:finished` are all run in this role.
  https://capistranorb.com/documentation/getting-started/flow/
  You can attach your tasks after finish publishing the deploy
  ```
  # lib/capistrano/tasks/puma.rake
  namespace :puma do
    desc 'Restart puma service'
    task :restart do
      on roles(:app) do
        execute 'sudo service puma restart'
      end
    end

    after 'deploy:published', 'puma:restart'
  end
  ```
* `:db` role is for mysql/postgresql database (requires `primary: true`) or just
  running migrations. `capistrano-rails` plugin provides the `deploy:migrate`
  https://github.com/capistrano/rails/blob/master/lib/capistrano/tasks/migrations.rake
  Here we are adding `public/assets/` to *linked_dirs* (which should also
  include `public/packs`)

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
occurs, so you can use your local keys on server with those option:
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
which uses your keys on remote server to download private repositories.

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
        // this forwards remote ip address to the app
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        access_log /home/deploy/myapp/current/log/nginx.rails.access.log;

        // I tried to pass scheme http or http to the app but the problem is nlb
        // does not send header to the app. Elb sends headers
        // https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html#request-routing
        // but NLB does not set headers
        // https://stackoverflow.com/questions/54558742/x-forwarded-proto-not-being-passed-through-aws-alb-sandwich-with-palo-alto-vm-fi
        proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
        // so only solution is to redirect in javascript
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

See multiple logs (like multilog or multi tail) in one terminal window

~~~
vagrant ssh
sudo apt-get install multitail
multitail /var/log/nginx/* /home/deploy/myapp/current/log/*
~~~

see logs of capistrano on local machine

```
less log/capistrano.log
```

# Rubber

This gem is used just as wrapper for common stack, like passenger_postgresql,
targeting EC2 instances and do all provision stepts.
It depends on capistrano 2.
[railscast](http://railscasts.com/episodes/347-rubber-and-amazon-ec2)
[google group](https://groups.google.com/forum/?fromgroups#!forum/rubber-ec2)
[wiki](https://github.com/rubber/rubber/wiki/Quick-Start)

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
to `config/rubber` which you can customize, usually only `yml` files.
Capistrano recepies files `.rb`  usually do not need to change, and also
configuration files `role/*.conf` are up-to-date.
You can start configuring `config/rubber/rubber.yml` and `config/deploy.rb`.

In `rubber.yml` add AWS root security credentials (or another IAM user) to
access API and EC2 keypairs to access machine.
User security credentials you can export in env.
Keypair you can download to home `~/.ec2` folder, rename without `pem` and
change permissions `chmod 600 ~/.ec2/gsg-keypair`.
No need to create public version `ssh-keygen -y -f ~/.ec2/aws_test >
~/.ec2/aws_test.pub`). In case of `The key pair  does not exist
(Fog::Compute::AWS::NotFound)` you should check if keypair is for same region.
Also `image_id` should be some of the
http://cloud-images.ubuntu.com/locator/ec2/ You can choose t2.micro since it
Free tier applicable:

~~~
# config/rubber/rubber.yml

# this should be one word and app will be in /mnt/myapp-production/releases/...
app_name: myapp

app_user: ubuntu

cloud_providers:
  aws:
    access_key: "#{ ENV['AWS_ACCESS_KEY_ID'] }"
    secret_access_key: "#{ ENV['AWS_SECRET_ACCESS_KEY'] }"
    account: "#{ ENV['AWS_ACCOUNT_ID'] }"

    key_name: us-east-1-gsg-keypair

    image_type: t2.micro
    # Ubuntu 16
    image_id: ami-04169656fea786776

prompt_for_security_group_sync: false

# this is just default prompt
staging_roles: "app,db:primary=true"
~~~

We use just web app and db role, not all roles:
apache,app,collectd,common,db:primary=true,elasticsearch,examples,graphite_server,graphite_web,graylog_elasticsearch,graylog_mongodb,graylog_server,graylog_web,haproxy,mongodb,monit,passenger,postgresql,postgresql_master,web,web_tools
Role `app` depends on `passenger` which depends on `apache`.
Role `"db:primary=true"` depends on `postgresql` and `postgresql_master`
I had a problem with `web` role since it depends on `haproxy`
(`config/rubber/rubber-complete.yml`) which has some errors, so I remove that
dependency:
~~~
# config/rubber/rubber-complete.yml
  web: []
  app: [passenger]
~~~
We could remove that role, but we need to assign security groups.

That imlies to change passenger port to 80

~~~
# config/rubber/rubber-passenger.yml
passenger_listen_port: 80
passenger_listen_ssl_port: 443
~~~

For Ubuntu 16 (which is ) we need to remove package libapache2-mod-proxy-html
for apache
https://groups.google.com/forum/#!topic/rubber-ec2/fut5WZ6TicE

~~~
# config/rubber/rubber-apache.yml
roles:
  apache:
    packages: [apache2, apache2-utils, libcurl4-openssl-dev, libapache2-mod-xsendfile]
  web_tools:
    packages: [apache2, apache2-utils, libcurl4-openssl-dev, libapache2-mod-xsendfile]
~~~

and remove apache2-mpm-prefork, apache2-prefork-dev for passenger and remove
passenger version

~~~
# config/rubber/rubber-passenger.yml
    packages: [libcurl4-openssl-dev, libapache2-mod-xsendfile, libapache2-mod-passenger]
~~~

and make sure it has the same ruby version which if going to be build in
`config/rubber/deploy-setup.rb` You can use later ruby-build https://github.com/rbenv/ruby-build/releases and some of ruby versions `ruby-build --definitions`

~~~
# config/rubber/rubber-ruby.yml
ruby_build_version: 20180822
ruby_version: 2.3.3
~~~

For rails you need to uncommend `gem 'mini_racer', platforms: :ruby` in Gemfile.

To provision you can run

~~~
cap rubber:create_staging
# Hit enter at prompts to accept the default value for params
# this is the same as manually
cap rubber:create
cap rubber:bootstrap
cap deploy
# or you can set params in env var like
# ALIAS=web01 ROLES=app cap rubber:create
~~~

This will create `config/rubber/instance-production.yml`. It will also reboot
after creating.  `ALIAS` is name ie subdomain and can not contain underscore. If
you run script again and change name it will add another InstanceItem `web02`
(new EC2 machine). It will also update your `/etc/hosts` so you can access in
browser with production.foo.com instead of ip address of newly created instance.
You can remove (terminate) all EC2 instances with

~~~
cap rubber:destroy
# type 'web01' or your alias
~~~

You can create additional instances:

~~~
ALIAS=web01 ROLES=app cap rubber:create
ALIAS=web01 ROLES=app cap rubber:bootstrap # this is idempotent and it is
# reading config/rubber/instance-production.yml for web01 InstanceItem
cap deploy:cold
~~~

and you can see current in `config/rubber/instance-vagrant.yml`

Roles should be defined in that instance file. Note that for first time we are
defining roles using env ROLES on rubber create task.

Note that if [current instance- file
does not exists](https://github.com/rubber/rubber/blob/master/lib/rubber/recipes/rubber/utils.rb#L20)
`rubber:create_staging` will create new instance using `staging_roles`. If that
instance file exists, than roles from it will be used.

Rails is running on port 7000 inside virtualbox. haproxy routes standard http
80 port to rails 7000, so you can access site on <http://default.foo.com/>

# ERRORS

To see logs you can

~~~
ssh -i $PEM_FILE ubuntu@web01.foo.com
tail -f /var/log/apache2/* /mnt/myapp-production/current/log/*
# in another terminal
curl localhost:7000
~~~

logs for other services you can find
https://github.com/rubber/rubber/wiki/Troubleshooting
or you can use existing task:

~~~
RAILS_ENV=vagrant cap rubber:tail_logs
~~~

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
error on index page. Probably for first time you need to create database

~~~
cd /mnt/myapp-production/current
bundle exec rake db:setup
~~~

For errors like
```
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
solution is
```
# sometimes you need to start agent, for example you ssh to machine, so for
error like: # Could not open a connection to your authentication agent.
eval "$(ssh-agent -s)"

ssh-add ~/.ssh/id_rsa
```

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

If you get errors like

~~~
Received disconnect from 54.210.7.47 port 22:2: Too many authentication failures
Connection to production.redirection.my-domain.com closed by remote host.
Connection to production.redirection.my-domain.com closed.
~~~

it might help to add username `ubuntu@`, like `ssh -i $PEM_FILE
ubuntu@54.254.110.113`

To add env secrets you can write them in a `keys.sh` file and source from
`~/.bashrc` before return if not interactivelly. If you are using rubber than
use system wide bash for example `/etc/profile` and put keys for example
`/root/keys.sh`.

~~~
# keys.sh
export SECRET_KEY_BASE=52b57...
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
inside rubber plugin does not take that name). All that plugin does is a call to
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

Tips:

I create shortcuts in home `/home/ubuntu` and `/home/deploy_user` for easier
access
```
ln -s /var/www/myapp/shared/ .
ln -s /var/www/myapp/current .
```

Secrets in capistrano is long issue
https://github.com/capistrano/capistrano/issues/1884
I put secrets and source in bashrc for both users

```
# /var/www/myapp/env.sh
export RAILS_ENV=production
export SMTP_USERNAME=username

# /home/ubuntu/.bashrc
...
# for ubuntu user, it can be at the end of a file
source /var/www/myapp/env.sh

# /home/deploy_user/.bashrc
# for deploy_user, it needs to be on first line, since it is used with ssh
source /var/www/myapp/env.sh
```

When you update secrets, than you need to restart the server `cap production
deploy:restart`. For background jobs to pick up new secrets you need to restart
`cap production sidekiq:restart`.

* if there is an error for
  ```
  sidekiq:monit:monitor
      01 sudo /usr/bin/monit monitor sidekiq_cablecrm_production_0
      01 sudo
      01 :
      01 no tty present and no askpass program specified
  ```
  than I tried to enable ssh with no password for monit with `sudo visudo` and
  add line
  ```
  deploy_user ALL= NOPASSWD: /usr/bin/monit

  ```
  but still same problem.

# RBENV

It is better than rvm since for installation it needs just PATH shims.

Choosing ruby version is using: `RBENV_VERSION` env variable, `.ruby-version`
file in directory of the script you are executing, or its parents to the root
of the filesystem, or `.ruby-version` file in the current working directory or
its parents to the root of the filesystem.
https://github.com/rbenv/rbenv
https://github.com/rbenv/rbenv-installer
https://github.com/rbenv/ruby-build
https://github.com/rbenv/rbenv-vars

```
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
git clone https://github.com/rbenv/rbenv-vars.git ~/.rbenv/plugins/rbenv-vars
exec $SHELL
rbenv install 2.6.3
rbenv global 2.6.3
```

Add env variables to `/home/orlovic/premesti.se/.rbenv-vars`

To use rbenb variables in capistrano task you can write
```
namespace :db do
  desc 'Load myapp users from shared/users.sql'
  task :load_myapp_users do
    on roles(:all) do
      execute "cd #{current_path} && $HOME/.rbenv/bin/rbenv exec bundle exec script/db_load_myapp_users.rb"

      # This will raise error Could not locate Gemfile or .bundle/ directory!!!
      # within current_path do
      #   execute "$HOME/.rbenv/bin/rbenv exec bundle exec script/db_load_myapp_users.rb"
      # end
    end
  end
end
```

Not so usefull post
https://piotrmurach.com/articles/working-with-capistrano-environment-variables-and-remote-commands/


# Passenger and NGINX

To configure nginx with passenger, and redirect from non www to www, you can use

```
# /etc/nginx/sites-available/trk.conf
server {
    listen 80;
    server_name www.trkapp.com;

    root /var/www/trkapp/current/public;

    passenger_enabled on;
    passenger_ruby /usr/local/rvm/gems/ruby-2.5.3/wrappers/ruby;
}
server {
    listen 80;
    server_name trkapp.com ;
    return 301 http://www.trkapp.com$request_uri;
}
```

Restart rails
```
cap production passenger:restart
```

# Redis

To start redis server on boot you need to change `vi /etc/redis/redis.conf` to
contains line:

```
supervised systemd
```

than you need to enable booting redis server on start
```
systemctl enable redis-server
```
This will create a symlink
```
/etc/systemd/system/redis.service → /lib/systemd/system/redis-server.service
```

You can reboot
```
sudo reboot
```
To test if redis is working and to clear cache you can run on server
```
redis-cli FLUSHALL
```

Instead of disabling protected mode we will bind on private network interface

```
# list all interface
ip a
# find private ip address like ens5 172.30.5.211
sudo vi /etc/redis/redis.conf
# add that ip address to bind command
bind 127.0.0.1 ::1 172.30.5.211
# and change the policy
maxmemory-policy allkeys-lfu

# test that we can not access from outside aws using public ip
redis-cli -h 3.81.27.218 ping
# pong
# but we can access from application server using internal network
redis-cli -h 172.30.5.211 ping

# to connect to the same database you need to use SELECT in redis-cli
# for example when redis url is: redis://172.30.5.211:6379/1
redis-cli -h 172.30.5.211
select 1
```

# Sidekiq

```
# use system instead user
set :sidekiq_service_unit_user, :system
```
You need to run on production
https://github.com/seuros/capistrano-sidekiq#integration-with-systemd
```
sudo loginctl enable-linger $USER
```
To create symlink in `~/.config/systemd/user/sidekiq-production.service`
(which is generated by gem's default using rvm) you can use shims
https://gist.github.com/Paprikas/11fc850f81b687d9cbb7a8efb5ead208
```
# config/deploy.rb
# https://github.com/seuros/capistrano-sidekiq#integration-with-systemd
set :bundler_path, '/home/deploy/.rbenv/shims/bundle'
set :init_system, :systemd
```
Than you need to install
```
cap production sidekiq:install
# output should be
      01 mkdir -p /home/deploy/.config/systemd/user
    ✔ 01 deploy@138.68.225.244 0.583s
      Uploading /home/deploy/.config/systemd/user/sidekiq-production.service 100.0%
      02 systemctl --user daemon-reload
    ✔ 02 deploy@138.68.225.244 0.623s
      03 systemctl --user enable sidekiq-production.service
      03 Created symlink /home/deploy/.config/systemd/user/default.target.wants/sidekiq-production.service → /home/deploy/.config/systemd/user/sidekiq-pro...
    ✔ 03 deploy@138.68.225.244 0.619s
```

# Tips

* check if you already pushed before update https://victorafanasev.info/tech/capistrano-automatically-check-if-remote-origin-updated-before-deploy
  ```
  # lib/capistrano/tasks/deploy_git_uptodate_check.rake
  # https://victorafanasev.info/tech/capistrano-automatically-check-if-remote-origin-updated-before-deploy
  namespace :deploy do
    desc 'Check if origin master synced with local repository before deploy'
    task :git_uptodate_check do
      if !`git status --short`.empty?
        raise 'Please commit your changes first'
      elsif `git remote`.empty?
        raise 'Please add remote origin repository to your repo first'
      elsif !`git rev-list master...origin/master`.empty?
        raise 'Please push your commits to the remote origin repo first'
      end
    end
  end

  before 'deploy', 'deploy:git_uptodate_check'
  ```
* on digital ocean 1GB RAM memory you can not compile webpack, and it stop
  without any error
  ```
 DEBUG [5c5758a9] 	Compiling...

 DEBUG [5c5758a9] 	Compilation failed:
  ```
  solution is to limit how much memory node can use by setting up env variable
  https://github.com/rails/webpacker/issues/2033
  https://github.com/rails/webpacker/issues/2143 (related)

  but if it is too low you can get this error
  ```
  FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory
  ```
  and you need to increase the memory to at least 1000MB
```
# /home/deploy/move_index/.rbenv-vars
# check current value of max-old-space-size, not that without bundle it will
# show original value
# $HOME/.rbenv/bin/rbenv exec bundle exec node -e 'console.log(v8.getHeapStatistics().heap_size_limit/(1024*1024))' # => 1024
node -e 'console.log(v8.getHeapStatistics().heap_size_limit/(1024*1024))' # => 994
NODE_OPTIONS=--max-old-space-size=1000
```
To check what memory limit is used you can write a task that is run every time
before assets:precompile
```
# lib/tasks/before_assets_precompile.rake
task :before_assets_precompile do
  # run a command which starts your packaging
  system("env | grep NODE_OPTIONS")
  system("node -e 'console.log(v8.getHeapStatistics().heap_size_limit/(1024*1024))' # => 994")
end

# every time you execute 'rake assets:precompile'
# run 'before_assets_precompile' first
Rake::Task['assets:precompile'].enhance ['before_assets_precompile']
```
so you can see in output
```
00:25 deploy:assets:precompile
      01 $HOME/.rbenv/bin/rbenv exec bundle exec rake assets:precompile
      01 ** Invoke assets:precompile (first_time)
      01 NODE_OPTIONS=--max-old-space-size=1000
      01 1048
      01 ** Invoke yarn:install (first_time)
      01 ** Execute yarn:install
```


not sure how to limit max memory for ruby bundler
* to run on specific host you can use host filtering
  ```
  bundle exec cap staging locally:delayed_job:start HOSTS=18.232.175.34
  bundle exec cap staging locally:delayed_job:start ROLES=worker
  ```

* upload files
  ```
  # lib/capistrano/tasks/upload.rake
  namespace :upload do
    desc <<~HERE_DOC
      Upload files

      cap production upload:files README.md
    HERE_DOC
    task :files do
      on roles(:all) do
        index = ARGV.index 'upload:files'
        ARGV[index + 1..-1].each do |file_name|
          puts "Uploading #{file_name} to #{current_path}/#{file_name}"
          begin
            upload! file_name, "#{current_path}/#{file_name}"
          rescue StandardError => e
            puts e.message
            puts 'Try to run cap upload:files_f to create folder first and upload files (slower version)'
          end
        end
        exit # so we do no proccess filename arguments as cap tasks
      end
    end

    desc <<~HERE_DOC
      Create folder and upload files (this is slower than just upload)

      cap production upload:files_f README.md
    HERE_DOC
    task :files_f do
      on roles(:all) do
        index = ARGV.index 'upload:files_f'
        ARGV[index + 1..-1].each do |file_name|
          puts "Create folders #{File.dirname("#{current_path}/#{file_name}")} and upload #{file_name}"
          execute :mkdir, '-p', File.dirname("#{current_path}/#{file_name}")
          upload! file_name, "#{current_path}/#{file_name}"
        end
        exit # so we do no proccess filename arguments as cap tasks
      end
    end
  end
  ```

* https://github.com/javan/whenever is used to create crontab listing using
  job type: `runner`, `rake` or `command`. `:task` is replaced with first
  argument. here are defaults https://github.com/javan/whenever/blob/e75fd0c21c73878a6140867ab7053cce9a1e5db6/lib/whenever/setup.rb
  https://github.com/javan/whenever/blob/e75fd0c21c73878a6140867ab7053cce9a1e5db6/lib/whenever/job_list.rb#L54
  You can not use capistrano commands even it looks like capistrano. You can set
  :whenever_variables and use that.
  ```
  job_type :command, ":task :output"
  job_type :rake,    "cd :path && :environment_variable=:environment bundle exec rake :task --silent :output"
  job_type :runner,  "cd :path && bin/rails runner -e :environment ':task' :output"
  job_type :script,  "cd :path && :environment_variable=:environment bundle exec script/:task :output"
  ```
  When using rbenv we need to prepend with `rbenv exec` so first we need to pass
  env variable in `config/deploy.rb`
  ```
  # config/deploy.rb
  # https://github.com/javan/whenever/blob/master/lib/whenever/capistrano/v3/tasks/whenever.rake
  set :whenever_roles, -> { :worker }
  # https://github.com/javan/whenever/wiki/rbenv-and-capistrano-Notes
  set :whenever_environment, fetch(:stage)
  set :whenever_variables, (lambda do
    "'environment=#{fetch :whenever_environment}" \
    "&rbenv_root=#{fetch :rbenv_path}'"
  end)
  ```
  and than use it in schedule.rb
  ```
  # config/schedule.rb
  # https://github.com/javan/whenever/issues/75#issuecomment-380747
  # https://github.com/javan/whenever/wiki/rbenv-and-capistrano-Notes
  set :path, '/home/ubuntu/movebase/current'
  if @rbenv_root
    # when calling using capistrano than set correct ruby and env
    set :rbenv_exec, "#{rbenv_root}/bin/rbenv exec"
  else
    set :rbenv_exec, ''
  end

  # Make sure that log/cron folder exists: mkdir ~/movebase/current/log/cron

  # Example: runner 'MyClass.some_function' (please use only one word, no space,
  # brackets...)
  job_type :runner, %(cd :path &&
     start=`date` &&
    :rbenv_exec bundle exec rails runner -e :environment :task 2>&1 |
    (echo $start :task started; cat -; echo `date` :task finished)
    >> ':path/log/cron/:task.log' 2>&1
  )

  job_type :rake, %(cd :path &&
     start=`date` &&
    :rbenv_exec bundle exec rake :task |
    (echo $start :task started; cat -; echo `date` :task finished)
    >> ':path/log/cron/:task.log' 2>&1
  )

  every :reboot do
    runner 'MyService.new.perform_and_puts'
    rake 'sitemap:refresh'
  end
  ```

  Use with capistrano
  https://github.com/javan/whenever/wiki/rbenv-and-capistrano-Notes
  ```
  # config/deploy.rb
  set :whenever_environment, fetch(:stage)
  set :whenever_variables, (lambda do
    "'environment=#{fetch :whenever_environment}" \
    "&rbenv_root=#{fetch :rbenv_path}'"
  end)
  ```
  so using capistrano task `cap production whenever:update_crontab` you update
  `whenever_variables` with `rbenv_root` so when ssh command is executed
  ```
  cap production whenever:update_crontab
  00:00 whenever:update_crontab
        01 $HOME/.rbenv/bin/rbenv exec bundle exec whenever --update-crontab myapp_production --set 'environment=production&rbenv_root=$HOME/.rbenv' --roles=app,db,web 
  ```
  we have defined rbenv_root and whenever can use it
  ```
  # config/schedule.rb
  set :output, error: 'cron.error.log', standard: 'cron.log'
  # https://github.com/javan/whenever/wiki/rbenv-and-capistrano-Notes
  if defined? rbenv_root
    job_type :delayed_job, %(cd :path :rbenv_root && :environment_variable=:environment :rbenv_root/bin/rbenv exec bundle exec bin/delayed_job :task --queues=webapp,mailers :output)
  else
    job_type :delayed_job, %(cd :path && :environment_variable=:environment bin/rbenv exec bundle exec bin/delayed_job :task --queues=webapp,mailers :output)
  end

  every :reboot do
    delayed_job 'start'
  end
  ```

  You can use interpolation with `#{name}` or `:name` inside `job_type` but for
  `set` you can only use `#{name}`. To check crontab syntax simply run
  `whenever` in development machine.
  ```
  set :path, '/home/ubuntu/myapp/current'
  set :p, "#{path}/log/cron.script.log"
  # this will not work since colon interpolation does not work for set
  # set :p, ":path/log/cron.script.log"

  job_type :p, ":path/log/cron.script.log"
  ```

* lets encrypt ssl cert
  ```
  # config/etc/letsencrypt/live/myapp.com/renew_and_upload_certificate.sh
  #!/bin/bash
  set -e # Any commands which fail will cause the shell script to exit immediately
  set -x # show command being executed

  # Run this script as root user.
  # You can run multiple times

  date
  cd /etc/letsencrypt/live/myapp.com

  # STEP 1: Generate certificate
  AWS_CONFIG_FILE=/home/ubuntu/efs/.elbas_keys certbot certonly -q --dns-route53 -d *.myapp.com
  # without -q quiet output should be like
  # - Congratulations! Your certificate and chain have been saved at:
  #   /etc/letsencrypt/live/myapp.com/fullchain.pem

  # exit if cert has not been changed
  cmp --silent cert.pem.uploaded cert.pem && exit 1
  cp cert.pem cert.pem.uploaded

  # STEP 2: Upload to ACM and get $CERTIFICATE Arn
  AWS_CONFIG_FILE=/home/ubuntu/efs/.elbas_keys aws acm import-certificate --certificate fileb://cert.pem --certificate-chain fileb://chain.pem --private-key fileb://privkey.pem > step1.json
  export CERTIFICATE=`cat step1.json | jq -r '.CertificateArn'`
  # like CERTIFICATE=arn:aws:acm:us-east-1:664559194543:certificate/16e41ded-6cad-4e6e-b161-018d38d1a378


  # STEP 3: Find load balancer ARN and set
  # AWS_CONFIG_FILE=/home/ubuntu/efs/.elbas_keys aws elbv2 describe-load-balancers
  export LOAD_BALANCER=arn:aws:elasticloadbalancing:us-east-1:664559194543:loadbalancer/app/alb-gfd-app/3d181aae37d3b54d

  # STEP 4: Find listener ARN
  # AWS_CONFIG_FILE=/home/ubuntu/efs/.elbas_keys aws elbv2 describe-listeners --load-balancer $LOAD_BALANCER
  export LISTENER=arn:aws:elasticloadbalancing:us-east-1:664559194543:listener/app/alb-gfd-app/3d181aae37d3b54d/827991f6232bcd5c

  # STEP 5: Set new certificate for listener
  AWS_CONFIG_FILE=/home/ubuntu/efs/.elbas_keys aws elbv2 add-listener-certificates --listener-arn $LISTENER --certificates CertificateArn=$CERTIFICATE
  ```
  and create a crontab task
  ```
  cap production ssh-worker

  sudo ln -s /home/ubuntu/myapp/current/config/etc/letsencrypt/live/myapp.com/renew_and_upload_certificate.sh /etc/letsencrypt/live/myapp.com/
  sudo su
  crontab -e
  # add task

  SHELL=/bin/sh
  PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/snap/bin

  0 0 * * * perl -e 'sleep int(rand(43200))' && /etc/letsencrypt/live/myapp.com/renew_and_upload_certificate.sh >> /home/ubuntu/myapp/current/log/renew_and_upload_certificate.log 2>&1
  ```

# ERRORS

If you have an error
```
Net::SSH::AuthenticationFailed: Authentication failed for user ubuntu@ec2...
```
than it is probably your PEM file in different location, so to fix this you can
copy to `~/config/keys/pems/gfd-keypair.pem` or export in PEM_FILE
```
export PEM_FILE=~/gfd-keypair.pem
```
I also got the same error but PEM was there.
I solved by downgrading and upgrading capistrano gem.
https://stackoverflow.com/questions/59556891/rails-capistrano-authentication-fail-error
https://stackoverflow.com/questions/72014495/capistrano-rails-6-netssh-authentication-failed-for-user-deploy-sshkitrun
https://stackoverflow.com/questions/63309504/ssh-authentication-fails-when-running-capistrano-cap-production-deploy

* error like
```
cap aborted!
Aws::Errors::MissingCredentialsError: unable to sign request without credentials set
```
is because you have not set up credentials, env or default location
```
# ~/.aws/credentials
[default]
aws_access_key_id = AKIA...
aws_secret_access_key = 545...
```
If you have an error
```
Aws::Errors::MissingRegionError: No region was provided. Configure the `:region` option or export the region name to ENV['AWS_REGION']
```
than it is probably you can not read development credentials so you need a
`config/credentials/development.key` and check if this works
```
rails credentials:edit -e development
```
You should also get a `config/master.key` to be able to edit production secrets.

* error like
```
cap aborted!
NoMethodError: undefined method `instances' for nil:NilClass
Did you mean?  instance_values
```
is solved by adding credentials, in my case I store in `rails credentials:edit`
so you need both `config/master.key` and `config/credentials.yml.enc`
* error like
```
00:01 git:check
      01 git ls-remote git@github.com:duleorlovic/myapp.git HEAD
      01 git@github.com: Permission denied (publickey).
      01 fatal: Could not read from remote repository.
      01
      01 Please make sure you have the correct access rights
      01 and the repository exists.
      01 git@github.com: Permission denied (publickey).
      01 fatal: Could not read from remote repository.
      01
      01 Please make sure you have the correct access rights
```
can be solved with
```
ssh-add ~/.ssh/id_rsa
```

* for error
```
Missing template layouts/application with
or
Manifest webpack
```
please make sure that instance contains all roles `roles: %w[app web db]`

# Errors

For error
```
cap production elbas:ssh
(Backtrace restricted to imported tasks)
cap aborted!
LoadError: cannot load such file -- elbas/capistrano
/home/dule/myapp/myapp/Capfile:13:in `<top (required)>'
(See full trace by running task with --trace)
```
solution is to use `bundle exec`
```
bundle exec cap production elbas:ssh
```

