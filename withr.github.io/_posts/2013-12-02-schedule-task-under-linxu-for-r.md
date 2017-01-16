---
layout: post
title: "Schedule tasks under Linxu for R"
date: 2013-12-02 09:13
comments: true
categories: R Shiny
---

Frequently encountered, we need to schedule tasks: run task automatically! To schedule your R task, edit **/etc/crontab** like this:

``` ruby
sudo nano /etc/crontab
```



At the end of the opened file, add such a line:

``` ruby 
0 * * * * username Rscript /var/shiny-server/www/Project/load_data.R
```

Where <code> 0 * * * * </code> is the [cron](http://en.wikipedia.org/wiki/Cron) formula to schedule your task. The above one means run the followed command per hour at 0 minute. The *username* is your logged name. Sometimes, the scheduled task doesn't run, then we should check two issues: 

- **Is the shell command correct?** To check it, just paste the shell command (e.g. *Rscript /var/shiny-server/www/Project/load_data.R*) in the terminal. If it works, then go to the next step:

- **Permission mode**. Some files have strict permission, it allow only special users to execute. To check the perission, run the following command in termial: 

``` ruby 
sudo ls -l /var/shiny-server
``` 

Please[take a look how the permissons were defined under Linux](http://linuxcommand.org/lts0070.php). We can change the permission mode using the following command:

``` ruby 
sudo chmod - R 777 /var/shiny-server/
``` 






