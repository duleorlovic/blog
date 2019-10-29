---
layout: post
---

# Other examples

* <https://jenkins.io/> https://github.com/jenkinsci/jenkins ci in java
* others <https://news.ycombinator.com/item?id=12703121>

# Flynn

<https://flynn.io/>

Install

~~~
L=/usr/local/bin/flynn && curl -sSL -A "`uname -sp`" https://dl.flynn.io/cli | zcat >$L && chmod +x $L
~~~

# Fastline Ci

<https://github.com/fastlane/ci> Open source, self hosted, mobile optimized CI
powered by fastlane (opensource tools for building mobile apps). I like it
because it is ruby.
It requires Fastfile


# Gocd

https://www.gocd.org/

After downloading client and server dev file, double click and install, you can
start with https://docs.gocd.org/current/installation/install/server/linux.html#debian-based-distributions-ie-ubuntu

~~~
sudo /etc/init.d/go-server start
# Started Go Server on http://main:8153/go

sudo /etc/init.d/go-agent start
# http://localhost:8153/go
~~~

Quite complicated...

# Pronto

You can create another github user bot (and invite him to collaborate on
your project) and use [pronto](https://github.com/mmozuras/pronto) to post
comments on commit, pull request or status

~~~
sed -i Gemfile -e '/group :development do/a  \
  gem "pronto", require: false\
  gem "pronto-rubocop", require: false\
  gem "pronto-brakeman", require: false\
  gem "pronto-eslint", require: false\
  gem "pronto-jshint", require: false\
  gem "pronto-poper", require: false\
  gem "pronto-rails_best_practices", require: false\
  gem "pronto-reek", require: false\
  gem "pronto-scss", require: false\
  gem "pronto-flay", require: false\
'
~~~

Generate [Personal Access
Token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)
for the bot user and export in your env file so pronto command can post comments

~~~
pronto run -f github
pronto run -f github_status
pronto run -f github_pr
~~~

You need to set up target commit (default is master) to which it needs to
compare current HEAD. It compare only changes that occurs between those two
(changes on master are ignored).

https://github.com/kulakajak/pronto/commit/69cf86b6ca031c7a62bfa4b24db986d81b95f12c
trkbot

<https://christoph.luppri.ch/articles/2017/03/05/how-to-automatically-review-your-prs-for-style-violations-with-pronto-and-rubocop>
to find last pull request id

# Github actions


https://boringrails.com/articles/building-a-rails-ci-pipeline-with-github-actions/


# Gitlab

https://about.gitlab.com/webcast/mastering-ci-cd/
