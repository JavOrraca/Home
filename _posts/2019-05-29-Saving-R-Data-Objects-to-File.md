---
layout: post
title: "Saving R Data Objects to File for Quick Loading"
---
<br>
![](https://raw.githubusercontent.com/JavOrraca/Home/gh-pages/assets/img/R-Data-Formats.png)

R users! Are you spending too much time reading large data sets, wrangling, then waiting to convert your data frames to matrices or arrays (and if you're like me, you might be doing a lot of this locally)... All of the sudden, you've got 25-50 GB (or more) worth of vector objects sitting in your environment and it's taken 10-15 minutes to even get to this point where you can begin modeling...

Here's a solution for you: Use R to save one or multiple data objects to file as RDS or RData files. I got this amazing tip from [Emil Hvitfeldt](https://www.hvitfeldt.me/blog/) and I've already saved significant time from this recommendation.

Source:
* [STHDA Guide for Saving RDS and RData files](http://www.sthda.com/english/wiki/saving-data-into-r-data-format-rds-and-rdata)