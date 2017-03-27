---
layout: post
title: Datatables
---


# Install

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

* default client side orocessing works for less than [10 000
rows](https://datatables.net/manual/data/#Processing-modes). For greater than
100 000 you should use server side processing.

# Export

[export to csv,
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

If you have special simbols, you can try to enable `bom: true` option.

# Core

Columns can be
[column-selector](https://datatables.net/reference/type/column-selector) by its
index (zero based), jQuery selector or columns.name (names should be
[defined](https://datatables.net/reference/option/columns.name)). You can define
name targeting using
[columnDefs](https://datatables.net/reference/option/columnDefs) targets which
use class name without dot `targets: 'username'` for `<th class="username">` so
we can continue with jQuery selector.

~~~

~~~

# DOM

[dom](https://datatables.net/reference/option/dom) control buttons: `l` for
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
~~~

# Seach (or filter is used for filtering out)

For search you can use html data attribute `<td data-search="Novembar">2.2.2012</td>`

* when using `data-search` then original text is not used, so it is better to
merge original text to `data-search`
* when you are using `<a href="some-text">` that `some-text` will be matched
also
* default [search(input, regex,
smart)](https://datatables.net/reference/api/search()) is `regex=false,
smart=true`, that means any order of strings and matching substring. For example
`my name` is matched with `my n a m m e e e e`. You can also use double quotes
`"my name"`.
* it is usefull to add param search term and using regex, for example
`dule|mile`

~~~
  table.search("<%= params[:search_term] %>", true, false).draw();
~~~

or you can search specific columns.


## Select column

[link](https://datatables.net/reference/api/column().search())

~~~
  table.columns( '.select-filter' ).every( function () {
    var that = this;

    // Create the select list and search operation
    var select = $('<select />')
      .appendTo(
        this.footer()
      )
      .on( 'change', function () {
        that
          .search( $(this).val() )
          .draw();
      } );

    // Get the search data for the first column and add to the select list
    this
      .cache( 'search' )
      .sort()
      .unique()
      .each( function ( d ) {
          select.append( $('<option value="'+d+'">'+d+'</option>') );
      });
  });
~~~

# Order (sort)

* disable ordering with
[orderable](https://datatables.net/reference/option/columns.orderable) property
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
for ordering data-sort is `<td data-order="<%= post.created_at.to_datetime.to_i
%>">2 Novemeber</td>`.
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
  json.user do
    json.display user.name
    json.username user.username
  end

  # app/views/users/index.html.erb
  columns: [
    { data: { _: 'my_date.display', sort: 'my_date.timestamp' } },
    { data: { _: 'amount.display', sort: 'amount.cents' } },
    { data: { _: 'user.display', filter: 'user.username' } },
    ]
  ~~~

* [ultimate date time
sorting](https://datatables.net/plug-ins/sorting/datetime-moment) [datetime
input](https://editor.datatables.net/examples/dates/datetime.html)
* [select page
length](https://datatables.net/examples/advanced_init/length_menu.html) with
[lengthMenu](https://datatables.net/reference/option/lengthMenu) option

# Ajax

* data can be inside table ([DOM](https://datatables.net/manual/data/#DOM)
data). Note that it will be discarded if `data` or `ajax` option is used.
[deferred loading client
side](https://datatables.net/forums/discussion/16440/best-way-to-do-deferred-loading-client-side)
is possible only with manually adding rows

# Responsive

There are a lot of tools for
[responsive](https://datatables.net/extensions/responsive/examples/):

* [column priority and
expand](https://datatables.net/extensions/responsive/examples/column-control/columnPriority.html) or you can show [details in modal](https://datatables.net/extensions/responsive/examples/display-types/bootstrap-modal.html)

# Tips

* [multiselect with ctrl or
shift](https://datatables.net/extensions/select/examples/styling/bootstrap.html)
* [drag and drop
  reorder](https://datatables.net/extensions/rowreorder/examples/)
* when you have a lot of columns (x-axis scrollbar is shown) than you can use
  [fixedColumns](https://datatables.net/extensions/fixedcolumns/examples/) which
  are always shown

* another nice
[bootstrap-table](http://issues.wenzhixin.net.cn/bootstrap-table/index.html)
