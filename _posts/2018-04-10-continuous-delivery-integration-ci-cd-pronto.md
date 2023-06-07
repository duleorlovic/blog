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


docs
https://help.github.com/en/github/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#in-this-article
forum https://github.community/t5/GitHub-Actions/bd-p/actions
ruby, mysql is included https://help.github.com/en/actions/automating-your-workflow-with-github-actions/software-installed-on-github-hosted-runners#ubuntu-1804-lts with root/root username/password
example
https://boringrails.com/articles/building-a-rails-ci-pipeline-with-github-actions/

```
# .github/workflows/ruby.yml
name: Ruby

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest
    env:
      RAILS_ENV: test
      # those are used to connect from Rails
      PGHOST: localhost
      PGDATABASE: rails_db_test
      PGUSER: rails_db_user
      PGPASSWORD: postgres

    services:
      postgres:
        image: postgres:11.8
        env:
          POSTGRES_DB: rails_db_test
          POSTGRES_USER: rails_db_user
          POSTGRES_PASSWORD: postgres
        ports: ["5432:5432"]
        # needed because the postgres container does not provide a healthcheck
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5

      mysql:
        image: mysql:5.7
        env:
          MYSQL_ROOT_PASSWORD: password
        ports:
        - 3306
        options: --health-cmd "mysqladmin ping" --health-interval 10s --health-timeout 5s --health-retries 10
    steps:
    - uses: actions/checkout@v1
    - name: Set up Ruby 2.6.3
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: 2.6.3
    - name: Set up Node
      uses: actions/setup-node@v2
      with:
        node-version: 14
    - name: Set up Chrome
      run: |
        wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
        sudo apt-get update && sudo apt-get install -y xvfb google-chrome-stable
    - name: Install dependecies
      run: |
        sudo apt-get -yqq install libpq-dev
        gem install bundler
        bundle install --jobs 4 --retry 3
        yarn install
        # not sure if we need this for system tests
        # bundle exec rake webdrivers:chromedriver:update # gem 'webdrivers'
    - name: Setup test database
      run: |
        bin/rails db:create
        bin/rails db:migrate
    - name: Run tests
      run: |
        bundle exec rake
        # minitest
        bundle exec rails test
        bundle exec rails test:system
    - name: Heroku staging
      env:
        # generate HEROKU_API_KEY with heroku auth:token
        HEROKU_API_KEY: ${{ secrets.HEROKU_API_KEY }}
        HEROKU_APP_NAME: "move-index"
      if: github.ref == 'refs/heads/master' && job.status == 'success'
      run: git push https://heroku:$HEROKU_API_TOKEN@git.heroku.com/$HEROKU_APP_NAME.git origin/master:master
```
Another way to deploy to heroku is using https://github.com/AkhileshNS/heroku-deploy
Note to use single quote instead of double quoted strings
```
  Deploy:
    if: ${ { github.event_name == 'push' && github.ref == 'refs/heads/master' }}
    needs: Test
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@master
      - name: Deploy to heroku
      - uses: akhileshns/heroku-deploy@v3.6.8 # This is the action
        with:
          heroku_api_key: ${{secrets.HEROKU_API_KEY}}
          heroku_app_name: "YOUR APP's NAME" #Must be unique in Heroku
          heroku_email: "YOUR EMAIL"
```

With `postgres: image: postgres:11` I got an error, so I switch to old 10.8
```
##[error]Failed to initialize, postgres service is unhealthy.
```
Secrets store
```
  env:
    MY_TOKEN: ${{ secrets.MY_TOKEN }}
```

Each job runs in fresh instance of the virtual environment specified by
`runs-on`.
`env` can be defined globaly, for each job, or step.
`jobs.<job_id>.steps.if` is used to filter which step is run
```
steps:
  - name: examples
    if: github.event_name == 'pull_request' && github.event.action == 'unassigned'
    if: github.ref == 'refs/heads/master' && job.status == 'success'
    # previous step failed
    if: failure()
```
`uses` is external steps, sometime needs parameter `with`. Format is
`{owner}/{repo}@{ref}` like
Those setup ruby can pick one of the versions https://help.github.com/en/actions/automating-your-workflow-with-github-actions/software-installed-on-github-hosted-runners
```
  uses: actions/heroku@master
  uses: actions/setup-node@v1
```

Github actions capistrano deploy is simple run `cap production deploy` but you
need to enable ssh keys. It is enabling using
https://github.com/marketplace/actions/webfactory-ssh-agent
* generate specific keys for the project, make sure you can access with deploy
  user using this key
```
# generate new key pair on remove server
ssh-keygen -t rsa -b 4096 -m pem -f my_app_ci
# copy my_app_ci.pub to .ssh/authorized_keys on $PRODUCTION_IP, this will
# success since ssh will offer your other keys ~/.ssh/id_rsa
ssh-copy-id -i my_app_ci deploy@$PRODUCTION_IP
# test if you can connect using only my_app_ci
ssh -o 'IdentitiesOnly=yes' -i my_app_ci deploy@$PRODUCTION_IP
```
  store private key `cat my_app_ci` as project secret `CI_SSH_PRIVATE_KEY`
* on production generate `ssh-keygen` and add public key to github ssh keys
  so that it can download source from github

Github actions do not support [ci skip] in commit message.
Workaround which marks as failed (send email message also)
https://github.com/srz-zumix/ci-skip/blob/master/docs/github/WORKAROUND.md

For other checks you can skip with two lines and skip check
https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-status-checks#skipping-and-requesting-checks-for-individual-commits
```
git commit -am'Try


skip-check: true'
```

Use github actions locally on docker using https://github.com/nektos/act
https://simplabs.com/blog/2021/03/15/trying-your-github-actions-locally/
```
# install act
curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash
```
You can configure act https://github.com/nektos/act#configuration using
`~/.actrc` or local `.actrc` file.
For example if you want to use another base image
```
act -P ubuntu-latest=ghcr.io/catthehacker/ubuntu:full-latest
```

For permission denied errors or bundle install
```
[Test/build] ⭐  Run Install dependecies
[Test/build]   🐳  docker exec cmd=[bash --noprofile --norc -e -o pipefail /home/orlovic/rails/gofordesi-webapp/workflow/4] user=
| Fetching bundler-2.3.8.gem
| Successfully installed bundler-2.3.8
| 1 gem installed
| There was an error while trying to write to
| `/home/orlovic/rails/gofordesi-webapp/.bundle/config`. It is likely that you
| need to grant write permissions for that path.
[Test/build]   ❌  Failure - Install dependecies
Error: exit with `FAILURE`: 23
```

solution is to add step before bundle install
```
sudo chown -R $(whoami):$(whoami) .
```

Using `services` is not yet supported in act
https://github.com/nektos/act/issues/173 so you need to run services locally
like postgres redis.

There is an error when I use credentials, it seems that for test env no
credentials are loaded even I commited test key (github ci works fine).


Use artifact to debug logs and screenshots

```
# .github/workflows/testyml
        bundle exec rails test:system || echo continue
    - name: Upload Artifact
      uses: actions/upload-artifact@v2
      with:
        name: my-artifact
        # https://github.com/actions/upload-artifact#upload-using-multiple-paths-and-exclusions
        path: |
          tmp/screenshots/*
          log/*
```

### Self hosted runners

Add self-hosted runner on Settings > Actions > Runners > New self-hosted runner
https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners#adding-a-self-hosted-runner-to-a-repository

Download and extract runner source
```
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.304.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.304.0/actions-runner-linux-x64-2.304.0.tar.gz
tar xzf ./actions-runner-linux-x64-2.304.0.tar.gz
```

Configure using MY-PROJECT-TOKEN
```
./config.sh --url https://github.com/trkin/kindergarten-exchange --token MY-PROJECT-TOKEN
# this creates .runner .credentials
```

Remove configuration
```
./config.sh remove --token MY-PROJECT-TOKEN
```

Create selfhosted container based on docker
https://dev.to/pwd9000/create-a-docker-based-self-hosted-github-runner-linux-container-48dh

## Cache

https://help.github.com/en/actions/automating-your-workflow-with-github-actions/caching-dependencies-to-speed-up-workflows

https://github.com/actions/cache/blob/master/examples.md#node---yarn
Cache folder needs to be inside working directory.

```
    steps:
      - uses: actions/checkout@v1
      - name: Set up Ruby 2.6.3
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: 2.6.x
      - uses: actions/cache@v1
        with:
          path: vendor/bundle
          key: ${{ runner.os }}-gem-${ { hashFiles('**/Gemfile.lock') }}
          restore-keys: |
            ${{ runner.os }}-gem-
      - name: Set up Node
        uses: actions/setup-node@v1
        with:
          node-version: 10.13.0
      - name: Get yarn cache
        id: yarn-cache
        run: echo "::set-output name=dir::$(yarn cache dir)"
      - uses: actions/cache@v1
        with:
          path: ${{ steps.yarn-cache.outputs.dir }}
          key: ${{ runner.os }}-yarn-${ { hashFiles('**/yarn.lock') }}
          restore-keys: |
            ${{ runner.os }}-yarn-
      - name: Install dependencies
        run: |
          sudo apt update
          sudo apt -yqq install libpq-dev
          gem install bundler
          bundle config path vendor/bundle
          bundle install --jobs 4 --retry 3
          yarn install
```

# Gitlab

https://docs.gitlab.com/ce/ci/yaml/README.html
https://www.youtube.com/watch?v=GCYLqbxm70A
https://www.digitalocean.com/community/tutorials/how-to-set-up-continuous-integration-pipelines-with-gitlab-ci-on-ubuntu-16-04
https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-gitlab-on-ubuntu-16-04

Installing

```
cd /tmp
curl -LO https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh
less script.deb.sh
sudo apt-get install gitlab-ee
sudo apt-get install --reinstall gitlab-ee
# sudo EXTERNAL_URL="https://gitlab.example.com" apt-get install gitlab-ee
```

Removing (uninstalling)
```
sudo gitlab-ctl uninstall
sudo gitlab-ctl cleanse
sudo gitlab-ctl remove-accounts
sudo dpkg -P gitlab-ce
sudo rm -rf /opt/gitlab /var/opt/gitlab /etc/gitlab /var/log/gitlab
```

Configure
```
sudo vi /etc/gitlab/gitlab.rb
# uncomment two lines
sudo apt-get install gitlab-ee
sudo apt-get install gitlab-ee
sudo apt-get install gitlab-ee

sudo gitlab-ctl reconfigure
```

See log
```
sudo gitlab-ctl tail
sudo gitlab-ctl tail unicorn/
```
For error
```
E, [2019-10-30T11:21:11.986549 #2433] ERROR -- : retrying in 0.5 seconds (0 tries left)
E, [2019-10-30T11:21:12.487244 #2433] ERROR -- : adding listener failed addr=127.0.0.1:8080 (in use)

==> /var/log/gitlab/unicorn/unicorn_stdout.log <==
bundler: failed to load command: unicorn (/opt/gitlab/embedded/bin/unicorn)

==> /var/log/gitlab/unicorn/unicorn_stderr.log <==
Errno::EADDRINUSE: Address already in use - bind(2) for 127.0.0.1:8080
  /opt/gitlab/embedded/lib/ruby/gems/2.6.0/gems/unicorn-5.4.1/lib/unicorn/socket_helper.rb:164:in `bind'
```

You should kill services that runs on 8080

```
sudo netstat -tupln | grep 8080
```

Default admin username is `root`

Install runner https://docs.gitlab.com/runner/install/
```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
sudo apt-get install gitlab-runner
```

Find token for the project `asd` in http://localhost/root/asd/-/settings/ci_cd#js-runners-settings
and register runner
```
sudo gitlab-runner register # create new runner
sudo gitlab-runner register --help
sudo gitlab-runner list
```

TODO shell executor https://www.youtube.com/watch?v=sACdT3Rv89E
You can use any cluster https://gitlab.com/help/user/project/clusters/index

To run a job localy you can use
https://docs.gitlab.com/runner/commands/README.html#gitlab-runner-exec
It reads local .gitlab-ci.yml and clone local git repository.
Two arguments to `exec`: executor and job_name. It does not look for stages
since it run only target job (it can see `variables` and `before_script`)
```
gitlab-runner exec docker job_name
```
Btw to run docker with tty session you can use
```
docker run -a stdin -a stdout -i -t ruby:2.4.5 /bin/bash
```

Example yaml file for Ruby is on
https://docs.gitlab.com/ee/ci/examples/test-and-deploy-ruby-application-to-heroku.html
Yaml https://docs.gitlab.com/ee/ci/yaml/
You can validate on https://gitlab.com/duleorlovic/rails_6/-/ci/lint
* `job1:` toplevel elements which contains `script` clause. Jobs are picked up
  by runners and independently performed in parallel.
  ```
  job1:
    stage: test
    script:
      - echo $CI_JOB_STAGE # test
    variables:
      DATABASE_URL: "postgresql://postgres:postgres@postgres:5432/$POSTGRES_DB"
    # https://gitlab.com/help/ci/environments#configuring-manual-deployments
    when: manual
    stage: deploy
    environment: # just a tag so you can group deployments
      name: production
      url: https://rails-6.herokuapp.com
  ```
* `image` is container with preinstalled tools and env variables like
  `$RUBY_VERSION`, examples are: `image: ruby:2.5.3`
* `services` pull another images which is accessible with hostname as image,
  name like `postgres`, it sets env variable like `$POSTGRES_DB`. You can
  configure it when you set global variables: `variables: { PSOTGRES_DB: my,
  POSTGRES_USER: duke, POSTGRES_PASSWORD: "" }`
  https://docs.gitlab.com/ee/ci/services/
  https://docs.gitlab.com/ee/ci/services/postgres.html
  https://gitlab.com/help/ci/docker/using_docker_images.md#passing-environment-variables-to-services
* `stages` you can define order of stages, default `stages: [build test deploy]`
  (it is used instead of `types`)
* `before_script` before every job
* `after_script`
* `variables` those vars are hardcoded and for all jobs. For keys you can use
  project settings ci/cd variables. Here are predefined vars like
  `$CI_COMMIT_SHORT_SHA`
  https://gitlab.com/help/ci/variables/predefined_variables.md to see all
  available vars you can use `script: [export]` (or `env`). You can use in most
  places but not all https://gitlab.com/help/ci/variables/where_variables_can_be_used.md
* `cache` used to define cached folders
  ```
  cache:
    path:
      - node_modules/
      - vendor/bundle
  ```

Environment and deployments https://gitlab.com/help/ci/environments for specific
jobs (like tags).
use `GIT_STRATEGY: none` when you do not need to chechout the code

Error when chrome is not found
  ```
      Webdrivers::BrowserNotFound:
              Failed to find Chrome binary.
  ```
  You can install https://discuss.circleci.com/t/installing-chrome-inside-of-your-docker-container/9067 https://gist.github.com/nicobytes/e92b9b4fb7070a925662a48dd038cf67
  https://stackoverflow.com/questions/56894827/gitlab-ci-config-for-rails-system-tests-with-selenium-and-headless-chrome
  https://medium.com/weareevermore/set-up-gitlab-ci-for-rails-applications-cf0117128bce
  ```
  sudo apt --fix-broken install

  wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -
  set -x && apt-get update && apt-get install -y xvfb google-chrome-stable

  wget -q -O /usr/bin/xvfb-chrome https://bitbucket.org/atlassian/docker-node-chrome-firefox/raw/ff180e2f16ea8639d4ca4a3abb0017ee23c2836c/scripts/xvfb-chrome
  ln -sf /usr/bin/xvfb-chrome /usr/bin/google-chrome
  chmod 755 /usr/bin/google-chrome
  ```

Errors when chrome can not be run
```
Selenium::WebDriver::Error::UnknownError: unknown error: Chrome failed to start: exited abnormally
  (unknown error: DevToolsActivePort file doesn't exist)
```
you need to use options `--headless --no-sandbox --disable-gpu` or in rails just
switch to `:headless_chrome`
```
# test/application_system_test_case.rb
require 'test_helper'
Capybara.register_driver :selenium_chrome_headless_docker_friendly do |app|
  Capybara::Selenium::Driver.load_selenium
  browser_options = ::Selenium::WebDriver::Chrome::Options.new
  browser_options.args << '--headless'
  browser_options.args << '--disable-gpu'
  # Sandbox cannot be used inside unprivileged Docker container
  browser_options.args << '--no-sandbox'
  Capybara::Selenium::Driver.new(app, browser: :chrome, options: browser_options)
end

Capybara.javascript_driver = :selenium_chrome_headless_docker_friendly
class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, screen_size: [1400, 1400]
  setup do
    Capybara.current_driver = Capybara.javascript_driver # :selenium by default
  end
end
```

Example https://medium.com/weareevermore/set-up-gitlab-ci-for-rails-applications-cf0117128bce


```
# .gitlab-ci.yml

# https://hub.docker.com/r/library/ruby/tags/
image: ruby:2.6.3

# Check out: http://docs.gitlab.com/ce/ci/docker/using_docker_images.html#what-is-a-service
services:
  - postgres:latest

variables:
  BUNDLE_PATH: vendor/bundle
  POSTGRES_DB: database_name
  RAILS_ENV: test

cache:
  paths:
    - vendor
    - node_modules

before_script:
  - ruby -v  # Print out ruby version for debugging
  - which ruby

  - apt-get update -q && apt-get install nodejs -yqq
  - node -v

   # Install yarn
  - wget -q -O - https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
  - echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list
  - apt-get update -yq
  - apt-get install -y yarn

  - gem install bundler --no-document
  - bundle install -j $(nproc) --path vendor
  - yarn install


tests:
  variables:
    DATABASE_URL: "postgresql://postgres:postgres@postgres:5432/$POSTGRES_DB"
  script:
    - bundle exec rails assets:precompile
    - bundle exec rails db:migrate
    - bundle exec rubocop
    - bundle exec rails test

# https://gitlab.com/help/ci/environments#complete-example
deploy_review:
  stage: deploy
  script:
    - echo "Deploy a review app"
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: https://$CI_ENVIRONMENT_SLUG.example.com
  only:
    - branches
  except:
    - master

deploy_staging:
  stage: deploy
  environment:
    name: staging
    url: https://rails-6-staging.herokuapp.com
  script:
    - gem install dpl
    - dpl --provider=heroku --app=$HEROKU_STAGING_APP_NAME --api-key=$HEROKU_API_KEY
  only:
    - master

deploy:
  stage: deploy
  environment:
    name: production
    url: https://rails-6.herokuapp.com
  script:
    - gem install dpl
    - dpl --provider=heroku --app=$HEROKU_APP_NAME --api-key=$HEROKU_API_KEY
  only:
    - master
  when: manual
```

Error:
Admin::BotsTest#test_creating_a_Bot:
Selenium::WebDriver::Error::UnknownError: unknown error: Chrome failed to start: exited abnormally
  (unknown error: DevToolsActivePort file doesn't exist)
  (The process started from chrome location /usr/bin/google-chrome is no longer running, so ChromeDriver is assuming that Chrome has crashed.)
    test/system/admin/bots_test.rb:19:in `block in <class:BotsTest>'
