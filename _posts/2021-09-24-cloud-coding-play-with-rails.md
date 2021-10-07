---
layout: post
---


TOOLS:
Katacode contains editor, terminal, preview
https://katacoda.com/scenario-examples/scenarios/create-scenario-101

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
```

TODO: Make examples for
https://thoughtbot.com/blog/hotwire-typeahead-searching
