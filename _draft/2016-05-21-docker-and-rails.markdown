---
layout: post
title: Docker and Rails
---

https://www.udemy.com/course/docker-mastery/?couponCode=HAPPYJULY2021
https://github.com/bretfisher/udemy-docker-mastery

https://docs.docker.com/get-started/

```
docker run hello-world
```
# Commands

container
* `docker container run name-of-image` run (download if does not exists)
  https://docs.docker.com/engine/reference/run/
  if you add another commands they will be added to ENTRYPOINT as arguments
  `docker container run --rm ruby_echo some arguments`
  * `-it` interactive and enable tty for example to start bash on ubuntu
    container, without need to ssh to it. This will start new container named
    proxy and start bash `docker container run -it --name proxy ubuntu bash`
    To run on existing container see `exec`
  * `-p local-port:image-port` open port: `docker container run -p 80:80 nginx`
    To see ports for existing container `docker container port nginx`
  * `-e` add ENV variable for example `docker container run -d --name mysql -e
    MYSQL_RANDOM_ROOT_PASSWORD=true mysql`
  * `-d` detach, `--rm` automatically remove container when exits, usefull just
    to test some distros `docker container run --rm -it ubuntu bash` so you do
    not neet to remove exited containers
* `docker container inspect name-of-image` to see how we started the container
  To format output you can use format `-f` option
* `docker container exec -it container-name` run additional command in existing
  container (docker ps will not show additional container)
* `docker container start -ai ubuntu` start stopped container and open a shell
* `docker container ls --all` list all containers (spawned by the image), if it
  still running than no need `--all`. List processes `docker top container-name`
* `docker ps` To see all running containers. Container is a running instance of
  an image. Also `docker container stats`
* `docker container logs my_nginx` or follow `docker container logs -f postgres`
* `docker container stop 323232`, stop all `docker kill $(docker ps -a)`, remove
  stopped container `docker rm my_nginx` (force `docker container rm -f
  my_nginx`. Remove stopped containers, not used networks and image images
  `docker system prune -a`. See system usage with `docker system df`

Image build
Image is app binaries and dependencies and metadata (docker inspect) about the
image data and how to run image (ports opened, env set, command to be run).
'ordered collection of root filesystem changes and the corresponding execution
parameters for use within a container runtime'. Host is providing a kernel and
drivers.
http://hub.docker.com/ Alpine means it is light
* `docker pull nginx:1.11.9` pull image from repository
* `docker image ls` list local images, also `docker images`
* `docker image tag name-of-source-image duleorlovic/my-image:latest` put a tag
  If a tag starts with your-docker-id/some-tag it will create a repo on hub.
  `:latest` tag is default tag.
* `docker image push duleorlovic/my-image` push to
  https://hub.docker.com/u/duleorlovic/ you need to login `docker login` (save
  in `/home/orlovic/.docker/config.json`)
* `docker image build -t my-image .` build image from current folder with
  Dockerfile and name it my-image
* `docker history nginx:latest` show layers how image was built

Network
* `docker network ls` bridge (attached to my card), host (skips virtual
  networks) and null. Inside same network containers can communicate without
  need to open ports.
* `docker network inspect bridge` you can see containers attached
  To see ip address of container
  `docker container inspect --format '{{ .NetworkSettings.IPAddress }}' webhost`
* `docker network create` create new network
* `docker network connect my_net my_web` to plug in cable
* `docker network disconnect my_net my_web` to disconnect
* default `bridge` network does not have DNS, but if you create new network
  (which includes DNS) and create containers with `--net my_net` so you can ping
  between containers using their name
  `docker container exec -it my_web curl my_new_web`
* round robin (poor man loadbalancer) is made using a `--network-alias` (or
  `--net-alias`)
  ```
  docker network create my_net
  docker container run -d --net my_net --net-alias duke elasticsearch:2
  docker container run -d --net my_net --net-alias duke elasticsearch:2
  docker container run --rm --net my_net alpine nslookup duke

  Non-authoritative answer:
  Name:	duke
  Address: 172.18.0.4
  Name:	duke
  Address: 172.18.0.5
  ```


Dockerfile stanzas (commands)
https://docs.docker.com/engine/reference/builder/

* `FROM ruby:2.7.0` set parent image
* `COPY local-file /docker/target/folder` copy from local to container
* `WORKDIR /app` change root to the folder
* `RUN bundle install` run commands is creating new layer so it is good to use
  `&&` to join all bash commands so it create a single layer
  https://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/
  To enable logging all you need to do is to redirect to stdout
  ```
  # forward request and error logs to docker log collector
  RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log
  ```
* `ENV` set key value enviroment variable
* `CMD ["nginx", "-g", "daemon off;"]` this command is run when container starts
  (created or started again from exited container). It is overwritten when
  container runs with arguments
* `ENTRYPOINT` it can be ruby script so you can pass arguments to it. It is used
  when running container as an executable.
  https://docs.docker.com/engine/reference/builder/#entrypoint
  Shell form (as string `ENTRYPOINT command param1`) starts as subcommand of
  `/bin/sh -c` so process does not receive `SIGTERM` from `docker stop`
  ```
  # Dockerfile
  RUN mkdir -p /app
  WORKDIR /app
  COPY . ./
  RUN chmod +x app.rb

  ENTRYPOINT ["/app/app.rb"]
  ```

  ruby file
  ```
  # app.rb
  #!/usr/bin/env ruby
  if __FILE__==$0
    puts "args=#{ARGV}"
  end
  ```

  so you can run
  ```
  docker image build . -t ruby_echo
  docker run ruby_echo param1
  ```
* `VOLUME /var/lib/mysql` To store data and make it persitent data (since we
  could redeploy containers which are ment to be immutable, instead of upgrading
  package on existing we change in config and deploy again). Volumes make
  special location outside of containers UFS Union File System, you can see by
  `docker container inspect mysql` (search for Mounts).
  List all volumes `docker volume ls`.  They need extra step to be removed
  `docker volume prune`. To reuse existing volume use `-v` option to use
  friendly name instead of ID
  `docker container run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=true -v
  dule-mysql:/var/lib/mysql mysql`.
  https://docs.docker.com/storage/volumes/

  Bind Mounts is mounting local file path (not inside docker installation
  folder as it is for volumes), it starts with slash and can not be
  used in Dockerfile. So instead of COPY we can mount and see our local
  index.html. To find correct path where to put a code you can see Dockerfile of
  official image on hub and search for VOLUME
  ```
  docker container run -d --name nginx -v $(pwd):/usr/share/nginx/html -p 80:80 nginx
  # to check that in container we see all local files
  docker container exec nginx ls /usr/share/nginx/html
  Dockerfile
  index.html
  ```
  To mount in read only use `ro` option: `orig:target:ro`
  To find on which port certain image will run the service you can click on tag
  and open dockerfile and search for `EXPOSE 80` command.
  Or you can download image and run inspect `docker container inspect drupal`
  and search for `ExposedPorts`. If you do not need access service from host
  (for example postgres service is can be used using internal network) you do
  not need to define `ports` in docker compose.

  Example to run yekyll serve which will generate `_site` folder and run server
  ```
  docker run --rm -v $(pwd):/srv/jekyll -p 80:4000 jekyll/jekyll jekyll serve
  ```

Follow https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
Also https://vitobotta.com/2020/04/06/optimised-docker-builds-rails-apps/

Docker compose yml file
```
# docker-compose.yml
version: '3.1'
services: # this is actual container
  #{servicename}: # this is --name in docker coommand, used as DNS name
    image: wordpress
    command: # CMD in Docker
      - 
    # you can build custom image using ./nginx.Dockerfile
    build:
      context: .
      dockerfile: nginx.Dockerfile
    image: nginx-custom


    environment: # -e in docker
      DB_PASSWORD: password
    volumes: # -v in docker, use . instead of $(pwd) Used to perseve data
      - ./data:/var/www/html
      - dule-mysql:/var/lib/mysql
    ports: # -v in docker
      - '80:4000'
    depends_on: # start also this service mysql-primary before self
      - mysql-primary
  #{servicename2}:

# when using friendly name for volumes
volumes:
  dule-mysql:

network:
```

To start containers use
```
docker-compose up

# this will remove containers and networks
docker-compose down
# this will also remove volumes
docker-compose down -v
# to remove custom build images that does not have a name
docker-compose down --rmi local

# to see logs
docker-compose logs -t

docker-compose ps
```

For rails we can use this Dockerfile
[link](http://blog.codeship.com/running-rails-development-environment-docker/)

~~~
# Dockerfile
# syntax=docker/dockerfile:1
# https://registry.hub.docker.com/_/ruby?tab=tags
# https://docs.docker.com/samples/rails/
FROM ruby:3.0.2

# Install apt based dependencies required to run Rails as
# well as RubyGems. As the Ruby image itself is based on a
# Debian image, we use apt-get to install those.
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg -o /root/yarn-pubkey.gpg && apt-key add /root/yarn-pubkey.gpg
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -
RUN apt-get update -qq && apt-get install -y nodejs postgresql-client yarn

# Configure the main working directory. This is the base
# directory used in any further RUN, COPY, and ENTRYPOINT
# commands.
WORKDIR /myapp

# Copy the Gemfile as well as the Gemfile.lock and install
# the RubyGems. This is a separate step so the dependencies
# will be cached unless changes to one of those two files
# are made.
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
RUN bundle install

# # Copy the main application. Not needed when useing volumes
# COPY . ./

# Add a script to be executed every time the container starts.
# COPY entrypoint.sh /usr/bin/
# RUN chmod +x /usr/bin/entrypoint.sh
# ENTRYPOINT ["entrypoint.sh"]

# Expose port 3000 to the Docker host, so we can access it from the outside.
EXPOSE 3000

# Configure the main process to run when running the image
CMD ["rails", "server", "-b", "0.0.0.0"]
~~~

```
# .dockerignore
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
version: '3.9'
services:
  postgres:
    image: postgres:latest
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
    # no need to open ports if it used only inside internal docker network
  rails:
    # https://stackoverflow.com/questions/58933929/yarn-packages-out-of-date-when-running-rails-app-in-docker
    # problem is that we can not run yarn since mount volue will overwrite
    # node_modules, so we need to run yarn 
    command: command: bash -c "bundle install && yarn install --check-files && bundle exec rails s -p 3000 -b '0.0.0.0'"
    build: .
    volumes:
      - .:/myapp
    ports:
      - '80:3000'
    depends_on:
      - postgres
~~~

In rails config you can use container name as DNS name for database
```
# config/database.yml

default: &default
  adapter: postgresql
  host: postgres
  username: postgres
  password: example

```

# PWD Try online in Play with docker

https://hub.docker.com/_/drupal
https://labs.play-with-docker.com/?stack=https://raw.githubusercontent.com/docker-library/docs/f81077b92e4522999836b8c5d098a103f568a431/drupal/stack.yml

TODO:
orlovic@main:~/rails/tmp/docker_rails$ 
https://training.play-with-docker.com/microservice-orchestration/

# Swarm

```
docker swarm init
docker node ls
```

# Instalation

Installing with script `curl -fsSL https://get.docker.com/ | sh` is shorter
script than installing using [installing commands](https://docs.docker.com/engine/installation/linux/ubuntulinux/)

`docker-ce` is package name (older is docker-engine)
Check version
```
sudo docker version
```

After you install you should add user to docker group so you do not need `sudo`

~~~
sudo usermod -aG docker ${USER}
~~~

You need to install two binaries docker machine and docker composer with
https://docs.docker.com/machine/install-machine/
https://docs.docker.com/compose/install/

~~~
base=https://github.com/docker/machine/releases/download/v0.16.0 \
  && curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine \
  && sudo mv /tmp/docker-machine /usr/local/bin/docker-machine \
  && chmod +x /usr/local/bin/docker-machine

sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
~~~

Change image folder to another bigger hard drive if you need, simply create a
file daemon.json and restart the computer
```
# /etc/docker/daemon.json
{
   "data-root": "/media/orlovic/bf12a7e5-a5d4-4532-8612-a3984f90b56c/var-lib-docker"
}

```

Online https://labs.play-with-docker.com/

# Running on windows

Install Docker desktop https://www.docker.com/get-started but it requires
Windows 10 Pro or Enterprise version 15063 :(
https://www.docker.com/products/docker-desktop

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
* tips https://nickjanetakis.com/blog/best-practices-around-production-ready-web-apps-with-docker-compose

