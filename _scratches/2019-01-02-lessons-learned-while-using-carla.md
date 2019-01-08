---
title: Lessons Learned while using CARLA
subtitle: I went through it all so you don't have to + code
tags: [self driving car, python, coding, simulator]
---
> Update: The self-driving RC car project now has a GitHub repository! You can find all the code that I end
up writing [in this repo](https://github.com/sagnibak/self-driving-car). There are detailed instructions
in the readme for you to be able to use all the code.

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

## The Simulator Server and the Python Client

CARLA is composed of two fundamental parts: the simulator and the Python client library.
The idea is that the simulator runs as a server, and the user can send commands to the server and
receive images and sensor measurements using TCP by running a Python script. It is necessary to
use the Python client library that comes with the simulator. It is written in C++ for
(a) performance and (b) smooth integration with the simulator which is also written in C++.

Now, if you want to follow along, you have two options:
1. If you have a computer with a decent GPU (I'd say at least a GTX 1060 or 1070), then you can go ahead
and download the prebuilt binaries from [here](https://github.com/carla-simulator/carla/releases/tag/0.8.4).
Do yourself a favor and don't try to build from source, unless you are on Linux and you know what you're doing.
I'm using version 0.8.4, as I mentioned earlier.
2. If you don't have the aforementioned setup but you want to play with the data, then ask me for the data I
have collected in the comment section, and suggest some nice way of sharing several gigabytes of data.

I am including the client code that I used to collect data in the [official repository](https://github.com/sagnibak/self-driving-car)
for this project, but keep in mind that you will need to get the simulator in order to run it.

<!-- Now, if you have the simulator, you need to grab [this script]() and put it in the `carla/PythonClient`
directory. Next, start the simulator with the command `./CarlaUE4.sh -carla-server` on Mac/Linux or
`CarlaUE4.exe -carla-server` on Windows. Finally, start the data collection script with `python3 <script>.py -i`.
The `-i` flag is to instruct the script to save images to disk. It takes a few seconds to start, but once
started, you should see a black PyGame window (this is because I turned off rendering on the PyGame window
to improve performance). This window needs to be in focus, because essentially it will take in WASD (and other)
keyboard input and pass it on to the server to control the car, and it will also listen for incoming image
data and save it. -->

You can follow the instructions in the [readme](https://github.com/sagnibak/self-driving-car#carla-simulator-scripts)
if you want to use my scripts to collect your own training data. Here I am going to explain how the scripts
work, but I will not go over how to use them (because I have already explained that part in the readme
linked above).



Note: CARLA, at its heart, is a moderately demanding game made using Unreal Engine. And capturing data while
driving is an even more graphically demanding task because it requires more images to be rendered. So you
will most certainly need a dedicated graphics card in order to run CARLA at reasonable framerates.
