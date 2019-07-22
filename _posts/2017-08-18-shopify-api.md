---
layout: post
---

# Creating new app

<https://help.shopify.com/api/getting-started> On <https://partners.shopify.com>
create account and development store, for example
<https://duleorlovic-test.myshopify.com/>.
Also create an App (url could be your local ngrok tunnel). Since shopify from
september uses only https, you need to use https ngrok url.
You need to start ngrok and than update in Apps -> App Info -> App
information App URL as <https://159abd37.ngrok.io/> and Whitelisted redirection
URL <https://159abd37.ngrok.io/auth/shopify/callback>
Use ngrok url when installing the app, since if you install from `<localhost>`
then `Oauth error invalid_request: The redirect_uri is not whitelisted` is
raised. This is also raised when you use `http` instead `https`.
Copy credentials from Apps -> App info -> App credentials and store them in
`SHOPIFY_API_KEY` and `SHOPIFY_SECRET_KEY`.

Note that protocoll should be https (admin store is always redirected to https),
because if we use `http://159abd37.ngrok.io` there will be a warning:
`Shopify.AppMessenger received message null from unexpected origin
https://159abd37.ngrok.io ; expecting http://159abd37.ngrok.io or subdomain
thereof`

Install <https://developer-tools.shopifyapps.com/install> to load sample
generated data.


Create new rails and add gem <https://github.com/Shopify/shopify_app>

~~~
rails new shopify_test -d postgresql
cd shopify_test
rake db:create
git init . && git add . && git commit -am "rails new"
cat > config/secrets.yml << HERE_DOC
shared:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] || 'some_secret' %>

  shopify_api_key: <%= ENV["SHOPIFY_API_KEY"] %>
  shopify_secret_key: <%= ENV["SHOPIFY_SECRET_KEY"] %>
HERE_DOC
echo "gem 'shopify_app'" >> Gemfile
bundle

rails generate shopify_app
#    generate  shopify_app:install
#      create  config/initializers/shopify_app.rb
#      create  config/initializers/omniauth.rb
#      insert  config/initializers/omniauth.rb
#      create  app/views/layouts/embedded_app.html.erb
#      create  app/views/layouts/_flash_messages.html.erb
#       route  mount ShopifyApp::Engine, at: '/'
#    generate  shopify_app:shop_model
#      create  app/models/shop.rb
#      create  db/migrate/20171103082230_create_shops.rb
#        gsub  config/initializers/shopify_app.rb
#      create  test/fixtures/shops.yml
#    generate  shopify_app:home_controller
rake db:migrate
sed -i config/initializers/shopify_app.rb -e '/<api_key>/c \
  config.api_key = Rails.application.secrets.shopify_api_key'
sed -i config/initializers/shopify_app.rb -e '/<secret>/c \
  config.secret = Rails.application.secrets.shopify_secret_key'
git add . && git commit -m "rails g shopify_app"
~~~

When you navigate to <https://159abd37.ngrok.io> for the first time you will be
asked for shop url, so enter <https://duleorlovic-test.myshopify.com/> or just a
name `duleorlovic-test` to install our app. Than enable Apps -> App info ->
Extensions -> "Embed in Shopify admin".

You can use `ngrok http 3003` while you are developing.
Also <https://requestb.in/> and <https://forwardhq.com/>

Only legacy app uses `embeded: false`. If you do not want to redirect to shopify
admin and be under iframe, you can disable redirect
<https://github.com/Shopify/shopify_app#testing-an-embedded-app-outside-the-shopify-admin>

~~~
// app/views/layouts/embedded_app.html.erb
        forceRedirect: <%= Rails.env.development? || Rails.env.test? ? 'false' : 'true' %>
~~~

Note that `app/views/layouts/embedded_app.html.erb` layouts always used (and
not application layout).

# Heroku

When you deploy to heroku you need to update Redirection URL or create another
app

~~~
heroku config:set SHOPIFY_API_KEY=$SHOPIFY_API_KEY SHOPIFY_SECRET_KEY=$SHOPIFY_SECRET_KEY SERVER_URL=https://`heroku domains | sed -n 2p`
~~~

Remember to reinstall application after the assets compilation is finished.
Also you can force oauth simply going on `/login` page, but this helps only with
webhooks. Script tags need reinstall. NOTE THAT USUALLY NEED TO RESTART RAILS
SERVER BEFORE THAT SO IT PICKS UP NEW DIGEST.

Use force ssl, so it is always using https (redirect url will be the same
protocol from which we started auth).

If there is a message for `/auth/failure` with `"message":"invalid_signature"`
than probably you already have existing shop with different API keys. So remove
any shops from database when you change API keys. Also might be that your API
SECRET is invalid.

If there is a message for your page
`duleorlovic-test.myshopify.com/admin/apps/12342134` with message `Not Found:
The page you were looking for does not exist.`... I do not know.


# OAuth

Client is application that would like access to shops data.
First step is to redirect resource owner to authorization server and ask for
scopes (write_orders,read_customers), define redirect_uri and use option for
offline (default) or online access mode (current session).
When resource owner click Install they will be redirected to redirect_url with
authorization code.

If offline access mode, than authorization code is exchanged with permament
access token using `POST https://{shop}.myshopify.com/admin/oauth/access_token`
with `authorization_code`, `client_id` and `client_secret`.


# Private app

You can generate keys for specific shop, got to you
`{shop_url}/admin/apps/private`. Enable what you need since by default not all
are enabled.

You can use [shopify-cli
console](https://github.com/Shopify/shopify_api#console) to access resources
using pry

~~~
shopify-cli add duleorlovic-test
shopify-cli console
ShopifyAPI::Product.find(:all)
~~~

Also, you can use postman to access requests.

# API

<https://help.shopify.com/api/reference>
When using api you can call `find` on two ways: `:all` or specific id.
When find is used with specific id it will raise exception if it is not found.
~~~
ShopifyAPI::Order.find 123456
ShopifyAPI::Order.find :all
~~~

If find is used with `:all` you can use additional `params` hash with:
* `fields` which fields to return, can be array
* `page` which page 1, 2, ...
* `limit` default is 50 usuall max is 250
* `ids` comma separated ids
* `vendor` or any other field

~~~
@orders = ShopifyAPI::Order.find(:all, params: {fields: "id,name,created_at,email,financial_status,total_price", financial_status: "pending", limit: 250})
~~~

`where` could also work (no need to nest inside `params` hash):

~~~
ShopifyAPI::Product.where(ids: "8931567365")
# NOTE that single 'id' does not work, it returns all
ShopifyAPI::Product.where(id: "8931567365") # THIS DOES NOT WORK
# other fields are ok in singular - original field name
ShopifyAPI::Product.where(vendor: "duleorlovic-test")

# fields array, page and limit also works
ShopifyAPI::Product.where(vendor: "duleorlovic-test", fields: [:id, :title],
limit: 1, page: 2)
~~~

# Webhooks

You can generate config and job for webhook
<https://github.com/Shopify/shopify_app#webhooksmanager>

Note that address needs to be different for each webhook and name is joined
item_action.

~~~
rails g shopify_app:add_webhook -t carts/update -a https://example.com/webhooks/carts_update
~~~

This will generate something but you can add server url config:

~~~
  config.webhooks = [
    {
      topic: 'carts/update',
      address: Rails.application.secrets.server_url.to_S +
        '/webhooks/carts_update',
      format: 'json'
    },
  ]
~~~

Webhook is hash, so you need to access like `webhook[:line_items]`
Webhooks are regenerated by uninstalling and installing again the app. Also
retriggering the oauth flow also update webhooks (so no need to uninstall).
Job is created

~~~
# app/jobs/carts_update_job.rb
class CartsUpdateJob < ActiveJob::Base
  def perform(shop_domain:, webhook:)
    shop = Shop.find_by(shopify_domain: shop_domain)

    shop.with_shopify_session do
      # do something with webhook hash
    end
  end
end
~~~

# Application proxy

You can load productions from external source. Enable it on partners.shopify.com
-> Apps -> Extension -> App proxy
sub path can be anything and proxy url should match exact path on server.
NOTE that this is settings for the app defaults. When user install the app, it
will use current settings, but they can customize subpath. You can change
subpath on the app and that will have no effects. But If you change proxy url
that will broke current applications unless you enable that url on server. So
that proxy url always should be working url.

On controller if you inherit from `ShopifyApp::AuthenticatedController` than you
should skip
~~
  skip_around_action :shopify_session, only: [:index]
  skip_before_action :login_again_if_different_shop, only: [:index]
~~~

Anyway for liquid you should set header and render without layout
~~~
    render layout: false, content_type: 'application/liquid'
~~~

# Polaris and react components

<https://polaris.shopify.com/components/get-started>


# Application admin links

You can register admin links so user can use it from other admin pages
<https://help.shopify.com/api/partner-dashboard/app-admin-access>
Go to Partners -> Apps -> Extensions -> Add an admin link


# Modify online store using ScriptTag

Remote javascript can be loaded on pages of shop's storefront so shop owner do
not need to change theme files directly (and to revert changes when uninstalling
your app).

You can use template from
[use javascript
responsibly](https://help.shopify.com/api/sdks/shopify-apps/modifying-online-store/use-javascript-responsibly)
and save it on `assets/storefront_javascript/loadSliders.js`.
Enable precompiling and rendering without digest

~~~
cat >> Gemfile << HERE_DOC
# compile assets without digest
gem 'non-stupid-digest-assets'
HERE_DOC
bundle

cat >> config/initializers/assets.rb << HERE_DOC
Rails.application.config.assets.precompile += %w[
  loadSliders.js
]
NonStupidDigestAssets.whitelist += %w[
  loadSliders.js
]
HERE_DOC
~~~

`rake assets:precompile` should create `public/assets/loadSliders.js`

Register script tag in `config/initializers/shopify_app.rb` as in example
<https://github.com/Shopify/shopify_app#scripttagsmanager>
~~~
  config.scripttags = [
    { event: 'onload',
      src: "#{Rails.application.secrets.server_url}/assets/loadSliders.js" }
  ]
~~~

You need to restart rails and reinstall shopify app if you want to change src
url.
You can use non-stupid-digest-assets or you can use lambda for src.

# Authorize

To authorize request:
https://help.shopify.com/api/tutorials/application-proxies#proxy-response

~~~
  shop = request.params['shop']
  code = request.params['code']
  hmac = request.params['hmac']
  # perform hmac validation to determine if the request is coming from Shopify
  h = request.params.reject{|k,_| k == 'hmac'}
  query = URI.escape(h.sort.collect{|k,v| "#{k}=#{v}"}.join('&'))
  digest = OpenSSL::HMAC.hexdigest(OpenSSL::Digest.new('sha256'), API_SECRET, query)
  if not (hmac == digest)
    return [403, "Authentication failed. Digest provided was: #{digest}"]
  end
~~~

# Apps

MULTI VENDOR LOKAL SELLER
* multistore forum discussion: https://ecommerce.shopify.com/c/shopify-discussion/t/multiple-domain-under-one-account-140477
* one store multi users/vendors discussion https://ecommerce.shopify.com/c/shopify-discussion/t/user-level-accounts-366021
* supplier offering storefronts to distributors/end_stores. not possible with
one store since price should be different to each distributor.


# Publishing your app

You should attract your merchants to make a review and put stars on you app.
For example: We see you're seeing success with our app we'd love it if you left
us a review [link].
When someone is talking to customer support and have good experience, customer
support says: Hey we'd love itif you leave a review.

# Opensource examples

<https://github.com/julionc/awesome-shopify>
