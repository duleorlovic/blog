---
layout: post
---


Similar for php
https://stackoverflow.com/questions/4616159/is-there-a-php-sandbox-something-like-jsfiddle-is-to-js

TOOLS:
Katacode contains editor, terminal, preview for creating tutorials
https://katacoda.com/scenario-examples/scenarios/create-scenario-101
https://learn.openshift.com/

online ide https://aws.amazon.com/cloud9/
https://docs.aws.amazon.com/cloud9/latest/user-guide/welcome.html

Does not support server side
https://jsfiddle.net/ https://jsbin.com
https://codesandbox.io/  https://www.codecademy.com
Interesting limits for hours, memory, hard, requests https://help.glitch.com/kb/article/17-what-are-the-technical-restrictions-for-glitch-projects/#Uptime

free editor https://theia-ide.org/
It is used by Redhat OpenShift https://developers.redhat.com/developer-sandbox,
example for Rails app https://github.com/sclorg/rails-ex/blob/master/README.md

Code-server uses VS https://coder.com/ (it is opensource alternative to VS Code
Codespaces)
can be installed on various platforms https://github.com/cdr/deploy-code-server
Install on ubuntu
https://github.com/cdr/code-server#getting-started
https://coder.com/docs/code-server/v3.12.0/install#installsh
```
curl -fsSL https://code-server.dev/install.sh | sh
sudo systemctl enable --now code-server@$USER
# Created symlink /etc/systemd/system/default.target.wants/code-server@ubuntu.service → /lib/systemd/system/code-server@.service.

```
Configuration is inside `~/.config/code-server/config.yaml` and data
`.local/share/code-server`
```
vi .config/code-server/config.yaml
# change bind-addr: 0.0.0.0:8080
# password
sudo service code-server@orlovic restart
```


https://www.youtube.com/watch?v=oILc0ywDVTk
Install Rancher and docker
```
# https://rancher.com/docs/rancher/v2.5/en/installation/resources/installing-docker/
curl https://releases.rancher.com/install-docker/19.03.sh | sh

# no need for sudo (you need to log out and log in again
sudo usermod  -aG docker orlovic

# run rancher container
# https://rancher.com/docs/rancher/v2.5/en/installation/other-installation-methods/single-node-docker/

sudo docker run --privileged -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher

# wait some time and that access ip of the virtual machine like https://192.168.0.25/
```

# Trk Samples play with rails

Virtual IT lab

Main mission to help organize your own code idea. For example when I implement
one feature in one project and than same thing on other project, than I improved
it in first project. Next time I want to implement that feature I want te be
able to find it easilly and last version.

Each example is pushed to our gitlab and new docker image is created and stored
to gitlab docker repository.

Based on Play with docker SDK

Here is example of tutorial and output panel
https://github.com/play-with-docker/play-with-docker.github.io
which is server under
https://training.play-with-docker.com
It just uses https://github.com/play-with-docker/sdk like
```
<html>
  <head>
    <title>PWD SDK</title>
  </head>
  <body>
    <div id="myTerm" style="width: 500px; height: 500px;"></div>
    <link
      rel="stylesheet"
      href="https://unpkg.com/pwd-sdk@0.0.12/dist/styles.css"
    />
    <script src="https://unpkg.com/pwd-sdk@0.0.12/dist/pwd.min.js"></script>
    <script>
      pwd = new PWD();
      pwd.newSession([{ selector: "#myTerm" }], { baseUrl: 'http://localhost'});
    </script>
  </body>
</html>
```
and inside docs you can use `.term1` to direct commands to terminal or
`data-term` to open server for current session
https://github.com/play-with-docker/play-with-docker.github.io/blob/master/writing-tutorials.md#auto-populating-code-in-the-terminal


AWS deploy
https://www.n0r1sk.com/post/running-play-with-docker-on-aws/

Similar:
https://cloud.google.com/blog/products/serverless/introducing-cloud-run-button-click-to-deploy-your-git-repos-to-google-cloud

TRK samples

Code can be run on providers like Heroku, AWS, Google cloud.
Or locally with two commands:  `git clone ... && docker-compose up`
Normal visitors can see only the screenshots or static page.
Paid PRO membership can play with the code online using codeserver and preview
using play with docker (or similar).
Everything should be open and visible, registration only when necessary.

Search projects by Topics

TODO: Make examples for
https://thoughtbot.com/blog/hotwire-typeahead-searching
nice ui ux like https://journey.cloud/ https://clockify.me/tracker
