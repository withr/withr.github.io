---
layout: post
title: "Setup Shiny Server on Ubuntu 14.04"
date: 2015-03-16 10:27
comments: true
categories: R Shiny
---

I have set up a Shiny Server on Ubuntu 12.04 in 2013. [Shiny](http://shiny.rstudio.com/) package has been under active development, and current version has reached **0.11** while the one on my server is still **0.8**. I should keep update with Shiny package's development, however, from version 0.8, Shiny package has changed a lot, and I have to modify all my apps to get them work which is huge task for me. Now, I have to update them because the topics discussed on [Shiny's Google group](https://groups.google.com/forum/#!forum/shiny-discuss) is not concerning my server any more :-( 


## Install Ubuntu 14.04 server. 

This progress was conducted by our IT administer, I will not describe the detail here. 


## Setup locale 

**Note: This step must be done before installing Shiny Server**

For detail description about how to set up locale on Linux, please have a look of this [post](http://withr.me/configure-character-encoding-for-r-under-linux-and-windows/). I just show how to install Norwegian locale and set it as the default locale. 

- Generate Norwegian locale (Bokmål ISO-8859-1).

```
sudo locale-gen nb_NO
sudo update-locale
locale -a
```
If you can see **nb_NO.iso88591**, then the Norwegian locale has been generated and installed successfully. 

- Set **nb_NO.iso88591** as the default locale by editing <code>sudo nano /etc/default/locale</code> to: 

```
LANG="nb_NO.ISO-8859-1"
LANGUAGE="nb_NO.ISO-8859-1"
LC_ALL="nb_NO.ISO-8859-1"

```

- Restart the server <code>sudo reboot</code>, and check the locale using <code>locale</code>. It should look like: 

```
LANG=nb_NO.ISO-8859-1
LANGUAGE=nb_NO.ISO-8859-1
LC_CTYPE="nb_NO.ISO-8859-1"
LC_NUMERIC="nb_NO.ISO-8859-1"
LC_TIME="nb_NO.ISO-8859-1"
LC_COLLATE="nb_NO.ISO-8859-1"
LC_MONETARY="nb_NO.ISO-8859-1"
LC_MESSAGES="nb_NO.ISO-8859-1"
LC_PAPER="nb_NO.ISO-8859-1"
LC_NAME="nb_NO.ISO-8859-1"
LC_ADDRESS="nb_NO.ISO-8859-1"
LC_TELEPHONE="nb_NO.ISO-8859-1"
LC_MEASUREMENT="nb_NO.ISO-8859-1"
LC_IDENTIFICATION="nb_NO.ISO-8859-1"
LC_ALL=nb_NO.ISO-8859-1
```
## Install R and shiny package

 Open [Shiny Server's installation page](http://www.rstudio.com/products/shiny/shiny-server/), and navigate to [Download Shiny Server](http://www.rstudio.com/products/shiny/download-server/). It said: "Before installing Shiny Server, you'll need to install R and the Shiny package". 

- To install the latest version of R, we need to add the CRAN repository to our system. For Ubuntu 14.04: open **/etc/apt/sources.list** using <code>sudo nano /etc/apt/sources.list</code>, and add this line to its end: <code>deb http://cran.uib.no/bin/linux/ubuntu trusty/</code>, save and exit *nano*. **trusty** is the version name for Ubuntu 14.04. 

- Install R using the following command: 

```
sudo apt-get install r-base

```

This process would take one minute. 

- Install the **Shiny** R package.

```
sudo su - \
-c "R -e \"install.packages('shiny', repos='http://cran.rstudio.com/')\""
```

This process could take one and half minute, because many dependent packages were also installed. 

- Install Shiny Server

```
sudo apt-get install gdebi-core
wget http://download3.rstudio.org/ubuntu-12.04/x86_64/shiny-server-1.3.0.403-amd64.deb
sudo gdebi shiny-server-1.3.0.403-amd64.deb
```

This process could take one and half minute. By default, Shiny Server will run automatically after installation, you will see a welcome page on port 3838 of your server's IP address, e.g. "http://192.168.202.16:3838/"

Installation of Shiny Server finished!


## Install R packages. 

 Before starting to install R packages using install.packages() under R, we need to install several dependencies or environment first.
  
- Install **curl** library used by R package *RCurl*:
  
```
  sudo apt-get install libcurl4-openssl-dev 
  
```
  
- Install **Java** used by R package *rJava* and add a system environment variable LDLIBRARYPATH.

```
sudo apt-get install openjdk-7-jdk
export LD_LIBRARY_PATH=/usr/lib/jvm/java-7-openjdk-amd64/jre/lib/amd64/server
R CMD javareconf  # Let R know the configuration of Java;
```



- When install the Shiny R package, it was installed at "/usr/local/lib/R/site-library" be default, however, the files under this directory is not allowed to write for normal users. So, in order to install packages to that directory, we need to start R with **sudo**: <code> sudo R </code>, and specify the option **lib**, like: 

```
install.packages("devtools", lib = "/usr/local/lib/R/site-library")
```

**Or**, we can set the default library path, so we don't need to type that option every time.

Use the following command to open and edit */usr/lib/R/etc/Renviron*

```
sudo nano /usr/lib/R/etc/Renviron
```
Comment the line: *R_LIBS_USER=${R_LIBS_USER-'~/R/x86_64-pc-linux-gnu-library/3.0'}* by adding a "#" at the beginning of that line. 

Now back to R, we will found the first library path is: */usr/local/lib/R/site-library* using <code>.libPaths()</code>. 



```
> .libPaths()
[1] "/usr/local/lib/R/site-library" "/usr/lib/R/site-library"
[3] "/usr/lib/R/library"
```



## Configure Shiny Server. 

You don't need to modify the configure file, but if you need, edit this file */etc/shiny-server/shiny-server.conf* like the following: 

```
# Instruct Shiny Server to run applications as the user "shiny"
run_as shiny;

# Define a server that listens on port 3838
server {
  listen 3838;

  # Define a location at the base URL
  location / {

    # Host the directory of Shiny Apps stored in this directory
    site_dir /srv/shiny-demo;

    # Log all Shiny output to files in this directory
    log_dir /var/log/shiny-server;

    # When a user visits the base URL rather than a particular ap$
    # an index of the applications available in this directory wi$
    directory_index on;
  }
}

server {
  listen 80;

  # Define a location at the base URL
  location / {

    # Host the directory of Shiny Apps stored in this directory
    site_dir /srv/shiny-server;

    # Log all Shiny output to files in this directory
    log_dir /var/log/shiny-server;

    # When a user visits the base URL rather than a particular ap$
    # an index of the applications available in this directory wi$
    directory_index on;
  }
}

```

## Add shiny-apps

By default, server directory is not allowed to write for normal user, we can change its mode: 


```
sudo chmod -R 777 /srv

```



## Install RStudio Server

```
wget http://download2.rstudio.org/rstudio-server-0.98.1103-amd64.deb
sudo gdebi rstudio-server-0.98.1103-amd64.deb

```
The RStudio-server will run on port 8787 of your server's IP address. Use your user name and password to log onto it. 

You may the some errors, like "ERROR: no permission to install to directory '/usr/local/lib/R/site-library'", that's because RStudio has no writing access to R's site-library directory. We can change the mode of all files in that directory to **777**, so any user can has full access to that directory. 

```
sudo chmod -R 777 /usr/local/lib/R

```
