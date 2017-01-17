---
layout: post
title: "Multiprocessing Web crawler with proxies"
date: 2017-01-17 15:27
comments: true
categories: web scrape
---


Multiprocessing is used to accelerate the scraping of web pages, while using proxies can hide your IP address, and avoid the server ban your crawlers. This post take an example to show you how to write such a crawler in Python. 

There are two popular libraries in Python which can be taken as parallelized calculation libraries: **[multiprocessing over threading](http://stackoverflow.com/questions/3044580/multiprocessing-vs-threading-python)**, both have pros and cons. In this post I use **multiprocessing**.


~~~~
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import time, re, sys, os, random, requests
from multiprocessing.dummy import Pool 
from bs4 import BeautifulSoup as BS

## Define constant variables;
URL = "https://m.finn.no/lookup.html?finnkode="
dir_prj = "/home/HDD1T/Finn/"

## Generate directory and log
pag = dir_prj + "pages_" + arg + "/"
log = dir_prj + "log_" + arg

if not os.path.exists(pag):
    os.makedirs(pag)

if not os.path.exists(log):
    os.system("touch " + log)

## All proxies expired
proxies_ls = [
    "https://91.20.3.107:8085",
    "https://91.23.94.107:8085",
    "https://91.24.14.113:8085",
    "https://146.15.204.70:8085",
    "https://193.10.171.7:8085"] 

~~~~

Above codes load libraries, prepare director and log file, and proxies pool. 


~~~~
####### Record log
def write_log(finn_code, msg):
    with open(log, "a") as l:
        l.write(str(finn_code) + " " +  msg + " " + time.strftime("%Y-%m-%d %H:%M:%S", time.gmtime()) + "\n")
        l.close()
    print  "\rPostnumber: " + str(finn_code) + " " + msg + "!"

~~~~

Log recording function, which is necessary for monitoring and error tracking.


~~~~
def download_page(finn_code):
    url = URL + str(finn_code)
    try:
        proxy = {"https" : proxies_ls[id % len(proxies_ls)]}
        r = requests.get(url, proxies = proxy, timeout=5)
        if r.status_code == 200:
            html = r.content
            bs = BS(html)
            try:
                div_menu = bs.find("div", {"class": re.compile("flex-grow1 truncate mrl flex.*")}).findAll("a")
                dir_menu = div_menu[0].get_text().encode('utf8', 'replace')
                if (len(div_menu) >= 2):
                    for m in range(1, len(div_menu)):
                        dir_menu = dir_menu + "/" + div_menu[m].get_text().encode('utf8', 'replace')
                file_name = pag + dir_menu + "/" + str(finn_code) + ".html"
                ## If the directory not exist, then create first;
                if not os.path.exists(os.path.dirname(file_name)):
                    os.makedirs(os.path.dirname(file_name))
                ## Save HTML file to disk
                with open(file_name, "w") as f:
                    f.write(html)
                    f.close()
                ## Random sleep seconds
                time.sleep(1)
                write_log(finn_code, "OK " + file_name)    
            except:
                write_log(finn_code, "ParseError")  
        else:
            write_log(finn_code, "NotExist")
    except:
        write_log(finn_code, "RequestError")   
        global id
        id = id + 1
        sleep(1)

~~~~

The main download function. This function use **id** to determine which proxy to be used, if the proxy was unexpected fail, and then use the next one. This was done by checking if the request to the web page encounts an error, and declare id as global variable. 


~~~~
## Initialize id
id = 1

## Unit: 10 million
codeLen = 10000000
arg = sys.argv[1]
codeFirst = codeLen * (int(arg) + 0)
codeLast  = codeLen * (int(arg) + 1) 


## Total codes;
codes_all = range(codeFirst, codeLast)
## Tried codes;
codes_tried = []
with open(log, "r") as f:
    for line in f:
        codes_tried.append(int(line[0:8]))
    f.close()
## Codes not tried;
codes_untried = list(set(codes_all) - set(codes_tried))


codes_downloaded = []
with open(log, "r") as f:
    for line in f:
        if line[9:10] == "O" or line[9:10] == "N":
            codes_downloaded.append(int(line[0:8]))
    f.close() 
codes_redownload = list(set(codes_all) - set(codes_downloaded))
~~~~

Above codes calculate two import array: **codes_untried** & **codes_redownload**. **codes_untried** are the codes, that the crawler never tried to download, and **codes_redownload** are the codes which encounted an error when downloading, or not tried. When the crawler starts, all codes are **codes_untried** and **codes_redownload**. If all codes have been tried, then **codes_redownload** are those that failed.

~~~~   
if len(codes_untried) > 0:
    print "There are " + str(len(codes_untried)) + " codes need to download!"
    time.sleep(5)    
    pool = Pool(8)
    pool.map(download_page, codes_untried) 
    pool.close()  
    pool.join() 
elif len(codes_redownload) > 0:
    print "There are " + str(len(codes_redownload)) + " codes need to redownload!"
    time.sleep(5)   
    pool = Pool(8)
    pool.map(download_page, codes_redownload) 
    pool.close()  
    pool.join()
else:
    print "All codes have been downloaded!"

~~~~

Schedule all above script to run, like each hour will first go through all codes, then check those codes failed, if all codes have successfully downloaded, the script just print: "All codes have been downloaded!". 

The schdule file can be writen like the following, to avoid open repeat processes. 

~~~~
#!/bin/bash

n1=`wc -l /home/tian/HDD1T/Finn/log_$1`
sleep 5
n2=`wc -l /home/tian/HDD1T/Finn/log_$1`

if [ "$n1" == "$n2" ]; then
    python /home/tian/HDD1T/Finn/crawler.py $1
    echo "Start crawler.py $1"
fi

~~~~

