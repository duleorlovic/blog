---
layout: post
---

Did you ever wanted to use some configuration options, but you find very
difficult to use secrets.yml or credentials file, tired of using `heroku config`
to often, or even worse, deploying each time you change some constant in your
code ?

Here is a nice solution how to use *Rails.cache* to quickly change some config
without need to restart rails application. Its Rails 3 ready. Some credits go
to
[catarse_settings](https://github.com/catarse/catarse_settings_db/blob/master/app/models/catarse_settings_db/setting.rb).

For Heroku, you need addon
[MemCachier](https://devcenter.heroku.com/articles/memcachier) since FileStore
is not shared between dynos (rails console is separate dyno).
[rack-cache](https://devcenter.heroku.com/articles/rack-cache-memcached-rails31)
is even better. It is not needed if you have activeAdmin page for updating
settings.

To find what is needed, the best way is to look at template and apply it
```
rails app:template LOCATION=https://raw.githubusercontent.com/duleorlovic/rails_settings_in_cache_and_beta_features/main/template.rb
```
Main files are
```
# app/models/my_setting.rb
# app/helpers/beta_helper.rb
```

Add routes
```
# config/routes.rb
  get "admin/settings_and_beta_features"
  post "admin/update_settings_and_beta_features"
```
and controller
```
# app/controllers/admin_controller.rb
```

In all places in my rails application I use this `beta` helper method that uses
two MySetting variable. `live_features` are comma separated list of feature
names that are live, and `beta_user_emails` are emails for users that can see
non live features (of course only when they are logged in).

To keep track of features I usually write them in const.file

```
# config/initializers/const.rb
  def beta_features
    hash_or_error_if_key_does_not_exists(
      landing_feature: "This is my landing feature",
    )
  end
```
To enable them, you can use admin page
```
# app/views/admin/settings_and_beta_features.html.erb
```

# Tests

In your tests simply add  `before { MySetting[:live_features] = 'phone_confirmation_feature' }`

# Administrate

For administrate gem, we need to call `MySetting[]` methods because we need to
update cache

~~~
rails g administrate:dashboard MySetting
# add :my_settings to DASHBOARDS in app/dashboards/dashboard_manifest.rb
# add following code to app/controllers/admin/my_settings_controller.rb
    def update
      super
      MySetting[resource_params[:name]] = resource_params[:value]
    end
    def create
      super
      MySetting[resource_params[:name]] = resource_params[:value]
    end
    def destroy
      MySetting[requested_resource.name] = ""
      super
    end
~~~

# ActiveAdmin

[activeadmin gem](https://github.com/activeadmin/activeadmin) is nice way to
edit stuff. It contains generators, for example `rails g active_admin:resource
Banner`

~~~
# app/admin/my_setting.rb
ActiveAdmin.register MySetting do
  permit_params :name, :value, :description
  controller do
    def update
      super
      MySetting[resource.name] = resource.value
    end
    def create
      super
      MySetting[resource.name] = resource.value
    end
    def destroy
      MySetting[resource.name] = ""
      super
    end
  end
end
~~~
