---
layout: post
title: "testing logstash configs with rspec"
date: 2013-01-20 19:51
comments: true
categories: [tdd,logstash,logging,rspec]
published: false
---

At work I'm supporting a rails app, developed by an external company, that we've been suffering performance problems with. 

To quantify this I've been using logstash to 


Since the log format is changing, I've taken the opportunity to clean up the logstash install & configuration. The first thing I wanted to do on this front is to devise an automated testing framework for the logstash configs.

Since logstash 1.1.5 it's been possible to run rspec tests using the monolithic jar, like

    java -jar logstash-monolithic.jar rspec <filelist>


