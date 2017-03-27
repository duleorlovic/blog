---
title: Ionic & Rails Direct S3 image upload
layout: post
tags: ionic aws s3 angular
---

In rails you can upload using [carrierwave for uploading]({{ site.baseurl }}
{% post_url 2015-04-05-common-rails-bootstrap-snippets %}
#carrierwave-for-uploading) which works also for direct uploading to AWS.

But [jQuery-File-Upload](https://github.com/blueimp/jQuery-File-Upload)
is much cleaner way of uploading and you can upload multiplefiles at once and
you can limit the size and choose name of uploaded files.
[heroku
tutorial](https://devcenter.heroku.com/articles/direct-to-s3-image-uploads-in-rails)
uses
[PresignedPost](http://docs.aws.amazon.com/sdkforruby/api/Aws/S3/PresignedPost.html)

~~~
cat >> Gemfile << HERE_DOC
# direct S3 uploading
gem 'aws-sdk', '~> 2'
HERE_DOC

sed -i config/secrets.yml -e '/^test:/i \
  # aws s3\
  aws_bucket_name: <%= ENV["AWS_BUCKET_NAME"] %>\
  aws_access_key_id: <%= ENV["AWS_ACCESS_KEY_ID"] %>\
  aws_secret_access_key: <%= ENV["AWS_SECRET_ACCESS_KEY"] %>\
  # region is important for all non us-east-1 regions\
  aws_region: <%= ENV["AWS_REGION"] || "us-east-1" %>\
'

cat >> config/initializers/aws.rb << HERE_DOC
S3_MAX_FILE_SIZE = 10 * 1024 * 1024 # 10 MB

Aws.config.update({
  region: Rails.application.secrets.aws_region,
  credentials: Aws::Credentials.new(
    Rails.application.secrets.aws_access_key_id,
    Rails.application.secrets.aws_secret_access_key
  ),
})

if Rails.application.secrets.aws_bucket_name
  S3_BUCKET = Aws::S3::Resource.new.bucket(
    Rails.application.secrets.aws_bucket_name
  )
else
  class FakeAws
    def presigned_post(args)
      Rails.logger.error "Please provide AWS credentials"
      OpenStruct.new fields: {}, url: "https://bucket-name.s3-ap-southeast-1.amazonaws.com"
    end
  end
  S3_BUCKET = FakeAws.new
end
HERE_DOC
~~~

To get aws keys add this to controller

~~~
# app/constollers/posts_controller.rb
before_action :set_s3_direct_post, only: [:new, :edit, :create, :update]

def set_s3_direct_post
  @s3_direct_post = S3_BUCKET.presigned_post(
    key: "uploads/#{SecureRandom.uuid}/${filename}",
    success_action_status: '201', # Aws will respond with XML
    acl: 'public-read',
    content_length_range: 1..S3_MAX_FILE_SIZE,
  )
end
~~~

This is how we can use plugin

~~~
cat > app/assets/javascripts/direct_upload.coffee.erb << 'HERE_DOC'
@directUpload = ($fileInput, callback) ->
  progressBar  = $("<div class='bar'></div>")
  barContainer = $("<div class='progress'></div>").append(progressBar)
  $fileInput.after(barContainer)

  url = $fileInput.data 'url'
  formData = $fileInput.data 'formData'
  $fileInput.fileupload(
    fileInput: $fileInput
    url: url
    type: 'POST'
    autoUpload: true
    formData: formData
    paramName: 'file'
    dataType: 'XML'
    replaceFileInput: true
    acceptFileTypes: /(\.|\/)(gif|jpe?g|png|pdf)$/i
    maxFileSize: <%= S3_MAX_FILE_SIZE.to_s %>
    maxNumberOfFiles: 3
    messages: {
      maxFileSize: 'File exceeds maximum allowed size of <%= ActionView::Base.new.number_to_human_size S3_MAX_FILE_SIZE %>'
      acceptFileTypes: 'File type should be: jpg, png or pdf'
    }
    progressall: (e, data) ->
      progress = parseInt(data.loaded / data.total * 100, 10)
      progressBar.css('width', progress + '%')

    start: (e) ->
      console.log 'start'
      progressBar.
        css('background', 'green').
        css('display', 'block').
        css('width', '0%').
        text("Loading...")

    done: (e, data) ->
      console.log 'done'
      progressBar.text("Uploading done")

      # extract key and generate URL from response
      key = $(data.jqXHR.responseXML).find("Key").text()
      url = '//' + $fileInput.data('host') + '/' + key
      callback(url)

    fail: (e, data) ->
      console.log 'fail'
      progressBar.
        css("background", "red").
        text("Failed")
  ).on 'fileuploadprocessalways', (e, data) ->
    currentFile = data.files[data.index]
    if (data.files.error && currentFile.error)
      # there was an error, do something about it
      console.log(currentFile.error)
      alert(currentFile.error)
HERE_DOC
~~~

~~~
# app/assets/javascripts/file_upload_adminlte.coffee
window.setupFileUpload = ($fileInput)->
  directUpload $fileInput, (url) ->
    randomId = Math.floor(Math.random() * 9999 + 100)
    # here should we create new element with random number
    #  name: 'customer[documents_attributes][' + randomId + '][key]'
    if url.match(/\.(png|jpeg|jpg|png)$/)
      # create with img
    else
      # create with pdf

    $('#file-list').append new_element
~~~

jQuery file upload [minimal
setup](https://github.com/blueimp/jQuery-File-Upload/wiki/Basic-plugin) requires
jQuery (usually already included in rails), jquery.ui.widget (you can find that
in download package), jquery.fileupload.js. To make sure it is installed run in
console `console.log($().fileupload)`
Just copy them to your asset path and include (`require_tree .` probably loads
in wrong order so it is better to manually include them).

~~~
// app/assets/javascripts/application.js
//= require jquery.ui.widget
//= require jquery.fileupload
//= require jquery.fileupload-process
//= require jquery.fileupload-validate
~~~

Add some styles

~~~
cat >> app/assets/stylesheets/main.scss << HERE_DOC
.document {
  padding: 5px;
  .document-thumbnail {
    width: 50px;
    // for fa pdf icon
    font-size: 50px;
  }
}
HERE_DOC
~~~

You can save one or many urls in `documents` field (type text) and use
`serialize :documents, Array` but if you have more logic its better to keep them
in separate model

~~~
rails g model document name key documentable:references{polymorphic}
rake db:migrate

sed -i app/models/document.rb -e '/^end/i \
  validates_presence_of :key\
\
  def file_name\
    "#{key}".split("/").last\
  end'

sed -i app/models/post.rb -e '/class Post/a \
  has_many :documents, as: :documentable\
  accepts_nested_attributes_for :documents, allow_destroy: true'

sed -i app/controllers/posts_controller.rb -e '/params.require/c \
      params.require(:post).permit(\
        :name, :title,\
        documents_attributes: [:key, :name, :_destroy, :id]\
      )'
~~~

In form you can add new

~~~
# app/views/posts/_form.html.erb
    <div id="file-list">
      <%= f.fields_for :documents do |document_form| %>
        <%= document_form.hidden_field :key %>
        <div class="document row form-inline">
          <%= link_to "#", "data-toggle" => "modal", "data-target" => "#documentModal", "data-set-modal-source" => "http:#{document_form.object.key}", class: 'col-sm-5' do %>
            <% if document_form.object.key =~ /.\.(png|jpeg|jpg|png)$/ %>
              <img src='<%= document_form.object.key %>' class='document-thumbnail'>
            <% else %>
              <i class="fa fa-file-pdf-o document-thumbnail" aria-hidden="true"></i>
            <% end %>
            <%= document_form.object.file_name %>
          <% end %>
          <div class="col-sm-5">
            <%= document_form.text_area :name, placeholder: "My File Description", skip_label: true %>
          </div>
          <div class="col-sm-2">
            <%= document_form.check_box :_destroy, label: "Mark for delete" %>
          </div>
        </div>
      <% end %>
    </div> <!--<div id="file-list">-->
    <div class="m-t-10 text-center">
      <a href="#" class="btn btn-success btn-block" onclick="$('#fileupload').trigger('click')">
        <i class="fa fa-plus" aria-hidden="true"></i>
        Add more files
      </a>
    </div>
    <%= file_field_tag "files[]", id: 'fileupload', multiple: true, data: { 'form-data': @s3_direct_post.fields, 'url': @s3_direct_post.url, 'host': URI.parse(@s3_direct_post.url).host }, style: 'position: absolute;left: -500px' %>
    <script>
      setupFileUpload($('#fileupload'))
    </script>
~~~

In view you can list

~~~
  <% if @post.documents.size.zero? %>
    No Documents available
  <% end %>
  <% @post.documents.each do |document| %>
    <div class="document row">
      <%= link_to "#", "data-toggle" => "modal", "data-target" => "#documentModal", "data-set-modal-source" => "http:#{document.key}", class: 'col-sm-5' do %>
        <% if document.key =~ /.\.(png|jpeg|jpg|png)$/ %>
            <img src='<%= document.key %>' class='document-thumbnail'>
          <% else %>
            <i class="fa fa-file-pdf-o document-thumbnail" aria-hidden="true"></i>
        <% end %>
        <%= document.file_name %>
      <% end %>
      <div class="col-md-6 document-name">
        <%= document.name %>
      </div>
    </div>
  <% end %>
~~~

And you can preview in bootstrap modal

~~~
<div id="documentModal" class="modal fade" role="dialog">
  <div class="modal-dialog modal-lg">
    <!-- Modal content-->
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal">&times;</button>
        <a href="" id="download-link">
          <h4 class="modal-title">Download document
            <i class="icon-download-alt"></i>
          </h4>
        </a>
      </div>
      <div class="modal-body">
        <img id="image-preview" src="" class="center-block">
        <iframe src="" frameborder="0" width="100%" class="iframe" id="pdf-preview"></iframe>
        <%= image_tag 'loader.gif', class: 'center-block loader' %>
      </div>
    </div>
  </div>
</div>
~~~



# AWS Bucket

* create a bucket **video-uploading-demo**. You can choose any region. But
  choose one word (without dots) because of *Insecure connection* error
* go to the [IAM page](https://console.aws.amazon.com/iam/home) to create user,
  save credentials and create [inline policy](
  {% post_url  2016-02-29-amazon-aws-s3 %}) to give access to the bucket
  **video-uploading-demo**
* I'm not sure about [step 2](https://github.com/asafdav/ng-s3upload) (grant
  put/delete and Upload/Delete permissions) since
  above inline policy works fine for me
* add CORS `<AllowedMethod>POST</AllowedMethod><AllowedHeader>*</AllowedHeader>` to Bucket -> Properties ->
  Permissions -> Add Cors configuration [2016-02-29-amazon-aws-s3#cors](
  {% post_url 2016-02-29-amazon-aws-s3 %}#cors)

# Record image

Read docs on
[cordova-plugin-camera](https://github.com/apache/cordova-plugin-camera) rather
than on [ngCordova camera](http://ngcordova.com/docs/plugins/camera/).

~~~
bower install ngCordova --save-dev
sed -i '/<\/head>/i \
    <!-- plugins -->
    <script src="lib/ngCordova/dist/ng-cordova.js"></script>\
' www/index.html
ionic plugin add cordova-plugin-camera
~~~

Example app [Angluarjs-Ionic-Schedule-App](https://github.com/sirius2013/Angluarjs-Ionic-Schedule-App)


Android emulator needs SD card before using camera, so add some MB in
`tools/android avd`.

~~~
cordova plugin add cordova-plugin-file-transfer
~~~

[AWS S3](http://aws.amazon.com/articles/1434) conditions can prevent user
uploading on different location.

starts-with should be set for all files...
