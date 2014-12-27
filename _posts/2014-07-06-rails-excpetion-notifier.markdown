
also from ajax requests

you can add new sections for email, to add your data

notification from background jobs http://andyatkinson.com/blog/2014/05/03/delayed-job-exception-notification-integration

  # custom 404
  # if you want to see this code in action in development env, comment this unless condition
  unless Rails.env.development?
    # https://github.com/smartinez87/exception_notification#manually-notify-of-exception
    # but its advisable to rescue from standard error (not all errors like memory http://robots.thoughtbot.com/rescue-standarderror-not-exception
    rescue_from StandardError do |exception|
      Rails.logger.error exception.message
      what_to_notify = "#{Rails.application.secrets.exception_notification_include}".split(',')

      if [ActiveRecord::RecordNotFound,
          ActionController::RoutingError,
          ActionController::UnknownController,
        #  ActionController::UnknownAction,
          ActionController::MethodNotAllowed].include? exception.class

        if what_to_notify.include? "page_not_found"
          ExceptionNotifier.notify_exception(exception, env: request.env)
        end
        redirect_helper(page_not_found_path)

      elsif [CanCan::AccessDenied].include? exception.class

        if what_to_notify.include? "access_denied"
          ExceptionNotifier.notify_exception(exception, env: request.env)
        end
        redirect_helper(root_url, exception.message)

      elsif [ActionController::InvalidAuthenticityToken].include? exception.class

        if what_to_notify.include? "invalid_auth"
          ExceptionNotifier.notify_exception(exception, env: request.env)
        end
        redirect_helper( new_user_session_path, 'There was a problem with your authentication. Please sign in again.')

      else

        # this exception should not be excluded
        ExceptionNotifier.notify_exception(exception, env: request.env)
        Rails.logger.error exception.backtrace.join("\n")
        redirect_helper(error_page_path)

      end
    end
  end

  private

  def redirect_helper(url, message = nil)

    respond_to do |format|
      format.html { redirect_to url, alert: message}
      format.js { render text: "window.location.assign('"+ url + "#authentication_error');" }
    end
  end
