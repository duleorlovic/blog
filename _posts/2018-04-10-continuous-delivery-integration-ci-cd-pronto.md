---
layout: post
---

# Continuous Delivery

https://www.gocd.org/

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

<https://christoph.luppri.ch/articles/2017/03/05/how-to-automatically-review-your-prs-for-style-violations-with-pronto-and-rubocop/?utm_source=rubyweekly&utm_medium=email>
to find last pull request id
