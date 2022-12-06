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

# you should `run` instead of `generate` or `rails_command`
# generate(:scaffold, "person name:string")
# rails_command("db:migrate")
run(<<BASH) or exit 1
set -e # Any commands which fail will cause the shell script to exit immediately
set -x # Show command being executed
rails g scaffold person name:string
rails db:migrate
BASH

# add route entry
route "root to: 'people#index'"

# adding a gem
# gem 'nokogiri'
run <<BASH
bundle add nokogiri
BASH

after_bundle do
  # user run instead of git commands
  # git :init
  # git add: "."
  # git commit: %Q{ -m 'Initial commit' }
  run "git add . && git commit -am'Initial commit'"
end

# environment config
environment 'config.action_mailer.default_url_options = {host: "http://yourwebsite.example.com"}', env: 'production'

# append to a file
run 'echo import \"bootstrap\" >> app/javascript/packs/application.js'

# for updating file with sed you need double back slash \\
run <<BASH
sed -i "" -e '/mailer_sender/c\\
  config.mailer_sender = Rails.application.credentials.mailer_sender
' config/initializers/devise.rb
BASH

# instead of run you can use gsub_file
# https://github.com/rails/thor/blob/master/lib/thor/actions/file_manipulation.rb#L265
gsub_file 'app/views/layouts/application.html.erb', /.*yield.*/, <<-ERB
    <div class='container'>
      <%= yield %>
    </div>
ERB

# note that thor actions are indempotent so you can rerun template
# insert into layout template file using after: "Base\n", or before: "  end"
# note that you should match beggining of the line otherwise it will insert in
# the middle of the line (or end of line like "Base\n" when using after).
# https://github.com/rails/thor/blob/master/lib/thor/actions/inject_into_file.rb
insert_into_file 'app/views/layouts/application.html.erb', <<-ERB, before: '  </head>'
    <%= javascript_pack_tag 'application', 'data-turbo-track': 'reload' %>
ERB

# https://www.rubydoc.info/gems/thor/Thor/Actions
create_file "test/controllers/my_controller_test.rb", <<-RUBY
RUBY

# create file, add force: true to overwrite existing content
file 'app/components/foo.rb', <<~RUBY
  class Foo
  end
~RUBY


# create folder
empty_directory "test/a/"

# overwrite

# creating credentials
# https://stackoverflow.com/questions/56118991/append-secrets-to-credentials-yml-enc-programmatically
EDITOR='echo "foo: bar" >> ' rails credentials:edit
```

Example of templates
https://github.com/duleorlovic/trk-rails-forms
