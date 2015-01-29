---
layout: post
title: "Configure Shiny-server Under Virtual Machine"
date: 2013-08-05 11:42
comments: true
categories: Shiny R
---

In previous [post](/blog/2013/07/23/configure-shiny-server-under-ubuntu/), I listed the issues of how to configure *Shiny-server* under *Ubuntu* which installed as parellel operation system with *Windows*. There are two shortcomings for that: I need to restart the computer to switch between the two systems; I can't test if my Shiny-server run correctly from other computer (I have only one computer in my office).  My colleague suggested me to install *Ubuntu* as virtual machine through [Virtualbox](https://www.virtualbox.org/), so I don't need to restart my computer, and the virtual machine can be run as a seperate computer. Indeed, it's much convenient to run *Ubuntu* as virtual machine, and I found there are two issues which I encountered when configure my *Shiny-server* app.

+ The first one is the libraries location of R. When I run my app, *Shiny-server* throwed up an error said it can not find the libraries required, but I am sure I have installed them. So, the problme is *Shiny-server* doesn't know where I installed my libraries, and the solution is add the following line to tell *Shiny-server* where the libraries were installed. This problem generally occured when we install R packages, R throwed up a message said the default location for installation is not accessable, would you like to choose another. If we said yes, R will choose one by default, and we should remember that <code>directory</code>. 

``` ruby
.libPaths("/home/tian/R/x86_64-pc-linux-gnu-library/3.0")

```

+ The second issue is how to configure the virtual machine as a seperate computer. This can be done easily by configuring the network connection as *Bridged Adapter*. After setting up the network connection, next time when we start Ubuntu, we can check the IP using shell:

``` ruby
ifconfig
```

The *inet addr* under *eth0* is the address that other computers can access my *Shiny-server* app, e.g. <code> 'eth0: inet addr':3838/app</code>

+ If you want to access documents stored on Network Drives, just add the directories to 'Share' folder of your virtual machine, and remember to mount those directories to '/media/share/' using the following shell:

``` ruby
sudo mount -t vboxsf C_DRIVE /media/share/C
```
where *C_DRIVE* is the name of shared directory, and 'C' is the target directory.

