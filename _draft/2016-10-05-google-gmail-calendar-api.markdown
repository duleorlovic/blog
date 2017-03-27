
# Youtube

## Youtube video in popup modal

It is simple iframe similar to [this
solution](http://jsfiddle.net/jeremykenedy/h8daS/1/) just with stop video after
modal is closed with Esc

~~~
<div class="modal fade" id="videoModal" tabindex="-1" role="dialog" aria-labelledby="videoModal" aria-hidden="true">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-body">
        <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
        <div>
          <iframe width="100%" height="350" src=""></iframe>
        </div>
      </div>
    </div>
  </div>
</div>

 <a href="#" class="btn btn-default" data-toggle="modal" data-target="#videoModal" data-theVideo="http://www.youtube.com/embed/loFtozxZG0s" >VIDEO</a>

<script>
$('[data-theVideo]').click(function () {
  var theModal = $(this).data( "target" ),
  videoSRC = $(this).attr( "data-theVideo" ), 
  videoSRCauto = videoSRC+"?autoplay=1" ;
  $(theModal+' iframe').attr('src', videoSRCauto);
  $(theModal+' button.close').click(function () {
      $(theModal+' iframe').attr('src', videoSRC);
  });
});

$('#videoModal').on('hidden.bs.modal', function() {
  $(this).find('iframe').attr('src', null);
});
</script>
~~~

# Google pdf preview

If you want to preview pdf or image file, you can use Google 

~~~
$('#viewerBox').attr('src','https://docs.google.com/viewer?embedded=true&url=<%= application.resume.media.url %>' );
~~~


or you can use directly [pdf.js](http://mozilla.github.io/pdf.js/getting_started/)

[yt](https://github.com/Fullscreen/yt) gem access [youtube
api](https://developers.google.com/youtube/). Creators
<http://fullscreen.github.io/yt/>. Private or non existing video url raise
exceptions so you need to `rescue Yt::Errors::NoItems,
Yt::Errors::RequestError`. Video could be public but not accessible (because of
deactived account).

# Gmail API

https://pramodbshinde.wordpress.com/2014/12/07/gmail-api-in-ruby-on-rails-a-piece-of-cake/
http://stackoverflow.com/questions/26005675/sending-html-email-using-gmail-api-in-ruby
https://developers.google.com/gmail/api/guides/sending
https://developers.google.com/api-client-library/ruby/apis/gmail/v1

Someone can break to your account simple by adding his email as recovery email.
Than he can easily change password.
