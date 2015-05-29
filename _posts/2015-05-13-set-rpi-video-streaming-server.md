---
layout: post
title: "Setup Raspberry Pi as a Video Streaming Server"
date: 2015-05-13 15:41
comments: true
categories: RPi
---


I have googled a lot, finally I found the guideline of [Mike Haldas](http://videos.cctvcamerapros.com/raspberry-pi/how-to-setup-video-streaming-server.html) is the easiest way to stream Raspberry Pi Video. Here I simplified the steps as: 

**1.** [Enable RPi camera](https://www.raspberrypi.org/documentation/usage/camera/README.md) if it was not.

```
sudo raspi-config
```

**2.** Install **Flask**

```
sudo apt-get install -y python-pip
sudo pip install flask
```

**3.** Download Miguel’s Flask video streaming project.

```
git clone https://github.com/miguelgrinberg/flask-video-streaming.git
```

**4.** Edit *app.py* file

```
sudo nano flask-video-streaming/app.py
```

 - **Comment** line <code>from camera import Camera</code>;
 - **uncomment** line <code>from camera_pi import Camera</code>.

**5.** Start Flask-server by running: 

```
python flask-video-streaming/app.py
```

**6.** Find RPi's IP address and run <code>RPi's IP:5000</code> in your web browser.

The shortcoming of this video streaming server is: **it supports only one client!** But, it's fast.

#### By default, flask uses the "index.html" in "flask-video-streaming/templates", we can modify that file to change its appearance. E.g. add 

``` ruby
style="width:100%;"
```

to "<img src= ..." tag. 
