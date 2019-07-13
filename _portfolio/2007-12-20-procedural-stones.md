---
layout: post
current: post
navigation: True
cover:  assets/images/portfolio/procedural-stones.jpg
title: Procedural marble and granite
excerpt: Using perlin noise to procedurally generate textures instead of photos for 3D visualization
date: 2007-12-20 10:00:00
class: post-template
subclass: 'post'
author: mmischitelli
---

Instead of using photos, which could be heavy on the VRAM and have tiling problems, we could use mathematical functions to generate textures.

In games or in real-time interactive programs, simple photos are often used as textures. While the result might be very good looking, there are also drawbacks: tiling and memory consumption just to name a few. One of the possible solutions to this problem is represented by *procedural textures*. There are many techniques that could be used to generate textures, one of which [was invented by Ken Perlin](https://en.wikipedia.org/wiki/Perlin_noise) and is thus called **Perlin Noise**.

The main idea behind Perlin noise is based on real-life observations: looking at a distant mountain it is possible to see just the biggest jaggies; the more one gets closer to it, the more smaller detail begin to emerge. Using the same concept, it is possible to add together different variations of the same mathematical function, each representing variations at a different frequency (or *scale*). The result is a distinct shape, characterized also by smaller jaggies that add detail on small scale.

![procedural-marble](/assets/images/portfolio/procedural-marble.jpg){:class="img-responsive"}