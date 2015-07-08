---
layout: post
title:  "Introduction"
date:   2015-07-07 21:28:48
categories: tech drupal
---
Among other things, I spend a lot of time working on a [Drupal][drupal] website.  The site's infrastructure is designed for redundancy: two data centers, each with its own crew of web, file, and database servers.  Either data center can go down and come back up with either site being fairly oblivious to the change.

The need for redundant services leads to a need for data replication, which has caused some unique problems for our configuration.  That will be the motivation behind a number of the posts here.

[drupal]: http://www.drupal.org
