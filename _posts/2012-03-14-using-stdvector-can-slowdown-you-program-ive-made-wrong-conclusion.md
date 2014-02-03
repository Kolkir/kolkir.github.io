---
title: Using std::vector can slowdown you program - I've made wrong conclusion
layout: post
tags: 
---


When you know sizes need to copy you have to use std::copy function - it
is faster for some cases even memcpy.
