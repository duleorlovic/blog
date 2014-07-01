---
layout: post
title:  Ruby on rails and Ajax javascript
date:   2014-07-01 14:06:33
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

ds


{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

