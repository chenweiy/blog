---
title: "Linux 60-Second Analysis"
author: ["Chenwei Yang"]
date: 2020-11-24T00:00:00+08:00
lastmod: 2020-12-26T15:03:20+08:00
tags: ["hi", "test", "linux"]
draft: false
author: "Chenwei Yang"
math: true
---

This checklist can be used for any performance issue, and reflects what I typically execute in the first sixty seconds after logging into a poorly-performing Linux system.&nbsp;[^fn:1]

The tools to run are:

1.  uptime
2.  dmesg | tail
3.  vmstat 1
4.  mpstat -P ALL 1
5.  pidstat 1
6.  iostat -xz 1
7.  free -m
8.  sar -n DEV 1
9.  sar -n TCP,ETCP 1
10. top

[^fn:1]: [Linux Performance Analysis in 60,000 Milliseconds](https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55)
