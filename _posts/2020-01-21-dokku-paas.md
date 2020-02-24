---
layout: post
---

# Dokku

<https://github.com/dokku/dokku>
Heroku-like self hosted platform as a service

To create app you can
```
dokku apps:create myApp

# to find help
dokku apps:help

# to list all apps
dokku apps:report

# similar command from docker
docker stats

# to remove destroy delete all exited docker containers
docker rm -v $(docker ps -a -q -f status=exited)
```

Redis
```
# Redis
sudo dokku plugin:install https://github.com/dokku/dokku-redis.git redis
dokku redis:create redis
```

Install swap on server, one time commands to run on server

```
# enable swap http://dokku.viewdocs.io/dokku~v0.17.9/getting-started/advanced-installation/#vms-with-less-than-1gb-of-memory
cd /var
touch swap.img
chmod 600 swap.img

dd if=/dev/zero of=/var/swap.img bs=1024k count=1000
mkswap /var/swap.img
swapon /var/swap.img
free

echo "/var/swap.img    none    swap    sw    0    0" >> /etc/fstab
```

Config
```
# add config environment variable: for buildpack based deployes envs are
# available during build and run time (Dockerfile based are only on run-time)
dokku config myApp
# when setting from local than you do not need to define myApp
dokku config:set RAILS_MASTER_KEY=`cat config/master.key`
# it is stored in: dokku run myApp cat  /app/.env and it is updated on next
# deploy or `ps:rebuild`
```

Restart
```
dokku ps:restart rails_6
```

From local machine add remote and push the code
```
# dokku.me should point to your server, without proxy
git remote add dokku dokku@dokku.me:myApp

git push dokku master
# output should be like:
=====> Application deployed:
       http://myApp.dokku.trk.in.rs

To dokku.trk.in.rs:myApp
 * [new branch]      master -> master
```

Run commands from local
```
ssh -t dokku@dokku.trk.in.rs run rails_6 ruby -v
# or just using client that is downloaded in ~/.dokku
# git clone git@github.com:dokku/dokku.git ~/.dokku
# alias dokku='$HOME/.dokku/contrib/dokku_client.sh'

dokku run ruby -v
dokku run rails console
```

Add custom domain. On cloudlfare you can map subdomains of subdomain to dokku
server, for example `*.a.trk.in.rs` so you can use any subdomain of that
subdomain. Only think is that you need to add to your app

```
dokku domains:add myApp rails.a.trk.in.rs
dokku domains:set myApp rails.a.trk.in.rs # this will remove all other domains

dokku domains:report
dokku urls rails_6
```

You can inspect other nginx configuration by inspecting it and you can customize
http://dokku.viewdocs.io/dokku/configuration/nginx/

```
less /etc/nginx/conf.d/dokku.conf # here we include all our apps configurations
cat /home/dokku/rails_6/nginx.conf
# upstream rails_6-5000 {
#   server 172.17.0.3:5000;
# }
# test your application inside server
curl 172.17.0.3:5000 -I
# HTTP/1.1 200 OK

# when accessing from outside and ssl is enabled
curl -I rails.trk.in.rs
# HTTP/1.1 301 Moved Permanently
# Location: https://rails.trk.in.rs:443/
```


Enabling https with letsencrypt need installation of plugin

```
sudo dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
# set global config so we do not need to set up for each app
dokku config:set --global DOKKU_LETSENCRYPT_EMAIL=salji@trk.in.rs
```
than you can enable ssl with
```
dokku letsencrypt myApp
# add cronjob for dokku user
dokku letsencrypt:cron-job --add myApp
```

Logs

```
dokku logs # when running from local machine
dokku logs myApp -t
dokku nginx:access-logs myApp -t
dokku nginx:error-logs myApp -t
```

## Posgres

Install plugin
```
# add postgres plugin
sudo dokku plugin:install https://github.com/dokku/dokku-postgres
```

Create service that usually contains only one database and name is stored in
`cat /var/lib/dokku/services/postgres/railsdatabase/DATABASE_NAME`
https://github.com/dokku/dokku-postgres/blob/master/config#L4 and is the same as
service so we need to create separate service for separate app (backup is for
one database with the same name as service)
https://github.com/dokku/dokku-postgres/blob/master/common-functions#L26

railsdatabase
```
# create postgres service with name railsdatabase
dokku postgres:create railsdatabase

# link with the app ie setting DATABASE_URL
dokku postgres:link railsdatabase myApp

# access postgres database
dokku postgres:connect railsdatabase
```

Automatic backups to amazon s3 https://github.com/dokku/dokku-postgres/blob/355952eb1f215d791c769fd320acb0e138525658/common-functions#L171
```
dokku postgres:backup-auth railsdatabase AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY
dokku postgres:backup-schedule railsdatabase CRON_SCHEDULE BUCKET_NAME
dokku postgres:backup railsdatabase BUCKET_NAME

# you can manually
dokku postgres:export railsdatabase > railsdatabase.dump
```

Use https://crontab.guru/ for example `@daily`

When you download dump you can extract and import in local database
```
tar -zxvf p.tgz
pg_restore -c -U postgres -d ruby-getting-started_development -v backup/export
```
or you can import this binary pg_dump to dokku
```
# no need to reset database, it will be automatically cleared and populated
dokku postgres:import railsdatabase < railsdatabase.dump

# if you have plain text SQL (not binary) dump you can use connect
dokku postgres:connect railsdatabase < ./plain.dump
```

Mount persistent storage
http://dokku.viewdocs.io/dokku/advanced-usage/persistent-storage/

# Herokuish

https://github.com/gliderlabs/herokuish
Heroku will
```
remote: -----> Removing BUNDLED WITH version in the Gemfile.lock
```

# Dockerfile

https://pawelurbanek.com/optimize-dokku-deployment-speed
http://dokku.viewdocs.io/dokku/deployment/methods/dockerfiles/
If there are `Dockerfile` than it will be used except there are `BUILDPACK_URL`
env variable or `.buildpacks` file.
If previously buildpack was used than we need to add flag to switch to docker
```
dokku config:unset --no-restart myApp DOKKU_PROXY_PORT_MAP
```

Since env variables are using only during runtime phase, you can use
docker-options plugin http://dokku.viewdocs.io/dokku/advanced-usage/docker-options/
```
dokku docker-options:add node-js-app build '--build-arg NODE_ENV=production'
```
so if your assets depends on some env variable, you should add it.

Dokku uses three phases:
* `build` the container that executes the appropriate buildpack
* `deploy` the container that executes your deployed application
* `run` the container that executes any arbitrary command via `dokku run`

You can add triggers for after deploy actions
```
# app.json
{
  "name": "My Rails app",
  "scripts": {
    "dokku": {
      "predeploy": "bundle exec rake assets:precompile",
      "postdeploy": "bundle exec rake db:migrate"
    }
  }
}
```
Zero downtime deploys http://dokku.viewdocs.io/dokku/deployment/zero-downtime-deploys/

# Similar
* https://github.com/caprover/caprover

https://pawelurbanek.com/rails-heroku-dokku-migration
https://github.com/githubsaturn/captainduckduck
