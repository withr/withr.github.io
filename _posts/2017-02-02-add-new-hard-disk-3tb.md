---
layout: post
title: "Add second hard disk (3TB) to Ubuntu 16.04"
date: 2017-02-02 15:27
comments: true
categories: Linux
---


I need more disk space of my computer, so I bought a 3TB hard disk. I want the new disk to be one partion, so I can easily distinguish it from others. 



After connecting the hard disk to computer and start the computer, I check if the disk was recognized by the computer using command <code>df</code>, and known that the disk was recognized as **/dev/sdc**. 

![]( /images/wd3t/fdisk_sdc.png )

Then I tried to create a partion using command: <code>fdisk /dev/sdc</code>, but encountered the following error: **can't create a partion larger than 2TB**.

![]( /images/wd3t/fdisk_error.png )

After googled some [blogs](https://www.cyberciti.biz/tips/fdisk-unable-to-create-partition-greater-2tb.html), I found that I should use tool **GPT**. One sentence said: *However, if you are using Debian or Ubuntu Linux, you need to recompile the kernel. Set CONFIG_EFI_PARTITION to y to compile this feature.*, however, I don't know how to recompile the kernel, but I checked that the **CONFIG_EFI_PARTITION ** of Ubuntu 16.04 has already setted to "y":

![]( /images/wd3t/config.png )

So, create a partion larger than 2TB seems to be easier (run the following commands one by one): 


~~~~
parted /dev/sdc
mklabel gpt
mkpart primary 1M 3T
quit
~~~~

Note: if you use command: <code>mkpart primary 0 3T</code>, you may have the waring message (in red): **Partition 1 does not start on physical sector boundary**, which is not a big issue, but can be easily fixed by setting to *1M*. 

![]( /images/wd3t/format.png )

The next step is format the new partition and mount it to somewhere of your system. 

~~~~
mkfs.ext3 /dev/sdc1

cd; mkdir WD3T
mount /dec/sdc1 ~/WD3T
~~~~







