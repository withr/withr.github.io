---
layout: post
title: "Data fetch, infomation extraction and web application: all in R!"
date: 2016-02-08 10:31
comments: true
categories: R Shiny
---



Several years ago, I created a web page to show the exam scores of schools in Norway on Google Map. The original intention of creating that web page is to help us avoid the area where the schools' score are low when we want to change our apartment. Recent days, I found many people have intresting of this kind information. So, I created a [web-app](http://shinyapps.me/NationalTest5/) for that purpose with R. In case somebody want to do the similar work, I wrote the steps how I created the web-app. You can modify some of my codes to create the app you needed.

## Step 1: fetch data from internet. 

**Note**, if you have got your data ready, just skip this step.

The exam scores data can be found in [https://skoleporten.udir.no](https://skoleporten.udir.no/rapportvisning/grunnskole/laeringsresultater/nasjonale-proever-5-trinn/nasjonalt?enhetsid=00&vurderingsomrade=11&underomrade=50&skoletype=0&skoletypemenuid=0&sammenstilling=1). We can see there are 4 figures, and what I have interesting is the percentage of the students fall into to level 3 (high) for each subject. There is an option allow you to download the data of the figures mannually, but I don't want to do that mannually, so I downloaded the source of this page and found the the figure's data was stored in JSON format. JSON format array can be easily converted into **list** class of R, which is much easier to mannage for me. The page contains the average scores of whole Norway, and we need to find all schools' URLs. By inspecting the elements of the page, we can find the there are 19 fylkes (counties), see the following screenshot: 


![]( /images/NationalTest5/screenshot-skoleporten.udir.no 2016-02-08 10-00-49.png )


If we click one fylke, we can find some schools inside that fylke, but not all. However, if you inspect the elements of the page again, we can see all the rest schools were also listed, but weren't shown.


![]( /images/NationalTest5/screenshot-skoleporten.udir.no 2016-02-08 10-07-45.png )


So, we can get all schools' URLs and download their pages. The following is the R codes for downloading all schools' pages.

