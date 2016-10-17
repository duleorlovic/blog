---
layout: post
title: Google Maps and Locations
categories: google location-suggestion maps
---

As for any other google api, [Google Maps Javascript
API](https://developers.google.com/maps/documentation/javascript/tutorial)
requires GOOGLE_API_KEY. Use it in loading

~~~
<script src="https://maps.googleapis.com/maps/api/js?key=GOOGLE_API_KEY&callback=initMap"
    async defer></script>
~~~

You can load additional libraries using `libraries=` parameter. If the map is
not the main feauture of the site, I prefer to load it in javascript when
needed.

Start with samples, for example:
* [geocoding](https://developers.google.com/maps/documentation/javascript/examples/geocoding-simple) get locatio based on address
* [autocomplete](https://developers.google.com/maps/documentation/javascript/examples/places-autocomplete)
  show suggestions for adress input

# Edit location for some object

Use this helper in edit.html with form object, and in show with object:

~~~
# app/views/contacts/_form.html.erb
  <%= edit_map f %>

# app/views/contacts/show.html.erb
  <%= show_map @contact %>

# app/views/contacts/index.html.erb
  <%= show_all_map @contacts %>

# app/helpers/map_helper.rb
module MapHelper
  def edit_map(form_object)
    if form_object.object.latitude
      already_have_position = true
      latitude = form_object.object.latitude
      longitude = form_object.object.longitude
      zoom_level = 17
    else
      already_have_position = false
      latitude = 20
      longitude = 80
      zoom_level = 5
    end
    content = form_object.hidden_field(:latitude, id: 'latitude-input')
    content << form_object.hidden_field(:longitude, id: 'longitude-input')
    content << text_field_tag(:address_suggestions, nil, placeholder: 'Write approximate address and drag')
    content << content_tag(:div, nil, id: 'preview-map', style: 'height:300px;width:100%' )
    content << %(
      <script>
        function updateFields(position) {
          $('#latitude-input').val(position.lat());
          $('#longitude-input').val(position.lng());
        }
        function initialize_google() {
          console.log('initializing google autocomplete');
          var map = new google.maps.Map(document.getElementById('preview-map'), {
            center: {lat: #{latitude}, lng: #{longitude} },
            zoom: #{zoom_level},
          });
          var autocomplete = new google.maps.places.Autocomplete(
            document.getElementById('address_suggestions'),
            { componentRestrictions: {country: 'in'} }
          );
          var marker = new google.maps.Marker({
            map: map,
            anchorPoint: new google.maps.Point(0, -29),
            draggable: true,
            animation: google.maps.Animation.DROP,
            #{form_object.object.latitude.present? ?
            "position: {
               lat: #{latitude},
               lng: #{longitude},
             }," : "" }
          });
          autocomplete.bindTo('bounds', map);
          autocomplete.addListener('place_changed', function() {
            marker.setVisible(false);
            var place = autocomplete.getPlace();
            if (!place.geometry) {
             window.alert("Autocomplete's returned place contains no geometry");
             return;
            }
            // If the place has a geometry, then present it on a map.
            if (place.geometry.viewport) {
             map.fitBounds(place.geometry.viewport);
            } else {
             map.setCenter(place.geometry.location);
             map.setZoom(17);  // Why 17? Because it looks good.
            }
            marker.setPosition(place.geometry.location);
            marker.setVisible(true);
            updateFields(place.geometry.location);
          });

          google.maps.event.addListener(marker, "dragend", function(e) {
            updateFields(marker.getPosition());
          });
        } // function initialize_google

      </script>
    ).html_safe
    content << async_load
  end

  def show_map(object)
    latitude = object.latitude
    longitude = object.longitude
    return "<h4>Map position is not set</h4>".html_safe unless latitude
    content = content_tag(:div, nil, id: 'preview-map', style: 'height:300px;width:100%' )
    content << %(
      <script>
        function initialize_google() {
          console.log('initializing google autocomplete');
          var map = new google.maps.Map(document.getElementById('preview-map'), {
            center: {lat: #{latitude}, lng: #{longitude} },
            zoom: 17,
          });
          var marker = new google.maps.Marker({
            map: map,
            animation: google.maps.Animation.DROP,
            #{latitude.present? ?
            "position: {
               lat: #{latitude},
               lng: #{longitude},
             }," : "" }
          });
        } // function initialize_google
      </script>
    ).html_safe
    content << async_load
  end

  def show_all_map(location_nodes)
    data = location_nodes.map do |location_node|
      {
        position: {
          lat: location_node.latitude,
          lng: location_node.longitude,
        },
        name: location_node.name,
        description: location_node.description,
        url: location_node_path(location_node),
      }
    end
    content = content_tag(:div, nil, id: 'preview-map', style: 'height:300px;width:100%' )
    content << %(
      <script>
        function initialize_google() {
          console.log('initializing google autocomplete');
          var map = new google.maps.Map(document.getElementById('preview-map'), {
            center: {lat: 20, lng: 80},
            zoom: 5,
          });
          var data = #{ data.to_json };
          var markers = [];
          var bounds = new google.maps.LatLngBounds();
          var infoWindow = new google.maps.InfoWindow({
            content: "<div><a href='' id='info-url'><h4 id='info-name'></h4></a><p id='info-description'></p></div>"
          });
          for(var i = 0; i < data.length; i++ ) {
            var marker = new google.maps.Marker({
              animation: google.maps.Animation.DROP,
              position: data[i].position,
              name: data[i].name,
              description: data[i].description,
              url: data[i].url,
            });
            bounds.extend(marker.getPosition());
            markers[i] = marker;
            setTimeout(dropMarker(i), i * 100);
            google.maps.event.addListener(marker, 'click', function() {
              infoWindow.open(map, this);
              $('#info-name').html(this.name);
              $('#info-description').html(this.description);
              $('#info-url').attr('href', this.url);
            });
          }
          map.fitBounds(bounds);
          function dropMarker(i) {
            return function() {
              markers[i].setMap(map);
            };
          }


        } // function initialize_google
      </script>
    ).html_safe
    content << async_load
  end

  private

  def async_load
    %(
      <script>
        // https://developers.google.com/maps/documentation/javascript/examples/map-simple-async
        function loadScript() {
          var script = document.createElement('script');
          script.type = 'text/javascript';
          script.src = 'https://maps.googleapis.com/maps/api/js?libraries=places' +
              '&key=#{Rails.application.secrets.google_api_key}&callback=initialize_google';
          document.body.appendChild(script);
        }
        // http://stackoverflow.com/questions/9228958/how-to-check-if-google-maps-api-is-loaded
        if (! (typeof google === 'object' && typeof google.maps === 'object')) {
          loadScript();
        } else {
          // it is loaded but we need to bind on newly created object
          initialize_google();
        }
      </script>
    ).html_safe
  end
end
~~~

# Location search field

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

# Geolocating

Here are the steps that I usually do for geolocating users:

  * try to get lat/long pair from browser
  * use free service <http://www.geoplugin.com/webservices/json>
  * ask user for address and geocode it on client [Google Maps Javascript
    API](https://developers.google.com/maps/documentation/javascript/geocoding)
    Geocoding Service
  * ask user for address and geocode it on server [Google Maps Geocoding
    API](https://developers.google.com/maps/documentation/geocoding/intro)

## Get Brower location

## Get Geoplugin location

Here is example in angular using their
[json](http://www.geoplugin.com/webservices/json) service. 
You can buy cheap [ssl](http://www.geoplugin.com/webservices/ssl) packet and use `https://ssl.geoplugin.net/json.gp?k=...` endpoint.

~~~
    this.getGeoPluginLocation = ->
      deferred = $q.defer()
      # $http has a problem with Access-Control-Allow-Origin since request is
      # OPTIONS instead of GET method
      # $http.get('http://www.geoplugin.net/json.gp?jsoncallback=?').then (response) ->
      $.getJSON('http://www.geoplugin.net/json.gp?jsoncallback=?').then (response) ->
        userLocationData.latLng.lat = parseFloat response.geoplugin_latitude
        userLocationData.latLng.lng = parseFloat response.geoplugin_longitude
        userLocationData.address = response.geoplugin_city + " " + response.geoplugin_countryName
        deferred.resolve()
      return deferred.promise
~~~

[Google Maps Geolocation
API](https://developers.google.com/maps/documentation/geolocation/intro#wifi_access_point_object)
can use wifi or tower ids to give you latLng.
