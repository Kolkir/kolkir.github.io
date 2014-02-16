---
published: false
title: C++ serialization with indexing
layout: post
tags: c++
---

I've started C++ serialization library implementation, the main difference from other similar libs, should be possibility to make effective search by keys through the archive. 
I've begin with creation the RecordFile class for C++ objects serialization to the file. When user save some object with this class, he receive file lacation object which can be used to rewrite original saved object. On top of this class I've made BTree indexation in separate file. 
Now there is an errors with removing keys from tree. Also it is not easy to make test data for unit test, so I've begin making tree editor with UI.
Link to the current code on [github](https://github.com/Kolkir/btree.git).
