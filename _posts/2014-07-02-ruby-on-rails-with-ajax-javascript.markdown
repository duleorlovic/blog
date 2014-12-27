---
layout: post
title:  Ruby on rails and Ajax javascript
date:   2014-07-01 14:06:33
categories: javascript ruby_on_rails
---

Do you like one page application ? Do you like Rails ? Here is their child...

Ajax with Rails
---

[Rails and javascript](http://edgeguides.rubyonrails.org/working_with_javascript_in_rails.html) can live together but some rules has to be established so you know what is going on. Anytime you can put `debugger;` anywhere in javascript code and it should stop if you have enabled some of js tools in your browser.

There are two approaches when dealing with ajax calls: server responds with final templates and javascript, or server responds with json so client side javascript is responsible for rendering.

For CRUD actions, ajax usually simulate get actions for links (new, show, edit), post actions for forms (create, update) and some patch actions for changing the state of item (destroy, activate, pause ...).

For example:

    # edit.js.erb
    $('#question-<%= @question.id %>').replaceWith(" <%= j render partial: 'questions/edit_question', { question: @question } %>");

and

    # update.js.erb
    $('#question-<%= @question.id %>').replaceWith(" <%= j render partial: 'questions/show_question', { question: @question } %>");
    

Do we need for each CRUD action to write this respond.js? No, we can simplify this by rendering respond.html templates and replacing content in javascript. If you need different rendering in index than it is in show action (for example show action has some new buttons etc), than you need to treat them separately, one partial for index and one template for show.

All data-attributes names should be lower case.

For ajax requests, if there are no coresponding respond.js, rails will fall back for rendering respond.html. We just need to stop layout rendering in this case with this line in controller:

    # app/controllers/surveys_controller.rb
    layout Proc.new { |controller| false if controller.request.xhr? }


Using this unobtrusive ajax, we do not need ID's for each element since we know what is `e.target` element so we can use `$(e.target).closest('p').replaceWith($new_elem);`.

For errors on form validations we could respond with json and some non successfully error code, for example `format.js { render json: @question.errors, status: :unprocessable_entity }`, but this is approach is more angular oriented.

Event listeners for new elements
---

If you use javascript code like `$('a[data-activate]').click(...` it will not bind to new elements created by ajax. Instead you should bind to body or another element that stays on page and and use delegation:

    $('body').on('click', 'a[data-activate]', function() {});` 
    
If you  bind on `document` element make sure it is called only once, because of turbolinks if you navigate 3 times to that page where you are calling the script, you will have 3 event listeners.

If you need something like datepicker you can bind on focus [link](http://stackoverflow.com/questions/10433154/putting-datepicker-on-dynamically-created-elements-jquery-jqueryui)

    $('body').on('focus',".datepicker_recurring_start", function(){
      $(this).datepicker();
    });


Order of binded events on the same element is by definition order. If you want to execute on click event **before** other is target that is closer, for example insted of `$(document).on('click','[data-1]',` use `$('body').on('click','[data-1]',`. If you want to execute **after** events that are binded to `$(document).on('click'` than you should wrapp all that in one function and call the code with prefered order.

Disable with "Adding..." 
----

When you work with ajax it is advisable to use rails ujs `data: { disable_with: "Adding..." }` because user can click several times on a link before server responds. This is very important for all buttons/links that are not idempotent actions (idempotent actions are ones that you can apply them as many times as you want without worries, for example abs(abs(abs(x))) ).

When you are using `<%= link_to path, data: { disable_with: "Saving"} do %> <div>..</div> <% end %>`, this will not work because whole `<div>..</div>` will be replaced with `Saving` (without div's). It is better to use `disable_with: "<div>Saving<span class='loading-icon'></span></div>".html_safe`.

Show ajax errors
---

In case of network failure or other errors, ajax links (with `[data-remote]`) do not inform the user. So it is advisable to attach event listener in case of errors (this is not error in form validations or some other response from server, it is a real error usually timeout because server did not respond).

    // error handling just for debugging
    $(document).on('ajax:error', '[data-remote]', function(e, xhr, status, error) {
      $('#flash-messages').append('Server responds with ' + status + ' ' + error);
      LOG && console.log("ajax:error [data-remote] " + status+' '+error );
    });
      


Flash messages
---

In case of some form validations error or some successul action we can send flash message to the user. In rails it is just flash[:notice]="Message text", but if we use ajax calls, we should discard flash object otherwise it will show up on next page reload [link](http://stackoverflow.com/questions/21032465/rails-doesnt-display-flash-messages-after-ajax-call). With ajax server can send error and notice message with X-Message-Flash.

    class ApplicationController < ActionController::Base
      
      after_filter :flash_to_headers                                       
      
      def flash_to_headers                                                 
        # http://stackoverflow.com/questions/21032465/rails-doesnt-display-flash-messages-after-ajax-call
        return unless request.xhr?
        # in ajax requests we return flash as X message                    
        response.headers['X-Message-Flash'] = flash.to_json unless flash.empty? 
        # discard flash object otherwise it will show up on next page load 
        flash.discard 
      end                                                                  
    end 

This approach should be coupled with writing respond.js block that do not redirect for example for update/create action `format.js { render :show, status: :ok, location: @survey, content_type: Mime::HTML }` so we have those messages in javascript. Another solution is not to discard flash and use native redirection (the same as for format.html) ie not writing forman.js as use default format.html responses. There is also `flash.now[:notice] = msg` if you want to customize it further, probably on change (PUT, PATCH) requests, but think it twice, because but not on form submit because forms are wrapped with classes errors, and usually on some index pages (where we change some of the object property and wants to stay on the same page).

GET INDEX, GET SHOW(:id), GET NEW, GET EDIT(:id), not flash messages, response is template so ajax is ok (format.js == format.html)
POST CREATE, usually flash message, respond is redirect and in ajax is ok, second request is GET(:id) (format.js == format.html)
PATCH/PUT UPDATE(:id), usually flash message, respond is redirect and in ajax is ok, second request is GET(:id) (format.js == format.html)
DELETE DELETE(:id), usually flash message, respond is redirect but in ajax is not fine, second request is stil DELETE in chrome so use `format.js { render nothing: true }` 


To summarize differences between using rails native format.js files and thiw rails unobtrusive ajax:
pros: unobtrusive ajax approach knows which link is clicked so it do not require specifix id (but it requres parameter which closest element should be replaced). This is specifically important when we generate forms for new objects (new records does not have id). You do not need to generate some random id-s to target that forms so they could be replaced with show partials.


// Used to add class 'active' to selector
// example <button data-activate=".popup"></button>
$(document).on('click','[data-activate]', function(e) {
  $( $(this).data('activate')).addClass('active');
  e.preventDefault();
  LOG && console.log("click a[data-activate]="+$(this).data('activate'));
});
// this could be toogle but it is clearer to use activate deactivate
// Used to remove class 'active' to selector
// example <button data-deactivate=".popup"></button>
$(document).on('click','[data-deactivate]', function(e) {
  $( $(this).data('deactivate')).removeClass('active');
  if ( $(this).data('prevent-default') != false)
    e.preventDefault();
  LOG && console.log("click a[data-deactivate]="+$(this).data('deactivate'));
});


debugging javascript can be easilly startet with command `debugger;` even from `format.js` response.
za processing controller/edit as JS
+
+* prvo se gleda da li postoji template edit.js
+* ako ne postoji onda bilo koji npr edit.html
+
+ajax:success event dolazi na objekat, ali vazi za sve njegove parents.
+vezati ajas:success za neki objekat unutar partiala, ali koji obavija zadati element sa remote: true (samo pazite npr <td></td> ne bi trebalo obavijati jer to kvari <tr>)
+
+$(document).ready ->
+  $(".wrapper").on("ajax:success", (e, data, status, xhr) ->
+    alert "Success"
+  ).on "ajax:error", (e, xhr, status, error) ->
+    alert "Error"
+
+Ne treba da koristimo .ajaxSuccess zato sto se on kaci samo na document, i treba obratiti paznju da je redosled agumenata razlicit kod success i error.
+
+https://github.com/rails/jquery-ujs/wiki/ajax
+
+Za updejt linkove, npr link_to activate, data: {disable_with: "Activating"} gde ne mora da se ucitava neka edit forma, najbolje je raditi zamenu posle responsa.
+Za menjanje edit i show template treba ih obaviti u div i raditi menjanje.
+Za menjanje parcijalnih delova najbolje koristiti neki data-target i koristiti isti updaejt.
+
+
+Ako se trazi #new as JS a ne postoji new.js, onda ce se renderovati new.html 

pros/cons: if there is an error respond.js you will not notice that in console, but in this approach, it will be shown as console error 

Custom js functions like select2 or autosize should be defined next to target elements. for example

   <%= f.textarea :content %>
   <% controller.js_functions << { 'textarea' => 'autosize' } %>

That way it is much clearer. when you load the partial you do not need to worry what it needs to call (like in .js files), scope is much narrower (you will not change all the textarea, because this function is called on children('textarea') of this partial).

When you are rendering back the form that has some _destoyed submodels, you should not display tham again:

dusan@trk-inovacije:~/www/rails/rails_ajax_without_respond_js

<% if f.object._destroy %>
<% end %>
<%= ubacuje &quot; &lt; i stale karakter u html escaping, \n /b ostaju isti
<%= j javacsript render escapuje /, ', " menja sa \/, \', \"

Adding new elements can be done in javascript (without server active) but then the user has to confirm that, ie submit the form.
You can add remove new elements with ajax, so there is no need for submitting.

We used both forms.

