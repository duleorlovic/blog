for oauth you need to create new project and enable Google+ API
and write both http https urls in both javascript and callback

~~~
# Authorized JavaScript origins
https://www.kontakt.in.rs
http://www.kontakt.in.rs
#Authorized redirect URIs
https://www.kontakt.in.rs/users/auth/google_oauth2/callback
http://www.kontakt.in.rs/users/auth/google_oauth2/callback
~~~


location places you need Google Maps Javascript API

~~~
<!-- app/views/
<script src="https://maps.googleapis.com/maps/api/js&libraries=places"></script>
~~~

~~~
<%# app/views/home/search.html.erb %>
<%= form_for @search, builder: MyFormBuilder, url: search_path, method: :get, html: { id: "filter_form" } do |f| %>
  <%= f.location_field :address, placeholder: "Address" %>
<% end %>
~~~

~~~
# app/models/search.rb
class Search

  include ActiveModel::Validations
  include ActiveModel::Conversion
  extend  ActiveModel::Naming

  attr_accessor :address, :food

  def initialize h={}
    @address = h[:address]
    @food = h[:food]
  end
end
~~~

~~~
# app/controllers/home_controller.rb
def search
  @search = Search.new params[:search]
end
~~~

~~~
# app/form_builders/my_form_builder.rb
# http://api.rubyonrails.org/classes/ActionView/Helpers/FormBuilder.html
class MyFormBuilder < ActionView::Helpers::FormBuilder

  # https://developers.google.com/maps/documentation/javascript/places
  def location_field( method, options={} )
        content = text_field(method, options) 
    if options.has_key?(:location_field_id)
      location_field_id = options[:location_field_id]
    else
      location_field_id = object_name + "_" + method.to_s
    end
    content << "
      <script>
        function initialize_google() {
          var options = {
          //  types: ['(cities)'],
            componentRestrictions: {country: 'us'}
          };
          console.log('initializing google autocomplete');
          input = document.getElementById('#{location_field_id}');
          var autocomplete = new google.maps.places.Autocomplete(input, options);
        }
        // https://developers.google.com/maps/documentation/javascript/examples/map-simple-async
        function loadScript() {
          var script = document.createElement('script');
          script.type = 'text/javascript';
          script.src = 'https://maps.googleapis.com/maps/api/js?v=3.exp&' + 'libraries=places&' +
              'callback=initialize_google';
          document.body.appendChild(script);
        }
        // http://stackoverflow.com/questions/9228958/how-to-check-if-google-maps-api-is-loaded
        if (! (typeof google === 'object' && typeof google.maps === 'object'))
        {
          loadScript();
        }
        else
        {
           // it is loaded but we need to bind on newly created object
           initialize_google();
        }
      </script>".html_safe
  end

end
~~~

