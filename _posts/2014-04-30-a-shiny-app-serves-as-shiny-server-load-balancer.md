---
layout: post
title: "A Shiny-app Serves as Shiny-server Load Balancer"
date: 2014-04-30 13:51
comments: true
categories: R Shiny 
---
The Shiny-app on open-source edition Shiny-server has only one concurrent, which means it can run only for one user at a time point. But it can host multiple Shiny-apps, which can run synchronously. So, if we create severl Shiny-apps with different names but same function, then we can let more users use our service at same time. But users don't how to choose the Shiny-app with small user number. This post will show you how to create a Shiny-app to redirect user to the Shiny-app with lower load. 

I have no knowledge about server load balancer, the following method is ONLY what I thought it can be. 

- First, we need to know the load information about Shiny apps on our server, like which apps are running, how many users for each app, etc. 

- Then,  create a normal Shiny app to detect which app has little user number than others, and using JavaScript to redirect user to that app. 

###1. CPU information about Shiny-app
The following is the R code than generates a data frame containing which Shiny-app are running and the user number of each Shiny-app. 

``` ruby
## Setup work directory;
setwd("/srv/shiny-system/Data") 
I <- 0
for (i in 1:60) {
  system("top -n 1 -b -u shiny > top.log")
  dat <- readLines("top.log")
  id <- grep("R *$", dat)
  Names <- strsplit(gsub("^ +|%|\\+", "", dat[7]), " +")[[1]]
  if (length(id) > 0) {
    # 'top' data frame;
    L <- strsplit(gsub("^ *", "", dat[id]), " +")
    dat <- data.frame(matrix(unlist(L), ncol = 12, byrow = T))
    names(dat) <- Names
    dat <- data.frame(Time = Sys.time(), dat[, -ncol(dat)], usr = NA, app = NA)
    dat$CPU <-as.numeric(as.character(dat$CPU))
    dat$MEM <-as.numeric(as.character(dat$MEM))
    # Check if connection number changed;
    for (i in 1:length(dat$PID)) {
      PID <- dat$PID[i]
      system(paste("sudo netstat -p | grep", PID, "> netstat.log"))
      system(paste("sudo netstat -p | grep", PID, ">> netstat.log2"))
      system(paste("sudo lsof -p", PID, "| grep /srv > lsof.log"))
      netstat <- readLines("netstat.log")
      lsof <- readLines("lsof.log")
      dat$usr[i] <- length(grep("ESTABLISHED", netstat) & grep("tcp", netstat))
      dat$app[i] <- regmatches(lsof, regexec("srv/(.*)", lsof))[[1]][2]
    }
    dat <- dat[, c("app", "usr")]
  } else {
    dat <- data.frame(app = "app", usr = 0)
  }
  write.table(dat, file = "CPU.txt")
}


```

To make it run automatically, schedule it under <code>/etc/crontab</code> like the following:

``` ruby

*  *  *  *  * root Rscript /<dir of shiny-app>/CPU.R

```

###2. Create the Shiny-app for redirecting.

**ui.R**

``` ruby
shinyUI(bootstrapPage(
  tags$style("#link {visibility: hidden;}"), # This app doesn't need user interface;
  textInput(inputId = "link", label = "", value = ""), # Redirecting link;
  tags$script(type="text/javascript", src = "redirect.js") # JavaScript for redirecting;
))
``` 

**server.R** 

``` ruby
shinyServer(function(input, output, session) {
  CPU <- read.table("Data/CPU.txt")
  App <- data.frame(app = c("app_1", "app_2", "app_3", "app_4"))
  App <- merge(App, CPU, all.x = TRUE)
  App$usr[which(is.na(App$usr))] <- 0
  Link <- paste("http://192.168.150.36/", App$app[which.min(App$usr)], sep = "")
  updateTextInput(session, inputId = "link", value = Link)
}) 

``` 

**redirect.js**

``` ruby
setInterval(function() {
  var link = document.getElementById('link').value;
    if (link.length >1) {
      window.open(link, "_top")
    }
}, 50)

``` 



















