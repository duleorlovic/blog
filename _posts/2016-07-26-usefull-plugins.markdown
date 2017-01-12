---
layout: post
title: Usefull plugins, services and opensource stuff
---

# Javascript plugins

* [jbox](http://stephanwagner.me/jBox) notifications, modals, tooltips, images
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
  window.flash_notice = (message) ->
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
  ~~~

* "One minute ago" can be updated automatically in this
  [timeago](http://timeago.yarp.com/) plugin. Similarly,
  [jquery.countdown](https://github.com/hilios/jQuery.countdown) for future
  dates
* [zeroclipboard](https://github.com/zeroclipboard/zeroclipboard) to copy some
text on click
* [jsPDF](https://github.com/MrRio/jsPDF) pdf generator
  [examples](http://mrrio.github.io/jsPDF/#)

* select something
  * [tokenize](http://zellerda.com/projects/jquery/tokenize) for tags,
    preselecting items
  * nice js filter [bouncy content
  filter](http://codyhouse.co/demo/bouncy-content-filter/)
  * javascript input filter
  [http://digitalbush.com/projects/masked-input-plugin/](http://digitalbush.com/projects/masked-input-plugin/)
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

* number input field [spinner](https://github.com/vsn4ik/jquery.spinner)
* toggle checkbox [draggable
  switch](http://www.bootstrap-switch.org/examples.html)

* calendar date and time picker
  * [pickdate](http://amsul.ca/pickadate.js/)
  * [jquery ui datepicker](https://jqueryui.com/datepicker/)
    Error on chrome if you use `type=date`
    [link](http://stackoverflow.com/questions/16890376/chrome-type-date-and-jquery-ui-date-picker-clashing)
    so use `type=text`
* image file upload drag and drop crop
  * [jquery.fileapi](http://rubaxa.github.io/jquery.fileapi/) supports
    webcam, crop, dragndrop
  * [jQuery-File-Upload](https://blueimp.github.io/jQuery-File-Upload/)
  * [browser-camera](https://davidwalsh.name/browser-camera) html5 getUserMedia
* rich text editor wyswyg
  * [quill](https://quilljs.com/)

  ~~~
  <!-- app/views/layouts/application.html.erb -->
  <script src="//cdn.quilljs.com/1.0.3/quill.min.js" type="text/javascript"></script>
  <link href="//cdn.quilljs.com/1.0.3/quill.snow.css" rel="stylesheet">
  ~~~

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
    <%=raw @page.content %>
  </div>
  <%= form_for @page, url: admin_page_path(@page) do |f| %>
    <%= f.hidden_field :content, id: 'editor-content' %>
    <%= f.submit 'Update', 'data-disable-with' => 'Processing...' %>
  <% end %>
  <%= link_to 'Preview', root_path + @page.permalink %>
  <%= link_to "Back", admin_pages_path %>
  ~~~

* [fastclick](https://github.com/ftlabs/fastclick)
* another nice
[bootstrap-table](http://issues.wenzhixin.net.cn/bootstrap-table/index.html)

# Bootstrap

Bootstrap grid is mobile first, so three `.col-md-4` will be stacked until
desktop and large desktop, where it will be three equal width columns.
Bootstrap Media queries:

* xs - phones (no need for media queary since this is default)
* sm - tablets `@media (min-width: @screen-sm-min) { // 768px and up`
* md - desktops ` @media (min-width: @screen-md-min) { // 992px and up`
* lg - larger desktops `@media (min-width: @screen-lg-min) { // 1200px and up`

So when you specify for sm, you do not need to specify for md or lg.

With Bootstrap you can use their predefined media queries with `min-width`
(mobile first), for example

~~~
@media(min-width:$screen-sm){
  .alert {
    position: absolute;
    bottom: 0px;
    right: 20px;
  }
}
~~~

Bootstrap `container` has fixed width for big devices.

You can use [responsive available
classes](http://getbootstrap.com/css/#responsive-utilities-classes) to toggle
between `display: block, inline, inline-block` or to hide, for specific viewport
size:

~~~
.visibe-sm-block
.hidden-md
~~~

You can write your own to match all greater (or smaller) sizes:

~~~
# when it is max-width than it is Non-Mobile First Method
# so we use le (less or equal then)
# (min-width is Mobile First and we use hide-ge-)
/* Large Devices, Wide Screens */
@media only screen and (max-width : 1200px) {
  .hide-le-lg {
    display: none;
  }
}
/* Medium Devices, Desktops */
@media only screen and (max-width : 992px) {
  .hide-le-md {
    display: none;
  }
}
/* Small Devices, Tablets */
@media only screen and (max-width : 768px) {
  .hide-le-sm {
    display: none;
  }
}
/* Extra Small Devices, Phones */
@media only screen and (max-width : 480px) {
  .hide-le-xs {
    display: none;
  }
}
~~~

* you can center bootstrap column with offset
* you can set icons on inputs, easy with generators, just add `prepend:
'password'`

## bootstrap modal

You can use ajax, but than `render layout: false` and provide only inner html of
`<div class="modal"><div class="modal-dialog">...` ie only `.modal-content` will
be replaced. So you should use same header and footer in initial and ajax
version.

To close modal with escape key you need to add tabindex `<div class="modal"
tabindex="-1">`. On activation button you do not need `data-keyboard="true"`
since that is default.

You can use `autofocus="true"` option to focus input field when modal shows up.

For [tooltips](http://getbootstrap.com/javascript/#tooltips) you can use html
version `data-html="true" data-toggle="tooltip" title="<h2>Hi</h2>"`.

# Fontawesome

FA icons are very usefull for quick icons. Look for
[examples](http://fontawesome.io/examples/)

* color can be set with `bg-red`
* spinner `<i class="fa fa-spinner fa-spin" style="font-size:24px"></i>`


# Datatables

[datatables.net](https://datatables.net/) install by simply [select packages and
download](https://datatables.net/download/#bs-3.3.7/jq-2.2.4/jszip-2.5.0/pdfmake-0.1.18/dt-1.10.13/b-1.2.3/b-html5-1.2.3/b-print-1.2.3)
but it will be 3M in one file. I think it is better to [set up bower]({{ site.baseurl }} {% post_url 2015-04-05-common-rails-bootstrap-snippets %}#bower)
and install all dependencies that you have selected [bower
packages](https://datatables.net/download/bower)


~~~
bower install --save datatables.net datatables.net-bs \
  Stuk/jszip pdfmake \
  datatables.net-buttons datatables.net-buttons-dt datatables.net-buttons-bs \
  select2 bootstrap-datepicker
~~~

~~~
// app/assets/javascript/application.js
//= require jszip/dist/jszip.js
//= require pdfmake/build/pdfmake.js
//= require pdfmake/build/vfs_fonts.js
//= require datatables.net/js/jquery.dataTables.js
//= require datatables.net-bs/js/dataTables.bootstrap.js
//= require datatables.net-buttons/js/dataTables.buttons.js
//= require datatables.net-buttons/js/buttons.html5.js
//= require datatables.net-buttons/js/buttons.print.js
//= require select2/dist/js/select2
//= require bootstrap-datepicker/dist/js/bootstrap-datepicker

// app/assets/stylesheets/application.scss
@import "datatables.net-bs/css/dataTables.bootstrap";
@import "datatables.net-buttons-dt/css/buttons.dataTables";
@import "datatables.net-buttons-bs/css/buttons.bootstrap";
@import "select2/dist/css/select2";
@import "bootstrap-datepicker/dist/css/bootstrap-datepicker3";
~~~

~~~
$('#clicks-table').DataTable({
  dom: 'Bfrtip',
    buttons: [
      'copy', 'csv', 'excel', 'pdf', 'print'
    ]
});
~~~

* [export to csv,
pdf](https://datatables.net/extensions/buttons/examples/initialisation/export.html)
  * you can configure that only visible columns will
  [print](https://datatables.net/extensions/buttons/examples/print/columns.html)

  ~~~
  $('#clicks-table').DataTable({
    buttons: [
      {
        extend: 'pdf',
        exportOptions: {
          columns: ':not(.non-for-export),:visible',
        }
      },
    ]
  });
  ~~~

* [dom](https://datatables.net/reference/option/dom) control buttons: `l` for
lenght menu, `B` buttons, `t` table, `p` pagination, `r` processing display.
You can add class for styling page info and menu.

  ~~~
  <%# app/views/clicks/index.html.erb
  $('#clicks-table').DataTable({
    dom: 'Bfrtp<"move-up"il>',
    processing: true,
    oLanguage: {
      // loader
      sProcessing: '<i class="fa fa-spinner fa-spin" style="font-size:24px"></i> Processing...'
    },
  });
  ~~~

  ~~~
  // app/assets/stylesheets/datatable.scss
  @media(min-width:$screen-sm) {
    .move-up {
      margin-top: -50px;
      position: relative;
      z-index: -1;
    }
  }

* [multiselect with ctrl or
shift](https://datatables.net/extensions/select/examples/styling/bootstrap.html)
* [reorder](https://datatables.net/extensions/rowreorder/examples/)
* read about [usage columns](http://legacy.datatables.net/usage/columns) to
define sort based on another column. Also hide some `<td class="hidden"><%=
index %></td>` column both in css and in datatable
[hidden_columns](https://datatables.net/examples/basic_init/hidden_columns.html)

~~~
  "aoColumnDefs": [
    { "aDataSort": [1], "aTargets": [0] },
    { "bVisible": false, "bSearchable": false, "aTargets": [1] },
  ],
~~~

* [html 5 attributes](https://datatables.net/manual/data/orthogonal-data#HTML-5)
for ordering `<td data-order="<%= post.created_at.to_datetime.to_i %>" data-search="Novembar">2 Novemeber</td>`
* you can also achieve sorting if you have ajax data

  ~~~
  # app/views/users/index.json.erb
  json.my_date do
    json.display user.my_date.to_s
    json.timestamp user.my_date.to_datetime.to_i
  end
  json.amount do
    json.display humanized_money_with_symbol user.amount
    json.cents user.amount.to_s
  end

  # app/views/users/index.html.erb
  columns: [
    { data: { _: 'my_date.display', sort: 'my_date.timestamp' } },
    { data: { _: 'amount.display', sort: 'amount.cents' } },
    ]
  ~~~

* [ultimate date time
sorting](https://datatables.net/plug-ins/sorting/datetime-moment)
* [select page
length](https://datatables.net/examples/advanced_init/length_menu.html) with
[lengthMenu](https://datatables.net/reference/option/lengthMenu) option
* default client side orocessing works for less than [10 000
rows](https://datatables.net/manual/data/#Processing-modes). For greater than
100 000 you should use server side processing.
* data can be inside table ([DOM](https://datatables.net/manual/data/#DOM)
data). Note that it will be discarded if `data` or `ajax` option is used.
* [deferred loading client
side](https://datatables.net/forums/discussion/16440/best-way-to-do-deferred-loading-client-side)
is possible only with manually adding rows


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

# Free tier services

* [formspree](http://formspree.io/) sending emails in javascript
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
* [memory_profiler](https://github.com/SamSaffron/memory_profiler)

# Rails

* crowdsourcing [catarse](https://github.com/catarse/catarse) [tilt](https://github.com/crowdtilt/crowdtiltopen/) is not actually opensource, since their api should be used

# Templates

* [gentelella](https://github.com/puikinsh/gentelella)
[preview](https://colorlib.com/polygon/gentelella/index.html) [rails
version](https://github.com/iogbole/gentelella_on_rails)
* [adminLTE]( {{ site.baseurl }} {% post_url 2016-10-28-adminlte-free-template-on-rails %})


# Nice design and ui tools

* [granim.js](https://sarcadass.github.io/granim.js/examples.html)
* [DemoHouse](https://airen.github.io/DemoHouse/)
