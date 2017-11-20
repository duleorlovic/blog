---
layout: post
---

# Install

<https://github.com/sharetribe/sharetribe>

If you have newer versions of `node 7.8` or `npm 4.2` than you should install
older `nvm install 7.8` and `npm install -g npm@4.2.0`.
If you dont see forms or other react components, than `rm -rf
client/node_modules/` and `npm install`

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
~~~

Scheduler tasks
Domain

For Heroku use the same steps.



# Examples

* interesting example <https://www.sharegrid.com/>
