---
layout: post
title: "Encrypt user's password with md5 for you Shiny-app"
date: 2014-02-14 14:50
comments: true
categories: R Shiny
---


In a previous [blog](http://withr.me/blog/2014/01/03/authentication-of-shiny-server-application-using-a-simple-method/), I post a simple authentication method for Shiny-app, and received several comments mainly concerning that I should encrypt user password. I agree, user's password can be intercepted when it was transferring. To secure users' personal information, I think we should consider both server and client sides. 

Your server could be hacked and all users' personal information could be copied. Such things have happened to some big websites last year. Besides, when data was transferring through internet, due to signal attenuation issue, all data will be enhanced through some deviance, such like network switches, where users' information could also be intercepted. So, how to secure your password? The best way is: don't store it on server and don't transfer it through internet!

What? Are you kidding? Sorry, I'm not! 

The idea is simple: before transferring your password, encrypt it first. This is common concept for IT guys, but maybe not for most R users.

Shiny has no such feature, which belong to Shiny-pro, the commercial version. But we add it ourselves because Shiny is open.

1. Includes md5.js to the head part of ui.R
2. Create and includes a new ShinyBinding to receive the encrypted password.

That's all!

Here is a [demo](http://spark.rstudio.com/withr/authentication/) (username: withr; passwd: 12345678) and you can find source code [here](https://gist.github.com/withr/9001831).
