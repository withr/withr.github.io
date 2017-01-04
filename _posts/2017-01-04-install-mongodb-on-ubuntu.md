---
layout: post
title: "Install mongodb on ubuntu 16.04 LTS"
date: 2017-01-04 11:27
comments: true
categories: Linux
---

The official instruction for install MongoDB on Ubuntu system can be found [here](https://docs.mongodb.com/getting-started/shell/tutorial/install-mongodb-on-ubuntu/). 

In the **Overview** part, it said: 


>While Ubuntu includes its own MongoDB packages, the official MongoDB Community Edition packages are generally more up-to-date.


## Install from Ubuntu's packages management tools

I am curious on the **more up-to-date**, so I installed Ubuntu's MongoDB packages first as following:

To start the MongoDB database, we can run it's shell by typing **mongo** in terminal. As I didn't install any MongoDB related packages, it's not surprise I got the following error message:


>The program 'mongo' is currently not installed. You can install it by typing:
>sudo apt install mongodb-clients


So I install the mongodb-clients using command:

~~~~
sudo apt install -y mongodb-clients
~~~~

Then, I run **mongo** again, but got the following error message:


>tian@acer:~$ mongo
>MongoDB shell version: 2.6.10
>connecting to: test
>2017-01-04T09:05:04.949+0000 warning: Failed to connect to 127.0.0.1:27017, reason: errno:111 Connection refused
>2017-01-04T09:05:04.949+0000 Error: couldn't connect to server 127.0.0.1:27017 (127.0.0.1), connection attempt failed at >src/mongo/shell/mongo.js:146
>exception: connect failed



After googled the error code, I realized that I need to start the MongoDB server first. 
 


>tian@acer:~$ mongod
>The program 'mongod' is currently not installed. You can install it by typing:
>sudo apt install mongodb-server


Again, the **mongod** is not installed, so I installed it using command:

~~~~
sudo apt install mongodb-server
~~~~

Then, I run **mongod** again, and got this error message:



>tian@acer:~$ mongod
>mongod --help for help and startup options
>2017-01-04T09:09:37.194+0000 [initandlisten] MongoDB starting : pid=8061 port=27017 dbpath=/data/db 64-bit host=acer
>2017-01-04T09:09:37.194+0000 [initandlisten] db version v2.6.10
>2017-01-04T09:09:37.194+0000 [initandlisten] git version: nogitversion
>2017-01-04T09:09:37.194+0000 [initandlisten] OpenSSL version: OpenSSL 1.0.2g  1 Mar 2016
>2017-01-04T09:09:37.194+0000 [initandlisten] build info: Linux lgw01-12 3.19.0-25-generic #26~14.04.1-Ubuntu SMP Fri Jul 24 21:16:20 UTC >2015 x86_64 BOOST_LIB_VERSION=1_58
>2017-01-04T09:09:37.194+0000 [initandlisten] allocator: tcmalloc
>2017-01-04T09:09:37.194+0000 [initandlisten] options: {}
>2017-01-04T09:09:37.194+0000 [initandlisten] exception in initAndListen: 10296
>*********************************************************************
> ERROR: dbpath (/data/db) does not exist.
> Create this directory or give existing directory in --dbpath.
> See http://dochub.mongodb.org/core/startingandstoppingmongo
>*********************************************************************
>, terminating
>2017-01-04T09:09:37.194+0000 [initandlisten] dbexit:
>2017-01-04T09:09:37.194+0000 [initandlisten] shutdown: going to close listening sockets...
>2017-01-04T09:09:37.194+0000 [initandlisten] shutdown: going to flush diaglog...
>2017-01-04T09:09:37.195+0000 [initandlisten] shutdown: going to close sockets...
>2017-01-04T09:09:37.195+0000 [initandlisten] shutdown: waiting for fs preallocator...
>2017-01-04T09:09:37.195+0000 [initandlisten] shutdown: lock for final commit...
>2017-01-04T09:09:37.195+0000 [initandlisten] shutdown: final commit...
>2017-01-04T09:09:37.195+0000 [initandlisten] shutdown: closing all files...
>2017-01-04T09:09:37.195+0000 [initandlisten] closeAllFiles() finished
>2017-01-04T09:09:37.195+0000 [initandlisten] dbexit: really exiting now


The error message said "dbpath (/data/db) does not exist", I created it using command:

~~~~
sudo mkdir -p data/db
~~~~

Now, I check the version MongoDB Shell and found :

**MongoDB shell version: 2.6.10**


## Install from MongoDB's official recommendation.

~~~~
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo service mongod start
~~~~
Check the version of MongoDB Shell, I got the following information:

~~~~
tian@acer:/$ mongod --version
db version v3.4.1
~~~~


Official version is **3.4.1** while Ubuntu's version is **2.6.10**, it's seems Ubuntu may drop their MongoDB packages because they are one and half year later than official release, while the software has [released more than 20 newer versions](https://jira.mongodb.org/browse/SERVER?selectedTab=com.atlassian.jira.jira-projects-plugin:versions-panel&subset=-1).

