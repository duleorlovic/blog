---
layout: post
---

# Install

<https://github.com/sharetribe/sharetribe>

~~~
PORT=3001 foreman start -f Procfile.static
bundle exec rake jobs:work
~~~

Open <http://lvh.me:5000/> or updated port <http://lvh.me:3001/>

Localy inspect mails that are send:

~~~
gem install mailcatcher
mailcatcher

# config/config.yml
development:
  mail_delivery_method: smtp
  smtp_email_address: "localhost"
  smtp_email_port: 1025
~~~

Open <http://localhost:1080>

# Development

<https://github.com/sharetribe/sharetribe/tree/master/docs>

# Deploy in production

<https://github.com/sharetribe/sharetribe#setting-up-sharetribe-for-production>

First checkout latest
[release](https://github.com/sharetribe/sharetribe/releases).

Keys for AWS S3 and stmp emails are stored in `config/config.defaults.yml`.
It does not accepts `<%= ENV[""] %>` but you can override with env with the same
**lowercase** names.

~~~
# config/keys/sharetribe.sh
# domain
export domain=my-domain.com

# aws is used from my_keys, but need lowercase
export s3_bucket_name=$AWS_BUCKET_NAME
export s3_region=$AWS_REGION
export s3_upload_bucket_name=$AWS_BUCKET_NAME
export aws_access_key_id=$AWS_ACCESS_KEY_ID
export aws_secret_access_key=$AWS_SECRET_ACCESS_KEY

# mail using my gmail
export mail_delivery_method=smtp
export smtp_email_address=smtp.gmail.com
export smtp_email_port=587
export smtp_email_user_name=$GMAIL_EMAIL
export smtp_email_password=$GMAIL_PASSWORD
export smtp_email_domain=gmail.com

# google api key for maps
export google_maps_key=$GOOGLE_API_KEY

# facebook
export fb_connect_id=$FACEBOOK_KEY
export fb_connect_secret=$FACEBOOK_SECRET

# all in heroku command
heroku config:set domain=my-domain.com s3_bucket_name=$AWS_BUCKET_NAME s3_region=$AWS_REGION s3_upload_bucket_name=$AWS_BUCKET_NAME aws_access_key_id=$AWS_ACCESS_KEY_ID aws_secret_access_key=$AWS_SECRET_ACCESS_KEY mail_delivery_method=smtp smtp_email_address=smtp.gmail.com smtp_email_port=587 smtp_email_user_name=$GMAIL_EMAIL smtp_email_password=$GMAIL_PASSWORD smtp_email_domain=gmail.com google_maps_key=$GOOGLE_API_KEY fb_connect_id=$FACEBOOK_KEY fb_connect_secret=$FACEBOOK_SECRET
~~~

Domain is root domain, and each community is under subdomain.

Sphinx needs access to mysql so download credentials from clearDB dashboard and
save to `lib/certificates` folder. Edit `config/thinking_sphinx.yml` to point to
those files. Key needs to be without password.

~~~
openssl rsa -in cleardb_id-key.pem -out cleardb_id-key-no-password.pem
vi config/thinking_sphinx.yml
production:
  mysql_ssl_ca: lib/certificates/cleardb-ca.pem
  mysql_ssl_cert: lib/certificates/cleardb_id-cert.pem
  mysql_ssl_key: lib/certificates/cleardb_id-key-no-password.pem

heroku run bundle exec flying-sphinx configure
heroku run bundle exec flying-sphinx rebuild
heroku run bundle exec flying-sphinx index
heroku run bundle exec flying-sphinx start
~~~

Scheduler tasks
<https://github.com/sharetribe/sharetribe/blob/master/docs/scheduled_tasks.md>

~~~
heroku addons:create scheduler:standard
heroku addons:open scheduler

# create three tasks, first is hourly other are daily
bundle exec rails runner ActiveSessionsHelper.cleanup
bundle exec rails runner CommunityMailer.deliver_community_updates
bundle exec rake sharetribe:delete_expired_auth_tokens
~~~

For Heroku use the same steps.

~~~
heroku config:set PASSENGER_MAX_POOL_SIZE=3
heroku run bundle exec rake db:structure:load
~~~

Problem with `Please check the output above for any errors and make sure that
`mysql` is installed in your PATH and has proper permissions.` is reported
https://github.com/sharetribe/sharetribe/issues/2981 and you need to install
buildpack <https://stackoverflow.com/a/47392242/287166>

# Adding linkedin

~~~
# linkedin
export linkedin_client_id=$LINKEDIN_CLIENT_ID
export linkedin_client_secret=$LINKEDIN_CLIENT_SECRET

heroku config:set linkedin_client_id=$LINKEDIN_CLIENT_ID linkedin_client_secret=$LINKEDIN_CLIENT_SECRET

# config/initializers/devise.rb
  config.omniauth :linkedin, APP_CONFIG.linkedin_client_id, APP_CONFIG.linkedin_client_secret
~~~

# Contribute

<https://github.com/sharetribe/sharetribe/blob/master/app/models/person.rb#L518>
logger is using Rails logger instead of SharetribeLogger.

# Examples

* interesting example <https://www.sharegrid.com/>
