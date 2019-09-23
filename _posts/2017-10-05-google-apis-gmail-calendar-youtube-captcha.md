---
layout: post
---

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

# Gmail shortcuts

* ? to open keyboard shortcut help
* Ctrl+Enter to send email
* j and k to older and newer conversation
* o to open conversation, u to back to threadlist
* g+a go to all mail, g+t to sent emails, g+i to inbox
* c to compose
* / to search

# ReCaptcha

Register on <https://www.google.com/recaptcha> and you can use in in javascript
`<div class="g-recaptcha" data-sitekey="6LdrgFQUAAAAALvyQvT3fpoagmnb-ik9f73Y0Zaz"></div>`
but on rails and devise follow
<https://github.com/plataformatec/devise/wiki/How-To:-Use-Recaptcha-with-Devise>
and gem <https://github.com/ambethia/recaptcha>

Example is https://github.com/duleorlovic/premesti.se

```
# Gemfile
# captcha on contact form
gem 'recaptcha'

# config/locales/en.yml
  recaptcha:
    errors:
      verification_failed: reCAPTCHA verification failed, please try again.

# app/form_objects/contact_form.rb
class ContactForm
  include ActiveModel::Model
  include Recaptcha::Adapters::ControllerMethods

  attr_accessor :email, :text, :g_recaptcha_response, :current_user, :remote_ip
  validates_format_of :email, with: Devise.email_regexp
  validates :text, :email, presence: true

  def save
    verify_recaptcha model: self, response: g_recaptcha_response
    return false if errors.present?
    return false unless valid?

    _send_notification
    true
  end

  def request
    OpenStruct.new remote_ip: remote_ip
  end

  def env
    nil
  end

  def _send_notification
    Notify.message("contact_form #{email} @ #{Time.zone.now}", email, text, remote_ip, current_user)
  end
end

# app/controllers/pages_controller.rb
  def contact
    @contact_form = ContactForm.new(
      email: current_user&.email
    )
  end

  def submit_contact
    @contact_form = ContactForm.new(
      email: current_user&.email || params[:contact_form][:email],
      text: params[:contact_form][:text],
      g_recaptcha_response: params['g-recaptcha-response'],
      current_user: current_user,
      remote_ip: request.remote_ip,
    )
    if @contact_form.save
      flash.now[:notice] = t('contact_thanks')
      contact
    else
      flash.now[:alert] = @contact_form.errors.full_messages.join(', ')
    end
    render :contact
  end

# app/assets/javascripts/window_functions.coffee
window.enableRecaptchaButtons = (e) ->
  console.log 'enableRecaptchaButtons'
  $('[data-recaptcha-button]').prop('disabled', false)

# config/initializers/recaptcha.rb
Recaptcha.configure do |config|
  config.site_key = Rails.application.secrets.google_recaptcha_site_key
  config.secret_key = Rails.application.secrets.google_recaptcha_secret_key
end

# config/secrets.yml
  # Google recaptcha
  google_recaptcha_site_key: <%= ENV['GOOGLE_RECAPTCHA_SITE_KEY'] %>
  google_recaptcha_secret_key: <%= ENV['GOOGLE_RECAPTCHA_SECRET_KEY'] %>
```

Use `Checkbox` v2 (`Invisible` is similar, just you need to trigger using js and
use callback to receive results). V3 is for scoring.

It is not possible to edit configuration using api
https://stackoverflow.com/questions/38197959/how-to-add-redirect-uris-programmatically
so use selenium script to update redirect URIs. You can use 50 domain with one
key.

I had an error when using STMP
~~~
  555 5.5.2 Syntax error. u25sm6671019wml.4 - gsmtp
  /home/orlovic/.rbenv/versions/2.6.3/lib/ruby/2.6.0/net/smtp.rb:969:in `check_response'
~~~
and the issue was on google's servers and lasts for one and half hours.
