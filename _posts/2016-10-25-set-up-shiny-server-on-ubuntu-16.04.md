---
layout: post
title: "Setup Shiny Server on Ubuntu 16.04"
date: 2016-10-25 10:27
comments: true
categories: R Shiny
---

I have written two blogs recording my experience of installing shiny-server on Ubuntu [12.04](http://withr.me/set-up-highcharts-export-server-on-ubuntu-server-12-dot-04-step-by-step/) & [14.04](http://withr.me/set-up-shiny-server-on-ubuntu-14-dot-04/), now it's the turn of Ubuntu 16.04. There is a little difference, just for memory. 

<!-- more -->


## Update system (not mandatory) 

~~~~
sudo apt-get update
~~~~

And upgrade the server, which will make sure install the latest version of R.

~~~~
sudo apt-get upgrade
~~~~


## Install R and shiny package

 Before installing Shiny Server, we need to install R and the Shiny package first. 

- To install the latest version of R, we need to add the CRAN repository to our system's sources list: **/etc/apt/sources.list**. 

~~~~
sudo nano /etc/apt/sources.list
~~~~

Add <code>deb http://cran.uib.no/bin/linux/ubuntu xenial/</code> to its end, and save the modification. Where *cran.uib.no* is the CRAN mirror for Norway (so you may need to change it to the mirror suitable for you) and **xenial** is the version name for Ubuntu 16.04.

- Install R using the following command: 

~~~~
sudo apt-get install -y r-base
~~~~

This process would take one minute. 

- Install the **Shiny** R package.

~~~~
sudo su - -c "R -e \"install.packages('shiny', repos='http://cran.rstudio.com/')\""
~~~~

This process could take one and half minute, because many dependent packages were also installed. 

- Install Shiny Server

~~~~
sudo apt-get install -y gdebi-core
wget http://download3.rstudio.org/ubuntu-12.04/x86_64/shiny-server-1.3.0.403-amd64.deb
sudo gdebi shiny-server-1.3.0.403-amd64.deb
~~~~

This process could take one and half minute. By default, Shiny Server will run automatically after installation, you will see a welcome page on port 3838 of your server's IP address, e.g. "http://192.168.202.16:3838/"

Installation of Shiny Server finished!


## Install R packages. 

 - Before starting to install R packages using install.packages() under R, we need to install several dependencies or environment first.


~~~~
sudo apt-get install -y libcurl4-openssl-dev  libssl-dev libxml2-dev
~~~~

where **libcurl4-openssl-dev** may needed in installing R package "RCurl", **libssl-dev** may needed in installing R package "devtools", and **libxml2-dev** may needed in installing R package "XML". 
 

 
- Install **Java** used by R package *rJava* and add a system environment variable LDLIBRARYPATH.

~~~~
sudo apt-get install -y default-jdk
export LD_LIBRARY_PATH=/usr/lib/jvm/java-8-openjdk-amd64/jre/lib/amd64/server/
sudo R CMD javareconf  
~~~~



- Install the following R packages:

~~~~
install.packages(c('RJDBC', 'XLConnect', 'devtools', 'RJSONIO'))
require(devtools)
install_github('rCharts', 'ramnathv')
~~~~


## Install RStudio Server

~~~~
wget https://download2.rstudio.org/rstudio-server-0.99.903-i386.deb
sudo gdebi rstudio-server-0.99.903-i386.deb
~~~~

The RStudio-server will run on port 8787 of your server's IP address. Use your user name and password to log onto it. 
