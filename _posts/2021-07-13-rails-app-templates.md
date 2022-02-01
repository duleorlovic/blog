---
layout: post
---

https://guides.rubyonrails.org/rails_application_templates.html

Rails templates used for new applications

```
# for new apps
rails new blog -m http://example.com/template.rb

# for existing apps
rails app:template LOCATION=./template.rb
```

Basic api example

```
# template.rb

# you can use run instead rails generator and commands
generate(:scaffold, "person name:string")
generate(:migration, 'add_company_to_users company:references')

rails_command("db:migrate")

# add route entry
route "root to: 'people#index'"

# adding a gem
gem 'nokogiri'

after_bundle do
  git :init
  git add: "."
  git commit: %Q{ -m 'Initial commit' }
end

# config
environment 'config.action_mailer.default_url_options = {host: "http://yourwebsite.example.com"}', env: 'production'

# file, add force: true to overwrite existing content
file 'app/components/foo.rb', <<~RUBY
  class Foo
  end
~RUBY

# run any command like backtick
run "rm README.rdoc"

# adding npm package
run 'yarn add bootstrap jquery popper.js'

# append to a file
run 'echo import \"bootstrap\" >> app/javascript/packs/application.js'

# gsub file
# https://github.com/rails/thor/blob/master/lib/thor/actions/file_manipulation.rb#L265
gsub_file 'app/views/layouts/application.html.erb', /.*yield.*/, <<-ERB
    <div class='container'>
      <%= yield %>
    </div>
ERB

# insert into layout template file using after: "Base\n", or before: "  end"
# https://github.com/rails/thor/blob/master/lib/thor/actions/inject_into_file.rb
insert_into_file 'app/views/layouts/application.html.erb', <<-ERB, before: '  </head>'
    <%= javascript_pack_tag 'application', 'data-turbo-track': 'reload' %>
ERB
```

Example of templates
https://github.com/duleorlovic/trk-rails-forms
