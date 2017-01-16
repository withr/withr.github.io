---
layout: post
title: "Who Touched My Shiny-app?"
date: 2014-02-11 14:30
comments: true
categories: R Shiny
---
When we created a Shiny-app, deployed it on a server and open it to public, we must have interesting of who visited our app, and if possible, where they from and what they have done. To achieve this, we need a **user behavior tracking** feature if we are not using the commercial version: *Shiny-server pro*. I wrote a JavaScript code to conduct such operation, and would like to share with you. 

The idea of the JavaScript code is: 

1. Trigger action when user change inputs.
2. Catch the changed object, get its value and label, and put it in a Shiny input.
3. Export the value stored in above input to local file.

So, we need some JavaScript to catch user's behavior and store the value and label to the new created Shiny input.

``` ruby
// Who touched my Shiny-app?
$(document).on("change", ".shiny-bound-input:not([id='tracking'])", function(evt) {
  var el = $(evt.target);
  if (el.prop("tagName").toLowerCase() === "select") {
    value = $("option:selected", el).map(function(){ return this.text }).get().join(", ");
    label = el.prev("label").text();
  } else if (el.attr("type") === "checkbox") {
    value = el.attr("checked"); 
    label = el.parent().text(); 
  } else if (el.attr("type") === "radio") {
    value = el.next().text(); 
    label = $(".control-label", el.parent().parent()).text();
  } else if (el.attr("type") === "text") {
    value = el.val(); 
    console.log(el.prev("label"))
    label = el.prev("label").text();
    if (el.attr("class") === "input-small") {
      label = label + el.index()
    }
  } else if (el.attr("type") === "number") {
    value = el.val(); 
    label = el.prev("label").text();
  };
  label = label.replace(/\n|\r/gm,"");
  label = label.replace(/^ *| *$/gm,"");
  var target = $("#tracking");
  target.val(label + "|" + value);
  target.trigger("change");
});

``` 

And, we need users' location and geographic coordinates. we can get them from many IP query websites, like [ipinfo.io](http://ipinfo.io).

``` ruby
// IP info;
$(document).ready(function(){
  var target = $("#tracking");
  $.get("http://ipinfo.io", function(response) {
    target.val("ipInfo|" + response.ip + "," + response.city + "," + response.loc);
    target.trigger("change");
  }, "jsonp");
});
``` 

A demo can be found [here](http://spark.rstudio.com/withr/tracking/), and its source codes are stored [here](https://gist.github.com/withr/8934848). 
