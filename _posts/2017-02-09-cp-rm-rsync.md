BigData basic: efficiently copy & delete directory containing huge number of files 


With more than 10 years' experience of using Linux, I have never been bothered by copy /delete a folder using: <code>sudo cp -R source dest</code> and <code>sudo rm -R folder</code>, untill I decided to play with BigData. 


I have a modern computer, which is very fast. The hardware setting of the computer is: 

-CPU: Intel(R) Core(TM) i7-6700 CPU @ 3.40GHz
-RAM: 8GB DDR4 

I installed Ubuntu 16.04 on the SSD (250GB), however I have two additional Hard Disks: one is 1T and one is 3T. On the 3T hard disk, I have a folder with huge number (3747834) of HTML files with size equal 400GB (400836788KB) in total (each HTML file's size is around 100KB). Now I want to test the speed of copy files using <code>cp</code> and <code>rsync</code>. To record the task process, I use the following bash script to monitor the disk space change. 

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

## Copy and delete inside of HDD



**The cp command is:** 

~~~~
sudo cp -r srcFolder destFolder & disown
~~~~

Note: **disown** in above command is used to make sure is the terminal disconnected with server, the command will still run. This is very useful for long time waiting task, because terminal will disconnect with server automatically if no action happens in a limited period

The total time of the copy task used 20000 seconds, which is around 20MB/s, and here is the monitored process: 

**The rsync command is:** 

~~~~
sudo rsync -a srcFolder destFolder & disown
~~~~

The total time of the copy task used 40000 seconds, which is around 10MB/s, and here is the monitored process: 

![](/images/cp/cp_400.png)

So, **cp** is around 2 times faster than **rsync**.








rm -rf Finn_rsync & disown




Deleting huge number of files is another chanlledge using command **rm**. The deleteing speed is quite fast in the beginning, but later it became slower and slower, which force me to abort the task, and search for better solutions. After searching some posts, it comes that **rsync** is the [simplest and fast solution](https://web.archive.org/web/20130929001850/http://linuxnote.net/jianingy/en/linux/a-fast-way-to-remove-huge-number-of-files.html). In that post, the author compared several methods on 1 million zero KB files, and result showed that command **rsync -a --delete empty/ folder/** use 1/3 time of others (*find folder -type f -delete*, *rm -R folder*). Now let's see how faster it is for my task.

The command: 

~~~~
mkdir tmp
rsync -a --delete tmp/ myfolder/ & disown
~~~~

The monitored log file showed that the deleting process used  2999 seconds, which is arround 100 files/seconds. The following the monitor figure: 

![]()

The above command is recommended by many posts for deleting large number of files. However 
[Matthew Ife](http://serverfault.com/questions/183821/rm-on-a-directory-with-millions-of-files/328305#328305), thought it is fast, but not fast enough, because the way **rsync** reads directory contents is in-efficiently, and he gave the code of efficiently reading directory contents. Let's try his method :




![](/images/cp/rm_400.png)

![](/images/cp/cp_800.png)




