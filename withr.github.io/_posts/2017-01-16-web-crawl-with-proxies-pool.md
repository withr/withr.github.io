---
layout: post
title: "Web scrape using proxies"
date: 2017-01-16 15:27
comments: true
categories: web scrape
---

Though, Ryan Mitchell said web scraping is legal in his book: **Web Scraping with Python Collecting Data from the Modern Web**, we still can read some reports that companies lose the court cases due to they scraped huge data from other companies. So for these who want to scrape a website, I suggest:


1. Read the **robots.txt** file of the web site, to see if the web site allow scraping. **robots.txt** general located under the root directory of a web site, for example, this is Google's robots.txt: https://www.google.com/robots.txt If it said you can scrape, then just go ahead, otherwise see the next suggest. 

2. Most web sites do not like web scrapers, that is because these crawlers may heavy their servers burden. Another reason may they want people visit their data, but don't want people to own ALL their data. How they know you own their data? The only way is you reveal the data to public. Therefore, if you can keep the data secret, I don't think you will get any trouble. 

Let's take [Posten](https://adressesok.posten.no/) as an example. With Posten, we can search all addresses under a postcode, so we can get all addresses of Norway, if we loop all possible postcodes.  Like this url: 
*https://adressesok.posten.no/nb/addresses/search?q=postnummer%3A0192&view=list*, which list all address under postcode *0192*. But is that legal? or does Posten allow to scrape? Let's take a look of it's *robots.txt* (https://adressesok.posten.no/robots.txt): 


~~~~
User-Agent: *
Disallow: /*?

User-agent: *
Crawl-delay: 15
~~~~

It said in the second line, it doesn't allow any scraper to scrape with address contain **?**. But the last line said, if you scrape this web site, the crawl-delay should be longer than 15 seconds, which means you can crawl less than 4 times in a minute, slower than human. I have tried, no matter we crawl using a delay or not, we can only crawl 200 times in one hour, then the server will block the crawler for one hour. The server block the crawler by identifying its IP address, if you want to crawl it still, the only method is using another IP address: **proxy server**. 

You may have read posts about web scraping, and some of them mentioned using free proxies, but I suggest you do not use the free proxies, because they are not stable, and you may waste too many hours on them. Just buy some proxies, like from here: https://buy.fineproxy.org/eng/mini.html. 5 proxies for 10 days cost only $2. 



Here is the source code:


~~~~
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from time import *
import re, sys, os, requests
from bs4 import BeautifulSoup as BS

## Define constant variables;
URL = "https://adressesok.posten.no/nb/addresses/search?view=list&q=postnummer%3A"
wd  = "/home/Posten/pages/"
log = "/home/Posten/log"

~~~~

Python library **requests** is the most popular web scraping library, while **BeautifulSoup** is the easies library for parsing a HTML page. 


~~~~
## Record log
def write_log(postnr, msg):
    log = "/home/tian/Posten/log"
    with open(log, "a") as l:
        l.write(postnr + " " +  msg + " " + strftime("%Y-%m-%d %H:%M:%S", gmtime()) + "\n")
        l.close()
    print  "\rPostnumber: " + postnr + " " + msg + "!"

~~~~

A log file is always necessary, because we can't predict what will happen in future, for example, power off/network disconnected, etc. 


~~~~
proxies_ls = [
    "https://37.9.45.242:8085",
    "https://91.243.94.120:8085",
    "https://93.179.89.230:8085",
    "https://146.185.204.86:8085",
    "https://91.243.89.89:8085"
]   
~~~~

Above is the proxies list I bought (they are all expired). 

~~~~
def download_page(postnr):
    url = URL + postnr
    notDownloaded = True
    t1 = time()
    while notDownloaded:
        proxies = {"https" : proxies_ls[id % 5]}
        r = requests.get(url, proxies = proxies, timeout=5)
        sleep(1)
        html = r.content
        bs = BS(html)
        try:
            search_count = bs.find("h2", {"class": "search_count"}).get_text().encode('utf8', 'replace')
            search_count = re.sub("^ *", "", search_count)
            search_parse = search_count.split(" ")
            if search_parse[1] == "adresser" and search_parse[0] != "0":
                file_name = wd + postnr + ".html"
                with open(file_name, "w") as f:
                    f.write(html)
                    f.close()
                write_log(postnr, "OK")
            else:
                write_log(postnr, "NoAddress")
            notDownloaded = False
        except:
            global id
            id = id + 1
            sleep(1)
            msg = "\rPostnumber " + postnr + " try later!" + " waited " + str(int(time()-t1)) + " seconds."
            sys.stdout.write(msg); sys.stdout.flush()
~~~~  

The function for downloading a page to local file. There is only one argument: postnr, which should be a string. **id** was used to choose which proxy to use, if one proxy was banned, the next proxy will be used, and this was conducted by adding one to global variable **id**.  



~~~~        
            
## postnr_all
p1 = int(sys.argv[1])
p2 = int(sys.argv[2])
postnr_all = []
xx = range(p1, p2)
for i in range(len(xx)):
    postnr_all.append('{:04d}'.format(xx[i]))

## postnr_downloaded
postnr_downloaded = []
f = open('/home/tian/Posten/log', 'r')
for line in f:
    postnr_downloaded.append(line[0:4])

f.close()

## postnr_todownload
postnr_todownload = list(set(postnr_all) - set(postnr_downloaded))
postnr_todownload.sort()

print "postnr_all: " + str(len(postnr_all))
print "postnr_downloaded: " + str(len(postnr_downloaded))
print "postnr_todownload:" + str(len(postnr_todownload))

~~~~ 

Above code was designed for unexpected collapse. Every time start the process, above code will check which postcodes have already been downloaded, and create a new postcode list to be downloaded. 



~~~~ 
id = 0
for postnr in postnr_todownload:
    download_page(postnr)
~~~~  

Start the downloading, and **id** could be any integer. 

  



