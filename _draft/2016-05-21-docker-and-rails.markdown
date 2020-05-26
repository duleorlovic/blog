---
layout: post
title: Docker and Rails
---

https://docs.docker.com/get-started/

```
docker run hello-world
```
# Commands

* `docker run name-of-image` run (download if does not exists) https://docs.docker.com/engine/reference/run/
  * `-i` interactive
  * `-P` open ports
  * `-t` by tag
* `docker build -t my-image .` build image from current folder with Dockerfile
  and name it my-image
* `docker image list` list local images, also `docker images`
* `docker container ls --all` list all containers (spawned by the image), if it
  still running than no need `--all`
* `docker tag 31313131 duleorlovic/my-image:latest` put a tag
* `docker push duleorlovic/my-image` push to
  https://hub.docker.com/u/duleorlovic/
* `docker pull` pull image from repository
* `docker ps` To see all running containers. Container is a running instance of
  an image. Also `docker stats`
* `docker stop 323232`, stop all `docker kill $(docker ps -a)`, remove also
  `docker rm $(docker ps -a -q)` remove not running docker containers
* `docker system prune -a` to remove all images from the system

Dockerfile
https://docs.docker.com/engine/reference/builder/

* `FROM ruby:2.7.0` set parent image
* `COPY local-file docker-target-location` copy from local to container
* `WORKDIR /app` change root
* `RUN bundle install` run commands is creating new image
  https://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/
* `ENV`
* `ENTRYPOINT` it can be ruby script so you can pass arguments to it
  https://docs.docker.com/engine/reference/builder/#entrypoint
  ```
  # Dockerfile
  RUN mkdir -p /app
  WORKDIR /app
  COPY . ./

  ENTRYPOINT ["/app/app.rb"]
  ```

  ruby file should be executable
  ```
  # app.rb
  #!/usr/bin/env ruby
  if __FILE__==$0
    puts "ergs=#{ARGV}"
  end
  ```

  so you can run
  ```
  docker build . -t aws
  docker run aws param1
  ```

Follow https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
Also https://vitobotta.com/2020/04/06/optimised-docker-builds-rails-apps/

# Rails

For rails we can use this Dockerfile [link](http://blog.codeship.com/running-rails-development-environment-docker/)

~~~
# Dockerfile
# https://registry.hub.docker.com/_/ruby?tab=tags
FROM ruby:2.6.5

# Install apt based dependencies required to run Rails as
# well as RubyGems. As the Ruby image itself is based on a
# Debian image, we use apt-get to install those.
RUN apt-get update && apt-get install -y \
  build-essential \
    nodejs

# Configure the main working directory. This is the base
# directory used in any further RUN, COPY, and ENTRYPOINT
# commands.
RUN mkdir -p /app
WORKDIR /app

# Copy the Gemfile as well as the Gemfile.lock and install
# the RubyGems. This is a separate step so the dependencies
# will be cached unless changes to one of those two files
# are made.
COPY Gemfile Gemfile.lock ./
RUN gem install bundler && bundle install --jobs 20 --retry 5

# Copy the main application.
COPY . ./

# Expose port 3000 to the Docker host, so we can access it
# from the outside.
EXPOSE 3000

# The main command to run when the container starts. Also
# tell the Rails dev server to bind to all interfaces by
# default.
CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]
~~~

```
# dockerignore
.git*
db/*.sqlite3
db/*.sqlite3-journal
log/*
tmp/*
Dockerfile
README.rdoc
```

Also you can mount volume with `docker run -P -v $(pwd):/app image-search`.
It is easier to write volume in composer file which is used to write all
communication between images

~~~
# docker-compose.yml
app:
  build: .
  command: rails server -p 3000 -b '0.0.0.0'
  volumes:
    - .:/app
  ports:
    - "3000:3000"
~~~

If you have only one you can run
```
# this will run default command:
docker-compose up

# to run arbitrary command
docker-compose run app rake db:setup
```
Than you can add postgres, by changing gem and config/database.yml

~~~
  links:
    - postgres
postgres:
  image: postgres:9.4
  ports:
    - "5432"
~~~


# Instalation

Follow [installing](https://docs.docker.com/engine/installation/linux/ubuntulinux/)

~~~
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
sudo apt-get install apt-transport-https ca-certificates
sudo apt-get install linux-image-extra-$(uname -r)
sudo apt-get install docker-engine
~~~

After you install you should add user to docker group so you do not need `sudo`

~~~
sudo usermod -aG docker ${USER}
~~~

You need to install composer with

~~~
sudo -i
curl -L
https://github.com/docker/compose/releases/download/1.7.1/docker-compose-`uname
-s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
~~~

# Running on windows

Install Docker desktop https://www.docker.com/get-started but it requires
Windows 10 Pro or Enterprise version 15063 :(

)
# Deploy

<http://blog.codeship.com/deploying-docker-rails-app>
https://github.com/docker/labs/tree/master/developer-tools/ruby
<http://blog.kontena.io/heroku-style-application-deployments-with-docker>

# Testing with docker

<https://blog.codeship.com/effectively-testing-dockerized-ruby-applications/>

# Tips

* `docker-compose up` will not generate TTY session but `docker-compose run`
  will do. Add option `--service-ports` to map container ports if you use them.
* `chown orlovic -R .` after initial build, since owner is set to root


