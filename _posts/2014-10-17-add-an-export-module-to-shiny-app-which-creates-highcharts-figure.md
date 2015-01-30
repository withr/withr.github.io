---
layout: post
title: "Add An Export Module to Shiny-app for Highcharts Figure"
date: 2014-10-17 15:01
comments: true
categories: R Shiny
---

Highcharts is the best JavaScript chart library, I feel. By default, there is an export button which can let you save the interactive figure to PNG, JPEG, PDF and SVG format, and one important point is, you can use it in your shiny app through R package **rCharts**. However, there are some limits in such a way, the default exporting engine is based **http**, so if your shiny-app's portal is **https**, then you can't use the exporting feature (It seems Highcharts has also a *https* based export engine). I have spent a lot of time to configure a Highcharts export server, it works internally, but not outside. I can't figure out the problem, but found another solution. 


The figures created by Highcharts are actually **SVG** element in HTML, we can easily extract that element using JavaScript, and fill it into a **textInput** of Shiny, when **textInput** received the *SVG* element, its value changed, so it triggered Shiny server to response, then we can let Shiny-server to save the *SVG* element onto our server. We can convert that *SVG* file into the format we want, and download it use *downloadHandler*. 


The most difficult parts to R users which are not familiar with JavaScript and Linux are: 

###1. Extract the SVG element and fill it to *textInput*.

{% highlight JavaScript %}
// Export module
function figExport() {
  // Select chart and get svg data;
  var chart = $(".shiny-html-output.rChart.highcharts.shiny-bound-output").highcharts()
  var svg = chart.getSVG();
  var svg = svg.replace(/<g class="highcharts-button".*?g>/, "")
  var svg = svg.replace(/<g class="highcharts-tooltip".*?g>/, "")
  // Empty target object and fill with new svg data;
  var target = $("#svg");
  target.val("");
  target.val(svg);
  target.trigger("change");
  $("#Export").attr("style", "height: 105px;")
  $(".ePanel").attr("style", "visibility: visible;");
  $(".wPanel:has(#export)").mouseleave(function(){
    $(".ePanel").attr("style", "visibility: hidden;");
    $("#Export").attr("style", "height: 30px;")
  });
}
{% endhighlight %}

We can bind this function to a button: when clicking that button, web browser will extract the SVG element, fill it into an *textInput* using JavaScript and tell Shiny-server that the *textInput* got a new value. After receiving that *textInput*'s value has changed, Shiny-server will save its value to a SVG file. 

{% highlight R %}
p <- reactiveValues()
observe({
  p$tempSVG <- paste(tempfile(), "svg", sep =".")
  p$tempPDF <- paste(tempfile(), "pdf", sep =".")
  p$tempPNG <- paste(tempfile(), "png", sep =".")
  if (!is.null(input$svg)) {
    if (nchar(input$svg)>0) {
      writeLines(input$svg, p$tempSVG)
    }
  }
})
{% endhighlight %}



###2. Convert SVG file to other formats under Linux.
There are several programs can convert svg file to other formats, I choose **[inkscape](https://apps.ubuntu.com/cat/applications/precise/inkscape/)**. To install it, run this command:

{% highlight ruby %}
sudo apt-get install inkscape
{% endhighlight %}

After installing it on your server, we can call it through R using **system** command: 

{% highlight R %}
# Export figure as PDF;
output$uiFigPDF <- renderUI({
  downloadLink(outputId = "figPDF", 
               label    = E2N("Export figure as PDF"))
})
output$figPDF <- downloadHandler(
  filename = function() {
    paste('Highcharts-', Sys.Date(), '.pdf', sep='')
  },
  content = function(file) {
    system(paste("inkscape -f", p$tempSVG, "-A", p$tempPDF))
    file.copy(p$tempPDF, file)
  }
)
{% endhighlight %}


### That's it!

We can also add an option to change the dimension of the figure: 

{% highlight R %}
output$uiDim <- renderUI({
  *textInput*(inputId = "dim", label = E2N("Figure dimensions (W x H):"), 
            value = "1000 x 1200")
})

## Inside of  'renderChart' in server.r     
if (!is.null(input$dim)) {
  H$exporting(sourceWidth = as.numeric(strsplit(input$dim, " *x *")[[1]][1]),
              sourceHeight= as.numeric(strsplit(input$dim, " *x *")[[1]][2]),
              enabled = FALSE)
}
{% endhighlight %} 

And to export the figure data is even simple: 


{% highlight R %}
# Export figure data;
output$uiFigDat <- renderUI({
  downloadLink(outputId = "figDat", 
               label    = E2N("Export figure data"))
})
output$figDat <- downloadHandler(
  filename = function() {
    paste('Highcharts data-', Sys.Date(), '.csv', sep='')
  },
  content = function(file) {
    write.table(p$data, file, row.names = FALSE, quote = FALSE, sep = ";", 
                na = "", dec = ifelse(input$lang == "no", ",", "."))
  }
)
{% endhighlight %}

where <code>p$data</code> was assign before. Here is a [demo](http://188.226.199.99/shiny-server/shinyExport/), and you can find the source code on my [gist](https://gist.github.com/withr/59cc48edc1658b61ea4e).





