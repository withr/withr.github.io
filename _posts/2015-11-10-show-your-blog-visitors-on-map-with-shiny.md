---
layout: post
title: "Show your blog’s visitors on map with Shiny"
date: 2015-11-10 11:43
comments: true
categories: R Shiny
---

You may have interest about who visited your web blog, and it’s better if you can see them on map, like this [one](http://188.166.116.72/visitMap/). In this blog, I will show you how to create a shiny-app to track your visitors.

A shiny-app is a web site which allow users to interactively input and get output. The idea of creating a visitor-tracking shiny-app is: when users open your blog site, a shiny-app will also be opened (which can be embedded in your web site), and once the shiny-app be opened, it will save user’s IP related information to your server.

The JavaScript code for getting the IP related information:

```
$(document).ready(function(){
  var I = 0
  var tt = setInterval(function(){
    var L = $("#log");
    $.get("http://ipinfo.io", function(e) {
      L.val(Date() + "  ipInfo|" + e.ip + "," + e.city + "," + e.loc);
      L.trigger("change");
    }, "jsonp");
    if(L.val().length > 0) {
      clearInterval(tt)
    }
  }, 100)
});

```
Where “#log” is a textInput input of shiny for storing the IP related information. 

The following part in above is used to get the IP related information and store it into “#log”, and tell shiny-app that the value of “#log” has changed. So, we can write some code in “server.R” to save the value of “#log” into server.

```
    $.get("http://ipinfo.io", function(e) {
      L.val(Date() + "  ipInfo|" + e.ip + "," + e.city + "," + e.loc);
      L.trigger("change");
    }, "jsonp");
```
And the following part was used to stop continue fetching IP information:

```
    if(L.val().length > 0) {
      clearInterval(tt)
    }
```


### In server.R

The code for map: 

```
  output$myMap <- renderMap({
    Dat <- rDat()
    Map <- Leaflet$new()
    Map$addParams(width="100%", height="800px;", layerOpts = list(maxZoom = 18, zoomControl = FALSE))
    Map$addParams(bounds = list(c(min(Dat$Lat), min(Dat$Lon)) - 10, c(max(Dat$Lat), max(Dat$Lon)) + 10))
    Points <- as.list(rep(NA, nrow(Dat)))
    for (i in 1:nrow(Dat)) {
      Points[[i]] <- c(lat = Dat$Lat[i], lon = Dat$Lon[i], 
                       value = paste(Dat$City[i], as.character(Dat$Time[i]), sep = ": "))
    }
    Map$addParams(cluster = Points)
    Map
  })

```
where “Map$addParams(cluster = Points)” creates an object called “spec.cluster”. We need to add the following JavaScript code to show the cluster points.

```
 if (spec.cluster != undefined) {
    var markers = L.markerClusterGroup();
		for (var i = 0; i < spec.cluster.length; i++) {
			var a = spec.cluster[i];
			var title = a.value;
			var marker = L.marker(new L.LatLng(a.lat, a.lon), { title: title });
			marker.bindPopup(title);
			markers.addLayer(marker);
		}
		map.addLayer(markers);
    }
```

We also need include some JavaScript libraries needed into “/usr/local/lib/R/site-library/rCharts/libraries/leaflet/external”, they are: 

MarkerCluster.css
MarkerCluster.Default.css
leaflet.markercluster-src.js

To include above libraries, we need to modify the file “/usr/local/lib/R/site-library/rCharts/libraries/leaflet/config.yml” as: 

```
leaflet:
  css: 
    - external/leaflet.css
    - external/MarkerCluster.css 
    - external/MarkerCluster.Default.css

  jshead:
    - external/leaflet.js
    - external/leaflet.markercluster-src.js


```


All scripts (and a demo data) can be downloaded [here](misc/visitMap.zip). Remember backup the “leaflet” folder before replacing it with the one in download folder (“www/leaflet”).
