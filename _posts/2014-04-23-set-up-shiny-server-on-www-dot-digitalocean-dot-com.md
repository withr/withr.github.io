---
layout: post
title: "Set Up Shiny-server on www.digitalocean.com"
date: 2014-04-23 11:49
comments: true
categories: R Shiny
---

[RStudio](http://www.rstudio.com/) supplies several servers for hosting user's apps, e.g. *http://www.shinyapps.io/*, *http://spark.rstudio.com*, *http://glimmer.rstudio.com*. Thanks a lot to RStudio who has contributed many excellent R [libraries](http://www.rstudio.com/projects/)! I want to set my shiny-server on a virtual private server (VPS), because I have some other tasks and I feel a VPS is more stable than the PC under my desk. 

Setting up a Shiny-server on [Digital Ocean](http://www.digitalocean.com) is easy, follow this blog you can get your shiny-server work in 10 minutes. 

###1. Create a Droplet
First, register an account on [Digital Ocean](http://www.digitalocean.com). Log in and register Billing details, then you are ready to create your first **Droplet** (server). Give your droplet a name, select its size, region, image (I prefer Ubuntu 12.04.4) and then click *Create Droplet*. In one minute, your droplet will be created, and you will receive an email containing the IP address, username and password of your server.

![]( /images/do-createDroplet-2.png )


###2. Remote control
Users can access their server using the *Console* of Digital Ocean's webpage, but there are a lot limitation. It's much easy to control your server using local terminal console. Under Windows, I recommend to use **Putty**. Download and install [Putty](http://www.chiark.greenend.org.uk/~sgtatham/Putty/download.html). Open it, set the *IP address* and *Saved Sessions*, then click *Open* to open Putty's console. 

![]( /images/do-Putty.png )

Login Putty use the username and password in your email. After log in, please change your password first using command <code>passwd</code>.

![]( /images/do-console.png )



###3. Install R and shiny package

For security reason, add R's secure APT key to system:

``` ruby
 sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9

``` 

Add this repository 

<code>
deb http://cran.uib.no/bin/linux/ubuntu precise/
</code> 

to the end of *sources.list* to ensure the R that will be installed consistent with your system version:

``` ruby
sudo nano /etc/apt/sources.list
``` 

Update and install R:

``` ruby
sudo apt-get update
sudo apt-get install r-base r-base-dev
``` 

Install shiny package:

``` ruby
sudo su - -c "R -e \"install.packages('shiny', repos='http://cran.rstudio.com/')\""
``` 

###4. Install Shiny-server

``` ruby
sudo apt-get install gdebi-core
wget http://download3.rstudio.org/ubuntu-12.04/x86_64/shiny-server-1.1.0.10000-amd64.deb
sudo gdebi shiny-server-1.1.0.10000-amd64.deb

``` 

If anything is OK, you can now access your shiny-server by typing your server's IP address: 3838.

![]( /images/do-shiny-server.png )

###5. Install RStudio-server
*Putty* is convenient  for remotely control your sever, but not intuitive for documents management. **RStudio-server** is one excellent solution for R project management and file transfer. Use the following commands to install  RStudio server on your droplet:

``` ruby
sudo apt-get install gdebi-core
sudo apt-get install libapparmor1  # Required only for Ubuntu, not Debian
wget http://download2.rstudio.org/rstudio-server-0.98.507-amd64.deb
sudo gdebi rstudio-server-0.98.507-amd64.deb
``` 

Due to RStudio Server will not permit logins by system users, so we need add a non-system user to our droplet, for example, I want to add a user named *withr*:

``` ruby
adduser withr
``` 

Set the privilege of the user as equal to *root* will avoid many troubles!


``` ruby
visudo
``` 

Add line  <code> withr    ALL=(ALL:ALL) ALL </code> after  <code> root    ALL=(ALL:ALL) ALL </code>, like the following: 

``` ruby
# User privilege specification
root    ALL=(ALL:ALL) ALL
withr   ALL=(ALL:ALL) ALL 
``` 
Save the edition.

Now, we can access the RStudio server through IP:**8787** using *withr* and its password. 

![]( /images/do-RStudio.png )

