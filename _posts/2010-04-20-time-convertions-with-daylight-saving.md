---
title: Time convertions with daylight saving
layout: post
tags: 
---


I've found how to convert time from UTC to local time with daylight
saving correction:

1.  Use mktime() to create time from string(or some thing else)
2.  get time from gmtime() and localtime()
3.  calculate difference between time to get timezone offset (or
    use\_get\_timezone)
4.  use tm::tm\_isdst to determine apply or not daylight saving
5.  use \_get\_daylight() to get value in hours of daylight correction.

