---
layout: post
title: "Add domain name (host by Godaddy) for your server on DigitalOcean"
date: 2016-02-01 10:31
comments: true
categories: 
---



I bought a domain name from Godaddy long time ago. I thought to configure a domain name will take some time, that's the excuse why I didn't use it until now (shame on me).  Now I have to link my server to the domain name, and I found it's very easy. So I post the steps here, in case I need it in future. 

Two steps in short: **add DigitalOcean domain servers to the domain name (you bought) on Godaddy, and add domain name (you bought) to your droplet on DigitalOcean**.

### 1. Add DigitalOcean domain servers to Godaddy


Log onto your Godaddy account, and choose the domain name you want to link to your DigitalOcean droplet. Choose "Manage" of **Nameservers**.

![]( /images/DG_domain/screenshot-dcc.godaddy.com 2016-02-01 10-17-44.png )

Add DigitalOcean domain servers as shown in following picture.

![]( /images/DG_domain/screenshot-dcc.godaddy.com 2016-02-01 10-10-15.png )


### 2. Add domain name to DigitalOcean


Go to your DigitalOcean account, select **Add a Domain**.

![]( /images/DG_domain/screenshot-cloud.digitalocean.com 2016-02-01 09-59-55.png )


Add the domain name you bought. 

![]( /images/DG_domain/screenshot-cloud.digitalocean.com 2016-02-01 10-01-17.png )

Done! Type your domain in the URL address of a web browser to check. 

**Note**, it may take some time for the DNS to propagate!

