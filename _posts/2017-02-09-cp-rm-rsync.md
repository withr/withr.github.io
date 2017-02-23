---
layout: post
title: "BigData basic: copy & delete folder containing large number of files"
date: 2017-02-23 13:20
comments: true
categories: BigData
---


With more than 10 years' experience of using Linux, I have never been bothered by copy/delete a folder using: <code>sudo cp -R source dest</code> and <code>sudo rm -R folder</code>, untill I decided to play with BigData. 


I have a modern computer, which is fast. The hardware setting of the computer is: 

- CPU: Intel(R) Core(TM) i7-6700 CPU @ 3.40GHz

- RAM: 16GB DDR4 

I installed Ubuntu 16.04 on the SSD (250GB), and I have two additional hard disks: one is 1TB and one is 3T. On the 3TB. I have a folder containing huge number (3747834) of HTML files which the total size is 400GB (400836788KB, each HTML file's size is around 100KB). Now I want to test the speed of copy files using <code>cp</code> and <code>rsync</code>. To record the task process, I wrote the following bash script to monitor the disk space change. 

~~~~
#!/bin/bash
touch $1
while true; do
  n1=`df | grep sdc1 | awk '{print $3}' `
  sleep 1
  n2=`df | grep sdc1 | awk '{print $3}' `
  echo "$n1"
  if [ "$n1" != "$n2" ]; then
    tm=`date +%s`
    echo "$tm;$n2" >> $1
    echo  recorded
  fi
done
~~~~

## Copy insides of HDD

**The cp command is:** 

~~~~
sudo cp -r srcFolder destFolder & disown
~~~~

Note: **disown** in above command is used to make sure: when the terminal disconnected with server, the command will still run. This is very useful for long time waiting task, because terminal will disconnect with server automatically if no action happens in a limited period, which will also abort the shell command.

The total time of the copy task used 24117 seconds, which is around 16.6MB/s. 

**The rsync command is:** 

~~~~
sudo rsync -a srcFolder destFolder & disown
~~~~

The total time of the copy task used 44795 seconds, which is around 8.9MB/s.

Here is the monitored process: 

![](/images/cp/cp_400.png)

So, **cp** is around 2 times faster than **rsync**.


## Delete

Deleting huge number of files is another chanlledge using command **rm**. The deleteing speed is quite fast in the beginning, but later it became slower and slower, which force me to abort the task, and search for better solutions. After searching some posts, it comes that **rsync** is the [simplest and fast solution](https://web.archive.org/web/20130929001850/http://linuxnote.net/jianingy/en/linux/a-fast-way-to-remove-huge-number-of-files.html). In that post, the author compared several methods on 1 million zero KB files, and result showed that command **rsync -a --delete empty/ folder/** use 1/3 time of others (*find folder -type f -delete*, *rm -R folder*). Now let's see how faster it is for my task.

**The rsync command:**

~~~~
mkdir tmp
rsync -a --delete tmp/ myfolder/ & disown
~~~~

The monitored log file showed that the deleting process used  5358 seconds, which is arround 700 files/second. 


**The rm command:**

~~~~
rm -rf Finn_rsync & disown
~~~~

The monitored log file showed that the deleting process used  3081 seconds, which is arround 1216 files/second. 

![](/images/cp/rm_400.png)

So, **rm** is actually better than **rsync**. 


## Copy between HDDs

I have also a folder with size around 805GB (8279692 files) need to copy to other hard disk. Let's see the copy process using **cp** & **rsync** method respectively. 


**cp command:**

I tried **cp** first according to above result. However, the copy process aborted when it copied 400GB files, so I tried one more time, and it aborted again at 600GB. The copying process force the hard disk unmounted from the system, which I have no idea why. Then I tried the command **rsync**, and the following figure showed the two process: 

![](/images/cp/cp_800.png)

The **cp** command used 97598 seconds to copy 629403468 KB, the speed is around 6.5 MB/second. The **rsync** command used 137785 seconds to copy 843968412 KB, the speed is around 6.1 MB/second. We can see that speeds are not of too much difference, however due to **rsync** wasn't aborted, so I recommend it. 


