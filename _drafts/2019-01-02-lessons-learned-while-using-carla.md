---
title: Lessons Learned while using CARLA
subtitle: I went through it all so you don't have to
tags: [self driving car, python, coding, simulator]
---
I've had quite an adventure over the last three days trying to get what I want out of [CARLA](http://carla.org),
my simulator of choice for the self-driving car project. There are a number of reasons I chose CARLA
(look at the [previous post]({% post_url 2018-12-29-self-driving-car-overview %})), one of the most important being
its ability to generate ground truth for semantic segmentation, which will simplify tasks like lane line
and object detection. But CARLA ain't no easy beast to tame, especially when the tamer in question is stupid
enough to try to build the cutting-edge version from source _on Windows_.

![build on Windows meme](/img/build_on_windows.png)

For the record, I was able to successfully build simulator [version 0.9.2](https://github.com/carla-simulator/carla/releases/tag/0.9.2)
from source (albeit after a whole night of sifting through GitHub issues). But the CARLA simulator is
unusable without the Python Client library, which also needs to be built from source. As it currently stands,
the client library does not compile on Windows. As you can see [here](https://github.com/carla-simulator/carla/issues/976),
the issue is yet to be resolved. So I did the most logical thing: I gave up on the cutting-edge.
I settled for [version 0.8.4](http://carla.org/2018/06/18/release-0.8.4/).

But hadn't I said in the [first post]({% post_url 2018-12-29-self-driving-car-overview %})
that I was going to use the [stable version](http://carla.org/2018/04/23/release-0.8.2/) (0.8.2)? Why did I
change my mind?

* Version 0.9.2 has _three different maps_! That means a lot of variation in training data, especially for
the semantic segmentation network.
* I wanted to make my own maps as well, which necessitates building from source.
* I was feeling adventurous and I wanted to try something new and challenging to start the year off.

Unlike my new year's 10K run (which was awesome!), the build turned out to be more of a failure. So I settled
for version 0.8.4, for the following reasons:

* There are 2 maps, unlike 0.8.2 which only has one.
* There is a [prebuilt binary](https://github.com/carla-simulator/carla/releases/tag/0.8.4) for Windows. Even
though it says "experimental" right beside it, it has run smoothly for me so far.
* There are more kinds of vehicles (including two-wheelers and a Tesla Model 3), so more variation in the
training data.

So here is a little tutorial on how to use CARLA effectively to generate data.

## Architecture

CARLA is composed of two fundamental parts: the simulator and the Python client library.
**_FINISH THIS_**

Note: CARLA, at its heart, is a moderately demanding game made using Unreal Engine. And capturing data while
driving is an even more graphically demanding task because it requires more images to be rendered. So you
will most certainly need a dedicated graphics card in order to run CARLA at reasonable framerates.