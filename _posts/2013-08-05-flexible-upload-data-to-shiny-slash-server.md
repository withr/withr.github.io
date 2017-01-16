---
layout: post
title: "Flexible upload data to Shiny-server"
date: 2013-08-05 17:54
comments: true
categories: Shiny R
---

There is a function called [fileInput](http://rstudio.github.io/shiny/tutorial/#uploads) comes with *Shiny-server*, which can upload data to server. In this post, I would like to share my experience of uploading different kind of data (*.txt, .csv, .xls, .xlsx*, etc.) to Shiny-server. 

The idea of flexible uploading data is: using *File Selection Dialog* to determine the format and location of target document, create different uploading control panel for corresponding format, then upload data using corresponding R package and command. Due to there could be a great number of arguments for an uploading function, e.g. <code>read.table</code> has more than 20 arguments, I included only the most frequently used arguments, such as *header*, *sep*, etc. and take the rest as a character string, so users can type the code they needed themselves. 

<iframe src="http://spark.rstudio.com/withr/flexibleUpload/" width="100%" height = "400px"> </iframe>

Open the above <a href="http://spark.rstudio.com/withr/flexibleUpload/" target="_blank"> application </a> in a new tab.

The flexibility of arguments input was achieved through executing a string as a command:

``` ruby
eval(parse(text = expr))
```
where *expr* is a string of command, e.g. *'read.table(file, header = TRUE)'*. The function <code>parse</code> was suggested to be carefully used by *Thomas Lumley*, however, I don't know why and I have no alternative choice.

``` ruby
require(fortunes)
Loading required package: fortunes
fortune(106)
If the answer is parse() you should usually rethink the question.
   -- Thomas Lumley
      R-help (February 2005)
```

The source code of [*ui.R*](https://gist.github.com/withr/c0cb61a2deddd1881980#file-ui-r) and [*server.R*](https://gist.github.com/withr/c0cb61a2deddd1881980#file-server-r) can be downloaded.




