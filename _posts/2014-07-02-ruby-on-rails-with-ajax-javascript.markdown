---
layout: post
title:  Ruby on rails and Ajax javascript
date:   2014-07-02
categories: javascript ruby_on_rails
---


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
