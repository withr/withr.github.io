---
layout: post
title: "Scrape dynamic web page (Google Group forum) using Selenium (2)"
date: 2016-10-28 16:27
comments: true
categories: Linux
---

In previous post, I described how to download all topics pages of Shiny Google Group. The download topics are in html format, in this post, I will describe how to extact useful information, and import the result into R.

There are several packages can do the information extraction from HTML file in python, like **BeautifulSoup**, **Scrapy**, **lxml**, etc. I like **BeautifulSoup** because it's easy to use and its package name is the shortest: "bs4" :-)

The codes for extracting information in python is quite simple, just remember to add codes to handle exception, error log and monitor the process. I save all topics into a list (topics), for each item of the list is another list(topic), for the topic list, each item is a dictory. Once all information extracted, topics list was save as binary using python package: **pickle**

Here is the codes:


~~~~
from bs4 import BeautifulSoup as BS
import time, re, os, pickle, sys

wd = "/home/tian/shinyExpert/"
path = wd + "topics/"
HTML = os.listdir(path)
HTML.sort()
topics = []

for i in range(0, len(HTML)):
    ## print f
    f = HTML[i]
    bs = BS(open(path + f).read())
    ## title
    try:
        title = bs.find("span", {"id": "t-t"}).get_text()
    except:
        title = ""
        print "\nWarning: file " + f + " has no title!"
    threads = [{"id": f,"title": title}]
    ## threads
    thread_items = bs.findAll("div", {"class": "IVILX2C-tb-x"})
    N = len(thread_items)
    if (N > 0):
        for n in range(0, N):
            try:
                author  = thread_items[n].find("table", {"class": "IVILX2C-tb-O"}).find("td", {"align": "left"}).get_text()
            except:
                author = ""
                print "\nWarning: index = " + str(i) + "; file " + f + " has no author!"
            try:
                td_date = thread_items[n].find("table", {"class": "IVILX2C-tb-O"}).find("td", {"align": "right"})
                date = td_date.find("span", {"class": "IVILX2C-tb-Q IVILX2C-b-Cb"})["title"]
            except:
                date = ""
                print "\nWarning: index = " + str(i) + "; file " + f + " has no date!"
            try:
                content = thread_items[n].find("div", {"class": "IVILX2C-tb-P"}).find("div", {"dir": "ltr"}).get_text()
            except:
                content = ""  
                print "\nWarning: index = " + str(i) + "; file " + f + " has no content!"
            thread = {"author": author, "date":date, "content":content}
            threads.append(thread)        
    else:
        with open(wd + 'log_extract.txt', "w") as log:
            log.write(f + "\n")
            log.close()
        print "\nError: file " + f + " wasn't downloaded completely!"
    topics.append(threads)
    msg = "\rProcessed " + str(i+1) + " files: " + f
    sys.stdout.write(msg); sys.stdout.flush()




dat_topics = wd + "topics.dat"
pickle.dump(topics, open(dat_topics, "wb"))
#topics = pickle.load(open(dat_topics, "r"))

~~~~

I still prefer to use R to manapulate data, to load the python binary data into R, we can use the [rPython](https://cran.r-project.org/web/packages/rPython/rPython.pdf) package:


~~~~
library(rPython)
python.exec("import pickle; topics = pickle.load(open('~/shinyExpert/topics.dat', 'r'))")
topics <- python.get("topics")
length(topics)



~~~~




