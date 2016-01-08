---
layout: post
title: Administrate gem
---

# Install

# Generate

~~~
rails g administrate:dashboard Candidate
rails g administrate:views:index Candidate
~~~

# Adding custom action

~~~
# config/routes.rb
  namespace :admin do
    DashboardManifest::DASHBOARDS.each do |dashboard_resource|
      resources dashboard_resource
    end

    root controller: DashboardManifest::ROOT_DASHBOARD, action: :index
    resources :candidates do
      member do
        patch :mark_as_registered
        get :register
      end
    end
  end
~~~

In views you can use `[:mark_as_registered, Administrate::NAMESPACE, resource]`
path or `mark_as_registered_admin_candidate_path`.



