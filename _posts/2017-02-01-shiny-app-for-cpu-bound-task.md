---
layout: post
title: "shiny app for CPU bound project"
date: 2017-01-18 09:27
comments: true
categories: shiny
---

 R is a single thread program, which can use only one CPU core, for multiple concurrent requests, it will conduct one by one until the last one was finished. Modern computers generally have several cores CPU, a simple way to deal multiple concurrent, or CPU bound task, is to start another R process, which will use another CPU core. Two years ago, I wrote two blogs [1](http://withr.me/shiny-server-system-monitoring-for-open-source-edition/) & [2](http://withr.me/shiny-server-open-source-edition-solution-for-cpu-bound-apps/) for creating shiny-app which for CPU bound project. In this blog, I simplified the process of creating such kind of shiny-app.  
 
Suppose we want to create a shiny-app called **myApp**.

- Create a directory called **myApp**. Note this directory should under the folder where your shiny-server holds shiny-app. By default, it is: */srv/shiny-server/*.

- Create an **index.html** file under folder *myApp*. Here is its content: 


~~~~
<!DOCTYPE html>
<html>
<head>
<script>
  function loadApp() {
    var xhttp = new XMLHttpRequest();
    xhttp.onreadystatechange = function() {
      if (this.readyState == 4 && this.status == 200) {
        document.getElementById("app").src = 
        document.URL + this.responseText;
      }
    };
    xhttp.open("GET", "min_user", true);
    xhttp.send();
  }
</script>
</head>
<body onload="loadApp()" style="margin: 0px;" >
  <iframe id="app" style="width:100%; height:100vh; border: none;" src=""></iframe>
</body>
</html>
~~~~

- Create an **min_user.sh** file under folder *myApp*. This bash script will continue generate a file call **min_user** under folder *myApp*, which contains the name of which shiny-app under folder *myApp* has least number of users, and will be used in above HTML file. Here is its code:

~~~~
#!/bin/bash
dir_apps='/srv/shiny-server/myApp/'
apps=`ls "$dir_apps" | grep app_*`   

while true; do
  USERS=99999
  for var2 in $apps; do
    pid=`sudo lsof | grep ^R | grep "$dir_apps" | awk '{print $2, $9}' | grep $var2'$' |awk '{print $1}' `
    if [ -z "$pid" ]; then     
      USERS=0
      APP=$var2
    else
      num=`sudo netstat -p | grep "$pid" | wc -l`
      echo 'PID: '$pid'; App: '$var2'; Users: '$num  
      if [ "$num" -lt "$USERS" ]; then
          USERS=$num
          APP=$var2
      fi
    fi 
  done
  echo 'App: '$APP'; Users: '$USERS  
  echo $APP >  $dir_apps'/min_user'
done
~~~~

Note: the second line in above defined the directory of the shiny-app: **myApp**, if you used another name, remember to change in this line. 

To run above bash script, we need change its access permission first using command: 

~~~~
sudo chown 777 /srv/shiny-server/myApp/min_user.sh
~~~~

Moreover, we can add a line to **cron** task, to make sure the script will run automatically after a reboot of the computer.

~~~~
@reboot root /srv/shiny-server/myApp/min_user.sh
~~~~

- The last step is create several shiny-apps under folder *myApp*, their names (directories) should start with characters: **app_**, because the *min_user.sh* will monitor the users number for shiny-app's names start with *app_*. E.g. *app_1*, *app_2*, *app_3*, ect. Note, if the shiny-app contains big size data, or same *scripts*, *www*, etc, these files can be shared by putting them under folder *myApp*. 

Now, run the *min_user.sh* script first if you didn't, then open the shiny-app **myApp** in your web browser, you will be directed to the shiny-app which has least user number. 






