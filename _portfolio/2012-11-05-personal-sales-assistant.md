---
layout: post
current: post
navigation: True
cover:  assets/images/portfolio/psa-cover.jpg
title: Personal sales assistant
excerpt: iPad app used by sales assistants to collect orders
date: 2012-11-05 10:00:00
class: post-template
subclass: 'post'
author: mmischitelli
---

Native iOS ordering app tailored on the needs of a big Italian food company.

Personal Sales Assistant has been in developed for the iPad for 2 years. Entirely coded in Objective-C with ARC, the app was released on iOS 7.0 for iPads only as it required a big screen to be effective.

The backend was built on the rock-solid foundations of an existing AS/400 server. Modern REST-based APIs were developed to hide the complexities of the IBM server, as well as being able, in the future, to transition to a different environment.

Locally, the data upon which the software relied was stored in a simple SQLite DB. This allowed easy querying of data, transactions and a more robust data storage solution compared to simple files. All the queries that the software made on the DB were created using an in-house object-oriented Objective-C library that supported SQLite grammar.

![psa-product-detail](/assets/images/portfolio/psa-img1.png){:height="350px" style="float:left; margin:5px"}
![psa-product-detail](/assets/images/portfolio/psa-img2.png){:height="350px" style="float:left; margin:5px"}
![psa-product-detail](/assets/images/portfolio/psa-img3.png){:height="350px" style="float:left; margin:5px"}