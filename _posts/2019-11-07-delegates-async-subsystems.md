---
layout: post
current: post
cover:  assets/images/posts/delegates-async-subsystems.jpg
navigation: True
title: UE4 - delegates, async and subsystems
where: MakeIt Modena
date: 2019-11-07 20:00:00
tags: [ue4, cpp, talks]
class: post-template
subclass: 'post'
author: mmischitelli
---

Talk held on the 7th of November 2019 at MakeIt Modena for the [Italian C++ Community](https://www.italiancpp.org/).

As with most game engines and game architectures, Unreal developers often rely on events and delegates to wire up different systems and make them easily interact with each others. In UE4 there are many (I count up to 6!) different types of delegates and many more so ways of binding to them: for this reason I deemed useful highlighting their main characteristics and use cases, along with some code showing how they are declared and used. 

Then I moved on talking about the asynchronous execution primitives and higher level constructs that the framework provides the developer with. We saw how to create threads, how to declare ad use a mutex and even how to spawn external processes, monitor and communicate with them, all through the various mechanics that have been implemented by Epic in the engine.

The last portion of the meetup was dedicated to Subsystems, an architectural pattern recently introduced in UE4. They are defined as automatically instantiated classes that have a well defined lifetime. During the meetup we see what they actually are, what they are not and how you can declare them.

### Resources
- Repository: [Example code on GitHub](https://github.com/mmischitelli/MeetupNov19)
- Slides: Delegates, Async and Subsystems [\[.pdf (1.2MB)\]](/assets/downloads/talks-delegates-async-subsystems.pdf)
- Video: Live event recording [YouTube (ITA)](https://www.youtube.com/watch?v=mB6afDMzTjQ)
