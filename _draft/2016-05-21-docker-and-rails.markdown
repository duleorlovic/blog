---
layout: post
title: Docker and Rails
---

https://www.udemy.com/course/docker-mastery/
https://github.com/bretfisher/udemy-docker-mastery tutorial code

https://docs.docker.com/get-started/

```
docker run hello-world
# if you have exposed ports you need to
docker run -p 3000:3000 my-image
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
* `docker container exec -it container-name ls` run additional command in
  existing specific container (docker ps will not show additional container)
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
  in `/home/orlovic/.docker/config.json`) or to make sure you use correct
  username try `docker login -u duleorlovic`
* `docker image build -t my-image .` build image from current folder with
  Dockerfile and name it my-image
* `docker history nginx:latest` show layers how image was built

Network
* `docker network ls` type: bridge (attached to my card), host (skips virtual
  networks) and null. Inside same network containers can communicate without
  need to open ports.
* `docker network inspect bridge` you can see containers attached
  To see ip address of container
  `docker container inspect --format '{{ .NetworkSettings.IPAddress }}'
  <container-id>`
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
Only `RUN`, `COPY` and `ADD` create layers (other instructions create temp
intermediate images, but do not increase the size of the build)

* `FROM ruby:2.7.0` set parent image
* `COPY local-file /docker/target/folder` copy from local to container. Any
  change to the files will cause rebuilt of this layer
* `WORKDIR /app` change root to the folder
* `RUN bundle install` run commands is creating new layer so it is good to use
  `&&` to join all bash commands so it create a single layer, huge RUN command
  is using cache busting. To efectivelly boost cache and keep images small use
  clean or rm in same RUN command since `RUN apt-get clean` will not
  affect previous images size since they are already created.
  Also using separate `RUN apt-get update` and `RUN apt-get install git` will
  cause that old cache is used when you change to `RUN apt-get install git curl`
  since all previous lines RUN commands are using from cached images, so the
  advice is to always use `update`, `install` and `rm` on the same line.
  ```
  RUN apt-get update && apt-get install -y \
    alphabetically_sorted_package \
  && rm -rf /var/lib/apt/lists/*
  ```
  Use version when installing apps so it not installed latest.
  Do not store secrets in image, better to use orchestration for that.
  Repeat CMD and ENTRYPOINT from base image as a comment so user do not need to
  search in base image.
  https://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/
  To enable logging all you need to do is to redirect to stdout
  ```
  # forward request and error logs to docker log collector
  RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log
  ```
* `ENV` set key value enviroment variable, but it is hardcoded in the layer so
  better is to unset also in the same line
  ```
  RUN export ADMIN_USER=mark \
    && echo $ADMIN_USER > ./mark \
    && unset ADMIN_USER
  ```
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

  Entry point can be a script (for example to load the db schema when docker
  image is going to be started) which can continue with other arguments. If you
  have both ENTRYPOINT and CMD, it will be concatenated `ENTRYPOINT CMD` when
  you do `docker run my-image`, but if you add other arguments (after `docker`
  you can have options like `-v`, and image name `my-image`) those arguments
  will replace default `CMD`.
  Entrypoing can be a script like
  [docker-entrypoint.sh](https://github.com/docker-library/mysql/blob/15fe7357b165ee8aaa3ce165386f910a53a75087/8.0/Dockerfile.debian#L93)
  that will read ENV variables and perform some tasks (make sure it works well
  if for example db already has loaded the schema), and at the end it will
  execute what you have passed (or default `CMD`) with a `exec $@`
  [link](https://github.com/docker-library/mysql/blob/15fe7357b165ee8aaa3ce165386f910a53a75087/8.0/docker-entrypoint.sh#L409)
* `VOLUME /var/lib/mysql` Use it for mutable ara, like store data and make it
  persitent data (since we could redeploy containers which are ment to be
  immutable, instead of upgrading package on existing we change in config and
  deploy again). Volumes make special location outside of containers UFS Union
  File System, you can see by `docker container inspect mysql` (search for
  Mounts).  List all volumes `docker volume ls`.  They need extra step to be
  removed `docker volume prune`. To reuse existing volume use `-v` option to use
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

# see running containers similar to docker ps
docker-compose ps

# to run proccesses on containers
docker-compose run <name-of-service> <command>
docker-compose run rails bundle exec rake db:setup
```

To see config that is generated from multiple config
(docker-compose.override.yml if exists, is default override docker-compose.yml)

```
docker-compose -f a.yml -f b.yml config
```

For rails we can use this Dockerfile
[link](http://blog.codeship.com/running-rails-development-environment-docker/)
Also checkout my app:
orlovic@main:~/rails/tmp/docker_rails$ 

~~~
# Dockerfile
# syntax=docker/dockerfile:1
# https://registry.hub.docker.com/_/ruby?tab=tags
# https://docs.docker.com/samples/rails/
FROM ruby:3.0.2

# update yarn repo
RUN curl https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list
# update node
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -

# Install apt based dependencies required to run Rails as
# well as RubyGems. As the Ruby image itself is based on a
# Debian image, we use apt-get to install those.
RUN apt-get update -qq && apt-get install -y nodejs postgresql-client yarn

# Configure the main working directory. This is the base
# directory used in any further RUN, COPY, and ENTRYPOINT
# commands.
WORKDIR /myapp

# Copy the Gemfile as well as the Gemfile.lock and install
# the RubyGems. This is a separate step so the layers with gems
# will be cached unless changes to one of those two files
# are made.
COPY Gemfile Gemfile.lock ./
# Install latest bundler
RUN gem install bundler
RUN bundle install

# Copy the main application. Not needed when using volumes
COPY . ./

RUN bundle exec rake assets:precompile

# Add a script to be executed every time the container starts.
# COPY entrypoint.sh /usr/bin/
# RUN chmod +x /usr/bin/entrypoint.sh
# ENTRYPOINT ["entrypoint.sh"]

# Expose port 3000 to the Docker host, so we can access it from the outside when
# we run with a docker run -p 3000:3000 myimage
EXPOSE 3000

# Configure the main process to run when running the image
CMD ["rails", "server", "-b", "0.0.0.0"]
~~~

Ignore all files that are not needed for the app, image should be smaller as
possible
```
# .dockerignore
.git*
db/*.sqlite3
db/*.sqlite3-journal
log/*
tmp/*
Dockerfile
README.md
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
    # Instead of building the Docker image every time we change something in app
    # we can run bundle and yarn in boot command, note that current folder will
    # overwrite WORKDIR /myapp folder (also node_modules)
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

Here is a source of PWD 
https://github.com/play-with-docker/play-with-docker
and you can run locally with a `docker-compose up`

# Swarm

https://www.youtube.com/watch?v=wYMLP1k9aWQ
You can use docker-machine to create three virtualbox instances
```
docker-machine create manager
docker-machine create worker1
docker-machine create worker2
docker-machine ls
```
note that they are not automatically started when you reboot the machine so you
need to run `docker-machine start manager` but once manager is starter it will
try to start its services on all workers. You can test that by manually shut
down and start worker machine.
Docker Desktop Enterprise includes `docker cluster` which can create nodes on
any provider.
Docker machine can deploy any where amazon, do, google but only for development.
Install docker on any machine using a script https://get.docker.com/
See env which you can use for docker cli to send comands to worker1
```
docker-machine env worker1
eval $(docker-machine env worker1)
docker info | grep Name
# Name: worker1

# remeber to unset in this shell
unset DOCKER_TLS_VERIFY DOCKER_CERT_PATH DOCKER_MACHINE_NAME DOCKER_HOST
```

Inside manager (it is a worker with a permission to controll the swarm)
```
docker-machine ssh manager
# find address with `ip addr` under `eth1`
ip addr show

docker swarm init --advertise-addr 192.168.99.101
# this will create certificate on first manager node, and generate join tokens
# raft is database for configuration, secrets, root CA, it is replicated to
# other mangaers
# copy command docker swarm join --token ...

# on manager you can see all nodes
docker node ls
```

inside workers you need to join
```
docker-machine ssh worker
docker swarm join --token asd123 192.168.99.101:2377
```

Run service on all nodes (service is similar to run, but swarm will execute it)
```
docker service create --name mynginx --publish 8080:80 --replicas 3 nginx
overall progress: 3 out of 3 tasks
1/3: running   [==================================================>]
2/3: running   [==================================================>]
3/3: running   [==================================================>]
verify: Service converged

# check their status
docker service ls
# see on which node it is running
docker service ps mynginx

# To find container id you can join service name and prefix name from ps command
# Note that you can access container by container name only on current manager
# node, you need to inspect service one by one to confirm it is running on
# current node, and to find prefix of container name
docker exec -it <service-name-on-manager-node>.<container-name-prefix><TAB> bash
docker exec -it psql.1.qepi7z3dgmb1<press tab>
docker exec -it psql.1.qepi7z3dgmb1adg1mdastap35 bash
```
You can access any node to see default page http://192.168.99.100:8080/
http://192.168.99.101:8080/ http://192.168.99.102:8080/
To change default page you can run bash
```
docker container ls # find id or name
docker container exec -it d33b9ef99995 bash
echo dule > /usr/share/nginx/html/index.html
```

To scale up the service you can change number of replicas
```
docker service update <service-id> --replicas 3
# easier way
docker service scale web=8 api=6
```
More examples for updating
```
docker service update --image myapp:1.2.1 <service-name>
docker service update --env-add NODE_ENV=production --publish-rm 8080 <service-name>

# you can rebalance, to is will clean up and start again
docker service update --force <service-name>
```

HEALTCHECK is running from inside the container. It is supported in Dockerfile,
docker-compose, stacks
```
docker run --health-cmd="curl -f localhost:9200/_cluster/health || false" elasticsearch
```
interval (default 30s), timeout (default 30s) start-period (default 0) retries
(default 3).
3 statuses: pending, healthy
```
docker service create --health-cmd='pg_isready -U user postgres || exit 1' --name p1 postgres
```

To test that machines are regenerated when they are killed
```
docker container ls # find container-id
docker container rf -f <container-id>
docker container ls
```

Create network `overlay` for intranetwork container-to-container traffic inside
single swarm (overlay across all nodes)
```
docker network create --driver overlay mynetwork
```
Service and networks are many-to-many relationships.

Example setting up drupal and postgresql
```
docker service create --name psql --network mynetwork -e POSTGRES_PASSWORD=mypass postgres
# for password-less use -e POSTGRES_HOST_AUTH_METHOD=trust
docker service create --name drupal --network mynetwork -p 80:80 drupal
```
Runing `docker service ps drupal` you can see that node on which drupal is
running but you can go to any api address of the node `docker node inspect
--format '{{ .Status.Addr }}' worker1` for example it http://192.168.99.102/ and
procceed with db name `postgres` db username `postgres` password `mypass` and
for the host use service name `psql`.
Routing mesh uses virtual ip (vip) address so it load balancing between all
nodes for that service (for example `psql` is dns name for the service).
Routing mash takes ingress (incomming) packets for a service to proper Task
(spans all nodes in a swarm, so you can access any node). External traffic for
all open ports can be accepted on ip of any node, and routed to any other node
(ingress network) for example it can receive on the node that is not even
running the container. This is stateless load balancing (does not work with
session cookies), OSI layer 3 (TCP) not layer 4 (DNS) so you need to use Nginx
or HAProxy LB proxy.

To enable bind mount for service you need to use
```
docker service create --mount type=volume,source=db-data,target=/var/lib/postgresql/data postgres:9.4
```

To see logs
```
docker service logs <my-service>
```

Stacks is for production grade compose, so instead `docker service create` we
use `docker stack deploy` once for all object: services, networks and volumes.
Similar to docker-compose but for swarm, it uses `deploy:` instead `build:`
(which is ignored if you use docker-compose) and docker-compose is not needed.
Stack is running only on one swarm.  Example on
https://github.com/BretFisher/udemy-docker-mastery/blob/main/swarm-stack-1/example-voting-app-stack.yml

```
# example-voting-app-stack.yml
version: "3"
services:
  redis:
    image: redis:alpine
    networks:
      - frontend
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```

to deploy stack
```
wget https://raw.githubusercontent.com/BretFisher/udemy-docker-mastery/main/swarm-stack-1/example-voting-app-stack.yml
docker stack deploy -c example-voting-app-stack.yml voteapp
docker stack services voteapp # similar to service ls
docker stack ps voteapp # similar to service ps
```

You can see where nodes are located using visualizer http://192.168.99.102:8080/

Secrets in the swarm
```
docker secret create psql_user <file_name.txt>
echo 'mydbpass' | docker secret create psql_pass -

docker secret ls
```
to access secrets in containers use file `/run/secrets/psql_pass`
```
docker service create --name psql-test --secret psql_pass -e POSTGRES_PASSWORD_FILE=/run/secrets/psql_pass postgres

docker service ps psql-test
docker exec -it psql-test.1.obksdv2dqxms9907c1t7ae7z1 bash
root@e9c3be01cc9f:/# cat /run/secrets/psql_pass
mydbpass
```
You can add or remove existing containers
```
docker service update --secret-rm psql_user
```
To use in stacks
```
# stack.yml
version: '3.1'
services:
  drupal:
    image: drupal
    secrets:
      - psql_password
      - psql_user
    enviroment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/psql_password
secrets:
  psql_user:
    file: ./psql_user.txt
  psql_password:
    external: true
```
you need to create password `psql_password` before deploying stack.
You can use secrets in `docker-compose -d` (-d is development) without swarm. It
will bind mounts at runtime into the container (-v with file based secrets, does
not work with external passwords).

# Run a private docker registry

https://hub.docker.com/_/registry from Dockerfile
https://github.com/docker/distribution-library-image/blob/ab00e8dae12d4515ed259015eab771ec92e92dd4/amd64/Dockerfile#L11
you can see where data is stored, ie path to volume
```
docker container run -d -p 5000:5000 --name registry -v $(pwd)/registry-data:/var/lib/registry registry
```
and use https://docs.docker.com/registry/#basic-commands

```
docker image tag hello-world:latest localhost:5000/hello-world
docker push localhost:5000/hello-world
```
You can see new repository on http://127.0.0.1:5000/v2/_catalog

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

