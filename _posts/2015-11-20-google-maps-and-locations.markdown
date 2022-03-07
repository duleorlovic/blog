---
layout: post
title: Google Maps and Locations
categories: google location-suggestion maps
---

# Installation

As for any other google api, [Google Maps Javascript
API](https://developers.google.com/maps/documentation/javascript/tutorial)
requires GOOGLE_API_KEY. You need it when you loading the script

~~~
<script src="https://maps.googleapis.com/maps/api/js?key=GOOGLE_API_KEY&callback=initMap"
    async defer></script>
~~~

You can load additional libraries using `libraries=` parameter. If the map is
not the main feauture of the site, I prefer to load it in javascript when
needed.

For your google console project you need to enable relevant api:

* *Google Maps Javascript API* (to load gmap on the page)
* *Google Static Maps API* (to show image with pin marker)
* *Google Places API Web Service* (to search by address, restauran or other
place) note that I can not find this in search apis but only using this link
<https://console.developers.google.com/apis/api/places_backend?project=_>

Start with samples, for example:

* [geocoding](https://developers.google.com/maps/documentation/javascript/examples/geocoding-simple)
  input is address, output is location
* [autocomplete](https://developers.google.com/maps/documentation/javascript/examples/places-autocomplete)
  show suggestions for adress input

# Edit location for some object

Use this helper in edit.html with form object, and on show page with object.
You need to provide input fields: `text_field` or `hidden_field` so they will be
updated in javascript callback.

If you show/hide whole map (with `display: none`), than you need to trigger
resize with setCenter and setZoom or map.fitBounds (probably inside setTimeout
if you are still in digest loop).
Better is to use opacity 0, 1 and move element below.

~~~
.hide-with-opacity {
  opacity: 0;
  &.active {
    opacity: 1;
  }
}
~~~

Note that when `resize` is triggered than center is changed, so you need to grab
center before resize, than call resize (so map is actually shown) and than call
setCenter or panTo the original center. To get map object we use mapObject data

~~~
$(document).on 'ready page:load', ->
  # if you do not have map object, you can trigger resize on window
  # google.maps.event.trigger(window, 'resize', {})
  # but the problem is when map is initialized to hidden element, than center is
  # on top right corner, so when you trigger resize, pin will be on top-right
  # We need to getCenter and call setCenter again after we trigger resize
  if $('#preview-map').length
    $('.nav-tabs').on 'shown.bs.tab', ->
      map = $('#preview-map').data('mapObject')
      originalCenter = map.getCenter()
      # google.maps.event.trigger(window, 'resize', {})
      # map.setCenter c
      google.maps.event.trigger map, 'resize'
      map.panTo originalCenter
~~~


~~~
# app/helpers/map_helper.rb
# rubocop:disable Metrics/ModuleLength
# rubocop:disable Metrics/MethodLength
# rubocop:disable Metrics/AbcSize
# rubocop:disable Style/MultilineTernaryOperator
# rubocop:disable Layout/SpaceInsideStringInterpolation
module MapHelper
  INITIAL_LATITUDE = 45.2671352
  INITIAL_LONGITUDE = 19.83354959999997
  INITIAL_ZOOM = 12
  GOOGLE_API_KEY = Rails.application.secrets.google_api_key

  # You need to provide input fields: `text_field` or `hidden_field` so they
  # will be updated in javascript callback.
  #  <%= f.text_field :latitude, id: 'latitude-input' %>
  #  <%= f.text_field :longitude, id: 'longitude-input' %>
  #  <%= edit_map f.object, "#latitude-input", "#longitude-input" %>
  #
  # If you are showing inside bootstrap tab or modal than you need to trigger
  # resize. If you do not have map object, you can trigger resize on window
  # google.maps.event.trigger(window, 'resize')
  # but the problem is when map is initialized to hidden element, than center is
  # on top right corner, so when you trigger resize, pin will be on top-right
  # We need to getCenter and call setCenter again after we trigger resize so I
  # did it using addDomListener on window object and fire that resize on modal
  # show:
  #
  # somewhere inside function initialize_google() {
  #        google.maps.event.addDomListener(window, "resize", function() {
  #          var center = map.getCenter();
  #          google.maps.event.trigger(map, "resize");
  #          map.setCenter(center);
  #          console.log('map resize');
  #        });
  # somewhere in your javascript/coffeescript:
  # $(document).on 'shown.bs.modal', ->
  #   window.dispatchEvent(new Event('resize'))
  #
  # If you are destroying modal on hide (so it loads new content based on
  # response), than map will be destroyed too.
  #
  # $(document).on 'hidden.bs.modal', '.modal', ->
  #   $(this).removeData('bs.modal')
  #
  # Problem is that javascript in form response (on this page) is run only once,
  # so you can not trigger initialize_google() from this page. but only on shown
  # good is that you do not need to trigger resize since we draw map again:
  #
  # $(document).on 'shown.bs.modal', '.modal', ->
  #   if this.innerHTML.indexOf('initialize_google') > -1
  #     initialize_google()
  def edit_map(object, latitude_input, longitude_input)
    if object.latitude
      latitude = object.latitude
      longitude = object.longitude
      zoom_level = 17
    else
      latitude = INITIAL_LATITUDE
      longitude = INITIAL_LONGITUDE
      zoom_level = INITIAL_ZOOM
    end
    address_id = "address-suggestion" # -#{SecureRandom.hex 3}"
    content = text_field_tag(
      address_id, nil, placeholder:
      I18n.t('write_address_than_move_marker'), size: 50
    )
    map_id = "preview-map" #-#{SecureRandom.hex 3}"
    content << content_tag(:div, nil, id: map_id, class: 'edit-map-container')
    content << %(
      <script>
        function updateFields(position) {
          $("#{latitude_input}").val(position.lat());
          $("#{longitude_input}").val(position.lng());
        }
        function initialize_google() {
          console.log('initializing google autocomplete');
          // https://wiki.openstreetmap.org/wiki/Google_Maps_Example
          var mapTypeIds = [];
          for(var type in google.maps.MapTypeId) {
           mapTypeIds.push(google.maps.MapTypeId[type]);
          }
          mapTypeIds.push("OSM");
          var map = new google.maps.Map(document.getElementById('#{map_id}'), {
            center: {lat: #{latitude}, lng: #{longitude} },
            zoom: #{zoom_level},
            mapTypeId: "OSM",
            mapTypeControlOptions: {
              mapTypeIds: mapTypeIds
            },
          });

          map.mapTypes.set("OSM", new google.maps.ImageMapType({
            getTileUrl: function(coord, zoom) {
              // See above example if you need smooth wrapping at 180th meridian
              return "http://tile.openstreetmap.org/" + zoom + "/" + coord.x + "/" + coord.y + ".png";
              },
            tileSize: new google.maps.Size(256, 256),
            name: "OpenStreetMap",
            maxZoom: 18,
          }));
          var autocomplete = new google.maps.places.Autocomplete(
            document.getElementById('#{address_id}'),
            { componentRestrictions: {country: 'rs'} }
          );
          var marker = new google.maps.Marker({
            map: map,
            anchorPoint: new google.maps.Point(0, -29),
            draggable: true,
            animation: google.maps.Animation.DROP,
            #{object.latitude.present? ?
              "position: {
                lat: #{latitude},
                lng: #{longitude},
              }," : '' }
          });
          autocomplete.bindTo('bounds', map);
          autocomplete.addListener('place_changed', function() {
            marker.setVisible(false);
            var place = autocomplete.getPlace();
            if (!place.geometry) {
             window.alert("#{I18n.t('autocomplete_contains_no_geometry')}");
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
            console.log('place_changed');
          });

          google.maps.event.addListener(marker, "dragend", function(e) {
            updateFields(marker.getPosition());
          });
        } // function initialize_google
      </script>
    ).html_safe
    content << async_load
  end

  # https://developers.google.com/maps/documentation/static-maps/intro
  def show_static_map(object, options = {})
    latitude = object.latitude
    longitude = object.longitude
    return "<div class='map-not-set'>Map position is not set</div>".html_safe unless latitude
    img_options = {}
    img_options[:class] = options[:class] if options[:class].present?
    url = "//maps.googleapis.com/maps/api/staticmap?"
    url += "size=70x70"
    # custom icon must be http:// not https://
    # markers=icon:http://developers.google.com/maps/documentation/javascript/examples/full/images/beachflag.png|
    url += "&markers=|#{latitude},#{longitude}"
    # you need to enable "Google Static Maps API"
    url += "&key=#{GOOGLE_API_KEY}"
    link_to "http://maps.google.com/?q=#{latitude},#{longitude}" do
      image_tag url, img_options
    end
  end

  def show_map(object, options = {})
    latitude = object.latitude
    longitude = object.longitude
    return "<div class='map-not-set'>Map position is not set</div>".html_safe unless latitude
    content = content_tag(:div, nil, id: 'preview-map', class: "show-map-container #{options[:class]}")
    content << %(
      <script>
        function initialize_google() {
          console.log('initializing google autocomplete');
          var previewMapElement = document.getElementById('preview-map');
          var map = new google.maps.Map(previewMapElement, {
            center: {lat: #{latitude}, lng: #{longitude} },
            zoom: 17,
          });
          var marker = new google.maps.Marker({
            map: map,
            animation: google.maps.Animation.DROP,
            #{object.latitude.present? ?
              "position: {
                lat: #{latitude},
                lng: #{longitude},
              }," : '' }
          });
          $(previewMapElement).data('mapObject', map);
        } // function initialize_google
      </script>
    ).html_safe
    content << async_load
  end

  def show_all_map(objects, options = {})
    data = objects.map do |object|
      next if !object.latitude.present? || !object.longitude.present?
      {
        position: {
          lat: object.latitude,
          lng: object.longitude,
        },
        name: object.name,
        description: object.description,
        url: location_node_path(object),
      }
    end.compact
    content = content_tag(:div, nil, id: 'preview-map', class: "show-all-map-container #{options[:class]}")
    content << %(
      <script>
        function initialize_google() {
          console.log('initializing google autocomplete');
          var map = new google.maps.Map(document.getElementById('preview-map'), {
            center: {lat: #{INITIAL_LATITUDE}, lng: #{INITIAL_LONGITUDE}},
            zoom: #{INITIAL_ZOOM},
          });
          var data = #{data.to_json};
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
              '&key=#{GOOGLE_API_KEY}&callback=initialize_google';
          document.body.appendChild(script);
        }
        // http://stackoverflow.com/questions/9228958/how-to-check-if-google-maps-api-is-loaded
        if (typeof google === 'object' && typeof google.maps === 'object') {
          // it is loaded but we need to bind on newly created object
          initialize_google();
        } else {
          loadScript();
        }
      </script>
    ).html_safe
  end
end
~~~

~~~
# app/views/contacts/_form.html.erb
  <%= edit_map f %>

# app/views/contacts/show.html.erb
  <%= show_map @contact %>

# app/views/contacts/index.html.erb
  <%= show_all_map @contacts %>
  <%= show_static_map @contact %>

# app/assets/stylesheets/maps.scss
.edit-map-container,.show-all-map-container {
  height: 300px;
  width: 100%;
}
.show-map-container {
  height: 30px;
}
~~~

# Stimulus and map in jbox modal

https://github.com/trkin/move-index/blob/master/app/javascript/controllers/google_map_address_controller.js

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

# Geocoder gem

You can geocode ActiveRecord object by latitude longitude or by ip address.
Results (reverse_geocode) can be written in addres, city, country. You can use
in console

~~~
Geocoder.coordinates("25 Main St, Cooperstown, NY")
# Google Geocoding API error: over query limit.
Geocoder.coordinates("178.222.169.10") #=> ["46.1", "19.6"]

Geocoder.search("Paris", :bounds => [[32.1,-95.9], [33.9,-94.3]])
~~~

You can download MaxMind local database (only for ip geocode) and import to psql

~~~
# generate migration to create tables
rails generate geocoder:maxmind:geolite_city

# download, unpack, and import data
rake geocoder:maxmind:geolite:load PACKAGE=city

# configure
Geocoder.configure(ip_lookup: :maxmind_local, maxmind_local: {package: :city})

# now you can use offline
Geocoder.coordinates("178.222.169.10") #=> ["46.1", "19.6"]
~~~

Since for GOOGLE API there is a limit of free 2_500 request per day you can
geocode if change in latitude and longitude is big
> Each degree of latitude is approximately 69 miles (111 kilometers) apart. The range varies (due to the earth's slightly ellipsoid shape) from 68.703 miles (110.567 km) at the equator to 69.407 (111.699 km) at the poles. This is convenient because each minute (1/60th of a degree) is approximately one [nautical] mile.
> A degree of longitude is widest at the equator at 69.172 miles (111.321) and gradually shrinks to zero at the poles. At 40° north or south the distance between a degree of longitude is 53 miles (85 km)

~~~
# app/models/user.rb
class User < ActiveRecord::Base
  MIN_LATITUDE_CHANGE_FOR_GEOCODE = 0.2 # 1 degree is 69 miles (111 km)
  MIN_LONGITUDE_CHANGE_FOR_GEOCODE = 0.3 # 1 degree on equator 69 miles (111km), on 40deg 53 miles (85km)
  reverse_geocoded_by :latitude, :longitude, address: :reverse_geocoded_address do |obj, results|
    # next unless (obj.latitude_changed? && obj.longitude_changed?) || !obj.current_city_by_latitude_longitude.present?
    if geo = results.first
      obj.current_city_by_latitude_longitude    = geo.city
      obj.current_address_by_latitude_longitude = geo.address
      obj.save!
    end
  end
  # do not run on each validation since api limit is 2_500
  # after_validation :reverse_geocode

  def submit_reverse_geocode
    return false if latitude.nil? || longitude.nil?
    if current_city_by_latitude_longitude.nil? || last_geocoded_latitude.nil? || last_geocoded_longitude.nil?  ||
        (last_geocoded_latitude - latitude).abs > MIN_LATITUDE_CHANGE_FOR_GEOCODE ||
        (last_geocoded_longitude - longitude).abs > MIN_LONGITUDE_CHANGE_FOR_GEOCODE
      reverse_geocode
      self.last_geocoded_latitude = latitude
      self.last_geocoded_longitude = longitude
      save!
    else
      false
    end
  end
end
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
  https://console.cloud.google.com/marketplace/product/google/geocoding-backend.googleapis.com?q=search&referrer=search&authuser=1&project=myapp&supportedpurview=project

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

# Apple MapKit

https://developer.apple.com/videos/play/wwdc2018/212/

# GeoJSON

https://tools.ietf.org/html/rfc7946
Feature object represents a spatially bounded thing, has `type='Feature'` and
`geometry` as Geometry object, `properties` member. Geometry object is one of
the seven geometry types: `Point, MultiPoint, LineString, MultiLineString,
Polygon, MultiPolygon and GeometryCollection` which contains `coordinates` array
first is Longitude and second is Latitude (then optionally elevation)
```
var geojsonFeature = {
    "type": "Feature",
    "properties": {
        "name": "Coors Field",
        "amenity": "Baseball Stadium",
        "popupContent": "This is where the Rockies play!"
    },
    "geometry": {
        "type": "Point",
        "coordinates": [-104.99404, 39.75621]
    }
};
```

To add GeoJSON to leaflet you can use
```
  L.geoJSON(stores).addTo(mymap)
// or add data later
  var myLayer = L.geoJSON().addTo(mymap)
  myLayer.addData(stores)
```

You can style objects with color, wight, opacity
```
L.geoJSON(stores, {
  style: {
    'color': 'red',
    'weight': 4,
    'opacity': 1,
  }
}).addTo(mymap)

// or using funcion

L.geoJSON(stores, {
  style: function(feature) {
    switch (feauture.properties.myProp) {
    case 'Red': return { color: 'red' }
    }
  }
)
```

For points you can use pointToLayer to render circle instead of default marker
```
  L.geoJSON(geojsonFeature, {
    pointToLayer: function (feature, latlng) {
      return L.circleMarker(latlng, geojsonMarkerOptions);
    }
  }).addTo(mymap)
```
Use iteration onEachFeature to add popups
```
function onEachFeature(feature, layer) {
  if (feature.properties && feature.properties.popupContent) {
    layer.bindPopup(feature.properties.popupContent);
  }
}

L.geoJSON(geojsonFeature, {
    onEachFeature: onEachFeature
}).addTo(map);
```
Use `filter` function to skip redenring feature.

# Leaflet

```
  var mymap = L.map('mapid').setView([45.26, 19.83], 10)

  L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.1w', {
    attribution: 'dule',
    id: 'mapbox/streets-v11', // mapbox/satellite-v9
    zoomOffset: -1,
    tileSize: 512,
  }).addTo(mymap)

// z is zoom level, x, and y are lat long, id is type of tile
// there is no logic explanation of zoomOffset: -1 and tileSize: 512
// On https://leafletjs.com/examples/quick-start/ I see
// Because this API returns 512x512 tiles by default (instead of 256x256), we will also have to explicitly specify this and offset our zoom by a value of -1.
```
Marker, polygon and Popups
```
  var marker = L.marker([45.26, 19.83]).addTo(mymap)
  var polygon = L.polygon([
    [45.20, 19.4],
    [45.50, 19.9],
    [45.00, 19.9],
  ]).addTo(mymap)
  marker.bindPopup('<b>Hi</b> I am a popup').openPopup()
  polygon.bindPopup('I amd a polygon')

  var popup = L.popup()
    .setLatLng([45.3, 19.5])
    .setContent('I am a standalong popup')
    .openOn(mymap) // .openOn is same as addTo but will close previously opened popups
```

Events

Each element has it's own events, but all have `click` with argument which has
`e.latlng` object.
https://leafletjs.com/reference-1.7.1.html#map-event
```
  var clickPopup = L.popup()
  function onMapClick(e) {
    clickPopup
      .setLatLng(e.latlng)
      .setContent(`You clicked on ${e.latlng.toString()}`)
      .openOn(mymap)
  }
  mymap.on('click', onMapClick)
```
Ask for browser location
```
map.locate({setView: true, maxZoom: 16});

function onLocationFound(e) {
    var radius = e.accuracy;

    L.marker(e.latlng).addTo(map)
        .bindPopup("You are within " + radius + " meters from this point").openPopup();

    L.circle(e.latlng, radius).addTo(map);
}
map.on('locationfound', onLocationFound);

function onLocationError(e) {
    alert(e.message);
}
map.on('locationerror', onLocationError);
```

# Mapbox

https://docs.mapbox.com/help/tutorials/building-a-store-locator/#add-structure
https://docs.mapbox.com/help/tutorials/building-a-store-locator-js/ (legacy)
https://docs.mapbox.com/help/troubleshooting/transition-from-mapbox-js-to-mapbox-gl-js/

On rails you need to prevent babel for transpiling nodeModules
```
# config/webpack/environment.js
const { environment } = require('@rails/webpacker')

// https://github.com/visgl/react-map-gl/issues/874#issuecomment-524404219
// Preventing Babel from transpiling NodeModules packages
environment.loaders.delete('nodeModules');

module.exports = environment
```
