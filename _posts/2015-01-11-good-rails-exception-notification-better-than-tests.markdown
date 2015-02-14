---
layout: post
title:  Good rails exception notifier better that tests
categories: ruby-on-rails exception-notification
---

Do not hide your mistackes, instead, make sure that every bug is noticeable. That way you will learn more and thinks that you did not know (that's why bug occurs).

Bugs that I have meet are something like: user uploaded file with non asci chars, linkedin identity text is missing graduation date, after_create hook on model with devise cause user.save to return true but user.errors is present and user is redirected to update his registration form... You cant write tests for those situations. Better is to have nice notification with all input/session/user data. So please never use rescue block without notification.

Excellent gem for notification is [exception_notification](https://github.com/smartinez87/exception_notification). It is rack middleware and configuration is very simple. After adding `gem exception_notification` put this in your config file:

{% highlight ruby %}
# config/application.rb
if Rails.application.secrets.exception_recipients.present?
  Whatever::Application.config.middleware.use ExceptionNotification::Rack,
    :email => {
      :email_prefix => "[Whatever] ",
      :sender_address => %{"notifier" <notifier@example.com>},
      :exception_recipients => "#{Rails.application.secrets.exception_recipients}".split(','),
      :delivery_method => :smtp,
    }
end
{% endhighlight %}

This will send email for any exception that occurs in production.

Set email receivers in config secrets:

{% highlight yaml %}
# config/secrets.yml
# leave this empty if you do not want to enable server error notifications, othervise comma separated emails
exception_recipients: <%= ENV['EXCEPTION_RECIPIENTS'] %>
# leave this empty if you do not want to enable javascript error notifications, othervise comma separated emails
javascript_error_recipients: <%= ENV["JAVASCRIPT_ERROR_RECIPIENTS"] %>
{% endhighlight %}

Second variable is for javascript errors. If there is an error in ajax reponse, or in some of your javascript code, you can send another request to server to send notification. In your javascript file you should have something like this

{% highlight javascript %}
// catch javascript errors
window.onerror = function(errorMsg, url, lineNumber, column, errorObj) {
  if (typeof flash_alert == 'function')
    flash_alert(errorMsg);
  else
    alert(errorMsg); // we use alert if error occurs in application.js so flash_alert is not defined

  // notify server for this error
  $.get('/notify-javascript-error', { errorMsg: errorMsg, url: url, lineNumber: lineNumber, column: column, errorObj: errorObj});
}
// ajax error handling
$(document).on('ajax:error', '[data-remote]', function(e, xhr, status, error) {
  //disable eventual popups so user can see the message
  $('.active').removeClass('active');
  flash_alert("Please refresh the page. Server responds with: \"" +status+ " " + error + "\".");
  // notify server for this error
  $.get('/notify-javascript-error', { status: status, error: error });
});
{% endhighlight %}

Create route for *notify-javascript-error* and use manual notification [ExceptionNotifier.notify_exception](https://github.com/smartinez87/exception_notification#manually-notify-of-exception).

{% highlight ruby %}
# app/controllers/pages_controller.rb
class PagesController < ApplicationController
  def notify_javascript_error
    if Rails.application.secrets.javascript_error_recipients.present?
      ExceptionNotifier.notify_exception(
        Exception.new("javascript error"),
        :env => request.env,
        # set env to nill if you do not want sections: request and session (backtrace is not shown for manual notification, data section is always shown if exists)
        :sections => %w(request message),
        :exception_recipients => "#{Rails.application.secrets.javascript_error_recipients}".split(','), # split method should be applied to string, but this could be nil, so we use "#{nil}"
        :data => { 
          current_user: current_user,
          params: params,
        }
      )
    end
    render nothing: true
  end
end
{% endhighlight %}

As you can see, you can add additional information using `:data` param. Only the first argument is required (default your can find [here](https://github.com/smartinez87/exception_notification/blob/df0b924e96a8f02c1fc61f88e6a1ed9c31ee43ec/lib/exception_notifier/email_notifier.rb#L162)). Probably, for less important notification you can change subject with `email_prefix` param. You can also set some nice looking text with custom sections. Look at param `:sections`. You need to write partial (in which you can access to `@data`,`@request`... varibales).

{% highlight ruby %}
# app/views/exception_notifier/_message.text.erb
Javascript error params
<%=raw @data[:params].inspect %>
<br>
HTTP_USER_AGENT=<%= @request.env["HTTP_USER_AGENT"] %>
{% endhighlight %}

If you want to render error-page and page-not-found with rails you can rescue from all StandardError exceptions.

{% highlight ruby %}
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  # custom 404
  if Rails.application.secrets.exception_recipients.present?
    # but its advisable to rescue from standard error (not all errors like memory http://robots.thoughtbot.com/rescue-standarderror-not-exception
    rescue_from StandardError do |exception|
      if [ActiveRecord::RecordNotFound,
          ActionController::RoutingError,
          ActionController::UnknownController,
        #  ActionController::UnknownAction,
          ActionController::MethodNotAllowed].include? exception.class
        redirection_path = page_not_found_path
      elsif [ActionController::InvalidAuthenticityToken].include? exception.class
        redirection_path = new_user_session_path
      else
        redirection_path = error_page_path
      end
      ExceptionNotifier.notify_exception(exception, env: request.env, data: {current_user: current_user} )
      Rails.logger.error exception.backtrace.join("\n")
      Rails.logger.error exception.message
      respond_to do |format|
        format.html { redirect_to redirection_path}
        format.js { render text: "window.location.assign('"+ redirection_path + "');" }
        format.json { render nothing: true } # sometimes json api calls could raise exception
      end
    end
  end
end
{% endhighlight %}


And you should create routes for those *page_not_found_path* and *error_page_path* and nice templates as well. I would add two more pages *example-error* and *example-javascript-error* just to have some pages for test if this notification works.

{% highlight ruby %}
get 'example-error', to: 'pages#example_error'
get 'example-error-in-javascript', to: 'pages#example_error_in_javascript'
get 'error-page', to: 'pages#error_page', as: :error_page
get 'page-not-found', to: 'pages#page_not_found', as: :page_not_found
get 'notify-javascript-error', to: 'pages#notify_javascript_error', as: :notify_javascript_error
get 'javascript-required-page', to: 'pages#javascript_required_page', as: :javascript_required_page
{% endhighlight %}


You can see that there is also `javascript-required-page` where people is redirected when js is not enabled. This happens very rare. Put this in layout file, for pages after user login (and search boots not).


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
