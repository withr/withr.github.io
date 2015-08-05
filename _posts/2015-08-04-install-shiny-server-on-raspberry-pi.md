---
layout: post
title: "Install Shiny Server on Raspberry Pi"
date: 2015-08-04 11:43
comments: true
categories: RPi R Shiny
---

## Not verified!!

Several people asked me if I ever installed Shiny Server on Raspberry Pi (RPi) because I introduce myself as a "R and Raspberry Pi fan" on my [blogs](http://withr.me/). Though I don't think RPi is suitable to hold a Shiny Server due to its low memory size, but other guys may have some good reason to do so, especially that RPi is getting more and more powerful. So I wrote this step-by-step guide to share my experience. 



### In short: **Yes!** It's possible to install Shiny Server on RPi. 

#### Note: the installing process may take up to 10 hours and to open a shiny application may take up to 30 seconds!

### Devices needed
Besides the Raspberry Pi (B+) and a 5V2A micro USB power cable, you need also the following devices: 

  - A Micro SD card (I just have a 32GB card and I didn't test smaller ones)
  - A Micro SD card reader
  - An Ethernet cable
  - A WiFi adapter (Not necessary if you can get your RPi connect to Internet through Ethernet cable)
  
### Prepare your RPi
  - Please go through this [blog](http://withr.me/get-access-to-raspberry-pi-without-screen/) to get your RPi connected to Internet. 
  - When your RPi get access to Internet, update it first using command: 
  
  ```
  sudo apt-get update
  ```
  
  - Then we need to extend the storage space of RPi and reboot it.

  ```
  sudo raspi-config

  ```
  
  
### Install R from source
To install R using official recommandation for *[Debian] (https://cran.r-project.org/bin/linux/debian/)* will fail. Because for Raspbain (which based on Debian), the defualt *r-base-core* is version 2.15.1-4, while it needs a higher version to install *r-base* under *Wheezy*. The simplest way to install R is install it from source, though it will take more than two hours.

 - **Install dependencies and reboot**. 
  
  ```
  sudo apt-get install -y gfortran libreadline6-dev libx11-dev libxt-dev libcairo2-dev
  ```
  The above installation may used most of the memory, you can simply release the memory by rebooting you RPi.
  
  
 - **Install R**. You can paste all the following commands into the terminal, then check back after two hours.
 
  ```
  wget http://cran.rstudio.com/src/base/R-3/R-3.1.2.tar.gz 
  tar zxvf R-3.1.2.tar.gz; cd R-3.1.2/ 
  ./configure; make; 
  sudo make install
   
  ```
 After the installation finished, type **R** in the terminal to check if R has been installed successfully.
 

  
### Install *shiny* package
  We can't install R packages in R console like what we usrally do. The reason is RPi has a small memory which is not enough to conduct such kind of installation. Instead, we can install them from their sources. The *shiny* package has several dependent packages, and we need to install them first before installing *shiny*. Download them first: 
  
  ```
  wget https://cran.r-project.org/src/contrib/Rcpp_0.12.0.tar.gz
  wget https://cran.r-project.org/src/contrib/httpuv_1.3.3.tar.gz
  wget https://cran.r-project.org/src/contrib/mime_0.3.tar.gz
  wget https://cran.r-project.org/src/contrib/jsonlite_0.9.16.tar.gz
  wget https://cran.r-project.org/src/contrib/digest_0.6.8.tar.gz
  wget https://cran.r-project.org/src/contrib/htmltools_0.2.6.tar.gz
  wget https://cran.r-project.org/src/contrib/xtable_1.7-4.tar.gz
  wget https://cran.r-project.org/src/contrib/R6_2.1.0.tar.gz
  wget https://cran.r-project.org/src/contrib/Cairo_1.5-8.tar.gz
  wget https://cran.r-project.org/src/contrib/shiny_0.12.1.tar.gz
   
  ```
  Install above packages from command line:
  
  ```
   sudo R CMD INSTALL Rcpp_0.12.0.tar.gz
   sudo R CMD INSTALL httpuv_1.3.3.tar.gz
   sudo R CMD INSTALL mime_0.3.tar.gz
   sudo R CMD INSTALL jsonlite_0.9.16.tar.gz
   sudo R CMD INSTALL digest_0.6.8.tar.gz
   sudo R CMD INSTALL htmltools_0.2.6.tar.gz
   sudo R CMD INSTALL xtable_1.7-4.tar.gz
   sudo R CMD INSTALL R6_2.1.0.tar.gz
   sudo R CMD INSTALL Cairo_1.5-8.tar.gz
   sudo R CMD INSTALL shiny_0.12.1.tar.gz 
    
  ```
  
  Check whether the *shiny* package was installed successfully in R console: 
  
  ```
  library(shiny)
  ```
  
### Install cmake 

The default *cmake* on RPi is version: , while to install Shiny Server we need a higher version (This step will also take a long time):

```
wget http://www.cmake.org/files/v2.8/cmake-2.8.11.2.tar.gz
tar xzf cmake-2.8.11.2.tar.gz
cd cmake-2.8.11.2
./configure; make; 
sudo make install
 
```
  
### Install Shiny-server
Just follow the instruction on RStudio's offical [website](https://github.com/rstudio/shiny-server/wiki/Building-Shiny-Server-from-Source): 

```
git clone https://github.com/rstudio/shiny-server.git
cd shiny-server; DIR=`pwd`; PATH=$DIR/bin:$PATH
mkdir tmp; cd tmp; PYTHON=`which python`
cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DPYTHON="$PYTHON" ../
make; mkdir ../build
(cd .. && ./bin/npm --python="$PYTHON" rebuild)
(cd .. && ./bin/node ./ext/node/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js --python="$PYTHON" rebuild)
sudo make install
 

```

### Download Shiny Server default configure [file](https://github.com/rstudio/shiny-server/blob/master/config/default.config)


```
cd /etc/shiny-server
wget http://withr.me/misc/shiny-server.conf

```

### Add your shiny appliations.
Create a demo shiny-app in **/srv/shiny-server**, like [this](http://shiny.rstudio.com/gallery/kmeans-example.html): 
  
```
cd /srv/shiny-server
sudo mkdir kmeans
cd kmeans
sudo nano ui.R
sudo nano server.R

```
### Change file permissions to let Shiny Server get access.


```
sudo chmod 766 -R /srv
sudo shiny-server

```

Now open your web browser and type: RPi-IP:3838 in its address input. 
### Good luck!
  
  
  
  
  