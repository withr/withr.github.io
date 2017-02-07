---
layout: post
title: "multiprocessing could even slower than single thread"
date: 2017-02-07 14:27
comments: true
categories: MongoDB
---



I have been always using **multiprocessing** in my web crawlers to accelerate the processing. Today, I want test how much fast the storageEngine WiredTiger vs MMAPV1 in MongoDB. The result is what expected for inserting document using single thread: 20% faster. As WiredTiger has a lower level lock (document) than MMAPV1 (collection), and support multiple-core CPU, so I test also the speed of inserting using multiprocessing. The result surprised me: 14% slower than single thread. 


Here is the python script: 

~~~~
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import re, sys, os, time
from pymongo import MongoClient
from multiprocessing.dummy import Pool

##connect to default MongoClient
client = MongoClient()
db = client['MM']

IDs = range(1000000)

def writeSample(i):
    item = {"name": "Huidong Tian",
            "title": "PhD",
            "id": i,
            "des": "I am a data scientist focusing on Python and R", 
            "other": "As for why the book was created, a theory which has gained considerable interest, although still controversial is Persian imperial authorisation. "}
    collection.insert(item)
    msg = "\rwrite " + str(i) + " documents!"
    sys.stdout.write(msg); sys.stdout.flush()
  
collection = db['single']
t1 = time.time()
for i in IDs:
    writeSample(i)

print "\nImporting used " + str(int(time.time() - t1)) + " seconds in total!\n"

collection = db['multiple']
t1 = time.time()
pool = Pool(8)
pool.map(writeSample, IDs) 
pool.close()  
pool.join()
print "\nImporting used " + str(int(time.time() - t1)) + " seconds in total!\n"

~~~~

By default, the storage engine of MongoDB 3.4 is WiredTiger, to change to mmap1v, follow the steps:

- We need to modify the configure file: **/etc/mongod.conf** to specify a new location and **mmapv1** engine: 

~~~~
dbPath: /home/tian/3T/db_mm
engine: mmapv1
~~~~

**Note**: there is a space after ":", otherwise, mongod will not start. 

The dbPath should be access by mongodb, to make ensure use the command to change its owner:group and access permission: 

~~~~
sudo chown mongodb:mongodb /home/tian/3T/db_mm
sudo chmod 755 /home/tian/3T/db_mm

~~~~

- Stop current mongod service. 

Using <code>sudo service mongod stop</code> may not always works, if so, simply kill the pid of **mongod** (using <code>top|grep mongod</code> to find the pid of mongod

~~~~
sudo kill xxxx
~~~~

- Start mongod service 

~~~~
sudo service mongod start
~~~~

Now, type **mongo** in terminal, we should can connect the mongod server. 


## Here is result: 

Inserting speed: 

![](/images/mmapv1/speed.png )

Inserting speed: 

![](/images/mmapv1/size.png )


