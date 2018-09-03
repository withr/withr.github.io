---
layout: post
title: "Setup Selenium headless (without display) on Ubuntu Server 16.04"
date: 2016-10-28 12:27
comments: true
categories: Linux
---


Data management is the first skill of a data scientist, in my opinion. The management task generally includes import data from one form into another one (generally a database). But sometimes, we need to collect the data ourselves, for example from Internet. 

The data on internet generally stored as HTML, XML, JSON, etc. Web pages can be classified into two categories: static and dynamic pages. There are a lot of [softwares/libraries/packages](http://www.octoparse.com/blog/top-30-free-web-scraping-software/) for extracting data from static web pages, however for dynamic pages, there is only several tools (Selenium/PhantomJS) available. Among them, Selenium is the most well knowned. 

I have been using the package *XML*, *Rcurl* of R to extract data from static web pages for years, I tried several times to extract information from dynamic web pages, but found no lucky. Now I feel more and more interesting data stored in dynamic web pages, so I gave myself a push to handle it. There is a R package called [**RSelenium**](https://github.com/ropensci/RSelenium), which can driver a web browser and extract data from dynamic web pages. Unfortunately, I can’t make it work on my Linux server. (It works on my Windows PC). 

I spend a lot time to setup the Selenium on my Ubuntu 16.04, but never get succeeded until I realized that I have no display device (screen). Selenium use web browser to render web page content, and web browser need a display device to show the content. Then, what we do? Can we use selenium with display? Sure, use a virtual display device.



## Step 1: install selenium

~~~~
//sudo apt-get install python-pip
//sudo pip install selenium
sudo apt install python3-selenium 
~~~~



Step 2: install geckodriver

~~~~
wget https://github.com/mozilla/geckodriver/releases/download/v0.11.1/geckodriver-v0.11.1-linux64.tar.gz
tar -zxvf geckodriver-v0.11.1-linux64.tar.gz
sudo mv geckodriver /usr/bin/

~~~~

Above command download and unzip **deckodriver**. The third line command move the excutable deckodriver to a directory where it located in computer's environment list. So, instead move it */usr/bin/*, you can add its path to system variable **PATH**: <code>export PAHT=$PATH:directory where geckodriver located.
</code>



## Step 3: install Xvfb & PyVirtualDisplay

**Xvfb** is a software that simulates a display doing everything in memory and not showing any screen output, while **PyVirtualDisplay** is a Python wrapper for Xvfb. It allows you to easily work with a virtual display in Python[ref](http://scraping.pro/use-headless-firefox-scraping-linux/). Use the following command to install them. 


~~~~
sudo apt-get install xvfb
sudo pip install pyvirtualdisplay
~~~~


OK, now open python in the terminal, and run the following script to test wheter it work.

~~~~
from pyvirtualdisplay import Display
from selenium import webdriver


display = Display(visible=0, size=(1920, 1080)).start()
browser = webdriver.Firefox()
browser.get("https://groups.google.com/forum/?hl=en#!forum/shiny-discuss")
print browser.title.encode('utf8', 'replace')

~~~~



## Another way: PhantomJS

Above step let Firefox run without display through Xvfb, so now the firefox is actually a headless web browser. But there is another web browser is designed as headless: (PhantomJS)[http://phantomjs.org/]. If you are familiar with PhantomJS, you can instead it, instead of running commands in above step.


~~~~
wget https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2
tar xjf phantomjs-2.1.1-linux-x86_64.tar.bz2
export PATH=$PATH:<DIRECT>/phantomjs-2.1.1-linux-x86_64/bin
~~~~

Run the following script to tes:

~~~~
from selenium import webdriver
browser = webdriver.PhantomJS()
browser.get("https://groups.google.com/forum/?hl=en#!forum/shiny-discuss")
print browser.title.encode('utf8', 'replace')

~~~~



