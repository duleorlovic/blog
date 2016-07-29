---
layout: post
title:  Good rails exception notifier better than tests
tags: ruby-on-rails exception-notification
---

Do not hide your mistakes, instead, make sure that every bug is noticeable.
That way you will learn more and thinks that you did not know (that's why bug
occurs).

Bugs that I have meet are something like: user uploaded file with non asci
chars, linkedin identity text is missing graduation date, after_create hook
on model with devise cause `user.save` to return *true* but *user.errors* is
present and user is redirected to update his registration form... You can NOT
write tests for those situations. Better is to have nice notification with all
input/session/user data. When you implement this, please don't use rescue block
without notification.

Excellent gem for notification is
[exception_notification](https://github.com/smartinez87/exception_notification).
It is rack middleware and configuration is very simple.

# Basic installation

After adding to your Gemfile

~~~
cat >> Gemfile <<HERE_DOC
# error notification to EXCEPTION_RECIPIENTS emails
gem 'exception_notification'
HERE_DOC
bundle
~~~

put this in your config file:

{% highlight ruby %}
# config/application.rb
    if (receivers = Rails.application.secrets.exception_recipients).present?
      config.middleware.use(
        ExceptionNotification::Rack,
        ignore_exceptions: %w{ActiveRecord::RecordNotFound
                              AbstractController::ActionNotFound
                              ActionController::RoutingError
                              ActionController::UnknownFormat
                              },
        ignore_crawlers: %w{Googlebot bingbot linkdexbot Baiduspider YandexBot},
        ignore_if: -> (env, exception) { ["Agent-007"].include? env["HTTP_ACCEPT"] },
        email: {
          deliver_with: :deliver, # only for rails < 4.2.1
          email_prefix: "[Your App Name] ",
          sender_address: Rails.application.secrets.default_mailer_sender,
          exception_recipients: receivers.split(','),
        },
      )
    end
{% endhighlight %}

This will send email for any exception if `expception_recipients` are present.

As delivery method you can use *mandrill* or very nice *letter_opener* for development.
Set email receivers in config secrets:

{% highlight yaml %}
# config/secrets.yml
  # for all outgoing emails
  default_mailer_sender: <%= ENV["DEFAULT_MAILER_SENDER"] || "My Company <support@example.com>" %>

  # leave this empty if you do not want to enable server error notifications
  # othervise comma separated emails
  exception_recipients: <%= ENV['EXCEPTION_RECIPIENTS'] %>
{% endhighlight %}

# Javascript notification and example error pages

I would add two pages *sample-error* and *sample-error-in-javascript* just
to have some pages for test if this notification works. First create routes

{% highlight ruby %}
# config/routes.rb
get 'sample-error', to: 'pages'
get 'sample-error-in-javascript', to: 'pages'
get 'notify-javascript-error', to: 'pages#notify_javascript_error'
{% endhighlight %}

than controller method that we will use for manual notification
[ExceptionNotifier.notify_exception](https://github.com/smartinez87/exception_notification#manually-notify-of-exception).
You can pass additional information using `:data` param. Only the first argument
is required (default values you can find
[here](https://github.com/smartinez87/exception_notification/blob/df0b924e96a8f02c1fc61f88e6a1ed9c31ee43ec/lib/exception_notifier/email_notifier.rb#L162)).
For less important notification you can change subject with `email_prefix`
param. Manual notification can be simply as one line
`ExceptionNotifier.notify_exception(Exception.new('this_user_is_deactived'),
env: request.env, email_prefix: 'just to notify that', data: { current_user:
current_user });`.
Here is what I use:

{% highlight ruby %}
# app/controllers/pages_controller.rb

  def sample_error
    fail "This is sample_error on server"
  end

  def sample_error_in_javascript
    render layout: true, text: %(
      Calling manual_js_error_now
      <script>
        function manual_js_error_now_function() {
          manual_js_error_now
        }
        console.log('calling manual_js_error_now');
        manual_js_error_now_function();
        // you can also trigger error on window.onload = function() { manual_js_error_onload }
      </script>
      <br>
      <button onclick="trigger_js_error_on_click">Trigger error on click</button>
    )
  end

  def notify_javascript_error
    js_receivers = Rails.application.secrets.javascript_error_recipients
    if js_receivers.present?
      ExceptionNotifier.notify_exception(
        Exception.new("javascript error"),
        env: request.env,
        # set env to nil if you do not want sections: request and session
        # backtrace is not shown for manual notification
        # data section is always shown if exists
        # sections: %w(request message),
        exception_recipients: "#{js_receivers}".split(','),
        data: {
          current_user: current_user,
          params: params,
        }
      )
    end
    render nothing: true
  end
{% endhighlight %}

If error occurs in ajax reponse, or in some of your javascript code, we will
send another request to server to trigger javascript notification.
You can send notifications using formspree service.
There is non jQuery fallback but loaded jQuery is preferred.
You should create separate file that will be loaded in `<head>` and before
application.js so it is loaded before any other js code.

{% highlight javascript %}
// app/assets/javascripts/exception_notification.js

// This should be loaded in <head> and separatelly from application.js
// notification will not work if some error exist in this file
function exception_notification(data) {
  if (window.jQuery) {
    console.log("notify server about exception");
    $.get('/notify-javascript-error', data);
  } else {
    console.log("notify server about exception using non jQuery functions");
    var xhr = new XMLHttpRequest();
    // http://stackoverflow.com/questions/5505085/flatten-a-javascript-object-to-pass-as-querystring
    function toQueryString(obj) {
      var parts = [];
      for (var i in obj) {
          if (obj.hasOwnProperty(i)) {
              parts.push(encodeURIComponent(i) + "=" + encodeURIComponent(obj[i]));
          }
      }
      return parts.join("&");
    }
    xhr.open('GET', '/notify-javascript-error?' + toQueryString(data));
    xhr.send();
  }
  // or use formspree service with your email
  // $.ajax({
  //   url: "https://formspree.io/your@gmail.com", 
  //   method: "POST",
  //   data: data,
  //   dataType: "json"
  // });
}

// https://developer.mozilla.org/en/docs/Web/API/GlobalEventHandlers/onerror
// https://blog.getsentry.com/2016/01/04/client-javascript-reporting-window-onerror.html
window.onerror = function(errorMsg, url, lineNumber, column, errorObj) {
  if (errorMsg && sessionStorage) {
    var errorMsgs = JSON.parse(sessionStorage.getItem("errorMsgs") || "[]");
    if (errorMsgs.indexOf(errorMsg) != -1) {
      console.log("Ignore already notified error for this session and tab");
      return;
    } else {
      sessionStorage.setItem("errorMsgs", JSON.stringify(errorMsgs + [errorMsg]));
    }
  }

  flash_alert(errorMsg);

  var data = { errorMsg: errorMsg, url: url, lineNumber: lineNumber, column: column, stack: errorObj.stack, errorObj: errorObj };
  exception_notification(data);
}

// another approach is with error event listener
// window.addEventListener('error', function (e) {
//     var stack = e.error.stack;
//     var message = e.error.toString();
//     if (stack) {
//         message += '\n' + stack;
//     }
//     var xhr = new XMLHttpRequest();
//     xhr.open('POST', '/log', true);
//     xhr.send(message);
// });

// ajax error handling
// $(document).on('ajax:error', '[data-remote]', function(e, xhr, status, error) {
//   flash_alert("Please refresh the page. Server responds with: \"" +status+ " " + error + "\".");
//   exception_notification({ status: status, error: error });
// });

function flash_alert(msg) {
  if (msg.length == 0) return;
  // disable eventual popups so user can see the message
  // $('.active').removeClass('active');
  // alert(msg);
  console.log(msg);
}

// Ensures there will be no 'console is undefined' errors
// http://stackoverflow.com/questions/9725111/internet-explorer-console-is-not-defined-error
window.console = window.console || (function(){
    var c = {}; c.log = c.warn = c.debug = c.info = c.error = c.time = c.dir = c.profile = c.clear = c.exception = c.trace = c.assert = function(s){};
    return c;
})();
{% endhighlight %}

* if your javascript asset `application.js` is not included in `<head>` than you
  need to include this exception notification so it is available before
  any other js code:

  ~~~
  # app/views/layouts/application.html.erb
    <head>
      <%= javascript_include_tag :exception_notification %>
    </head>

  # config/initializers/assets.rb
  Rails.application.config.assets.precompile += %w( exception_notification.js )
  ~~~

# Custom Templates

You can also set some nice looking text with custom sections (uncomment param
`:sections`). You need to write partial in which you can access to
`@data`,`@request`... varibales.

{% highlight ruby %}
# app/views/exception_notifier/_message.text.erb
Javascript error params
<%=raw @data[:params].inspect %>
<br>
HTTP_USER_AGENT=<%= @request.env["HTTP_USER_AGENT"] %>
{% endhighlight %}

Do not forget to define javascript receivers

{% highlight yaml %}
# config/secrets.yml
  # leave this empty if you do not want to enable javascript error notification
  # othervise comma separated emails
  javascript_error_recipients: <%= ENV["JAVASCRIPT_ERROR_RECIPIENTS"] %>
{% endhighlight %}

# Render custom error pages

If you want to render error-page and page-not-found with rails you can rescue
from all StandardError exceptions.

{% highlight ruby %}
# app/controllers/application_controller.rb
  if Rails.application.secrets.exception_recipients.present?
    rescue_from StandardError do |exception|
      case exception.class
      when ActiveRecord::RecordNotFound,
          ActionController::RoutingError,
          ActionController::UnknownController,
          ActionController::MethodNotAllowed
        redirection_path = page_not_found_path
      when ActionController::InvalidAuthenticityToken
        redirection_path = new_user_session_path
      else
        redirection_path = error_page_path
      end
      ExceptionNotifier.notify_exception(exception,
                                         env: request.env,
                                         data: { current_user: current_user }
                                        )
      Rails.logger.error exception.backtrace.join("\n")
      Rails.logger.error exception.message
      flash[:alert] = exception.message if exception.message.present?
      respond_to do |format|
        format.html { redirect_to redirection_path }
        format.js do
          render text: "window.location.assign('#{redirection_path}');"
        end
        format.json { render nothing: true }
        format.text { render nothing: true }
        format.csv { render nothing: true }
      end
    end
  end
{% endhighlight %}

And you should create routes for those *page_not_found_path* and *error_page_path* and nice templates as well.

{% highlight ruby %}
# config/routes.rb
get 'error-page', to: 'pages'
get 'page-not-found', to: 'pages'
get 'javascript-required-page', to: 'pages#javascript_required_page', as: :javascript_required_page
{% endhighlight %}

Javascript required page is not needed since all browser use javascript
nowadays. But if you really want to show that notification use this in layout
file, for pages after user logs in (and search bots does not).

{% highlight ruby %}
# app/views/layout/application.html.erb
<% if params[:controller] == "requests" || params[:controller] == "contacts" %>
  <noscript>
    <meta http-equiv="refresh" content="2;url=/javascript-required-page">
  </noscript>
<% end %>
{% endhighlight %}


# Notificaiton in background jobs

If you are using background jobs like delayed job, then you should add
notification there also

{% highlight ruby %}
# config/initializers/delay_job.rb

# when you change this file, make sure that you restart delayed_job process
# bin/delayed_job stop && bin/delayed_job start
# if it configured, it will try 3 times, so probably 3 notification emails
# do not rescue in worker because no notification email will be send
# 
# http://andyatkinson.com/blog/2014/05/03/delayed-job-exception-notification-integration

# Chain delayed job's handle_failed_job method to do exception notification
Delayed::Worker.class_eval do
  def handle_failed_job_with_notification(job, error)
    handle_failed_job_without_notification(job, error)

    # only actually send mail in production
    if Rails.env.production?
      # rescue if ExceptionNotifier fails for some reason
      begin
        # ExceptionNotifier.notify_exception(error) do not use standard notification, use delayed_job partial
        env = {}
        env['exception_notifier.options'] = {
          :sections => %w(backtrace delayed_job),
          :email_prefix => "[MyApp Delayed Job Exception] ",
        }
        env['exception_notifier.exception_data'] = {:job => job}
        ::ExceptionNotifier::Notifier.exception_notification(env, error).deliver

      rescue Exception => e
        Rails.logger.error "ExceptionNotifier failed: #{e.class.name}: #{e.message}"

        e.backtrace.each do |f|
          Rails.logger.error "  #{f}"
        end

        Rails.logger.flush
      end
    end
  end

  alias_method_chain :handle_failed_job, :notification
end
{% endhighlight %}

To see log in rake task, you should use `heroku run:detached rake routes`
instead of `heroku run rake routes`
[link](https://devcenter.heroku.com/articles/one-off-dynos#running-tasks-in-background).
Exception notifications can be used there also with this
[exception_notification-rake](https://github.com/nikhaldi/exception_notification-rake).
Just add `gem 'exception_notification-rake'` to *Gemfile* and
`ExceptionNotifier::Rake.configure` to *config/secrets.yml*. Also works manual
notifications.
# Custom Error

If you need custom exception than you can try with

~~~
# raise CustomException.new bla: 'bla'
class CustomException < StandardError
  def initialize(data)
    @data = data
  end
end
~~~

# Deliver later

If you use ActiveJob than you can try to deliver later but there are some issues
<https://github.com/smartinez87/exception_notification/issues/319>
