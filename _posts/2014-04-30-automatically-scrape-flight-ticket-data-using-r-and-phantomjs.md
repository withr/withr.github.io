---
layout: post
title: "Automatically Scrape Flight Ticket Data Using R and Phantomjs"
date: 2014-04-30 10:20
comments: true
categories: Phantomjs R
---

I used to scrape static web pages with the R package *RCurl*. It's a great package! When it comes to dynamic web pages, *RCurl* comes to be difficult to set up (actually, I never get it works). Then I met **[Phantomjs](http://phantomjs.org/)**. *PhantomJS is a headless WebKit scriptable with a JavaScript API. It has fast and native support for various web standards: DOM handling, CSS selector, JSON, Canvas, and SVG.* In this blog, I will show how you to scrape flight tickets data using Phantomjs, extract and filter the entries that meet some restriction, and then send the result to your email automatically. 

- ### Install Phantomjs
Run the following command **line-by-line** in your system console.

``` ruby
sudo apt-get remove phantomjs
sudo unlink /usr/local/bin/phantomjs
sudo unlink /usr/local/share/phantomjs
sudo unlink /usr/bin/phantomjs
cd /usr/local/share
sudo wget https://phantomjs.googlecode.com/files/phantomjs-1.9.0-linux-x86_64.tar.bz2
tar xjf phantomjs-1.9.0-linux-x86_64.tar.bz2
sudo ln -s /usr/local/share/phantomjs-1.9.0-linux-x86_64/bin/phantomjs /usr/local/share/phantomjs 
sudo ln -s /usr/local/share/phantomjs-1.9.0-linux-x86_64/bin/phantomjs /usr/local/bin/phantomjs
sudo ln -s /usr/local/share/phantomjs-1.9.0-linux-x86_64/bin/phantomjs /usr/bin/phantomjs
``` 
Check if PhantomJS was successfully installed using: 

``` ruby
phantomjs -v
``` 

- ### Create the scraping script using Phantomjs

An example looks like this: 

``` ruby
var url = "http://www.finn.no/reise/flybilletter/resultat?numberOfChildren=0&tripType=roundtrip&requestedDestination=PEK.AIRPORT&requestedReturnDate=15.09.2014&requestedOrigin=OSL.AIRPORT&requestedDepartureDate=01.09.2014&numberOfAdults=1"

var fs = require('fs');
var page = new WebPage();

function waitFor(testFx, onReady, timeOutMillis) {
  var maxtimeOutMillis = timeOutMillis ? timeOutMillis : 30000 
  var start = new Date().getTime()
  var condition = false
  var interval = setInterval(function() {
    if ( (new Date().getTime() - start < maxtimeOutMillis) && !condition ) {
      condition = (typeof(testFx) === "string" ? eval(testFx) : testFx()); 
    } else {
      if(!condition) {
        console.log("'waitFor()' timeout");
      } else {
        console.log("'waitFor()' finished in " + (new Date().getTime() - start) + "ms.");
      }
      typeof(onReady) === "string" ? eval(onReady) : onReady(); 
      clearInterval(interval); 
    }
  }, 250); 
};


page.open(url, function (status) {
    if (status !== "success") {
      output.error = 'Unable to access network';
      phantom.exit();
    } else {
      waitFor(function() {
        return page.evaluate(function() {
          return document.getElementById("progressBar").style.width == "100%";
          });
        }, function() {
          result = page.evaluate(function () {
            var A = document.getElementsByClassName("line r-margin mhm toggler linkBlock")
            var Item = [];
            for (var i = 0; i < A.length; i++) {
              var a = A[i];
              var item = [];
              item.push(a.getElementsByClassName('img prs')[0].title);
              item.push(a.getElementsByClassName('man no-break')[0].innerText.replace(',-', ''));
              item.push(a.getElementsByClassName('line mbs')[0].innerText);
              item.push(a.getElementsByClassName('line mbs')[1].innerText);
              Item.push(item.join());
            }
            return Item.join(';') 
          });
          fs.write('/home/tianhd/Finn/01.09.2014-15.09.2014.txt', result);
          console.log('success');
         phantom.exit();
      });        
    }
});

``` 

What you need to change is: **(a)** the URL in the first line (I use www.finn.no, which searches flight tickets over 500 airlines); **(b)** the output file name and directory in line 50. Save above code with a JavaScript suffix, e.g. **fly.js**

Now, run the following command in your system console to see if the scraping works.

``` ruby
phantomjs /<directory>/fly.js  
``` 

If you can see the "success" message in your console, then the scraping work finished, and ticket data in the first page of the searched result will be saved in the file specified in line 50.

- ### Parse the scraped result, select the entries meet your restriction.

``` ruby
setwd("/home/tianhd/Finn")
DAT <- NULL
for (f in list.files(pattern= "[0-9].txt$")) {
  dat <- readLines(f)
  dat <- strsplit(dat, ';')[[1]]
  dat <- t(sapply(dat, FUN = function(x) strsplit(x, ",")[[1]]))
  row.names(dat) <- 1:nrow(dat)
  
  Dat <- data.frame(Flight = dat[, 1], Info = gsub('.txt', '', f), Price = as.numeric(gsub(" ", "", dat[, 2])))
  p_dur <- "[0-9]{2}t [0-9]{2}"
  Dat$DepDur <- gsub("t ", ".", regmatches(dat[, 3], regexpr(p_dur, dat[, 3])))
  Dat$ArrDur <- gsub("t ", ".", regmatches(dat[, 4], regexpr(p_dur, dat[, 4])))
  
  p_tid <- "^[0-9]{2}:[0-9]{2} - [0-9]{2}:[0-9]{2}"
  Dat$DepTid <- gsub(" ",  "",  regmatches(dat[, 3], regexpr(p_tid, dat[, 3])))
  Dat$ArrTid <- gsub(" ",  "",  regmatches(dat[, 4], regexpr(p_tid, dat[, 4])))
  
  DAT <- rbind(DAT, Dat)
}

id <- which(as.numeric(substr(DAT$ArrTid, 1,2)) >= 9 & 
              as.numeric(substr(DAT$DepTid, 1,2)) >= 9 &
              DAT$ArrDur <= 16 & DAT$DepDur <= 16 &
              DAT$Price <= 4500 )
DAT <- DAT[id, ]
DAT <- DAT[order(DAT$Price), ]
DAT 
``` 

The above R code will parse the scraped context into a data frame, and select the tickets that depart after 9:00, travel hours less than 16, and prices not higher than 4500. Save the above code to **preProcess.R**. An example of the result looks like: 

``` ruby
Flight             Info        Price DepDur ArrDur DepTid      ArrTid
Air China 28.08.2014-11.09.2014 4276 11.40 15.05 16:00-09:40 13:50-22:55
Air China 28.08.2014-11.09.2014 4318 12.30 15.05 15:10-09:40 13:50-22:55
Air China 28.08.2014-11.09.2014 4318 14.50 15.05 12:50-09:40 13:50-22:55
Air China 29.08.2014-12.09.2014 4328 11.40 12.25 16:00-09:40 13:50-20:15
Air China 29.08.2014-12.09.2014 4328 11.40 13.00 16:00-09:40 13:50-20:50
Air China 29.08.2014-12.09.2014 4328 12.30 12.25 15:10-09:40 13:50-20:15
``` 


- ### Send the result as attachment to your email 

There is a great R package called **mailR**, which can send the search result as attachment to your email address. This following code will send you an email using your Gmail account.

``` ruby
library(mailR)
send.mail(from = "<your email account>@gmail.com",
          to  =  "<some email address>",
          subject = "Fly ticket",
          body = "From www.Finn.no",
          smtp = list(host.name = "smtp.gmail.com", port = 465, 
                      user.name = "<your user name>", passwd = "<your password>", ssl = TRUE),
          authenticate = TRUE,
          attach.files = "/home/tianhd/Finn/OSL-PEK.csv",
          send = TRUE)
``` 

**Note!** To install package *mailR*, we need to install Java Runtime first. Type the following commands in your system console line-by-line:

``` ruby
sudo apt-get install openjdk-6-jdk
export LD_LIBRARY_PATH=/usr/lib/jvm/java-6-openjdk-amd64/jre/lib/amd64/server
R CMD javareconf # Let R know the configuration of Java;
```

- ### Schedule the job to run everyday automatically.

Use the following command to open the schedule list: 

``` ruby
sudo nano /etc/crontab
```

Add the following two lines to the end of above opened file:


``` ruby
03  12  *  *  * root phantomjs /<dir>/fly.js
05  12  *  *  * root  R --vanilla < /<dir>/dataPreprocess.R
```

Above code will run the web scraping code at 12:03 and run the R script at 12:05. It supposes your will receive an email at 12:05 every. For further detail about schedule task under Linux, please take a look of [this post](http://withr.me/blog/2013/12/02/schedule-task-under-linxu-for-r/)



