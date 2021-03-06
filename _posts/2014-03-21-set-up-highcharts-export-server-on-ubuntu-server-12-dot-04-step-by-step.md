---
layout: post
title: "Set Up Highcharts Export Server On Ubuntu Server 12.04: Step by Step"
date: 2014-03-21 10:29
comments: true
categories: Highcharts
---

There are a lot of JavaScript [libraries](http://techslides.com/50-javascript-charting-and-graphics-libraries/) for generating elegant and interactive charts, such as *d3.js*, *RGraph*, *Google Charts*, etc. However, most of them have no feature that can let users save their charts as images, except **Highcharts**! The charts generated by *Highcharts* contain a export button, users can use it to save charts as images. The mechanism of exporting feature of *Highcharts* is: by clicking the exporting button, users send a request to *Highcharts*'s export server with chart data, then the export server convert chart data (SVG) into images, e.g. *JPEG*, *PNG*, *PDF*, through [**PhantomJS**](http://phantomjs.org/), and send the generated images back to clients. So, there is a security problem! Even though *Highcharts* said they will [never gather](http://www.highcharts.com/docs/export-module/privacy-disclaimer) any information send to their export server, users' chart data still can be intercepted by third part organizations. That's a common problem of **http** protocol! In this blog, I will show you how to set up your own local *highchart-export* server step by step, which can be further set up to be accessible outside under **https** protocol.  

The tutorial of setting up export server on *Highcharts*'s [website](http://www.highcharts.com/docs/export-module/setting-up-the-server) is really vague, especially to users that have little knowledge about Java and Tomcat-server. I spent two days to get my server work, but I hope you can get it through in 10 minutes by following this tutorial. 

###1. Install Java
The Highcharts export server is an Apache-tomcat based server, which run under Java run environment, so we need to install Java first. And for Highcharts-export server, we need at least version 1.7. Open your terminal, and type the following command: 


``` ruby
sudo apt-get install openjdk-7-jdk openjdk-7-jre
``` 

To check whether Java was installed correctly, you can use the command:
 
``` ruby
java -version
``` 

If you can see the message: *java version "1.7.*"*, then the Java was successfully installed. We need to tell computer the main path of Java (**JAVA_HOME**) by appending this line "export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64" into " ~/.bashrc". 

``` ruby
sudo nano ~/.bashrc
``` 

After opening the "~/.bashrc" file, use arrow key to navigate to its bottom, type the export command, and press "Ctrl+x" to exist and save the modification. 


###2. Install Tomcat7
You can call Tomcat as the server, because it hosts the *highcharts-export-web* module which will be used for converting charts to images. 

``` ruby
sudo apt-get install tomcat7
``` 

Create System variable **CATALINA_HOME** by appending this line "export CATALINA_HOME= /usr/share/tomcat7" to the end of " ~/.bashrc".  Save it and execute the following command to make all modification work: 

``` ruby
. ~/.bashrc
``` 


If you can see the message: * Starting Tomcat servlet engine tomcat7   [OK]*, then the Tomcat7 was successfully installed. 



###3. Install Maven

``` ruby
sudo apt-get update
sudo apt-get install maven
``` 

Check if Maven was successfully installed using: 

``` ruby
mvn -version
``` 

###4. Install PhantomJS

``` ruby
sudo apt-get remove phantomjs
sudo unlink /usr/local/bin/phantomjs
sudo unlink /usr/local/share/phantomjs
sudo unlink /usr/bin/phantomjs
cd /usr/local/share
sudo wget https://phantomjs.googlecode.com/files/phantomjs-1.9.0-linux-x86_64.tar.bz2
tar xjf phantomjs-1.9.0-linux-x86_64.tar.bz2
sudo ln -s /usr/local/share/phantomjs-1.9.0-linux-x86_64/bin/phantomjs /usr/local/share/phantomjs 
sudo ln -s /usr/local/share/phantomjs-1.9.0-linux-x86_64/bin/phantomjs /usr/local/bin/phantomjs
sudo ln -s /usr/local/share/phantomjs-1.9.0-linux-x86_64/bin/phantomjs /usr/bin/phantomjs
``` 
Check if PhantomJS was successfully installed using: 

``` ruby
phantomjs -v
``` 

###5. Compile file "highcharts-export-web.war" using Maven
Go to Highcharts' github repository [highcharts.com](https://github.com/highslide-software/highcharts.com), and click the *Download ZIP* at the right-bottom corner. Unzip the download file, you can find the "highcharts-export" folder under "highcharts.com-master\exporting-server\java", copy it to a convenient place. For example, 

``` ruby
sudo cp -R theFolderDirectory /home/local/../highcharts-export/
``` 

Navigate to the directory "highcharts-export", and compile "highcharts-export-web.war".

``` ruby
cd /home/local/KRG/huti/highcharts-export/
mvn install
cd highcharts-export-web
mvn clean package
mvn install
``` 

If you can see *BUILD SUCCESS* message, then the "highcharts-export-web.war" was successfully compiled. 

Now copy the compiled file into your server's **webapps** folder. For example, mine is:

``` ruby
sudo cp -R /home/local/KRG/huti/highcharts-export/highcharts-export-web/target/highcharts-export-web.war /var/lib/tomcat7/webapps/
``` 

Finally, the setting up is done! You can access the server through interval IP (start with 192), which we can get using: 

``` ruby
ifconfige
``` 

Type the address "http://IP:8080/highcharts-export-web" in your web browser you will see the demo of *Highcharts Export Server*!


**Note!** If there are some kinds of error occurred, and you need to remove your Java or Tomcat, or re-installed them, you can use the following command: 

``` ruby
sudo apt-get purge openjdk*
sudo apt-get autoremove tomcat*
``` 

Also, to start and stop Tomcat7, you can use the command: 

``` ruby
service tomcat7 start
service tomcat7 stop
``` 


**Tips:**, To use the new export-server instead of the default one "http://export.highcharts.com/", we need specify the option *url* in our JavaScript, like: 

``` ruby
exporting:{
    url:'http://IP:8080/highcharts-export-web/'
}
``` 

Beware! The address must contain last "/", otherwise, export button will navigate to the demo of highcharts export server!





