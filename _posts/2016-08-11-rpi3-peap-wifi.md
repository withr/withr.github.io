---
layout: post
title: "Get access to PEAP WiFi from Raspberry pi 3"
date: 2016-08-11 13:32
comments: true
categories: R Shiny
---

Most of the tutorials available are for EAP WiFi, which require just a password. PEAP WiFi network used generally for company, and each user has an ID and a password. To access this kind of WiFi network is also very easy: 

### Step 1: check the IP address of your wifi module: 

```
ifconfig
```

You will see that “wlan0” has no IP address. Then you need edit the content of **/etc/wpa_supplicant/wpa_supplicant.conf** as the following: 


```
ctrl_interface=/var/run/wpa_supplicant
network={
      ssid="wifi name"
      key_mgmt=WPA-EAP
      eap=PEAP
      identity="your ID"
      ca_cert="/etc/certs/radius.pem"
      password="your password"
   }

```

If any item you don’t know, like the “ca_cert”, just comments that line by place "#" in the beginning of that line.

### Step 2: edit the config of wlan0 in **/etc/network/intefaces** to: 


```
iface wlan0 inet dhcp
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

```

### Step 3: restart the network, and check if wlan0 has its IP address

```
sudo /etc/init.d/networking restart

```

After couple of seconds, if you can see a green “ok”, then it means your raspberry pi has connected to the WiFi.  You can check the network connection using command:

```
Ping www.google.com

```
