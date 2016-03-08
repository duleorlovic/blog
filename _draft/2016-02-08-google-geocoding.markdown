---
layout: post
title: Google Geocoding
---

# Introduction

Here are the steps that I usually do for geolocating users:

  * try to get lat/long pair from browser
  * user free service <http://www.geoplugin.com/webservices/json>
  * ask user for address and geocode it on client [Google Maps Javascript
    API](https://developers.google.com/maps/documentation/javascript/geocoding)
    Geocoding Service
  * ask user for address and geocode on server [Google Maps Geocoding
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


