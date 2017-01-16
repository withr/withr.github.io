---
layout: post
title: "Make Your Date Folder Clean with Function unzip & unz"
date: 2013-02-26 17:52
comments: true
categories: R
---

I am a somewhat minimalist R user. I feel uncomfortable if something is not in a good order, such as the names of variables and documents, the structures of my codes and projects.  I prefer my data stored in *.txt* or *.csv* so I can load them to R using *read.table* or *read.csv*. For most of the time we got along well, until I got a huge number of *.txt* files. One of my research need to assign oxygen density value to our field observation. There are more than 600 oxygen files with total size round 1GB for different periods. It's annoying because: first, they occupy a lot of space, even larger than 90 percent of the whole project; second, it's time consuming when you copy or synchronize them to cloud server, like *Google Drive*.



At last, I found one way to deal with such problem: using the native functions **unzip** and **unz** of R. What you need to do is compress all *.txt* files into a *.zip* file. Here is an example: suppose you have compressed all your *.txt* files into a *.zip* file named "TSOC 1961 2010.zip";

``` ruby 
## List all files names inside of a .zip file;
file_ls <- as.character(unzip("TSOC_1961_2010.zip", list = TRUE)$Name)
## Read each .txt file into R;
for (i in file_ls) dat <- read.table(unz("Material/TSOC_1961_2010.zip", i))
```

Now, 600 files came to one file, size decreased to 100 MB, no more code lines added in the script. More important, it made my mind clean and conveniented project management.

R always surprise me!
