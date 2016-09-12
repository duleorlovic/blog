---
layout: post
title: Capistrano
---

# Install

~~~
gem install capistrano
~~~

Capistrano is not server provisioning tool. You need manually to boot up server
and install ssh, apache, git... usually all tasks that requires sudo should be
done manually. Capistrano use single non priviliged user in non interactive ssh
session.
