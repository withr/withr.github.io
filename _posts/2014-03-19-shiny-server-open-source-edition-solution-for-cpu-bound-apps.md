---
layout: post
title: "Shiny-server Open Source Edition: Solution for CPU Bound Apps"
date: 2014-03-19 10:35
comments: true
categories: R Shiny
---

Shiny-server integrates the statistic power of R with web, convenients the statistical analysis through web. Users can do complecate statistical anlysis through their web browsers. However, most of statistical analysis of R are **CPU bound** computation, that means CPU utilization is high, perhaps at 100% for many seconds. This leads to the problem of **thread blocking**: if one user is doing some calculation, the other user must wait until that process finished! For example, if one calculation needs 10 seconds to finish, and 4 users are doing the same calculation at same time, one user may need 10 seconds to get the result, and the unlucky one may need to wait 40 seconds! That's annoying, but the native character of statistical analysis and R (R is single-thread: it uses only one core of your CPU). 

The commercial edition Shiny-server has a *multiple processes* feature, which allow multiple users access the same shiny application at same time and don't need to wait. For example, it has a 20-concurrent version which allow 20 users access the same application at same time. But remember, the calculation time of the application (statistic analysis, model fitting, etc.) is fixed, and commercial version Shiny-server can not accelerate statistical computation, but take the advantage of multiple-core CPU to do calculation parallelly. So, if your Shiny-server has only a single-core CUP, commercial edition Shiny-server will take no advantage than the open source edition. And if you want take the full advantage of a 20-concurrent Shiny-server, you'd better have a 20-core CPU!


**Luckly**, the open source edition Shiny-server can host multiple applications on a single server! That means if your server has a 4-core CPU, and there are 4 applications on the server, 4 users access the 4 applications seperately at same time, the 4 applications will use all CPU cores, just like the commercial edition.

![]( /images/cpu_4.jpg )

So, if we can re-direct user to the application that is less busier than the visited, we acctually are doing the commercial version's job!

Let's take a look of the performance of single process and multiple processes. I have a open source edition Shiny-server and 4 apps (they are same, just 4 copies) on my 4-core CPU server. The app needs 22 seconds to get its first result. If 4 users are running the same app, the time they need wait to get their first results are: 21, 41, 61, 79 seconds, seperately. If 4 users are running the 4 applications seperately at the same time, the time they needs are: 35, 39, 38, 39 seconds, seperately. So, the time needed using parallel computation is not same as single thread computation: CUP need time to balance the assignment of computation (I am not clear about the machenism).


But users don't know which core is idle, how could they choose which application to load? A naive method is: create a txt file contains the applications' IP addresses and the number of their users, when users log in one application, increase its user number by one, then check if other application have lower users number, if ture, then re-direct user to the application with smallest user number, if false, continue to load current application, and when user log out the application, its user number will decrease by one. 

We can create a shiny input containing the re-directed application address when re-directing happens, and create a JavaScript to monitor that element, when capture that element, re-direct user to the application using <code> windown.open('http://....', '_top')</code>

Re-direct R code:

``` ruby
  observe({
    app <- session$clientData$url_pathname
    CPU <- read.table("../cpu.txt", header = TRUE)
    id <- grep(app, CPU$href)
    CPU$users[id] <- CPU$users[id] + 1
    write.table(CPU, "../cpu.txt", row.names = FALSE)
    session$onSessionEnded(function(){
      CPU <- read.table("../cpu.txt", header = TRUE)
      CPU$users[id] <- CPU$users[id] - 1
      write.table(CPU, "../cpu.txt", row.names = FALSE)
    })
  })
``` 

Re-direct JavaScript code:

``` ruby
var I = 0;
var tt = setInterval(function() {
  var link = $("div#redirect a").attr("href");
  if (typeof link != "undefined") {
    if (link.length >1) {
      window.open(link, "_top")
      clearInterval(tt);
    }
  }
}, 50)
``` 

As we see, parallel calculation is not *so* efficient on a single server. It will be more efficient if we can re-direct user to other *server* (only need a single-core CPU, can be set using VPS) instead of other application on the same server. 








