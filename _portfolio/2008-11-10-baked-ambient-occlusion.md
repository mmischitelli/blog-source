---
layout: post
current: post
navigation: True
cover:  assets/images/portfolio/ambient-occlusion.jpg
title: Baked ambient occlusion
excerpt: A naive approach to computing and storing ambient occlusion data for arbitrary meshes
date: 2008-11-10 10:00:00
class: post-template
subclass: 'post'
author: mmischitelli
---

During my trainee at the CNR of Pisa, I had the chance of working on various techniques for improving the visual perception of 3D meshes: ambient occlusion was one of those.

In particular, I was tasked with the job of writing an algorithm for pre-computing ambient occlusion values for any given mesh and store it in a texture for later use. This algorithm had to be put in a QT/OpenGL plugin for the open-source software [MeshLab](http://meshlab.sourceforge.net/). Using the classical definition of Ambient Occlusion (which is often related to *accessibility shading*), each fragment is more or less visible to others according to how much of it is covered or shown by the surrounding geometry.

According to a research conducted in 2000 by [H.H. Buelthoff and M.S. Langer](http://dx.doi.org/10.1068/p3060), humans use a more accurate model than dark-means-deep to perceive shape-from-shading under diffuse lighting, and ambient occlusion is a component of this model.

By integrating on the hemisphere surrounding a given point, it’s possible to determine the visibility for that fragment. This approach is quite heavy on the hardware, but it’s one of the most accurate ways for calculating ambient occlusion. While a CPU-only approach required even minutes to compute, using the GPU to elaborate evertything (by writing and accumulating data on a single texture) reduced total time by one order of magnitude. By using multiple render targets is possible to work with very complex geometry, up to hundreds of milions of polygons.

![540k-polys-under-6-seconds](/assets/images/portfolio/540k-poly-hw-acc.jpg){:class="img-responsive"}