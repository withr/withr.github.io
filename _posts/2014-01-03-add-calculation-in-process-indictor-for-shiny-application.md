---
layout: post
title: "Add 'Calculation In Process' Indicator for Shiny Application"
date: 2014-01-03 13:31
comments: true
categories: R Shiny
---

A *Calculation in process* busy indicator for shiny application is useful especially for the application that take long time to calculate result. I used to developed an application do model fitting which need more than 10 seconds! Users may lose their patience if the application doesn't response after several seconds! 

One solution is add a process bar, take a look of the threads in [Shiny Google Group](https://groups.google.com/forum/#!topic/shiny-discuss/ZfPCt0QqUuA) and [Stackoverflow](http://stackoverflow.com/questions/18237987/show-that-shiny-is-busy-or-loading-when-changing-tab-panels). The shortcoming of a process bar is you need to give the arguments *min* and *max* values, which for a lot of application is not available. Here I share a simple method add a busy indictor when calculation is undergoing.

Take a look of the html source of a Shiny application which need some time to calculate result, we can find the <code> <html></code> tag has a class named "shiny-busy" when the calculation is under going and disappear when calculation is finish. So, the simple idea comes out: write JavaScript code to monitor the class of <code> <html></code>, if the class equal to "shiny-busy", then show a busy indictor (gif picture), otherwise hide the indictor!

Create a busy indictor <code>div</code> in "ui.R":

``` ruby
div(class = "busy",  
    p("Calculation in progress.."), 
    img(src="ajaxloaderq.gif")
)
```

Include the JavaScript in <code>head</code>:

``` ruby
tagList(
  tags$head(
    tags$link(rel="stylesheet", type="text/css",href="style.css"),
    tags$script(type="text/javascript", src = "busy.js")
  )
)
``` 

Here is the souce code for "busy.js": 

``` ruby
setInterval(function(){
  if ($('html').attr('class')=='shiny-busy') {
    $('div.busy').show()
  } else {
    $('div.busy').hide()
  }
},100)
``` 

We can also use "style.css" to makeup busy indictor's appearance: 

``` ruby
div.busy { 
  position:absolute;
  top: 40%;
  left: 50%;
  margin-top: -100px;
  margin-left: -50px;
  display:none;
  background: rgba(230, 230, 230, .8);
  text-align: center;
  padding-top: 20px;
  padding-left: 30px;
  padding-bottom: 40px;
  padding-right: 30px;
  border-radius: 5px;
}
``` 


Here is the [source code](https://gist.github.com/withr/8799489) and [demo](http://spark.rstudio.com/withr/busyIndictor/).
