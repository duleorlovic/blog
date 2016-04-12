~~~
layout: post
title: Google API snippets
categories: google auth location-suggestion maps
~~~

# Google Auth

For oauth you need to create new project and enable Google+ API
and write both **http** and **https** urls in both *javascript* and *callback*

~~~
# Authorized JavaScript origins
https://www.kontakt.in.rs
http://www.kontakt.in.rs
#Authorized redirect URIs
https://www.kontakt.in.rs/users/auth/google_oauth2/callback
http://www.kontakt.in.rs/users/auth/google_oauth2/callback
~~~


# Google location

For location places you need Google Maps Javascript API. This is example usage with custom form builder.

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

  # http://api.rubyonrails.org/classes/ActiveModel/Model.html
  include ActiveModel::Model

  attr_accessor :address, :food

  def initialize( attributtes = {} )
    super
    @address ||= "New York"
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
    if options[:text_area] == true
      content = text_area(method, options) 
    else
      content = text_field(method, options) 
    end
    if options[:location_field_id].present?
      location_field_id = options[:location_field_id]
    else
      location_field_id = object_name + "_" + method.to_s
    end
    content << "
      <script>
        function initialize_google() {
          var options = {
          //  types: ['(cities)'],
            componentRestrictions: {country: '#{ options[:country] || 'us' }'}
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

# Maps

Map with markers

~~~
# app/controllers/home_controller.rb
  def search
    @search = Search.new params[:search]
    @target_location = Geocoder.search(@search.address)
    @users = User.chef.near(@search.address)
  end
~~~

When creating a map, you need to provide zoom option.

~~~
<%# app/views/home/search.html.erb %>
<div id="all_map"></div>
<ul>
  <% @users.each_with_index do |user,i| %>
    <li data-marker-array-index="<%= i %>">
      <h4><%= user.name %></h4>
    </li>
  <% end %>
</ul>

<script>
  var map = new google.maps.Map(document.getElementById('all_map'), {
    zoom: 4,
    <% center = { lat: (@target_location.try(:latitude)||25), lng: (@target_location.try(:longitude)||131) } %>
    center: <%=raw center.to_json %>,
    });
  var chefs = <%=raw @users.select {|u| u.geocoded? }.map {|u| { name: u.name, position: { lat: u.latitude, lng: u.longitude } }}.to_json %>;
  var markersArray = [];
  for(var i=0; i < chefs.length; i++ ) {
    var marker = new google.maps.Marker({
        title: chefs[i].name,
        map: map,
        position: chefs[i].position,
      });
    markersArray.push(marker);
  }
  bounds = new google.maps.LatLngBounds();
  for (var i = 0; i < markersArray.length; i++) {
    marker = markersArray[i];
    bounds.extend(marker.position);
  }
  if(markersArray.length > 0){
    map.fitBounds(bounds);
    var mapzoom = map.getZoom();
    if(mapzoom > 16){
      map.setZoom(16);
    };

  }
  $('[data-marker-array-index]').hover(function() {
      console.log("hover");
      var marker =markersArray[ parseInt($(this).data('marker-array-index'))]
      marker.setAnimation(google.maps.Animation.BOUNCE);
    }, function() {
      console.log("out hover");
      var marker =markersArray[ parseInt($(this).data('marker-array-index'))]
      marker.setAnimation(null);
    });
</script>
~~~

* static map with custom icon (must be http:// not https://)

  ~~~
  img(border='0', ng-src='//maps.googleapis.com/maps/api/staticmap?center=\
    {{ location.latitude }},{{ location.longitude }} \
    &size=100x100&markers=icon:http://developers.google.com/maps/documentation/javascript/examples/full/images/beachflag.png|{{ location.latitude }},{{ location.longitude }}')
  ~~~
