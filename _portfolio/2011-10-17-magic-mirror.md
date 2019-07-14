---
layout: post
current: post
navigation: True
title: Magic mirror
excerpt: Multithreaded video recording and playback software controlled via proximity sensors
date: 2011-10-17 10:00:00
class: post-template
subclass: 'post'
author: mmischitelli
---

Also called Virtual Fashion, Hypersoft's vision for retail-focused kiosk consisted of a high luminosity LCD panel mounted on the back of a two-way mirror.

The user could interact with it just by waving his hands, even meters away from the kiosk. The software was constantly polling a proximity sensor that, when hit an obstacle at a target distance range, would start the main app's process.

Usually the screen would just display a black picture so that from the other side of the mirror it seemed that nothing was behind it. When previously mentioned process started, the software started two different processes: the first would start recording wathever was happening in front of the kiosk; the other one played a simple video inviting the user to move.

Technically speaking, the challenges in developing such solution were mostly tied to multithreading and playing/recording high framerate video streams. Being this a R&D project, the budget was limited and had to rely on freely available libraries or system APIs (DirectShow).

<iframe width="560" height="315" src="https://www.youtube.com/embed/ylF5Lb-UdK8?controls=0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>