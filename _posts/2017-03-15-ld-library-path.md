---
layout: post
title: "LD_LIBRARY_PATH: Rstudio-server, Shiny-server and Tensorflow"
date: 2017-03-15 11:20
comments: true
categories: ML
---

I used to create a shiny-app to conduct image classification using Tensorflow on DigitalOcean, however it didn't work on my own PC station. The main problem is: Rstudio-server and Shiny-server can't find the shared object: *libcudart.so.8.0* which located under */usr/local/cuda/lib64* and should be binded to environment variable: "LD_LIBRARY_PATH". Recently, I noticed that the R package "tensorflow" has released, so I tried that package, and fixed the previous problem. 


There are three snarios: R console, Rstudio-server and Shiny-server, each has its only LD_LIBRARY_PATH value.

## R console

To open R console, just type **R** in terminal. Then use the R command: **Sys.getenv("LD_LIBRARY_PATH")** to check LD_LIBRARY_PATH's value. You will see that "/usr/local/cuda/lib64" is included if your tensorflow has been setup correctly. Then, we can conduct the image classification in R console using command: **system("python /home/tian/models/tutorials/image/imagenet/classify_image.py --image_file /tmp/imagenet/cropped_panda.jpg", intern = TRUE)**


## Rstudio-server

If we run command **Sys.getenv("LD_LIBRARY_PATH")** from Rstudio-server, you may get result like: "/usr/lib/R/lib::/lib:/usr/lib/x86_64-linux-gnu:/usr/lib/jvm/default-java/jre/lib/amd64/server", which not include "/usr/local/cuda/lib64". To add the directory to **LD_LIBRARY_PATH**, we need add a line **rsession-ld-library-path=/usr/local/cuda/lib64** in "/etc/rstudio/rserver.conf", then restart Rstudio-server.


~~~~
sudo nano /etc/rstudio/rserver.conf
## rsession-ld-library-path=/usr/local/cuda/lib64
sudo rstudio-server restart
~~~~

Now, run command **Sys.getenv("LD_LIBRARY_PATH")** again, you will find the path "/usr/local/cuda/lib64" has included. So we can run image classification in Rstudio-server using command: **system("python /home/tian/models/tutorials/image/imagenet/classify_image.py --image_file /tmp/imagenet/cropped_panda.jpg", intern = TRUE)**

## Shiny-server

Though we can driver tensorflow from R console and Rstudio-server, however the shiny-app still doesn't work. Include a line in shiny-app like: **writeLines(Sys.getenv("LD_LIBRARY_PATH"), "log.txt")** will tell us the value of "LD_LIBRARY_PATH" for Shiny-server, and we will find that the paths not include "/usr/local/cuda/lib64".  I tried to find the configure file to setup "LD_LIBRARY_PATH" for Shiny-server, but didn't succeed. Luckly, we can setup using R command: Sys.setenv(), the only unconvenience is that we always need to include that command for Tensorflow related shiny-app. For example, include this line into the first line of *server.R*

~~~~

Sys.setenv(LD_LIBRARY_PATH="/usr/local/cuda/lib64:/usr/lib/R/lib:/usr/lib/x86_64-linux-gnu:/usr/lib/jvm/default-java/jre/lib/amd64/server")

~~~~
 
After above setting, we may encounter the problem that the shiny-app still doesn't work. The problem is that directory */tmp* is not accessiable for *shiny*. We can change the access permissions of that folder to let user *shiny* can access.

~~~~
sudo chmod 777 -R /tmp
~~~~


















