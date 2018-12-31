---
title: Self Driving Car Overview
subtitle: A plan for our self-driving car
tags: [self driving car, python, coding]
---
Over the next couple of months, I want to make a self-driving car. Okay, I know that sounds really fancy,
and honestly even I am not sure ~~how much of a success~~ how big a failure it will be, but there's no harm
in trying, and it will be a _learning experience_ either way. I have decided to share this journey with the
world through my personal blog so keep following me over the next couple months to see how far I can go. And
if you have any suggestions to help me along the way, I'm all-ears! I am doing taking up this challenge with
the goal of learning, after all. (Learning what, you ask? I will be able to tell you only after I am done
with this project.)

Turns out, programming a self-driving car is not as difficult as all the big companies make it seem:

![Self driving car programming](/img/self_driving_car_programming.jpg)

In all seriousness though, we need to have a solid plan about how we are actually going to make the self-driving
car. First, let's clearly define the scope of this project; after all, people have wildly different ideas about
what constitutes a self-driving car.

## Goal

The ultimate goal of this project is to build a self-driving RC car (1/10<sup>th</sup> scale) that will be able to compete in
the [DIY Robocars](https://diyrobocars.com/) [Oakland races](https://diyrobocars.com/diy-robocar-races-110th-and-116th/).
You can check out the links for more information, but here is a summary: about once every 2 months or so a bunch
of smart people who've made their own self-driving RC cars meet up in a warehouse in Oakland and race their
creations. Since I happen to live only about 12 miles from there, I have no reason not to go there.

So I will call this project a success only when **I successfully make a self-driving car that is
able to meaningfully compete in one of those races.**

## How to get there

Now that we have a clear picture of what I want to do, let's break this problem down into simple, tangible parts
that can be coded or built.

### Hardware

As a simplification, I have decided not to worry too much about hardware until the software is ready. The only
thing I can say with certainty at this point about the hardware is that it will be a 1/10<sup>th</sup> scale
RC car with a single-board computer onboard (a Raspberry Pi or some slightly more powerful board) along with
a camera and an IMU. As for LIDAR, I am not sure whether I will need it or not, and I certainly don't want to
buy a LIDAR sensor right now only to find out later that I don't need it, so I am going to take care of the
software side first.

### Software

On the software side, we have two tasks: steering the car and setting the throttle. Contrary to what some
of you might think, I will not use *just* a neural network to steer the car and set the throttle of the car,
because it is impossible to make a neural network that can take in raw images and output steering angles.

![can't throw a neural network at everything nvidia](/img/cant_throw_nn.png)

Actually, Nvidia showed in [this paper](https://arxiv.org/abs/1604.07316)
and [this blogpost](https://devblogs.nvidia.com/deep-learning-self-driving-cars/)
that end-to-end neural networks can do just that–take in raw images and output steering angles.
But it is not exactly a good idea to just throw a neural network at every problem. This is because neural
networks are complicated functions and their behavior is pretty much impossible to predict when it encounters
an input that is far from the training data, i.e., there is no easy way to predict how a neural network will
extrapolate beyond the training data, even if it can reliably *intra*polate. For example, if we train a
neural network to drive a car in a track, we will have no idea about how it will behave on a different
track until it actually drives on it. In fact, given training data that is biased enough, neural networks
can learn to pick up non-obvious objects surrounding the track ([look here](https://youtu.be/qvRja-Veec4?t=751)
for a neural network that learned to steer whenever it saw a particular table next to the track).

On the other hand, if we make a lane-detecting algorithm, we will know for certain that it will
behave just as reliably on any track as long as the lane lines are visible. However, such feature-engineering
has its own drawbacks, namely the inability to generalize. While making such an algorithm, we must come up
with and handle all possible cases that the algorithm will encounter, because if it encounters a situation
that we did not consider, we know exactly what will happen: it won't work.

So we need to find some middle-ground, some Goldilocks zone between an end-to-end deep learning model
and a pipeline consisting of feature-engineering only, which will combine the best of both worlds. This
will be the primary challenge of this project.

I have decided to approach this by starting with some feature engineering for finding lanes and other objects
in the scene and then helping the model generalize using reinforcement learning (RL). I will use the
[CARLA simulator](http://carla.org/) (version 0.8.2, the stable release) to train my RL models and later
test my models using both the CARLA simulator and the [donkeycar simulator](http://docs.donkeycar.com/guide/simulator/),
which has tracks that more accurately represent the environment in which the donkeycar Oakland races are
held. Instead of starting with a deep model, I will start with feature engineering and then
augment/refine it with deep RL.

Making a lane detector is not a *very* complex problem. The Udacity self-driving car nanodegree includes
exactly that as their fourth project, so there are no less than a dozen GitHub repos that contain code for
lane detection (just google it or search it up on YouTube). In fact, once you understand all the concepts
involved, it can it can be implemented in an afternoon. At this point, I would suggest you watch a video on
that or read through some blogpost that explains the pipeline, otherwise the following might not fully make
sense.

The biggest problems with the "advanced lane detection" algorithm are that:
1. the perspective transform fails if the camera tilts (due to linear acceleration or turning)
2. the algorithm can fail at hairpin bends and sharp turns in general if the lanes turn back into the camera frame
3. the algorithm assumes that there is a single lane bounded by two lines
4. the algorithm has no way to detect other vehicles, and even fails to detect lanes when another car blocks its field
   of view


I can think of the following solutions to these problems (in order):
1. Assuming that the camera and the IMU are rigidly attached to the car, we can determine the tilt of the
camera tilt measurements from the IMU and factor that into the perspective transform that is applied to
the image.
2. One way I can come up with to solve this problem is to somehow change the region of interest depending
on the scene, but it is not obvious to me how we can do that. Maybe we can use deep learning for this? I am
actually looking forward to suggestions in the comments about this.
3. This may or may not be a problem depending on the rest of our code (I will look into this later).
4. In order to detect other vehicles, we can use an object detector like a YOLO neural network, but it is not
trivial (and impossible without some sort of calibration) to figure out how far the detected vehicles are.
Finding the lanes when blocked by some vehicle/object will require some sort of simultaneous localization and
mapping (SLAM) algorithm in order to synthesize information from previously seen frames (because any given
section of the track will be captured in multiple consecutive frames and be visible in at least some of them
even if it is blocked in some others; think about how you drive when there is a box truck right in front of
you on the highway).
