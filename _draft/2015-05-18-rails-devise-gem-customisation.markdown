
http://sourcey.com/rails-4-omniauth-using-devise-with-twitter-facebook-and-linkedin/

If you want to override some devise controllers, you should write your own controllers that inherits from devise controllers.

For example when user click on confirmation link, they can be logged in automatically:
~~~
# app/controllers/confirmations_controller.rb

# GET /resource/confirmation?confirmation_token=abcdef
  def show
    super do
      sign_in resource if resource.errors.empty?
    end
  end

  def send_first_deal
    flash[:notice] = "First deal sent."
    redirect_to refer_path
  end

# config/routes.rb
  devise_for :users, controllers: { confirmations: 'confirmations' }
  devise_scope :user do
    # devise looks for user_root after confirmation and signedin
    get 'user_root', to: 'confirmations#send_first_deal'
  end

~~~

