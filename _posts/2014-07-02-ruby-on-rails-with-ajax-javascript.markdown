---
layout: post
title:  Ruby on rails and Ajax javascript
date:   2014-07-01 14:06:33
categories: javascript ruby_on_rails
---

Ajax with Rails
===

[Rails and javascript](http://edgeguides.rubyonrails.org/working_with_javascript_in_rails.html) can live together but some rules has to be established so we know what is going on. Anytime you can put `debugger;` anywhere in javascript code and it should stop if you have enabled some of js tools in your browser.

There are two approaches when dealing with ajax calls. Server can respond with json so client job is to parse and render the message, or we can respond with html partials.

#Respond with json

When there are not a lot of design changes on pages, and when style of a forms are unified for whole site, you can use *data-* attributes to activate/respond to ajax calls. For example is you 

{% highlight javascript %}
////////////////////////////////
// ajax success and error events for links
////////////////////////////////


  $(document).on('ajax:success', 'a[data-function]', function(e, data, status, xhr){

    var intended_function = $(this).data('function');
    var parameter = $(this).data('parameter');
    var $new_elem = $(data);

    switch( intended_function ) {
      // replace closest
      case "replace":
       $(e.target).closest(parameter).replaceWith($new_elem); break;
      // hide closest
      case "hide":
        $(e.target).closest(parameter).css('background-color','yellow').fadeOut(400); break;
      // add before closest
      case "before":
        $(e.target).closest(parameter).before($new_elem); break;
    };

    var $flash_messages = xhr.getResponseHeader('X-Message-Flash');
    if ($flash_messages)
      parseXMessageFlash($.parseJSON( $flash_messages ));

    var $js_functions = $.parseJSON(xhr.getResponseHeader('X-Message-JS-Functions'));
    if ($js_functions)
      parseXMessageJSFunctions($new_elem, $js_functions );
    //  e.stopPropagation();
    //  e.preventDefault();
    LOG && console.log("ajax:success a[data-function]="+intended_function);
  });

{% endhighlight %}

delovi nekog objekta mogu da se menjaju, show_title.html, edit_title.html koji sadrze <div id='holder-show-title'>, pa onda edit_title.js radi $('#holder-show-title').replaceWith('<%= j render 'edit_title' %>');

ti delovi mogu da idu na isti update, ako se koristi hidden field ali onda mora i renderuje posebno

kada se u .js template poziva render "partial", mora se staviti oznaka, render partial: "partial"


ukoliko imate neki event.listener na objekat koji se potpuno zamenio, onda nece raditi. treba postaviti even listener na neki parent i delegirati ga na taj element, npr.
$('.wrap').on('click','a[data-toggle=collapse]', function() {
  $(this).siblings('posto na novim objektima nisu vezani event listeneri


jquery radi quoting npr https://github.com/jquery/jquery/blob/90b43de21296cf59af7dd37c49c1a9a4f8483f68/src/sizzle/dist/sizzle.js#L95

<%= ubacuje &quot; &lt; i stale karakter u html escaping, \n /b ostaju isti
<%= j javacsript render escapuje /, ', " menja sa \/, \', \"


ako definisemo respond_to onda radi format.js bez layouta (ako nadje ili ako ne nadje onda radi respond with html as HTML).
ako ne definisemo respond_to onda trazi js i mora biti bez novi redova, 

Form errors.

Form use data-model to determine how to wrap error div. We need to find input[name='model']['title'].

I putted all event listiners inside a function ready(), and call that function when $(document).ready(ready);
If I used $(document).on('page:load', ready); I will have multiple (identical) events attached to same object. That is another reason why we should bind all events to document.

Since we use remote-true, we should add data: { disable_with: "Adding" } for all links and button that are not idempotent actions (idempotent actions are ones that you can apply them as many times as  you want without worries for example abs(abs(abs(x))) ).


flash messages
http://stackoverflow.com/questions/21032465/rails-doesnt-display-flash-messages-after-ajax-call
we should discard flash object on ajax request otherwise it will show up on next page reload


we send alert and notice with X-Message-Alert.

flash are rendered usually on change (PUT, PATCH) requests, but not on form submit because forms are wrapped with classes errors, and usually on some index pages (where we change some of the object property and wants to stay on the same page).

You can not bind event on 'load' for new elements, but you can 
http://stackoverflow.com/questions/6781661/jquery-how-to-call-a-jquery-plugin-function-on-an-element-that-hasnt-yet-been

If you need something like datepicker you can bind on focus
http://stackoverflow.com/questions/10433154/putting-datepicker-on-dynamically-created-elements-jquery-jqueryui

    $('body').on('focus',".datepicker_recurring_start", function(){
      $(this).datepicker();
    });

execute custom function, data-function=select2(),autosize()

when you are using link_to path, data: { disable_with: "Saving"} do <div>..</div> end, this will not work because whole <div>..</div> will be replaced with saving

for before use that element, for example link_to path, data: { before: "a"}

Adding new elements can be done in javascript (without server active) but then the user has to confirm that, ie submit the form.
You can add remove new elements with ajax, so there is no need for submitting.

We used both forms.


All data-attributes names should be lower case.
Use X-Messages for something that is known only on server side, like flash message, custom functions

ajax form success data-replace should be set on form on a submit button


This ajax approach have advantage that rails respond, and javascript will render where the data attribute tels. respond.js does not know on which link it was clicked, and we have to pass additional parameters.
We do not need id-s for collection of objects. You can have more than one forms for new records (new records does not have id). You do not need to generate some random id-s to target that forms so they could be replaced with show partials.
Since we are using the same respond.js file for success and error, we need to have if statement there, and use flash.now[:alert]
Destroy action do not know what to hide, form or show partial.

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

<% if f.object._destroy %>
<% end %>
