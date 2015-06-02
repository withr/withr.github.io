---
layout: post
title: "Setup Shiny Server on Ubuntu 14.04"
date: 2015-03-16 10:27
comments: true
categories: R Shiny
---

I have set up a Shiny Server on Ubuntu 12.04 in 2013. [Shiny](http://shiny.rstudio.com/) package has been under active development, and current version has reached **0.11** while the one on my server is still **0.8**. I should keep update with Shiny package's development, however, from version 0.8, Shiny package has changed a lot, and I have to modify all my apps to get them work which is huge task for me. Now, I have to update them because the topics discussed on [Shiny's Google group](https://groups.google.com/forum/#!forum/shiny-discuss) is not concerning my server any more :-( 


## Install Ubuntu 14.04 server. 

This progress was conducted by our IT administer, I will not describe the detail here. Anyway, please update the server:

```
sudo apt-get update
```


## Install R and shiny package

 Before installing Shiny Server, we need to install R and the Shiny package first. 

- To install the latest version of R, we need to add the CRAN repository to our system's sources list: **/etc/apt/sources.list**. 

```
sudo nano /etc/apt/sources.list
```
Add <code>deb http://cran.uib.no/bin/linux/ubuntu trusty/</code> to its end, and save the modification. Where *cran.uib.no* is the CRAN mirror for Norway (so you may need to change it to the mirror suitable for you) and **trusty** is the version name for Ubuntu 14.04.

- Install R using the following command: 

```
sudo apt-get install -y r-base
```

This process would take one minute. 

- Install the **Shiny** R package.

```
sudo su - -c "R -e \"install.packages('shiny', repos='http://cran.rstudio.com/')\""
```

This process could take one and half minute, because many dependent packages were also installed. 

- Install Shiny Server

```
sudo apt-get install -y gdebi-core
wget http://download3.rstudio.org/ubuntu-12.04/x86_64/shiny-server-1.3.0.403-amd64.deb
sudo gdebi shiny-server-1.3.0.403-amd64.deb
```

This process could take one and half minute. By default, Shiny Server will run automatically after installation, you will see a welcome page on port 3838 of your server's IP address, e.g. "http://192.168.202.16:3838/"

Installation of Shiny Server finished!


## Install R packages. 

 Before starting to install R packages using install.packages() under R, we need to install several dependencies or environment first.
  
- Install **curl** library used by R package *RCurl*:
  
```
  sudo apt-get install -y libcurl4-openssl-dev 
  
```
- Install **libxml2-dev** library used by R package *XML*:

```
  sudo apt-get install -y libxml2-dev
  
```
- Install **Java** used by R package *rJava* and add a system environment variable LDLIBRARYPATH.

```
sudo apt-get install -y openjdk-7-jdk
export LD_LIBRARY_PATH=/usr/lib/jvm/java-7-openjdk-amd64/jre/lib/amd64/server
R CMD javareconf  
```



- When install the Shiny R package, it was installed at "/usr/local/lib/R/site-library" be default, however, the default library path is not that one. We change **/usr/lib/R/etc/Renviron** to set it as the default one.

```
sudo nano /usr/lib/R/etc/Renviron
```
Comment the line: <code>R_LIBS_USER=${R_LIBS_USER-'~/R/x86_64-pc-linux-gnu-library/3.0'}</code> by adding a "#" at the beginning of that line. 

Now, let's check the default library path in R: 

```
> .libPaths()
[1] "/usr/local/lib/R/site-library" "/usr/lib/R/site-library"
[3] "/usr/lib/R/library"
```
So, "/usr/local/lib/R/site-library" is the first choice (default) of the library paths.

- Install the following R packages:

```
install.packages(c('RJDBC', 'XLConnect', 'devtools', 'RJSONIO'))
require(devtools)
install_github('rCharts', 'ramnathv')
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

<h6 style="color: red"> The following installation and modification may not suitable for you.</h6>
## Install [inkscape](https://apps.ubuntu.com/cat/applications/precise/inkscape/) for converting svg to pdf & png format.
```
sudo apt-get install inkscape
```

## Shiny package:

- Add the following codes to *bootstrap.min.css* (v3.3.1) in directory */usr/local/lib/R/site-library/shiny/www/shared/bootstrap/css/*: 

```
.radio,.checkbox{margin-top:5px;margin-bottom:5px}
.radio+.radio,.checkbox+.checkbox{margin-top:5px}
.form-group{margin-bottom:10px}
select[multiple],select[size]{height:60px;}
```

## rCharts package:
 Replace the **highcharts.js** library with version **4.1.4** in **/usr/local/lib/R/site-library/rCharts/libraries/highcharts/js**. And replace the source code in **/usr/local/lib/R/site-library/rCharts/libraries/highcharts/layout/chart.html** with the following: 

```
<script type='text/javascript'>
// Get the ISO week date week number  
Date.prototype.getWeek = function () {  
    // Create a copy of this date object  
    var target  = new Date(this.valueOf());  
  
    // ISO week date weeks start on monday  
    // so correct the day number  
    var dayNr   = (this.getDay() + 6) % 7;  
  
    // ISO 8601 states that week 1 is the week  
    // with the first thursday of that year.  
    // Set the target date to the thursday in the target week  
    target.setDate(target.getDate() - dayNr + 3);  
  
    // Store the millisecond value of the target date  
    var firstThursday = target.valueOf();  
  
    // Set the target to the first thursday of the year  
    // First set the target to january first  
    target.setMonth(0, 1);  
    // Not a thursday? Correct the date to the next thursday  
    if (target.getDay() != 4) {  
        target.setMonth(0, 1 + ((4 - target.getDay()) + 7) % 7);  
    }  
    // The weeknumber is the number of weeks between the   
    // first thursday of the year and the thursday in the target week  
    return 1 + Math.ceil((firstThursday - target) / 604800000); // 604800000 = 7 * 24 * 3600 * 1000  
}  

// Get the ISO week date year number 
 
Date.prototype.getWeekYear = function ()   
{  
    // Create a new date object for the thursday of this week  
    var target  = new Date(this.valueOf());  
    target.setDate(target.getDate() - ((this.getDay() + 6) % 7) + 3);  
      
    return target.getFullYear();  
}  
</script>


<script type='text/javascript'>
  (function($){
    (function (Highcharts) {
      var each = Highcharts.each;
      Highcharts.wrap(Highcharts.Legend.prototype, 'renderItem', function (proceed, item) {
        proceed.call(this, item);
        var series = this.chart.series,
        element = item.legendGroup.element;
        element.onmouseover = function () {
          each(series, function (seriesItem) {
            if (seriesItem !== item) {
              each(['group', 'markerGroup'], function (group) {
                seriesItem[group].attr('opacity', 0.25);
              });
            }
          });
        }
        element.onmouseout = function () {
          each(series, function (seriesItem) {
            if (seriesItem !== item) {
              each(['group', 'markerGroup'], function (group) {
                seriesItem[group].attr('opacity', 1);
              });
            }
          });
        }           
      });
    }(Highcharts));  

    $(function () {
      Highcharts.setOptions({
        global: {
          useUTC: false
        }
      });
      Highcharts.dateFormats = {
        W: function (timestamp) {
          var date = new Date(timestamp)
          return  date.getWeek() + '-' + date.getWeekYear()
        }, 
				M: function (timestamp) {
          var date = new Date(timestamp)
          return  date.getMonth()+1 + '-' + date.getWeekYear()
        }
      };
      var chart = new Highcharts.Chart({{{ chartParams }}});
    });
  })(jQuery);

</script>
  
  <script>
  for (var i = 0; i < $("svg").length; i++) {
    var s = "svg:eq(" + i + ") g.highcharts-button:not(:last)"
    $(s).remove()
  };
</script>
```


