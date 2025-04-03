---
layout: post
title: "The Comet Robot"
author: "Paul Dreyer"
categories: documentation
tags: [documentation]
image: comet_front_purple.jpeg
---

## Background

"Comet" is a (soon-to-be-open-source) project I started in 2024. It is an autonomous assistant robot platform 
which acts as a brand new way to interface with software in the home / office. It moves around a 
room it is set up in via [suspension actuation](https://en.wikipedia.org/wiki/Cable_robots), which 
means that thin wires suspend the robot in space, and they change length to drive the robot around 
a room. This results in a robot which appears to hover in place very quietly, can answer questions, 
show apps / games on it's LCD screen, and deliver a rediculously cool experience

## Key Features / Efforts

{% include carousel.html %}

#### -- LCD Screen -- 

The robot is designed around it's 5" diameter LCD screen, which is used as the primary UXUI 
interface. This screen can be used to launch apps such as the camera, opening YouTube, or just
surfing the web.

#### -- UXUI -- 

The robot is equipped with a microphone and a forward facing camera which tracks your hand 
positions. This means you can use your hands to control a cursor on Comet's screen, navigating apps
or just interacting with the robot

It also has little wings on either side it can 'flap' in 2 degrees of freedom. These wings are here
for the sole purpose of giving the robot the ability to physically emote, allowing it to wave at
you, act happy or worried, or even point at things with the laser pointers embedded in the wings!

#### -- Motion Control -- 

There are a ton of devils hiding in the details for controlling a system like this; including cable 
stretch, uncertainties in motor mounting positions, and stochastic spooling of the tension wires to 
name a few. Because of this, I chose to use a deep RL based control scheme, training a network via 
model-free PPO, utilizing PyBullet as the simulator. Importantly, the network is making use of force
control, which it can estimate via load cells inline on each suspension wire. This is mainly to
attemnpt to avoid all of the issues with position control uncertainties that are nearly impossible 
to squash at a low price point. I built custom infrastructure to enable detailed curriculum 
scheduling, allowing the network to learn the basics like stabilizing the robo first, then 
progressing to velocity tracking as well. 

## Design considerations

Comet is born from a dream to have a cost-feasible autonomous robot in the home. To achieve this 
dream, every decision is built on the basis of:

1. Accesible manufacturing techniques for most DIY makers
2. Widely sourcable component selection


## Web-app

<div style="text-align: center;">
  <img src="{{ 'assets/img/comet_webapp_phone.png' | relative_url }}" 
       alt="photo not loaded"
       style="width: 50%; max-width: 500px; height: auto;">
</div>

Over winter break this year, I decided to build a web-app for the robot as well to branch out the 
functionality and give some additional control features. I built this from scratch primarily using 
React and React JS on the frontend, and almost entirely Python + SQL on the backend. The web app
hosts some really cool features listed below!:

#### -- Marketplace: In-house sales + App Store beta --

This tool is dedicated to listing the robot and its variants for sale on the web app. Payments are
not yet integrated, but the general layout and scaling routine is built out!

#### -- Generative Studio --

Whats a project in 2025 without a little gen AI. Generative studio is my take on the current state
of what's possible for the integration of generative AI into user-facing code bases. Here, a user
can query the model to do some unique customization for the robot, and the studio directly updates 
the code base in a hot-reload safe manner, and pushes changes to the robot instantly! My current
demo includes allowing users to create truly unique and fully customizable Comet Character face 
designs (i.e. the purple floating circles are Comet's stock character face). The secret sauce here 
is code formatting and giving truly enveloping context wrapping to the LLM integrating the user
requests with background detail.

#### -- Pilot --

This page will allow users to control the robot from anywhere with an internet connection! Currently
I have the control interface developed from smartphone screens, and the communication scheme built,
but this feature needs a mobile robot to fully test on first!

#### -- User Data base --

I'm using SQL to manage a user database on the backend. This stores limited user information such as 
account login, customization preferences, etc.

