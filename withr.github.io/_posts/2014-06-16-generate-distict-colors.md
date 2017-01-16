---
layout: post
title: "Generate Distinct Colors"
date: 2014-06-16 09:42
comments: true
categories: R Shiny
---

When plot with R (or some other software), it doesn't bother you a lot for choosing colors if you have only several data series. For example, if I want to create a figure for 5 data series, I can easily choose *red*, *green*, *dark green*, *blue* and *black*. But what if I need to create a figure having 100 data series? Of course, we can choose different *shape* or *pattern*, like *dash line*, *circle*, *triage*, etc. However, to me, I feel that make the figure messy, I prefer all series in same style, except their colors. I created a [Shiny-app](http://188.226.199.99:3839) which can generate random distinct colors in a certain color plate, would like to share it. 

![]( /images/DistictColor1.png )
  
![]( /images/DistictColor2.png )


The design idea is like this: 

1. Generate N random points. Some points are very close to each other, so they are not easy distinguished.
2. Find the point-pair which have the shortest distance, pick up one point of it and put it to a new location where it has the largest distance to all the rest points.
3. Continue the above step, until there is no new location meet requirement.

