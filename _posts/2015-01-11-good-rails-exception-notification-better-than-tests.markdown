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
It is rack middleware and configuration is very simple. After adding to your
Gemfile

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
        email: {
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
  # leave this empty if you do not want to enable server error notifications
  # othervise comma separated emails
  exception_recipients: <%= ENV['EXCEPTION_RECIPIENTS'] %>
  # leave this empty if you do not want to enable javascript error notification
  # othervise comma separated emails
  javascript_error_recipients: <%= ENV["JAVASCRIPT_ERROR_RECIPIENTS"] %>
{% endhighlight %}

Second variable is for javascript errors. If error occurs in ajax reponse, or
in some of your javascript code, we will send another request to server to
trigger notification. In your main javascript file you should have something
like this

{% highlight javascript %}
// app/assets/javascripts/exception_notification.js

// Ensures there will be no 'console is undefined' errors
// http://stackoverflow.com/questions/9725111/internet-explorer-console-is-not-defined-error
window.console = window.console || (function(){
    var c = {}; c.log = c.warn = c.debug = c.info = c.error = c.time = c.dir = c.profile = c.clear = c.exception = c.trace = c.assert = function(s){};
    return c;
})();

// catch javascript errors with https://developer.mozilla.org/en/docs/Web/API/GlobalEventHandlers/onerror
window.onerror = function(errorMsg, url, lineNumber, column, errorObj) {
  if (typeof(flash_alert) == 'function')
    flash_alert(errorMsg);
  else
    alert(errorMsg); // we use alert if error occurs in application.js so flash_alert is not defined

  // notify server for this error
  $.get('/notify-javascript-error', { errorMsg: errorMsg, url: url, lineNumber: lineNumber, column: column, stack: errorObj.stack, errorObj: errorObj});
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
$(document).on('ajax:error', '[data-remote]', function(e, xhr, status, error) {
  // disable eventual popups so user can see the message
  $('.active').removeClass('active');
  flash_alert("Please refresh the page. Server responds with: \"" +status+ " " + error + "\".");
  // notify server for this error
  $.get('/notify-javascript-error', { status: status, error: error });
});

function flash_alert(msg) {
  if (msg.length == 0) return;
  alert(msg);
}
{% endhighlight %}

Create route `get 'notify-javascript-error', to:
'pages#notify_javascript_error'` in your *config/routes.rb* and we will use
manual notification
[ExceptionNotifier.notify_exception](https://github.com/smartinez87/exception_notification#manually-notify-of-exception).
You can pass additional information using `:data` param. Only the first
argument is required (default your can find
[here](https://github.com/smartinez87/exception_notification/blob/df0b924e96a8f02c1fc61f88e6a1ed9c31ee43ec/lib/exception_notifier/email_notifier.rb#L162)).
For less important notification you can change subject with `email_prefix`
param. Manual notification can be simply as one line
`ExceptionNotifier.notify_exception(Exception.new('this_user_is_deactived'),
env: request.env, email_prefix: 'just to notify that', data: { current_user:
current_user });`. Here is what we use for javascript notification:

{% highlight ruby %}
# app/controllers/pages_controller.rb
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


And you should create routes for those *page_not_found_path* and *error_page_path* and nice templates as well. I would add two more pages *sample-error* and *sample-error-in-javascript* just to have some pages for test if this notification works.

{% highlight ruby %}
get 'sample-error', to: 'pages'
get 'sample-error-in-javascript', to: 'pages'
get 'error-page', to: 'pages'
get 'page-not-found', to: 'pages'
get 'notify-javascript-error', to: 'pages#notify_javascript_error', as: :notify_javascript_error
get 'javascript-required-page', to: 'pages#javascript_required_page', as: :javascript_required_page
{% endhighlight %}


You can see that there is also `javascript-required-page` where people is redirected when js is not enabled. This happens very rare. Put this in layout file, for pages after user login (and search bots not).


{% highlight ruby %}
# app/views/layout/application.html.erb
<% if params[:controller] == "requests" || params[:controller] == "contacts" %>
  <noscript>
    <meta http-equiv="refresh" content="2;url=/javascript-required-page">
  </noscript>
<% end %>
{% endhighlight %}

you are using background jobs like delayed job, then you should add notification there also

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
