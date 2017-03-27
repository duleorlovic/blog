---
layout: post
title: Docker and Rails
---

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

# Commands

* `docker run name-of-image` # run (download if does not exists)
* `docker images` # list local images
* `docker build -t my-image .` # build image from current folder with Dockerfile
* `docker tag 31313131 duleorlovic/my-image:latest` # put a tag
* `docker push duleorlovic/my-image` # push to
  https://hub.docker.com/u/duleorlovic/

# Rails

For rails we can use this Dockerfile [link](http://blog.codeship.com/running-rails-development-environment-docker/)

~~~
# Dockerfile
FROM ruby:2.2 
MAINTAINER marko@codeship.com

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

and run with `docker run -itP image-search` (`-P` is to open ports, `-i`
interactive, `-t` is find by tag).
Also you can mount volume with `docker run -P -v $(pwd):/app image-search`.
It is easier to write volume in composer file.

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

and run `docker-compose build` `docker-compose up`
Than you can add postgres, by changing gem and config/database.yml

~~~
  links:
    - postgres
postgres:
  image: postgres:9.4
  ports:
    - "5432"
~~~

and running `docker-compose run app rake db:setup`

To see all running containers run `docker ps`. To kill `docker stop 323232`


# Deploy

<http://blog.codeship.com/deploying-docker-rails-app>
<http://blog.kontena.io/heroku-style-application-deployments-with-docker/?utm_source=rubyweekly&utm_medium=email>

# Testing with docker

<https://blog.codeship.com/effectively-testing-dockerized-ruby-applications/>

# Tips

* `docker-compose up` will not generate TTY session but `docker-compose run`
  will do. Add option `--service-ports` to map container ports if you use them.
* `chown orlovic -R .` after initial build, since owner is set to root


