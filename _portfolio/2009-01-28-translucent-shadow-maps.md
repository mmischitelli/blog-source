---
layout: post
current: post
navigation: True
cover:  assets/images/portfolio/subsurface-scattering.jpg
title: Subsurface scattering using TSM
excerpt: A fully real-time technique for computing the subsurface scattering of light in materials
date: 2009-01-28 10:00:00
class: post-template
subclass: 'post'
author: mmischitelli
---

The third and last technique explored during my trainee at the CNR of Pisa involved implementing a real-time algorithm for simulating the so-called subsurface scattering.

When photons hit the surface of a solid, they can either bounce back or go through the various layers of which the material is composed. During this journey through the material, photons constantly interact and bounce (scatter) in pseudo-random directions, eventually coming out of it from a different angle and direction.

![scattering-in-materials](/assets/images/portfolio/tsm_scattering.png){:width="300px"}

Physically simulating this requires perfect knowledge of the material's composition, as well as tons of resources. For this reason, real-time simulation of this phenomenon requires approximation: one such technique was presented by C. Dachsbacher and M. Stamminger in 2003 and is named [Translucent Shadow Maps](https://dl.acm.org/citation.cfm?id=882404.882433).

![tsm-in-a-nutshell](/assets/images/portfolio/tsm_xin_xout.png){:width="500px"}

As the name suggests, this technique is similar to the traditional shadow mapping, extending it. Translucent Shadow Maps contain depth and incident light information. Sub-surface scattering is computed on-the-fly during rendering by filtering the shadow map neighbourhood. This filtering is done efficiently using a hierarchical approach, as shown in the following image:

![tsm-filtering-samples](/assets/images/portfolio/tsm_filter.png){:width="200px"}

This algorithm is usually combined with lambertian and specular reflectance functions to form a rough approximation of the BSSRDF. In the sample image below you can compare the same image with and without the scattering contribution:

![tsm-filtering-samples](/assets/images/portfolio/tsm_comparison.jpg)