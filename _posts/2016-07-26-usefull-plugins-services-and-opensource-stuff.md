---
layout: post
title: Usefull plugins, services and opensource stuff
---

Find other software

https://www.libhunt.com/r/awesome-selfhosted
https://github.com/awesome-selfhosted/awesome-selfhosted#ticketing

# Misc

* https://cantunsee.space/ practice tests design details css question answer
  pick better solution left or right
* similar to design questions, here is examples of do and donts
  https://material-minimal.com/learn/design-hacks/tips-and-tricks/
  also tutorial https://mdbootstrap.com/learn/mdb-foundations/basics/introduction/
* tailwind questions https://twwordle.hyperui.dev/?
* [fastclick](https://github.com/ftlabs/fastclick)
* [zeroclipboard](https://github.com/zeroclipboard/zeroclipboard) to copy some
text on click
* [jsPDF](https://github.com/MrRio/jsPDF) pdf generator
  [examples](http://mrrio.github.io/jsPDF/#)
  preview pdf in browser http://mozilla.github.io/pdf.js/
  https://github.com/mozilla/pdf.js/wiki/Setup-pdf.js-in-a-website
  in rails there is `gem 'pdfjs_viewer-rails'` but check for domain is enabled
  so there is an error https://github.com/mozilla/pdf.js/issues/7153 Related
  rails issues https://github.com/senny/pdfjs_viewer-rails/issues/12
  https://github.com/senny/pdfjs_viewer-rails/issues/42 (gem also uses old pdfjs
  version).
  Better approach is to copy from
  [examples](https://github.com/mozilla/pdf.js/wiki/Setup-pdf.js-in-a-website#from-examples)

  ```
  # app/views/pages/pdf_viewer.html.erb
  <%# https://github.com/mozilla/pdf.js/wiki/Setup-pdf.js-in-a-website#from-examples %>
  <head>
    <%= stylesheet_link_tag 'plugins/pdf_viewer', media: nil %>
    <!-- This snippet is used in production (included from viewer.html) -->
    <link rel="resource" type="application/l10n" href="plugins/pdf_viewer.properties">
    <%= javascript_include_tag 'plugins/pdf_viewer_build' %>
    <%= javascript_include_tag 'plugins/pdf_viewer' %>
  </head>


  # cp web/viewer.css to app/assets/stylesheets/pdf_viewer.scss and replace all
  # url with asset-url('pdf_viewer/image.svg') # note that for
  # pdf_viewers/grab.cur there is double quote
  # copy all web/images to app/assets/images/pdf_viewer/
  # cp web/viewer.js and build/* to app/assets/javascripts/pdf_viewer/
  # add your domain to HOSTED_VIEWER_ORIGINS viewer.js.erb:1547
  # update to point to worker script
   workerSrc: {
-    value: '../build/pdf.worker.js',
+    value: '<%= asset_path 'pdf_viewer/pdf.worker.js' %>',
  ```

  For error `Message: Failed to fetch` (console error `Access to fetch at
  'http://s3.amazonaws.com/...' from origin 'http://localhost:3004' has been
  blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on
  the requested resource. If an opaque response serves your needs, set the
  request's mode to 'no-cors' to fetch the resource with CORS disabled.`) you
  need to add cors to a bucket https://stackoverflow.com/questions/25104448/pdf-js-cors-issue-for-s3-file
  ```
<CORSConfiguration>
    <CORSRule>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <AllowedMethod>HEAD</AllowedMethod>
        <MaxAgeSeconds>3000</MaxAgeSeconds>
        <AllowedHeader>Authorization</AllowedHeader>
    </CORSRule>
</CORSConfiguration>
  ```

* number input field [spinner](https://github.com/vsn4ik/jquery.spinner)
* toggle checkbox: [bootstrap
  switch](http://bootstrapswitch.com/examples.html)

  ~~~
  bower install bootstrap-switch
  //= require bootstrap-switch/dist/js/bootstrap-switch.js
   *= require bootstrap-switch/dist/css/bootstrap3/bootstrap-switch.css
  ~~~

  * [abpetkov/switchery](https://github.com/abpetkov/switchery)
  * [animated smile checkbox](https://codepen.io/Rplus/pen/LdjwZz)
  * [bootstraptoggle](http://www.bootstraptoggle.com/) only for bootstrap 3, for
    B4 use https://github.com/gitbrent/bootstrap4-toggle
  ```
  # http://www.bootstraptoggle.com/
  # https://github.com/gitbrent/bootstrap4-toggle
  $('[data-bootstrap-toggle').each ->
    $(this).bootstrapToggle(
      # this class will enable vimium
      style: 'demo-button'
    )
  ```
* trigger a function on scroll to an element [waypoints](http://imakewebthings.com/waypoints/)
* drag and drop elements <https://shopify.github.io/draggable/>

# Alerts and notifications

## Jbox

[jbox](http://stephanwagner.me/jBox) notifications, modals, tooltips, images
and confirm windows. You need to install plugins that you use

~~~
yarn add jbox

// app/assets/javascripts/application.js
//= require jbox/src/js/jBox.js
//= require jbox/src/js/plugins/jBox.Notice.js

// app/assets/javascripts/document_on_turbolinks_load.coffee
  // add tooltip to all elements with title <a title='Hi'>
  new jBox('Tooltip', {
    attach: '[title]'
  })

// app/assets/javascripts/document_on.coffee
# <%= button_tag 'Edit', 'data-modal': new_admin_role_path, 'data-modal-title': 'Edit role', class: 'btn' %>
$(document).on 'click', '[data-modal]', (e) ->
  openModal(
    url: e.currentTarget.dataset.modal
    title: e.currentTarget.dataset.modalTitle
    width: e.currentTarget.dataset.modalSize
  )
  console.log "data-modal #{e.currentTarget.dataset.modal}"

// app/assets/javascripts/plugins/jbox_open_modal.coffee
# https://stephanwagner.me/jBox/options#ajax
window.openModal = ({url, title, width = 'auto'}) ->
  modal = new jBox 'Modal', {
    title: e.currentTarget.dataset.modalTitle
    ajax: {
      url: e.currentTarget.dataset.modal
      reload: 'strict'
      setContent: true
      complete: ->
        # we need to reposition when ajax returns
        this.position()
    }
    onCloseComplete: ->
      $("##{this.options.id}").html('')
  }
  modal.open()
  console.log "data-modal #{e.currentTarget.dataset.modal}"

window.initializeJboxTooltip = ->
  # note that this will remove title attribute
  new jBox 'Tooltip', {
    attach: '[title]'
    theme: 'TooltipDark'
  }

// app/assets/stylesheets/application.sass
@import 'jbox/src/scss/jBox.scss'
@import 'jbox/src/scss/plugins/jBox.Notice.scss'


# app/controllers/application_controller.rb
  after_action :_add_flash_message, if: -> { request.xhr? }
  after_action :_add_js_plugins_initialisations, if: -> { request.xhr? }

  # rubocop:disable Metrics/AbcSize
  def _add_flash_message
    return unless request.format == 'application/javascript'

    response.body += "flash_alert('#{view_context.j flash.now[:alert]}');" if flash.now[:alert].present?
    response.body += "flash_notice('#{view_context.j flash.now[:notice]}');" if flash.now[:notice].present?
  end
  # rubocop:enable Metrics/AbcSize

  def _add_js_plugins_initialisations
    code = 'initializeJboxTooltip();' if response.body.match? /title=/
    return unless code.present?

    response.body += if request.format == 'application/javascript'
                       view_context.j(code)
                     else
                       "<script>#{code}</script>"
                     end
  end


# app/views/layouts/application.html.erb
      <div class="d-none-not-important" id='notice-holder'><%= notice %></div>
      <% if notice %>
        <script>
          flash_notice('<%= j notice %>');
        </script>
      <% end %>

      <div class="d-none-not-important" id='alert-holder'><%= alert %></div>
      <% if alert %>
        <script>
          flash_alert('<%= j alert %>');
        </script>
      <% end %>
    </body>
  <% end %>
</html>

# app/assets/javascripts/flash_messages.coffee
# https://stephanwagner.me/jBox/get_started
window.flash_alert = (message) ->
  flash_message message, 'red'

window.flash_notice = (message) ->
  flash_message message, 'blue'

window.flash_message = (message, color) ->
  # https://stephanwagner.me/jBox/documentation says that we need to wait for
  # ready for page load, so we use setTimeout
  setTimeout(
    ->
      new jBox 'Notice',
        autoClose: 10000
        attributes:
          x: 'right'
          y: 'bottom'
        stack: true
        animation:
          open:'bounce'
          close:'fadeOut'
        content: message
        color: color
    50 # tried with 0, 1, but it was triggered before page load and not working
  )
~~~

# Input mask

* <https://unmanner.github.io/imaskjs/>
* jquery form validation https://github.com/jquery-validation/jquery-validation
  to validate only part of a form, you can use element http://jqueryvalidation.org/Validator.element/ but the problem is that you do not know if it is changed
  https://forum.jquery.com/topic/using-jquery-validation-on-textboxes-in-a-specific-div

# Select something

* [tokenize](http://zellerda.com/projects/jquery/tokenize) for tags,
  preselecting items
* nice js filter [bouncy content
filter](http://codyhouse.co/demo/bouncy-content-filter/)
* javascript input filter
<http://digitalbush.com/projects/masked-input-plugin/>
* [selectize](http://brianreavis.github.io/selectize.js/) and
[selectize-rails](https://github.com/manuelvanrijn/selectize-rails)
* [multiselect](http://loudev.com/) nice way to select multiple items from
  opened select box. If you show in popup with different data, then call
  `select.multiSelect('refresh');` after you initialize multiSelect.
  * if you want to keep order `keepOrder` option shows nice order, but
    submitting the form do not preserve order. You can use this
    [solution](http://stackoverflow.com/a/22271944/287166)
    [issue](https://github.com/lou/multi-select/issues/184)

  ~~~
  select.multiSelect({
    keepOrder: true,
    afterSelect: function(value){
      $('#templates-select option[value="'+value+'"]').remove();
      $('#templates-select').append($("<option></option>").attr("value",value).attr('selected', 'selected'));
    },
  });
  ~~~

  * `keepOrder` does not work for preselected items, solution could be to
    manually select (not already select: selected):

  ~~~
  $.each(selectedTemplateIds, function(index, template_id) {
    select.multiSelect('select', [String(template_id)]);
  });
  ~~~

* [multiple select](https://github.com/wenzhixin/multiple-select) it creates ul
lists, if inside `overflow: hidden` you can use
[container](https://github.com/wenzhixin/multiple-select/pull/34) and
`dropWidth: '100px'` options

## Select2

This nice [select2 examples](https://select2.github.io/examples.html) works.

* problem is when you have animations (like in modal, or collapsed-box) select
can calculate full width. solution is to set inline `style="width:100%"` or to
call with

  ~~~
    $('#customer-name-select').select2({ placeholder: 'Search by Customer Name or Username', width: '100%'});
  ~~~

* to make it `inline` you can wraps inside `inline-block` element and set some
  size with `style='width:100px'` on select element (select2 will use that to
  calculate size).

  ~~~
  # app/assets/select2.sass
  .select2-inline
    display: inline-block
    // by default .select2-container is block, which move it slightly to the top
    // and then it is not inline with other inline text or buttons
    .select2-container
      display: inline-block
  ~~~

  one solution to make select wider is to use `select_tag :id, '',
  include_blank: 'Long sentence that make select width wider'`
* options like
[dropdownParent](https://select2.github.io/options.html#dropdownParent) could
help

You can use `data-(any option)` so for example you can use
`data-placeholder="Please select"` to set select2 placeholder, or
`data-tags="true"` to enable tagging (also need `multiple: true` for select to
accept multiple options)

I use this initializer `data-select2-initialize` (do not use `data-select2`
since it is used already) and if you use turbolinks and have all select elements
on main page

~~~
# app/assets/javascripts/turbolinks_load.coffee.rb
$(document).on 'turbolinks:load', ->
  console.log 'turbolinks:load'
  $('[data-select2-initialize]').each ->
    options = {}
    if $(this).attr 'placeholder'
      # select_tag :id, options, include_blank: true, placeholder: 'Choose one'
      options['placeholder'] = $(this).attr 'placeholder'
    else
      options['placeholder'] = '<%= I18n.translate 'please_select' %>'

    options['language'] = determine_language()
    $(this).select2 options
~~~

or without turbolinks or if you have select elements on some ajax responses, you
can manually call `<script>initializeSelect2()</script>`)
~~~
# app/assets/javascripts/data_select2_initialize.coffee
@initializeSelect2 = ->
  $('[data-select2-initialize]').each ->
    if $(this).attr 'placeholder'
      placeholder = $(this).attr 'placeholder'
    else
      placeholder = "Please select"
    $(this).select2(
      placeholder: placeholder
    )
~~~

~~~
  <%= f.select :customer, options_from_collection_for_select(Customer.all, 'id', 'name_and_username'),  { include_blank: true }, 'data-select2-initialize': true, 'data-placeholder': 'Select customer' %>
  <script>
    initializeSelect2();
  </script>
~~~

~~~
# default options for all select
# https://select2.github.io/options.html#setting-default-options
# issue when select2 is used in modal or other elements that fades in
$.fn.select2.defaults.set("width", "100%")
~~~

Problem is when select2 is inside modal with `tabindex="-1"`
[example](https://github.com/select2/select2/issues/119)

To select item on select2 with ajax source, you need to initialize with data and
keep ajax source

~~~
    $(selectTarget).select2({
      data: [{id: selectValue, text: selectText}],
      ajax: {
        url: $(selectTarget).data('select2AjaxInitialize'),
        dataType: 'json'
      }
    });
~~~

or for all

~~~
$(document).on 'turbolinks:load', ->
  $('[data-select2-ajax-initialize]').each ->
    url = $(this).data('select2AjaxInitialize')
    options = {
      ajax: {
        url: url
        dataType: 'json'
      }
    }
    $(this).select2 options
    # using label for select does not open select2 on focus, so on label click
    # we should trigger open
    # https://github.com/select2/select2/issues/2311#issuecomment-180666626
    $(this).focus ->
      $(this).select2('open')
~~~

Search not only on text but on custom data attributes https://stackoverflow.com/questions/36021892/search-on-select-attribute-in-select2
https://select2.org/searching
I tried to add hidden divs to `<option>` but it seems that there could be only
the text as inner object. So I add title and copy `matcher` function from
Select2 and add
```
# app/assets/javascripts/data_select2_initialize.coffee
matcher = (params, data) ->
  # ... copy from original matcher ...
  title = stripDiacritics(data.title).toUpperCase()
  # check if title exists and contains the term
  if title.indexOf(term) > -1
    return data

@initializeSelect2 = ->
  $('[data-select2-initialize]').each ->
    # we need to remove tabindex attribute from modal if exists
    # https://github.com/select2/select2/issues/119
    $(this).parents(".modal").removeAttr('tabindex')
    $(this).select2(
      matcher: matcher
    )
```

Install with stimulus
```
```


# Calendar date and time picker

* calendar https://dev.to/timnans/rails-calendar-with-tui-and-stimulusjs-3je
  
* Best option is [daterangepicker](http://www.daterangepicker.com/) it can be
used for singleDatePicker, dateTime picker
  * download with `bower install bootstrap-daterangepicker --save` (git should
  be <https://github.com/dangrossman/bootstrap-daterangepicker>)
  * autoApply (can't be used when timePicker enabled)
  * autoUpdateInput (when you want to parse user selection, custom format)
  * it can contains predefined ranges (Last 7 days)
  * note that is uses `data-daterangepicker` so use with your own
  `data-bootstrap-datarangepicker`
  It has some problems with module as CommonJS (AMD works fine) since you need
  to pass jquery
  https://github.com/dangrossman/daterangepicker/issues/1294
  ```
  // let moment = require('moment')
  // https://github.com/dangrossman/daterangepicker/issues/1294
  // use from vendor until pull request is accepted
  // https://github.com/dangrossman/daterangepicker/pull/2026
  // require('./vendor/daterangepicker')(moment, $)
  ```
* [pickdate](http://amsul.ca/pickadate.js/)
* [jquery ui datepicker](https://jqueryui.com/datepicker/)
  Error on chrome if you use `type=date`
  [link](http://stackoverflow.com/questions/16890376/chrome-type-date-and-jquery-ui-date-picker-clashing)
  so use `type=text`
* [bootstrap-datepicker](https://bootstrap-datepicker.readthedocs.io/en/latest/index.html)
* time picker <http://jonthornton.github.io/jquery-timepicker/>

* "One minute ago" can be updated automatically in this
  [timeago](http://timeago.yarp.com/) plugin. Similarly,
  [jquery.countdown](https://github.com/hilios/jQuery.countdown) for future
  dates


# Image file upload drag and drop crop

* http://plugins.krajee.com/file-input/demo

* [jquery.fileapi](http://rubaxa.github.io/jquery.fileapi/) supports
  webcam, crop, dragndrop
* https://foliotek.github.io/Croppie/
* [jQuery-File-Upload](https://blueimp.github.io/jQuery-File-Upload/)
* [browser-camera](https://davidwalsh.name/browser-camera) html5 getUserMedia

For file input you can show preview file using
[FileReader](https://developer.mozilla.org/en-US/docs/Web/API/FileReader)

```
# app/assets/javascripts/ready_page_load.coffee
$(document).on 'ready page:load', ->
  # file image preview
  $("[data-image-preview]").each ->
    return unless typeof FileReader != "undefined"
    $preview_target = $($(this).data('imagePreview'))
    $(this).change ->
      regex = /^([a-zA-Z0-9\s_\\.\-:])+(.jpg|.jpeg|.png)$/
      if regex.test $(this).val().toLowerCase()
        reader = new FileReader()
        reader.onload = (e) ->
          $preview_target.html $('<img />', src: e.target.result)
        reader.readAsDataURL this.files[0]

# app/views/users/_form.html.erb
      <div class="admin-image-preview">
        <div class="admin-image-preview__thumbnail" id='preview-book-cover-picture'>
          <% if f.object.book_cover_picture.present? %>
            <%= image_tag f.object.book_cover_picture.url(:medium) %>
          <% end %>
        </div>
        <div>
            <%= f.file_field :book_cover_picture, 'data-image-preview': '#preview-book-cover-picture' %>
        </div>
      </div>
```

# Rich text editors wysiwyg

list:

* <https://summernote.org/>
* <http://wagn.org> wiki with cards with editor
* <https://beefree.io/>
* [fireEdit](https://github.com/coltaemanuela/FireEdit)
* [ckeditor](https://ckeditor.com/docs/index.html) for 4.1 you can add your own
  style https://ckeditor.com/docs/ckeditor4/latest/api/CKEDITOR_stylesSet.html
  https://ckeditor.com/docs/ckeditor4/latest/features/styles.html
  https://ckeditor.com/docs/ckeditor4/latest/api/CKEDITOR_config.html#cfg-contentsCss
  rails integration https://github.com/galetahub/ckeditor
  ```
  // app/assets/javascripts/cheditor/config.js
  CKEDITOR.stylesSet.add('default', [
      // Block-level styles
    { name: 'Blue Header', element: 'h2', attributes: { 'class': 'blue-marker', style: 'color:blue' } },
  ]);
  ```
* counter how many characters left https://github.com/rikschennink/short-and-sweet
* quill

## Quil

[quill](https://quilljs.com/) [examples](https://quilljs.com/playground)
You can edit by selecting content
[bubble](https://quilljs.com/docs/themes/#bubble)

Usually you do not need any additional styles when you want to show html (bold,
italic, h1, quote, code, list). Color and background colors are defined inline
with `style` attribute so it works (unless you use rails `sanitize` which remove
inline styles). Other styles requires quill classes (center, right, direction,
align, indent, large, huge, fonts) so you need to inlude that css and to wrap
output inside `ql_editor` class (or you can use [inline
style](https://quilljs.com/playground/#class-vs-inline-style) but it seems )

~~~
  <div class="ql-editor">
    <%= raw @location.about_html %>
  </div>
  <script>
    loadFile("//cdn.quilljs.com/1.2.2/quill.snow.css");
  </script>
~~~

You can load in head section

~~~
<!-- app/views/layouts/application.html.erb -->
<script src="//cdn.quilljs.com/1.2.2/quill.min.js" type="text/javascript"></script>
<link href="//cdn.quilljs.com/1.2.2/quill.snow.css" rel="stylesheet">
~~~

or you can load dynamically using this helper

~~~
# app/assets/javascripts/load_external_css_javascript.coffee
# all files will be loaded defered so you can use callbak
window.alreadyLoadedFiles = []
window.loadFile = (filename, callback) ->
  if callback and filename in alreadyLoadedFiles
    console.log "File #{filename} already loaded"
    callback()
    return
  if filename.slice(-2) == "js"
    fileref = document.createElement('script')
    fileref.setAttribute("type","text/javascript")
    fileref.setAttribute("src", filename)
    fileref.onload = callback if callback
  else if filename.slice(-3) == "css"
    fileref = document.createElement("link")
    fileref.setAttribute("rel", "stylesheet")
    fileref.setAttribute("type", "text/css")
    fileref.setAttribute("href", filename)
  if (typeof fileref!="undefined")
    document.getElementsByTagName("head")[0].appendChild(fileref)
    alreadyLoadedFiles.push filename
    console.log "Adding file #{filename}"
~~~

and use it with `loadQuill('editor-content', 'editor-container');`

~~~
# app/assets/javascripts/quill.coffee
window.loadQuill = (editorContent, editorContainer) ->
  loadFile "//cdn.quilljs.com/1.2.2/quill.snow.css"
  loadFile "//cdn.quilljs.com/1.2.2/quill.min.js", ->
    console.log "Loading quill editorContent=#{editorContent} " +
      "editorContainer=#{editorContainer}"
    # stripped example from https://quilljs.com/docs/modules/toolbar/
    # so we do not need to load quill classes
    options =
      # debug: 'info'
      modules:
        toolbar: [
          [{ 'header': [1, 2, 3, 4, 5, 6, false] }]
          ['bold', 'italic', 'underline']
          ['blockquote', 'code-block']
          [{ 'list': 'ordered'}, { 'list': 'bullet' }]
          ['clean']
          ['link']
        ]
      theme: 'snow'

    $editorContainer = $("##{editorContainer}")
    quill = new Quill($editorContainer.get(0), options)
    toolbar = quill.getModule 'toolbar'
    toolbar.addHandler 'link', (value) ->
      if (value)
        href = prompt('Enter the URL')
        href = "http://#{href}" if href.substring(0,4) != "http"
        this.quill.format('link', href)
      else
        this.quill.format('link', false)
    quill.on 'text-change', ->
      newHtml = $($editorContainer).find('.ql-editor').html()
      $("##{editorContent}").val newHtml
~~~

You can also get html on [form
submit](https://quilljs.com/playground/#form-submit)

~~~
# app/assets/javascript/quill.coffee
$(document).on 'ready page:load', ->
  options =
    debug: 'info'
    modules:
      toolbar: [
        ['bold', 'italic', 'underline']
        ['blockquote', 'code-block']
        [{ 'header': 1 }, { 'header': 2 }]
        [{ 'list': 'ordered'}, { 'list': 'bullet' }]
        [{ 'script': 'sub'}, { 'script': 'super' }]
        [{ 'indent': '-1'}, { 'indent': '+1' }]
        [{ 'direction': 'rtl' }]
        [{ 'size': ['small', false, 'large', 'huge'] }]
        [{ 'header': [1, 2, 3, 4, 5, 6, false] }]
        [{ 'color': [] }, { 'background': [] }]
        [{ 'font': [] }]
        [{ 'align': [] }]
        ['clean']
      ]
    theme: 'snow'
  syncHtml = ->

  $('[data-quill-editor]').each ->
    editoContainer = $(this).get(0)
    quill = new Quill(editoContainer, options)
    $targetInput = $(this.dataset.quillEditor)
    quill.on 'text-change', ->
      newHtml = $(editoContainer).find('.ql-editor').html()
      $targetInput.val newHtml
~~~


~~~
<%# app/views/admin/pages/show.html.erb %>
<div data-quill-editor="#editor-content">
  <%# you can test with raw content but sanitized is safer %>
  <%= sanitize @page.content %>
</div>
~~~

~~~
<%# app/views/admin/pages/_form.html.erb
    <div id="editor-container">
      <%= sanitize @location.about_html %>
    </div>
    <script>
      loadQuill('editor-content', 'editor-container');
    </script>
~~~

# Fontawesome

Font awesome FA icons are very usefull for quick icons. Look for
[examples](http://fontawesome.io/examples/)

* color can be set with `bg-red`
* spinner `<i class="fa fa-spinner fa-spin" style="font-size:24px"></i>`
* in FA 5, by default should be used `fas` (solid), `far` (regular) and `fal`
  (light) is only for pro, but some icons like window-close is free for regular
  https://fontawesome.com/icons/window-close?style=regular
* size can be with setting `style='font-size: 2rem'` or using class `fas fa-2x`
  https://fontawesome.com/how-to-use/on-the-web/styling/sizing-icons
* padding between icon and text is with margin, and center with fixed width
  `<i class='fas fa-tasks fa-fw mx-1' aria-hidden='true'></i><%= t('home') %>`

Other free fonts can be found on http://fontello.com/ where you can select icons
and download only that batch of icons.
* page loader progress bar <http://ricostacruz.com/nprogress/>
* page loader car <https://codepen.io/igor0ser/pen/amJkvp>

# Images

* free images photos high resolution <https://unsplash.com/> and free online
  sample images https://unsplash.it/images
* free stock photos <https://burst.shopify.com>
* free photo https://barnimages.com/
* free https://picjumbo.com/
* sample placeholder https://placeholder.com/
* free vector images kajak pictogram https://pixabay.com/en/sport-pictogram-olympia-water-swim-1580667/
* flags http://flagpedia.net/india
* create passport photo id card https://www.idphoto4you.com needs flash plugin
  so open in chrome
* free animations https://lottiefiles.com/49964-running-on-treadmill

# Slides

* <https://github.com/jaredwilli/io-slides> presentations
 https://code.google.com/archive/p/io-2012-slides/
 [video](https://www.youtube.com/watch?feature=player_embedded&v=WRvECXyWj80)

# Graphs

* for rails you can use https://chartkick.com (which uses Chart.js)
  https://github.com/duleorlovic/chartkick_groupdate_graphs_in_rails
  ```
  # for graphs
  gem 'chartkick'
  gem 'groupdate', '~> 6.1'

  # in view
      <%= line_chart @registrations_per_day_form.data, height: '400px', width: '99%' %>
  ```
* [graphs
gallery](https://github.com/mbostock/d3/wiki/Gallery) and
[bl.ocks.org/mbostock](http://bl.ocks.org/mbostock)
[slider](http://square.github.io/crossfilter/)
[people](http://www.findtheconversation.com/concept-map/#population)
[parallel](http://exposedata.com/parallel/) and the best is
[grafana](http://play.grafana.org/)
* [d3 for charts](http://plottablejs.org/examples/)
* [google
  charts](https://developers.google.com/chart/interactive/docs/gallery/linechart#curving-the-lines)
  can be curved but can not be area in the same time https://github.com/google/google-visualization-issues/issues/2110
* [highcharts](https://www.highcharts.com/demo) $500
* [echarts](https://echarts.apache.org/examples/en/index.html) free
* [chartjs](http://www.chartjs.org/). Labels and datasets should be inside
  `data` attribute. Labels are for all charts.

  ~~~
  var myChart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: ['2018-1-1', '2018-1-2'],
      datasets: [
      {
        data: [1, 2],
        label: 'Tavan',
      },
      ]
    }
  })
  ~~~
  To add new data you can push to datasets and labels
  https://www.chartjs.org/docs/latest/developers/updates.html

* arrows and grouping
  <http://marvl.infotech.monash.edu/webcola/examples/smallgroups.html>

* To draw diagrams and graphs
  <http://js.cytoscape.org/demos/compound-nodes/>
  Define elements with array of `{data: { id: 'a' }}`. Edges have
  `data: { id: 'ab', source: 'a', target: 'b' }`. Alternativelly you can define
  elements as objects of `nodes: [{data: {id:'a'}}], edges: [{data: {id: 'ab',
  source: 'a', target' b'}}]`.
  When you want to group inside squares (compound) than use `data: { parent: 'a'
  }` (in this case `a` will become square).
  Layout http://js.cytoscape.org/#layouts
  ```
  layout: { name: 'grid' }
  ```
  Style is array of selectors style keys:
  ```
  style: [
    {
      selector: 'node',
      style: {
        'label': 'data(id)'
      }
    }
  ]
  ```

  Events like `dragfree` `tapdrag` http://js.cytoscape.org/#collection/events
  http://js.cytoscape.org/#core/events
  You can get position http://js.cytoscape.org/#notation/position
* draw sketchy hand drawn styled charts https://github.com/jwilber/roughViz


# Email services

* [formspree](http://formspree.io/) sending emails in javascript, it is
  opensource so you can host it https://github.com/formspree/formspree written
  in python
* <http://www.freecontactform.com/free.php> free contact form

# User tracking services

[mixpanel](https://mixpanel.com/)

  * create tracking plan that defines each event and its properties (name and
  type: string, number, boolean), In plan we should define: Event name,
  description, why this event and for each property: property name, property
  description, property type, why this property
  * call `identify` when user signup, or if there are existing users, when the
  logs in, or use the app
  * you can also send push notifications from mixpanel

https://www.fullstory.com/
https://segment.com/ collect customer data

* https://github.com/ankane/ahoy is free version that stores data in our db
  https://gorails.com/episodes/internal-metrics-with-ahoy-and-blazer
  https://blog.engineyard.com/ruby-unbundled-track-how-customers-use-new-features

# User support services

* intercom <https://demos.intercom.com/>
* [user deck](https://userdeck.com)
* [chatwoot](https://www.chatwoot.com/docs/channels/website) opensource rails
  customer support engagement
* https://github.com/chaskiq/chaskiq opensource rails


# User intro tour

User navigation to website with help tips should be only one, when user clicks
something (so help is to interactivelly show tips, not on page load). Users will
not remeber if you show them 8 steps for the page they just arrived.

* [introjs](http://introjs.com/) commercial lincence after 2.1
* <http://bootstraptour.com>
[source](https://github.com/sorich87/bootstrap-tour)
  * intersting value is `backdrop: true` so element is highlighted `reflex:
  true`  so click on target element dismiss page tour
  * if you want to repeat tour you can use `yield` and `content_for`
  ~~~
    <%# somewhere in support dropdown menu %>
    <% if content_for? :page_tour_script %>
      <li>
        <%= yield :page_tour_script %>
        <a href="#" onclick="startTour()">
          Start Page Tour
        </a>
      </li>
    <% end %>
  ~~~

  ~~~
  <%# on a page where we need tour %>
  <% content_for :page_tour_script do %>
    <script>
      function startTour() {
        // http://bootstraptour.com/api/
        var tour = new Tour({
          storage: false,
          steps: [
          {
            element: "[data-target='#publish-location-package-modal']",
            title: "Publish Package",
            content: "You can publish package using this button",
            placement: 'right',
            backdrop: true,
            reflex: true,
          },
        ]});

        // Initialize the tour
        tour.init();

        // Start the tour
        tour.start();
      }
    </script>
  <% end %>
  ~~~

# Visitor user analytics

* <https://www.hotjar.com/tour>
* <https://www.fullstory.com/features>

To find new customers you can find sites that link to similar content. For
example in google `link:kulakajak.org -site:kulakajak.org` will give all sites
that links to kulakajak.org. You can scrape them to find admin contact email and
suggest them to add your content that will be interesting to their audience

> Do you feel it could be a good fit for your audience?  Might be worth a
> mention :)
> Here is the Link:
> I'd love to know what you think!
> Sincerely,


## Opensource heatmaps

* example how to to track with firebase
<https://github.com/seeyourvisitors/seeyourvisitors/blob/master/gg.dev.js> and
how to show results with <https://github.com/pa7/heatmap.js> example js is here
<https://github.com/seeyourvisitors/heatmap>

* another solution is <https://github.com/dugwood/clickheat>

# Marketing

* <https://submit.co> where to submit news about your project
* <http://www.communitybuildingguide.com/> brick by brick guide for awesome
communities

# Optimisation

* [caniuse.com](http://caniuse.com/) to see what is supported and what is not
in modern browsers
* optimization for speed [google
insights](https://developers.google.com/speed/pagespeed/insights/)
[gtmetrix](https://gtmetrix.com/reports/www.scuddle.com/zP3xxAuZ)

# Error tracking

* airbrake
* sentry

# Payment

* https://www.braintreepayments.com/
* https://www.paddle.co/
* https://www.payola.io/

# Ruby

* [yml_gtranslate](https://github.com/zenchief/yml_gtranslate) automatic
generate translated yml locale files for rails localisations using google
translate `for f in $(find config/locales/ -type d);do yml_gt en rs $f;done`.
Also usefull when want to grep only en.yml `grep -i catar config/locales
--include *.en.yml -R`

# Rails

<https://github.com/eliotsykes/real-world-rails> realworldrails opensource rails
more are using rspec than test: `cd real-world-rails/apps` and `find . -name
spec -maxdepth 2 | wc -l # 134` and `find . -name test -maxdepth 2 | wc -l # 52`

Those are nice to explore:
* crowdsourcing [catarse](https://github.com/catarse/catarse) [tilt](https://github.com/crowdtilt/crowdtiltopen/) is not actually opensource, since their api should be used
* [sharetribe]({{ site.baseurl }} {% post_url 2017-11-17-sharetribe %})
opensource market peer-to-peer marketplace
* diaspora
  * when you run localy, you can import new contacts and their posts, for
  example: hq@pod.diaspora.software
* openstreetmap
* http status live site check if site is down using simple url ping 
  https://github.com/brotandgames/ciao
* chat support intercom https://www.chatwoot.com/docs/channels/website

# Userscripts

https://wiki.greasespot.net/Greasemonkey_Manual:API
https://www.oreilly.com/library/view/greasemonkey-hacks/0596101651/ch01.html

Greasemonkey Hacks: Tips & Tools for Remixing the Web with Firefox 1st Edition
By Mark Pilgrim. Google book, some pages are missing
https://books.google.rs/books?id=KWRE2C_S4YsC&pg=PR19&lpg=PR19&dq=Greasemonkey+hacks
To create a script click on Tampermonkey firefox extension icon and 'Create
a new script'
```
// ==UserScript==
// @name         New Userscript
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  try to take over the world!
// @author       You
// @match        https://wiki.greasespot.net/Metadata_Block
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // Your code here...
    fetch('
})();
```

Index of scripts:
* https://www.userscript.zone/
* https://openuserjs.org/
* https://greasyfork.org/en

## Slither io

* http://slither.io/ bots https://github.com/joetex/SlitherAStar
  http://dahquan.github.io/rattlesnake/examples/feedme.html
  https://github.com/j-c-m/Slither.io-bot
  https://github.com/ErmiyaEskandary/Slither.io-bot#visual-tutorial
  https://www.youtube.com/watch?v=eicx_UkeVhI
  https://greasyfork.org/scripts/21520-mybot-with-100k-to-circle-was-name-slither-io-bot-championship-edition/code/Mybot%20with%20100K%20to%20circle%20(was%20Name%20Slitherio%20Bot%20Championship%20Edition).user.js


```
```

# Templates

* [gentelella](https://github.com/puikinsh/gentelella)
[preview](https://colorlib.com/polygon/gentelella/index.html) [rails
version](https://github.com/iogbole/gentelella_on_rails)
* [adminLTE]( {{ site.baseurl }} {% post_url 2016-10-28-adminlte-free-template-on-rails %})
* free landing pages templates <https://freehtml5.co>
[booster](https://freehtml5.co/booster-free-html5-bootstrap-template/) or
[elate](https://freehtml5.co/demos/elate/)
* bootstrap 4
  checkout original bootstrap themes https://themes.getbootstrap.com/preview/?theme_id=6743&show_new=
  https://coderthemes.com/hyper_2/light/dashboard-crm.html
  <https://github.com/BlackrockDigital/startbootstrap-stylish-portfolio> 
  <https://blackrockdigital.github.io/startbootstrap-freelancer/>
  <https://preview.tabler.io/index.html>
* [four images](http://barbajs.org/demo/grid/index.html)
* [trophy jekyll](http://thomasvaeth.com/trophy-jekyll/) nice template for jekyll

Free illustrations kits
* index of various sources https://freeillustrations.xyz/
* https://undraw.co/illustrations

Email templates

* responsive <https://github.com/InterNations/antwort> documentation
<http://internations.github.io/antwort/> guides
<https://github.com/InterNations/antwort/wiki/HTML-Styleguide-for-Email>
* three templates <https://github.com/mailgun/transactional-email-templates>
[example](http://mailgun.github.io/transactional-email-templates/action.html)



# Animations

* <http://animejs.com/>
* <http://daneden.me/animate> animate.css you need to add `class='animated
  fadeIn`. Check how it looks in Mac Safari.
* <https://alexcambose.ro/motus/#/demos/svg> animate on scroll
* <http://daybrush.com/scenejs/>
* rain drop https://codepen.io/nealagarwal/pen/xxKWVJY
* https://oimo.io/works css animations

* detect element visibility and start animating when it is visible
<https://xtianmiller.github.io/emergence.js/>

* <https://tympanus.net/Development/PerspectivePageViewNavigation/index6.html#>
* <https://tympanus.net/Tutorials/AnimatedBorderMenus/index5.html#>
* news menu https://tympanus.net/Development/TripleViewLayout/
* <https://tympanus.net/Tutorials/3DShadingWithBoxShadows/>
* click to copy css effects for buttons and inputs https://cssfx.dev/
* shikoba effect on button https://codepen.io/iamryanyu/embed/RNjRZz?height=410&theme-id=1&slug-hash=RNjRZz&default-tab=result&user=iamryanyu&embed-version=2&pen-title=Modern%20Button%20Collection#js-box
* checkbox in css https://codepen.io/himalayasingh/pen/EdVzNL
* rotate and shake on hover https://codepen.io/duleorlovic/pen/jdjYNE
* radio button animation https://codepen.io/milanraring/pen/NWqbvxe
* link underline animate https://codepen.io/aardrian/details/qBPgYzW
* create your own logo cvg animation in
  _posts/2015-08-22-svg-css3-transforms-and-animations.markdown
* animate images flip load https://tympanus.net/codrops/2022/05/19/image-to-grid-transition/

# Scroll

* <https://russellgoldenberg.github.io/scrollama/basic/> scrolling and changing
the content
* menu active item changes when you have long text and scroll to sections
  https://codepen.io/jpdanks/pen/LYVbbwK
* scroll whole page, not stuck at the middle of the slide, it is using
  ```
  html {
    scroll-snap-type: y mandatory;
  }

  section {
    scroll-snap-align: start;
    scroll-snap-stop: always;

    display: flex;
    align-items: center;
    justify-content: center;
  }
  ```
  https://codepen.io/argyleink/pen/yLovWjz

# Nice design and ui tools

* [granim.js](https://sarcadass.github.io/granim.js/examples.html)
* [DemoHouse](https://airen.github.io/DemoHouse/)
* CSS Tricks button rotate on lost hover
<https://css-tricks.com/examples/DifferentTransitionsOnOff/>
* split cards <https://designshack.net/tutorialexamples/splitreveal/index.html>
* cards which you can click (mobile) or hover (desktop) http://devjam.com/
* nice project portfolio showcase <https://madewithenvy.com/>
* nice example site sith custom drawings <http://www.parkrun.com/>
* animations <https://www.kronologic.ai/>
* red bull <https://www.bullandbeard.com/>
* project portfolio animated <http://celialopez.fr/>
* menu rotate on round image <https://codepen.io/JoseRosario/pen/PeERry>
* menu icon hamburger animation https://codepen.io/aaroniker/pen/abzZbzR?utm_campaign=CSS%20Animation%20Weekly&utm_medium=email&utm_source=Revue%20newsletter
* menu isometric https://codepen.io/ClementRoche/pen/yEPogx
* menu in css, select sibling and opacity on all but the one being hovered https://medium.freecodecamp.org/how-to-make-the-impossible-possible-in-css-with-a-little-creativity-bd96bb42b29d
* menu slidey navigation https://github.com/duleorlovic/slidey-nav
  https://codepen.io/EricPorter/pen/YzKrqMp
* carousel slider
  https://codepen.io/Akimzzy/full/JjGKMoX
* carousel alternative with expanding flex cards https://codepen.io/z-/pen/OBPJKK?utm_campaign=CSS%20Animation%20Weekly&utm_medium=email&utm_source=Revue%20newsletter

# Fonts and icons

* [ico Moon Free](https://github.com/Keyamoon/IcoMoon-Free)
   <https://icomoon.io/#preview-free>
* [simple-line-icons](http://simplelineicons.com/)
* [bootstrap glyphs](http://getbootstrap.com/components/)
* [google webfonts](https://fonts.google.com/)
  * [cabin sketch](https://fonts.google.com/specimen/Cabin+Sketch)

* set specific font styles <http://www.cssfontstack.com/Arial-Narrow>


# Web framework

<https://github.com/kemalyst/kemalyst> based on <https://crystal-lang.org/>

# Static analysis

* [shellckeck](https://github.com/koalaman/shellcheck) static analysis tool for
shell scripts

# Mockups and wireframes

* https://balsamiq.com/products/ trial one month
* https://moqups.com/ free up to 300 objects
* https://wireframe.cc/ free for public one page

To create mockups you can use: Invision, Marvel, Baslamiq, Sketch. There is one
opensource free mokup tool: Pencil Project https://pencil.evolus.vn/

# Continuous integration

see [ci cd post]{{ site.baseurl }} {% post_url 2018-04-10-continuous-delivery-integration-ci-cd-pronto %})

#  DDNS

* [duckdns install](https://www.duckdns.org/vascript:; install.jsp)
* https://www.nsupdate.info to run on ubuntu just `sudo apt-get install
ddclient` and provide config params (or update `/etc/ddclient.conf`) and it
will run every hour
Example installing ddclient on ubuntu for cloudflare
https://github.com/mcblum/ddclient-cloudflare-ubuntu
```
# /etc/ddclient.conf
use=web, web=checkip.dyndns.org/, web-skip='IP Address' # found after IP Address

## CloudFlare (www.cloudflare.com)
##
protocol=cloudflare,        \
zone=premesti.se,            \
ttl=1,                      \
login=email@trk.in.rs,     \
password=123123             \
premesti.se,www.premesti.se
```

debug with
```
ddclient -daemon=0 -debug  -noquiet
```
I got no SUCCESS or FAILED (just DEBUG and WARNING) response
```
ddclient -daemon=0 -debug  -noquiet
DEBUG:    proxy  = 
DEBUG:    url    = checkip.dyndns.org/
DEBUG:    server = checkip.dyndns.org
DEBUG:    get_ip: using web, checkip.dyndns.org/ reports 109.93.13.1
```
but when I remove cache

```
rm /var/cache/ddclient/ddclient.cache
```
it works
```
SUCCESS:  en-premesti-se.trk.in.rs -- Updated Successfully to 109.93.13.9
```

# Duckdns

Problem with dynamic dns is that their servers (hosted on AWS) sometime fails.
Currently it shows correct ip

```
dig +noall +answer trkcam.duckdns.org
trkcam.duckdns.org.	60	IN	A	77.46.227.151
```

but sometimes it resolves to `ec2-54-80-18-240j.compute-1.amazonaws.com` or
`ec2-52-91-66-51.compute-1.amazonaws.com`, and errors is raised:
```
Mysql2::Error::ConnectionError: Access denied for user 'cablecrm_staging_user'@'ec2-52-91-66-51.compute-1.amazonaws.com' (using password: (redacted)))
```
or
```
A Neo4j::Core::CypherSession::ConnectionFailedError occurred in chats#show:

  Faraday::ConnectionFailed: Couldn't resolve host name
```

Best way is to use static ip address.

# Find alternatives

https://alternativeto.net

# SMS

* twilio gem

# Jira

* use smart commit messages. Email of author must match email of user
  https://confluence.atlassian.com/fisheye/using-smart-commits-960155400.html
  * `Support small devices APP-123 #comment Added a media query form small dev`
  * `Message APP-123 #done`

# Status page

* https://github.com/adamcooke/staytus on rails
* others https://github.com/topics/statuspage?l=ruby
* opensource alternative to google drive https://nextcloud.com/
  Official on ubuntu is for apache
  https://docs.nextcloud.com/server/latest/admin_manual/installation/example_ubuntu.html
  for nginx config https://docs.nextcloud.com/server/latest/admin_manual/installation/nginx.html
```
sudo su
wget https://download.nextcloud.com/server/releases/nextcloud-18.0.3.zip
unzip nextcloud-18.0.3.zip -d /usr/share/nginx
chown www-data:www-data /usr/share/nginx/nextcloud/ -R

mysql
create database nextcloud;
create user nextclouduser@localhost identified by 'your-password';
grant all privileges on nextcloud.* to nextclouduser@localhost identified by 'your-password';
flush privileges;

# copy configuration from instructions to /etc/nginx/sites-available/nextcloud
systemctl reload nginx
```
Initial configuration
```
sudo vi /usr/share/nginx/nextcloud/config/config.php
# you can see that data folder inside installation, but we will move up
  'datadirectory' => '/usr/share/nginx/nextcloud-data',
```


Backup google drive is done with downloading with
https://github.com/astrada/google-drive-ocamlfuse
or with https://takeout.google.com/settings/takeout

* pihole on nginx https://docs.pi-hole.net/guides/nginx-configuration/
```
vi /etc/pihole/setupVars.conf
```

Block porn sites by simply goind to Settings -> Blocklists -> https://raw.githubusercontent.com/chadmayfield/pihole-blocklists/master/lists/pi_blocklist_porn_top1m.list


# Time tracking

* harvest
  It contains a cli https://github.com/zenhob/hcl
  ```
  gem man hcl
  ```
  Issue with status so you need to update
  ```
  gem open hcl
  lib/hcl/commands.rb
  # change http:// to https://
  ```
* https://github.com/laughedelic/gtm is old unmaintained
* https://github.com/git-time-metric/gtm
  Install linux brew https://brew.sh/
  ```
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```
  Than install gtm
  ~~~
  brew tap git-time-metric/gtm
  brew install gtm
  ~~~

  Use in your project
  ```
  gtm init

  gtm status
  gtm report -last-month -format summary
  ```
* approximate based on git commit times
  https://github.com/kimmobrunfeldt/git-hours
  ```
  git-hours
  # yesterday  or 2021-01-01 yesterday
  git-hours --since lastweek
  ```
*  Glass https://github.com/timeglass/glass

  Download and create links in your `/usr/local/bin` and run `sudo glass install`
  <https://github.com/timeglass/glass/blob/master/docs/manual_installation.md>

  ~~~
  glass init
  glass status
  git commit -am "Work"
  git log -n 1 --show-notes=time-spent
  ~~~

  To disable adding commit message use space (not empty)

  ~~~
  cat >> /var/lib/timeglass/timeglass.json << HERE_DOC
  {
    "mbu": "1m",
    "commit_message": " ",
    "auto_push": false
  }
  HERE_DOC
  ~~~

https://github.com/veggiemonk/awesome-docker#paas
* paas like dokku captainduckduck caprover https://github.com/caprover/caprover

* tor browser can use proxy in any country, on mac edit
```
vi "/System/Volumes/Data/Users/dule/Library/Application Support/TorBrowser-Data/Tor/torrc"
# and add folowwing lines
EntryNodes {us} StrictNodes 1
ExitNodes {us} StrictNodes 1
# and restart tor browser
```

* gdpr in rails https://github.com/prey/gdpr_rails
* create image with command line generate image email
  ```
  magick -gravity center -size 100x20 -font /System/Library/Fonts/Times.ttc caption:"ruby@trk.in.rs" email.png && open email.png
  ```
* server monitor tools:  Munin, Nagios, Cacti, Hyperic
  for internet accessible: Pingdom or Uptime Robot.

