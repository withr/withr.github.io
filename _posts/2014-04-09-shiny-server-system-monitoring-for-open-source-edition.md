---
layout: post
title: "Shiny-server System Performance Monitoring for Open Source Edition"
date: 2014-04-09 09:37
comments: true
categories: R Shiny Linux
---

If you have deployed your Shiny-app on internet, you may curious about: how many users are using my app? Is the server powerful enough for hosting the app? You can get answers through the *[Server monitoring](http://www.rstudio.com/shiny/server/)* feature if you are using the Professional edition of Shiny server. What if we are using the *Open source* edition? Officially, there is no such a feature, but we can create it ourselves using *Linux shell commands*.  

Suppose that we have three shiny-apps: app#1, app#2 and app#3. What we want to know is: the CPU/Memory performance and user number for each app. There are many shell commands for [monitoring Linux performance](http://www.tecmint.com/command-line-tools-to-monitor-linux-performance/), here I use command **top**, **netstat** and **lsof** for getting CPU/Memory usage, connection number (user number) and the name of each app, respectively. 


Using command **top**, we can get the Process ID (PID), CPU/Memory usage for each shiny app.

``` ruby
top -u shiny
``` 

The above command lists CPU/Memory performance for all shiny apps, e.g. 
![]( /images/top.png )
The **PID** should be paid more attention: each shiny app will get a PID, once a shiny app was launched, its PID will not change no matter how many users are using it. 


**netstat** is a command-line tool that displays all network connections. It can list all connections to a shiny app by specifing its Process ID (PID) using command: 

``` ruby
sudo netstat -p | grep 27180
``` 
Note, this command need the user log as an administer of the computer. The above command lists all connections to shiny app with PID 27180.
![]( /images/netstat.png )
We can see, there are 6 connections **ESTABLISHED**.

Now, we have known the CPU/Memory performance and connections number for each shiny app, however, we still don't know the name of each shiny app, which can be get using command **lsof**: 

``` ruby
sudo lsof -p 27180  | grep DIR
``` 

This command lists all directories (a shiny app is a directory) related with PID 27180, we can easily recognize which line contains the shiny app's name. 

![]( /images/lsof.png )

Now, we have got all information we need *manually*, and for sure we need them be generated automatically. The idea is: make a R script to run above shell commands repeatedly, read the outputs of each command into R, extract the information we need and write them into a .RData file. 

``` ruby
## Setup work directory;
setwd("/srv/shiny-system/Data") 
RData <- "sysLoad.RData"
if (!file.exists(RData)) {
  Dat <- NULL
} else {
  load(RData)
}
I <- 0
repeat{
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
    if (!is.null(Dat)) {
      dat.a <- Dat[which(Dat$Time == max(Dat$Time)),]
      con.a <- dat.a$usr[order(dat.a$app)]
      con.b <- dat$usr[order(dat$app)]
      if (paste(con.a, collapse = "") == paste(con.b, collapse = "")) {
        changed <- FALSE
      } else {
        changed <- TRUE
      }
    } else {
      changed <- TRUE
    }
    # Keep only the lines containing important informatin to same storage space;
    if (any(dat$CPU > 5) | any(dat$MEM > 50) | changed) {
      Dat <- rbind(Dat, dat)   
      Dat <- Dat[which(Dat$Time > (max(Dat$Time)-30*24*60*60)), ]
      save(Dat, file = RData)
    }
  }
  Sys.sleep(5)
  I <- I + 5
  if (I >= 60) {break}
}
``` 

**Note!** We can make above code run and never stop, but a safer way is create a Linux task to run above R script as scheduled, so even if the server stopped (e.g. restart), when it run again, it will continue to conduct the shell commands. The shell commands in above code contain *sudo*, which means they will ask user to input a password, that annoying, but we can avoid that by scheduling the task with user name as **root**. 

Use the following command to schedule a task.

``` ruby
sudo nano /etc/crontab
``` 

Add the following task to the end of the opened file (<code>/etc/crontab</code>). The five * in front of the line mean run the R script every minute.

``` ruby
*  *  *  *  * root Rscript /srv/shiny-system/Data/sysLoad.R
``` 

With the generated .RData file, we can create a shiny app to show the CPU/Memory performance and connection number automatically using function **reactiveTimer**  of Shiny package.





