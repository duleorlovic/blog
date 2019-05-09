---
layout: post
title: Datatables
---


# Install

[datatables.net](https://datatables.net/) install by simply [select packages and
download](https://datatables.net/download/#bs-3.3.7/jq-2.2.4/jszip-2.5.0/pdfmake-0.1.18/dt-1.10.13/b-1.2.3/b-html5-1.2.3/b-print-1.2.3)
but it will be 3M in one file. I think it is better to use yarn
and install all dependencies that you have selected [bower
packages](https://datatables.net/download/bower)

```
yarn add datatables.net-bs4
```
include them in application.js
```
//= require datatables.net/js/jquery.dataTables.js
//= require datatables.net-bs4/js/dataTables.bootstrap4.js
```
and application.sass
```
@import 'datatables.net-bs4/css/dataTables.bootstrap4.css'
```

Here is old bower example
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

You can use
[jquery-datatables-rails](https://github.com/rweng/jquery-datatables-rails)
gem. Add gem to Gemfile and run `rails g jquery:datatables:install`.
If you have `Uncaught TypeError: Cannot read property 'mData' of undefined` than
you need to replace `<th colspan="3"></th>` with three `<th></th> <th></th>
<th></th>`

# Data

Initial data could be inside ([DOM](https://datatables.net/manual/data/#DOM)) or
you can use ajax call to load all items. That is client side processing and it
works for less than [10 000
rows](https://datatables.net/manual/data/#Processing-modes). For greater than
100 000 you should use server side processing. But problem is delay that occurs
(at leat on rails) when it need to render 1000 rows, so we have to use server
side processing. In that case we need to implement searching and ordering on
server.

When using ajax you need to point where [data
array](https://datatables.net/manual/ajax#Data-array-location) is in JSON object
(`dataSrc: ''` if we return array, or `dataSrc: 'data'` if we return object with
data array).
We need to point where each column is on array item. If item is object
we can use their keys `columns: [ { data: 'name' }, { data: 'price' } ]`.
If item is also array we can use their index `columns: [ { data: 0 }, { data: 1
} ]`. We can also omit `columns` definition than it will use first column, first
data from array, second column second data ...

# Server side processing

[Server side processing](https://datatables.net/manual/server-side) is enabled
with `serverSide: true` and use `data` or `ajax` options. Note
that in this case any existing data will be discarded (you can not mix).

There was a forum question to add data manually
[deferred loading client
side](https://datatables.net/forums/discussion/16440/best-way-to-do-deferred-loading-client-side)
Best option is [defer
loading](https://datatables.net/examples/server_side/defer_loading.html)
Just add option `deferLoading: <%= @users.count %>` than initially data from the
page will be shown, than on any search, reorder or page, server side request
will be made. Note that you can not use `data-order` and `data-search` on
initial page (error is like `datatables warning requested unknown parameter
'[ojbect Object]' for row 0 column 7`).
Note that when deferred loading is enabled, you can not use `stateSave: true`
option (which will remember sort and search inside localstorage).

If you have long column names and a lot of columns (for example 10), than it
could be that url is too long, you can see in console `414 (Request-URI Too
Large)`. Than you should switch to method POST and since post is used to create
items, you need to use another route `search_products_path format: :json`

Request send a lot of params:

* `"draw"=>"1"` sequence counter so we know which one is last
* `"start"=>"0", "length"=>"10"` for pagination
* `"search"=>{"value"=>"323", "regex"=>"false"}` global search for all coulmns
with searchable true
* `"order"=>{"0"=>{"column"=>"0", "dir"=>"asc"}}` for ordering
* `"columns"=>{"0"=>{"data"=>"name", "name"=>"", "searchable"=>"true",
  "orderable"=>"true", "search"=>{"value"=>"", "regex"=>"false"}}` name and
  other properties of columns

Server need to returns:

* `draw` should be echo
* `recordsTotal` total number before filtering
* `recordsFiltered` total number after filtering
* `data` array with data used in ajax option

Example

~~~
# Gemfile
# for pagination
gem 'will_paginate'
~~~

~~~
# app/assets/javascripts/products.coffee
jQuery ->
  $('#products').dataTable(
    ajax:
      url: $('#products').data('source')
      dataSrc: 'data'
      error: (xhr, message, error) ->
        flash_alert("Please refresh the page. #{error}")
    columns: [
      { data: 'name' }
      { data: 'price' }
      { data: 'description' }
    ]
    deferLoading: $('#products').data('total')
    serverSide: true
  )
~~~

~~~
# app/datatables/products_datatable.rb
# http://railscasts.com/episodes/340-datatables?autoplay=true
class ProductsDatatable
  delegate :params, :page, :per_page, to: :@view
  def initialize(view)
    @view = view
  end

  def as_json(options = {})
    {
      draw: params[:draw].to_i,
      recordsTotal: Product.count,
      recordsFiltered: products.total_entries,
      data: data,
    }
  end

  private

  def data
    products.map do |product|
      product.slice :name, :price, :description
    end
  end

  def products
    @products ||= fetch_products
  end

  def fetch_products
    products = Product.order("#{sort_column } #{sort_direction}")
    products = products.page(page).per_page(per_page)
    if params[:search][:value].present?
      # ILIKE is case insensitive LIKE
      sql = "name ILIKE :search OR description ILIKE :search"
      # you can reference association but than references is needed
      # sql += " OR customers.name ILIKE :search"
      # sql += " OR DATE_FORMAT(updated_at, '%d-%b-%Y') ILIKE :search"
      products = products
        .where(sql, search: "%#{params[:search][:value]}%")
        # .include(:customer)
        # .references(:customer)
    end
    products
  end

  def page
    params[:start].to_i / per_page + 1
  end

  def per_page
    params[:length].to_i > 0 ? params[:lenght].to_i : 10
  end

  def sort_column
    columns = %w[name price description]
    columns[params[:order]["0"][:column].to_i]
  end

  def sort_direction
    params[:order]["0"][:dir] == "desc" ? "desc" : "asc"
  end
end
~~~

~~~
# app/controllers/products_controller.rb
class ProductsController < ApplicationController
  def index
    @products = Product.all.limit 10
  end

  def search
    render json: ProductsDatatable.new(view_context)
  end
end
~~~

~~~
<%# app/views/products/index.html.erb %>
<table id="products" data-source="<%= products_path format: :json %>" data-total="<%= Product.count %>">
  <thead>
    <tr>
      <th>Name</th>
      <th>Price</th>
      <th>Description</th>
    </tr>
  </thead>

  <tbody>
    <% @products.each do |product| %>
      <tr>
        <td><%= product.name %></td>
        <td><%= product.price %></td>
        <td><%= product.description %></td>
      </tr>
    <% end %>
  </tbody>
</table>
~~~

If you have nested models, you can order by `customers.name asc` and search
`customers.name like '%a%'`, but you should use references since includes works
only with hash params: `includes(:customer).where("customers.name like
'%d%').references(:customer)`.

# Using data from datatables

Columns are defined using

* [columns](https://datatables.net/reference/option/columns) option which define
all columns by its index (zero based). Note that all columns need to be defined
* [columnDefs](https://datatables.net/reference/option/columnDefs) `targets` key
which can use: number (index or negative index), string (class name without dot
on th element) `targets: 'username'` for `<th class="username">`. Note that we
do not need to define all columns, just those that we need to set up some
property `columnDefs: [ { targets: '_all', visible: false } ]`

Columns can be selected
[column-selector](https://datatables.net/reference/type/column-selector) by:

* its index
* jQuery selector
* columns.name (names should be
[defined](https://datatables.net/reference/option/columns.name)).

# Export

[export to csv,
pdf](https://datatables.net/extensions/buttons/examples/initialisation/export.html)

* you can configure that only visible columns will
[print](https://datatables.net/extensions/buttons/examples/print/columns.html)

~~~
$('#clicks-table').DataTable({
  dom: 'B',
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

Export server side datatable
https://diogenesggblog.wordpress.com/2017/02/02/how-to-export-all-rows-from-datatables-using-server-side-ajax/

# DOM

[dom](https://datatables.net/reference/option/dom) control buttons: `l` for
lenght change menu, `B` buttons, `t` table, `p` pagination, `r` processing
display, `f` filtering.
You can add class `move-up` for info and length `il`.

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

For search you can use html data attribute `<td
data-search="Novembar">2.2.2012</td>`

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
`dule|mile`. Note that you HAVE TO disable smart search and use second param
(false).

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
or as data attribute on th element: `<th data-orderable="false">`
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

* orthogonal data can be defined using [html 5
attributes](https://datatables.net/manual/data/orthogonal-data#HTML-5) for
ordering data-sort is `<td data-order="<%= post.created_at.to_datetime.to_i
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

# Responsive

There are a lot of tools for
[responsive](https://datatables.net/extensions/responsive/examples/):

~~~
responsive: true # enable responsive styles
scrollX: true # enable horizontal scroll, problem on big screens because it does
              # not occupy 100% width. if false, than only data will scroll but
              # header will still be fixed
scrollY: "55vh" # 55% of the vertical height of the browser window. Usefull
                # since user can see footer navigation links
scrollCollapse: true # if there are not data (when filtering) than table body
                     # will collapse
autoWidth: false # disable auto calculation for column width so on big screens
                 # it occupy 100% width
~~~

Note that bootstrap tooltip won't work since it is inside `overflow-y: scroll;`
element (or `overflow: hidden`) so you need to change where it is attached
`$.fn.tooltip.Constructor.DEFAULTS.container = 'body'`. Only `<select>` element
is natively shown over borders. You can use trick `position:fixed`

Without horizontal scrolling you can hide some columns by [column priority and
expand](https://datatables.net/extensions/responsive/examples/column-control/columnPriority.html)
or you can show [details in
modal](https://datatables.net/extensions/responsive/examples/display-types/bootstrap-modal.html)
* you can use [fixed
columns](https://datatables.net/extensions/fixedcolumns/examples/initialisation/left_right_columns.html)
so they always stays fixed, but all other columns can be scrolled
* user can select which columns to see
[colvis](https://datatables.net/extensions/fixedcolumns/examples/initialisation/colvis.html)
note: retired

# Custom types

[type detection](https://datatables.net/development/type-detection) is automati
for numeric, date and string, so default sorting works fine. If you want to do
custom searching than you need to use
[$.fn.dataTableExt.afnFiltering.push](https://datatables.net/forums/discussion/36925)
or newer
[$.fn.dataTable.ext.search](https://datatables.net/manual/plug-ins/search)
Note that is used on global search. If we add manually, than we need to remove
also.This
[codepen](https://codepen.io/markmichon/pen/tmeGD) show how to add and remove
filter

~~~
~~~

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

# Ajax datatable rails

For this [gem](https://github.com/jbox-web/ajax-datatables-rails) you need to
define view_columns, by default `orderable` and `searchable` are true, and
`cond` is `:like` (`:start_with`, `:end_with`, `:string_in`,... )

```
  def view_columns
    @view_columns ||= {
      name: { source: "User.name", cond: :like }
    }
  end
```


https://www.youtube.com/watch?v=bn9arlhfaXc&t=302s

https://datatables.net/forums/discussion/32542/datatables-and-webpack
https://www.datatables.net/download/npm
https://datatables.net/manual/server-side
