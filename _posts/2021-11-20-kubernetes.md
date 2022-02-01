---
layout: post
---

# Learn

Online learn deploy
https://kubernetes.io/docs/tutorials/kubernetes-basics/
(it uses https://www.katacoda.com/ ) Another online playground
https://labs.play-with-k8s.com/

# Install Kubernetes locally

Kind, k3d, minikube or microk8s

## Kind 

https://kind.sigs.k8s.io/

## K3d

https://k3d.io/

Example 3 nodes
```
k3d cluster create gaia --servers 1 --agents 2 --port 80:80@loadbalancer --port 443:443@loadbalancer --api-port 6443 --k3s-arg '--no-deploy=traefik@server:*'
```

## Minikube

Install minikube
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
minicube itself is running inside docker
```
minikube start
minikube ip

# to stop
docker ps
minikube stop
```

## Microk8s

For ubuntu you can use https://microk8s.io/docs
```
sudo snap install microk8s --classic --channel=1.21
sudo usermod -a -G microk8s $USER
newgrp microk8s

microk8s status --wait-ready
# microk8s is running

# to stop
microk8s stop
```

You can use alias
```
alias k=microk8s.kubectl
```
or you can copy config
```
microk8s config > ~/.kube/config
```

It is running as `/snap/microk8s/2694/kubelite` proccess.
To stop run
```
microk8s stop
```

# Kubectl

Install kubectl https://kubernetes.io/docs/tasks/tools/
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
Check if kubectl can access
```
export KUBECONFIG=~/.kube/kubernetes-rails-example-kubeconfig.yaml
kubectl version

# this will create Deployment, ReplicaSet and Pod
kubectl run my-nginx --image nginx

kubectl create deployment httpenv --image=bretfisher/httpenv
kubectl delete pod/httpenv-6fdc8554fb-f568c
kubectl scale deployment/httpenv --replicas 5
# create ClusterIP
kubectl expose deployment/httpenv --port 8888

kubectl get nodes -o wide # find node ip address
kubectl get pods -o wide # pods are running on nodes, always replaced
kubectl get pods -w # watch
kubectl get pod my-pod -o jsonpath="{.spec.containers[0].ports[0].containerPort}"
kubectl get services
kubectl get svc hello
kubectl get events
kubectl get deployments
kubectl get all

kubectl describe service my-load-balancer
kubectl describe pods

kubectl config view

kubectl logs -l app=rails-app -f # print logs from rails-app pods
kubectl logs my-pod -p # log for specific pod of previous crashed CrashLoopBackOff instance

kubectl apply -f config/kube/terminal.yml

# live edit kubernetes
kubectl edit service/my-service
kubectl edit replicaset/app-nginx-deployment-56845f96cc

kubectl exec terminal -- date # run command in first container inside pod
kubectl exec terminal -it -- bash # run bash inside first container
kubectl exec terminal -c ruby-container -it bash # run ssh inside specific con
kubectl exec terminal -it -- bundle exec rails console

# to trigger refresh deploy you can simply remove
kubectl delete pods -l app=rails-app

kubectl create -f config/kube/migrate.yml
kubectl wait --for=condition=complete --timeout=600s job/migrate
kubectl delete job migrate

export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=application-1,app.kubernetes.io/instance=chart-1" -o jsonpath="{.items[0].metadata.name}")
export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
echo "Visit http://127.0.0.1:8080 to use your application"
# $POD_NAME is chart-1-application-1-6bcd5f68bd-kjmqr $CONTAINER_PORT is 80
kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```

To list all kinds that we can create
```
kubectl api-resources
NAME        SHORTNAMES   APIVERSION NAMESPACED   KIND
bindings                 v1         true         Binding
componentstatuses cs     v1         false        ComponentStatus
nodes       no           v1         false        Node
deployments deploy       apps/v1    true         Deployment
```

To list all versions that are supported
```
kubectl api-versions
```
Find docs for specific API object
```
kubectl explain services
kubectl explain services --recursive
kubectl explain services.spec.type
# note that this is a documentation of the client (not from the server, so maybe
server does not support that version)
```
Full references is on https://kubernetes.io/docs/reference/

To see what generated configuration is used from cli arguments
```
kubectl create deployment sample --image nginx --dry-run -o yaml > deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: sample
  name: sample
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample
      ...

kubectl expose deployment/test --port 80 --dry-run -o yaml
```

You can see changes on server that differs from yaml
```
kubectl diff -f app.yml
```

Declarative is more `what` needs to be created, but impperative is more `how`.
Declerative is using apply command and we do not care how it will be performed
to the desired state.

# Docs

Learn Kubernetes with videos
https://www.youtube.com/hashtag/kubernetesessentialsfromgooglecloud
and documentation for example Specs for pod
https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#PodSpec

Control plane is a set of containers that manage the cluster, includes: API
server, scheduler, controller manager, etcd, CoreDNS
On nodes we have: kubelet (agent running on nodes) and kube-proxy.

## Yaml

Four root keys are: `apiVersion`, `kind`, `metadata` and `spec`.
Inside `metadata` you can use `labels` (for example: `app: api`, `env: prod`)
and for large objects use `annotations`

## Pods

Pod is one or more containers running together on one Node. Pod is basic unit
of deployment. Containers are always in pods.
Controller is used for creating and updating pods, many types of controlles:
Deployment, ReplicaSet, DaemonSet, Job

Pod can be in Pending Running Succeded Failed Unknown phase
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase
Container state (old has Pending state - insufficient resource)
* Waiting - Pod can't run (there are enough resource but maybe image can not be
  downloaded yet)
* Running - you can debug using exec or logs
* Terminated - either ran to completion or failed for some reason

Use probe to check container status
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes

Example
```
# pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.17.3
    ports:
    - containerPort: 80
```
but that does not create replica set so better is to create Deployment

```
# config/kube/deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-rails-example-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rails-app
  template:
    metadata:
      labels:
        app: rails-app
    spec:
      containers:
      - name: rails-app
        image: duleorlovic/kubernetes-rails-example:latest
        ports:
        - containerPort: 3000
```

## Services

Services route traffic to pods matching `app: MyApp` and redirects traffic from
80 to 9376 only on internal cluster ip address.
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
https://www.youtube.com/watch?v=uGm_A9qRCsk
https://kubernetes.io/docs/concepts/services-networking/service/
https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0
Service types:
`ClusterIP` (default) is reachable only inside cluster.
`NodePort` is opening a specific (high) port on a specific machine
`LoadBalancer` is used for using cloud-provided load balancer to route traffic
between services pods but since you pay for every exposed service it is cheaper
to use `Ingress` which can run on one port and route based on domain or path
`ExternalName` adds CNAME DNS record to CoreDNS

```
# create ClusterIP
kubectl expose deployment/httpenv --port 8888
# create terminal
kubectl run tmp-shell --rm -it --image bretfisher/netshoot -- bash
curl httpenv:8888
# find ip from kubectl get services
curl 10.152.183.129:8888

# create NodePort
kubectl expose deployment/httpenv --port 8888 --name httpenv-np --type NodePort
kubectl get services # this returns ports 8888:31471
# from host
curl localhost:31471
curl 10.152.183.129:8888
curl <hostname>.<namespace>.svc.cluster.local
```

3rd party proxy: Traefik, HAProxy, Nginx

## Namespaces

Namespaces is used to filter group of object in cluster
https://www.youtube.com/watch?v=plB3kyZLHe8
Debug services and dns with exec
https://www.youtube.com/watch?v=CSKRy7Ldqis

Namespace Authentication and cluster is defined inside `~/.kube/config`
You can see all clusters with
```
kubectl config get-contexts
CURRENT   NAME       CLUSTER            AUTHINFO   NAMESPACE
*         microk8s   microk8s-cluster   admin
```

## ConfigMap

```
kubectl create configmap nginx-frontend.conf --from-file nginx/frontend.conf
```

You can create using yml
```
# config/kube/env.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: env
data:
  RAILS_ENV: production
  RAILS_LOG_TO_STDOUT: enabled
  RAILS_SERVE_STATIC_FILES: enabled
  DATABASE_URL: postgresql://example.com/mydb
  REDIS_URL: redis://redis.default.svc.cluster.local:6379/
```
and use in your container
```
envFrom:
- configMapRef:
    name: env
```

You can create from text
```
kubectl create secret generic rails-secrets --from-literal=rails_master_key='example'
```
and use in container
```
env:
  - name: RAILS_MASTER_KEY
    valueFrom:
      secretKeyRef:
        name: rails-secrets
        key: rails_master_key
```

## Secrets

```
kubectl create secret generic tls-certs --from-file tls/
```

## StatefulSets

Use db-as-a-service whenever you can.
`Volumes` tied to lifecycle of a Pod, all containers in a single Pod can share
them.
`PersistentVolumes` created at cluster level, outlives a Pod, multiple Pods can
share them.

# Templating YAML

https://kustomize.io/ is template free, using overrides

`docker app` and compose-on-kubernetes https://kompose.io/

## Helm

It is similar to liquid shopify template language.
```
sudo snap install helm --classic
```
```
microk8s.enable helm3
```

Example

```
heml install chart-1 .
heml list
helm uninstall chart-1
```

You can see generated template or apply them
```
helm template .
helm template . | kubectl apply -f -
```

https://helm.sh/docs/chart_template_guide/getting_started/
Template

It is using sprig library https://helm.sh/docs/howto/charts_tips_and_tricks/
* `{{/*` and `*/}}` comment
* `{{- define "labels" -}}` and use with `{{ include "application-1" . |
  nindent 4 }}`
* `{{- if eq .Values.container1.enabled true -}}` or short `{{- if
  .Values.container1.enabled -}}` flow control
  https://helm.sh/docs/chart_template_guide/control_structures/

Chart repository
https://helm.sh/docs/topics/chart_repository/
you can run your own chartmuseum https://chartmuseum.com/docs/#helm-chart
For grafana we use
```
helm repo add grafana https://grafana.github.io/helm-charts
helm search repo grafana
helm install my-grafana grafana/grafana

# if you want to customize
helm pull grafana/grafana
tar -xf grafana-6.20.3.tgz

# remove chart
helm uninstall my-grafana
```

# Dashboard

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml
kubectl proxy
Starting to serve on 127.0.0.1:8001
```

# Context

https://blog.mikesir87.io/2019/08/using-ssh-connections-in-docker-contexts/
You can access docker using ssh (no need to expose docker port, only ssh with
keys authentication is enough)
```
# create
docker context create ssh-box --docker "host=ssh://user@my-box"


# Set the context for a single command
docker --context=ssh-box ps

# OR set the context globally
docker context use ssh-box
docker ps

# OR use the DOCKER_CONTEXT env var
DOCKER_CONTEXT=ssh-box docker ps
```

# Security

https://docs.docker.com/engine/security/
https://github.com/BretFisher/ama/issues/17
* namespace: access to own cpu, ram... (by default they can use all, so you need
  to limit) and view only its own filesystem (not from other containers)
* use non root user `USER node` and `COPY --chown=node`

# Ruby

https://kubernetes.io/docs/reference/using-api/client-libraries/#community-maintained-client-libraries


# Rails

Deploy rails to kubernetes https://kubernetes-rails.com/
```
# Dockerfile
FROM ruby:2.6.3

# update yarn repo
RUN curl https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list
# update node
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -
RUN apt-get update && apt-get install -y nodejs yarn postgresql-client

RUN mkdir /app
WORKDIR /app

COPY Gemfile Gemfile.lock ./
RUN gem install bundler
RUN bundle install
COPY . ./

RUN bundle exec rake assets:precompile

EXPOSE 3000
CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]
```
Create, test and push image to duleorlovic/kubernetes-rails-example:latest
```
docker build -t duleorlovic/kubernetes-rails-example .
docker run -p 3000:3000 duleorlovic/kubernetes-rails-example
docker push duleorlovic/kubernetes-rails-example
```

Enable kubernetes to download docker from private repo
```
kubectl create secret docker-registry my-docker-secret --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL
kubectl edit serviceaccounts default
```

TODO
http://blog.jedrychowski.org/2018/rails-kubernetes-basic-deployment/
rancher
https://www.youtube.com/watch?v=LK6KbAlQRIg
kubernetes
https://www.youtube.com/watch?v=X48VuDVv0do

https://evilmartians.com/chronicles/kubing-rails-stressless-kubernetes-deployments-with-kuby

# Digital ocean

Digital ocean install
```
sudo snap install doctl
doctl auth init
doctl kubernetes cluster kubeconfig save 901f6f80-2728-4ac7-bc5e-77365d839e39
```
Check if we can access
```
export KUBECONFIG=~/.kube/kubernetes-rails-example-kubeconfig.yaml
kubectl version
kubectl get nodes
```
Setup Kubernetes using Vagrant Virtuarbox
https://medium.com/swlh/setup-own-kubernetes-cluster-via-virtualbox-99a82605bfcc
https://github.com/mbaykara/k8s-cluster


# Vim

Using coc-yaml plugin for coc https://github.com/neoclide/coc-yaml so we have
code completion and documentation similar to VSCode
https://octetz.com/docs/2020/2020-01-06-vim-k8s-yaml-support/
https://www.youtube.com/watch?v=eSAzGx34gUE

Install coc https://github.com/neoclide/coc.nvim/wiki/Install-coc.nvim using
native package manager vim8
```
mkdir -p ~/.vim/pack/coc/start
cd ~/.vim/pack/coc/start
git clone --branch release https://github.com/neoclide/coc.nvim.git --depth=1
```
inside vim run
```
:CocInfo
:CocInstall coc-yaml
```

It is installing https://github.com/redhat-developer/vscode-yaml so run
`:CocConfig` to edit `~/.vim/coc-settings.json`
```
{
  "yaml.schemas": {
      "kubernetes": ["/*.yaml"]
  }
}
```
Copy paste keys https://github.com/neoclide/coc.nvim#example-vim-configuration

Control-space or tab is used to code complete. Use `<c-p>` and `<c-n>` to go to
next or prev item, or just type letters.

# Sidecar pattern

Book Designed Distributed Systems” by Brendan Burns
https://azure.microsoft.com/en-us/resources/designing-distributed-systems/ 
