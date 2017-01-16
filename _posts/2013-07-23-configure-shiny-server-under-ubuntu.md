---
layout: post
title: "Configure Shiny-server under Ubuntu 12.04"
date: 2013-07-23 15:55
comments: true
categories: Shiny R
---
[Shiny](http://www.rstudio.com/shiny/) is a R package allows you to create interactive web-based application which is getting more and more [popular](http://www.r-bloggers.com/?s=shiny) in R communities. It makes creating web application super simple if you are familiar with R. Though it requires no knowledge about HTML, CSS or JavaScript, but if you are familiar with them, that's definitely a plus. [Shiny-server](https://github.com/rstudio/shiny-server) is a server program that makes *Shiny* applications available on web, currently it works only on Linux systems.


Recently, I built a *Shiny* application to monitor the progress of data processing in our institute. The function of this application is: connect to our [Firebird](http://www.firebirdsql.org/) databases and Excel documents to fetch data, analyze the data and display the result. To connect Firebird database with R, I use the R package [RJDBC](http://cran.r-project.org/web/packages/RJDBC/index.html), while connect Excel documents, I use the R package [XLConnect](http://cran.r-project.org/web/packages/XLConnect/index.html). The *Shiny* application runs well locally on my Windows 7, however I need it be run as a server, which means it should be run under a Linux. I am not a Linux user, though I tried several times to get familiar with [Ubuntu](http://www.ubuntu.com/). To me, it's not easy to use R under Linux, even the installation of R is much difficult than under Windows. I guess that there are many R users like me, not familiar with Linux environment, so I write this blog to record how I get my *Shiny* application works under *Ubuntu 12.04 LTS*.


## 1. Install Ubuntu under Windows. 
[Wubi](http://www.ubuntu.com/download/desktop/windows-installer) is a software that allows you to install *Ubuntu* under a Windows system. Install *Ubuntu* under Windows is just like install normal software: download and install it by double clicking and follow the guide. *Wubi* will create a folder named 'ubuntu' in 'C' disk, currently it installs **Ubuntu 12.04**. To remove *Ubuntu* is also very easy, like uninstall a normal software using *Programs and Features* in *Control Panel*. Restart the computer to log in *Ubuntu*. The first time to use *Ubuntu*, I suggest update it first by typing the following code in terminal (Note: press 'Ctrl+Alt+T' to open terminal console). <code>
sudo apt-get update</code>. If the sound of computer's fan is too aloud, then we need update a driver. Click the *Dash Home* at the left-top corner of the desktop, and type 'Additional  Driver' in the search bar, select and run it. In the pop up window, active 'Experimental AMD binary Xorg driver and kernel model'. When the installation finished, restart the computer. 


## 2. Install R on Ubuntu. 
Unlike other Linux, we need to follow the following steps to install R on Ubuntu:

``` ruby   
gpg --keyserver keyserver.ubuntu.com --recv-key E084DAB9
gpg -a --export E084DAB9 | sudo apt-key add -
```

Open *sources.list*, and add the repository *deb http://cran.uib.no/bin/linux/ubuntu precise/* to its end, where *precise* means the R version for Ubuntu 12.04. Note, if you use 'nano' as the editor, press *Ctrl+X* to exit nano editor and press *Y* to save the change, then press *Enter*.

``` ruby
sudo nano /etc/apt/sources.list
```

Update *Ubuntu*, and install R.

``` ruby
sudo apt-get update
sudo apt-get install r-base r-base-dev
```

To test the installation, just type <code>R</code> in the terminal, if the *R console* appeared, then installation successes.	


## 3.	Install R packages. 
Before starting to install R packages using *install.packages()* under R, we need to install several dependencies first.

Install **curl** library used by R package *RCurl*:

``` ruby
sudo apt-get install libcurl4-openssl-dev 
```

Install Java used by R package *rJava* and add a system environment variable *LD_LIBRARY_PATH*. Note: install version **6**, because *rJava* uses this version.

``` ruby
sudo apt-get install openjdk-6-jdk
export LD_LIBRARY_PATH=/usr/lib/jvm/java-6-openjdk-amd64/jre/lib/amd64/server
R CMD javareconf # Let R know the configuration of Java;
```

Install R packages under R, which means we enter *R console* first by typing  <code>R</code> in the terminal.

``` ruby
install.packages(c('RJDBC', 'XLConnect', 'devtools', 'RJSONIO'))
require(devtools)
install_github('rCharts', 'ramnathv')
```


## 4. Install Node.js.

``` ruby
sudo apt-get update
sudo apt-get install software-properties-common python-software-properties python g++ make
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
```

## 5. Install Shiny in system-wide library. 
**Note**: If this step fails, very possibly due to some dependent packages were not installed successfully, which can be re-installed using the following command, just replace 'shiny' with the package's name.

``` ruby
sudo su - -c "R -e \"install.packages('shiny', repos='http://cran.rstudio.com/')\""

sudo npm install -g shiny-server
```

## 6. Install Upstart script. 

``` ruby
sudo wget\
  https://raw.github.com/rstudio/shiny-server/master/config/upstart/shiny-server.conf\
  -O /etc/init/shiny-server.conf
```

## 7.	Create a system account to run Shiny applications.

``` ruby
sudo useradd -r shiny
# Create a root directory for your website
sudo mkdir -p /var/shiny-server/www
# Create a directory for application logs
sudo mkdir -p /var/shiny-server/log
```


## 8. Copy the work directory to folder 'www'. 

``` ruby
sudo cp -R /host/Users/YourApp /var/shiny-server/www/
```

Due to in my application, I need connect *Firebird* with R, so I need the JDBC driver of *Firebird*, which can be downloaded [here](http://www.firebirdsql.org/en/jdbc-driver/). Note: select the version for Java 6 (Jaybird-2.2.3JDK_1.6.zip). The '.jar' file *jaybird-full-2.2.3.jar* is the driver, so we extract it from the zip file.

## 9. Start, reload and stop Shiny-server.
Using the following code to start, reload and stop Shiny-server:

``` ruby 
sudo start shiny-server
sudo reload shiny-server
sudo stop shiny-server
```

When the server was started, we can access the application in a web-browser using address: 
<code> http://IP:3838/YourApp </code>
Where the <code>IP</code> is the IP of your computer, which can be found using <code> ifconfig </code> in Ubuntu's terminal.



	



