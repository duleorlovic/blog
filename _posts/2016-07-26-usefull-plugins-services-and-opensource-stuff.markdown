---
layout: post
title: Usefull plugins, services and opensource stuff
---

# Javascript helpers

* [fastclick](https://github.com/ftlabs/fastclick)
* "One minute ago" can be updated automatically in this
  [timeago](http://timeago.yarp.com/) plugin. Similarly,
  [jquery.countdown](https://github.com/hilios/jQuery.countdown) for future
  dates
* [zeroclipboard](https://github.com/zeroclipboard/zeroclipboard) to copy some
text on click
* [jsPDF](https://github.com/MrRio/jsPDF) pdf generator
  [examples](http://mrrio.github.io/jsPDF/#)
* number input field [spinner](https://github.com/vsn4ik/jquery.spinner)
* toggle checkbox:
  * [draggable
  switch](http://www.bootstrap-switch.org/examples.html)
  * [abpetkov/switchery](https://github.com/abpetkov/switchery)
* trigger a function on scroll to an element [waypoints](http://imakewebthings.com/waypoints/)

# Alerts and notifications

## Jbox

[jbox](http://stephanwagner.me/jBox) notifications, modals, tooltips, images
and confirm windows. You need to install plugins that you use

~~~
bower install jbox --save

// app/assets/adminlte/application_adminlte.js
//= require jbox/Source/jBox
//= require jbox/Source/plugins/Notice/jBox.Notice

// app/assets/adminlte/application_adminlte.scss
@import "jbox/Source/jBox";
@import "jbox/Source/plugins/Notice/jBox.Notice";


# app/controllers/application_controller.rb
after_action :check_flash_message
def check_flash_message
  return unless request.xhr? && request.format.js? && new_layout?
  response.body += "flash_alert('#{view_context.j flash.now[:alert]}')" if flash.now[:alert].present?
  response.body += "flash_notice('#{view_context.j flash.now[:notice]}')" if flash.now[:notice].present?
end

# app/assets/javascripts/flash_messages.coffee
# https://stephanwagner.me/jBox/get_started
window.flash_alert = (message) ->
  # https://stephanwagner.me/jBox/documentation says that we need to wait for
  # ready for page load, so we use setTimeout
  setTimeout(
    ->
      new jBox 'Alert',
        autoClose: 10000
        attributes:
          x: 'right'
          y:'bottom'
        stack: false
        animation:
          open:'bounce'
          close:'fadeOut'
        content: message
        color: 'red'
    1
  )
window.flash_notice = (message) ->
  setTimeout(
    ->
      new jBox 'Notice',
        autoClose: 10000
        attributes:
          x: 'right'
          y:'bottom'
        stack: false
        animation:
          open:'bounce'
          close:'fadeOut'
        content: message
        color: 'blue'
    1
  )
~~~

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

* options like
[dropdownParent](https://select2.github.io/options.html#dropdownParent) could
help

You can use `data-(any option)` so for example you can use
`data-placeholder="Please select"` to set select2 placeholder, or
`data-tags="true"` to enable tagging (also need `multiple: true` for select to
accept multiple options)

I use this initializer `data-select2-initialize` (do not use `data-select2`
since it is used already)

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

# Calendar date and time picker

* Best option is [daterangepicker](http://www.daterangepicker.com/) it can be
used for singleDatePicker, dateTime picker
  * download with `bower install bootstrap-daterangepicker --save` (git should
  be <https://github.com/dangrossman/bootstrap-daterangepicker>)
  * autoApply (can't be used when timePicker enabled)
  * autoUpdateInput (when you want to parse user selection, custom format)
  * it can contains predefined ranges (Last 7 days)
  * note that is uses `data-daterangepicker` so use with your own
  `data-bootstrap-datarangepicker`
* [pickdate](http://amsul.ca/pickadate.js/)
* [jquery ui datepicker](https://jqueryui.com/datepicker/)
  Error on chrome if you use `type=date`
  [link](http://stackoverflow.com/questions/16890376/chrome-type-date-and-jquery-ui-date-picker-clashing)
  so use `type=text`
* [bootstrap-datepicker](https://bootstrap-datepicker.readthedocs.io/en/latest/index.html)

# Image file upload drag and drop crop

* [jquery.fileapi](http://rubaxa.github.io/jquery.fileapi/) supports
  webcam, crop, dragndrop
* [jQuery-File-Upload](https://blueimp.github.io/jQuery-File-Upload/)
* [browser-camera](https://davidwalsh.name/browser-camera) html5 getUserMedia

# Editors

## Wiki style

<http://wagn.org> is whole project

## Rich text editors wyswyg

* [fireEdit](https://github.com/coltaemanuela/FireEdit)

* quill

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

FA icons are very usefull for quick icons. Look for
[examples](http://fontawesome.io/examples/)

* color can be set with `bg-red`
* spinner `<i class="fa fa-spinner fa-spin" style="font-size:24px"></i>`

# Images

* free high resolution <https://unsplash.com/>
* free online sample images https://unsplash.it/images
* sample placeholder https://placeholder.com/

# Graphs

* [graphs
gallery](https://github.com/mbostock/d3/wiki/Gallery) and
[http://bl.ocks.org/mbostock](http://bl.ocks.org/mbostock)
[slider](http://square.github.io/crossfilter/)
[people](http://www.findtheconversation.com/concept-map/#population)
[parallel](http://exposedata.com/parallel/) and the best is
[grafana](http://play.grafana.org/)
* [d3 for charts](http://plottablejs.org/examples/)
* [chartjs](http://www.chartjs.org/)
* arrows and grouping
  <http://marvl.infotech.monash.edu/webcola/examples/smallgroups.html>

* smooth transitions between different svg
[flubber](https://github.com/veltman/flubber)

# Email services

* [formspree](http://formspree.io/) sending emails in javascript

# User support services

* [user deck](https://userdeck.com)

# Visitor analytics

* <https://www.hotjar.com/tour>
* <https://www.fullstory.com/features>

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

* crowdsourcing [catarse](https://github.com/catarse/catarse) [tilt](https://github.com/crowdtilt/crowdtiltopen/) is not actually opensource, since their api should be used
* opensource market peer-to-peer marketplace [sharetribe](https://github.com/sharetribe/sharetribe)

# Templates

* [gentelella](https://github.com/puikinsh/gentelella)
[preview](https://colorlib.com/polygon/gentelella/index.html) [rails
version](https://github.com/iogbole/gentelella_on_rails)
* [adminLTE]( {{ site.baseurl }} {% post_url 2016-10-28-adminlte-free-template-on-rails %})
* free templates <https://freehtml5.co>
[booster](https://freehtml5.co/booster-free-html5-bootstrap-template/) or
[elate](https://freehtml5.co/demos/elate/)
* [four images](http://barbajs.org/demo/grid/index.html)
* [trophy jekyll](http://thomasvaeth.com/trophy-jekyll/) nice template for jekyll


# Nice design and ui tools

* [granim.js](https://sarcadass.github.io/granim.js/examples.html)
* [DemoHouse](https://airen.github.io/DemoHouse/)

# Fonts and icons

* [ico Moon Free](https://github.com/Keyamoon/IcoMoon-Free)
<https://icomoon.io/#preview-free>
* [simple-line-icons](http://simplelineicons.com/)
* [bootstrap glyphs](http://getbootstrap.com/components/)

* set specific font styles <http://www.cssfontstack.com/Arial-Narrow>


# Web framework

<https://github.com/kemalyst/kemalyst> based on <https://crystal-lang.org/>

# Static analysis

* [shellckeck](https://github.com/koalaman/shellcheck) static analysis tool for
shell scripts
