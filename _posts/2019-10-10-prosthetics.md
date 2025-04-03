---
layout: post
title: "Low Cost Haptic Prosthetics"
author: "Paul Dreyer"
categories: documentation
tags: [documentation]
image: prosthetics.JPG
---

## Background

This was one of my earliest projects and took place during my gap between my active duty service and
becoming a full time student. I was ~4 months into a backpacking trip through South America, but
actively learning computere aided design (CAD) knowing that I wanted to go to school for engineering
after the trip concluded. I wanted to push myself to do something new, and interfacing with the 
human body seemed like an awesome challenge. I decided to dip my toes into the world of below-elbow
prosthetics, which just means the devices are designed for amputees who have some portion of their
forearm remaining post-surgery. I had spent roughly a month designing this arm, not really thinking
this could ever actually be useful for someone when something serendipitous happened. I was having a
coffee in a cafe in Medellin, when I noticed a donation box in the cafe, with a 3D printed 
prosthetic arm in it that I recognized from some research I had done trying to gain some foundation
knowlege in the space. Well, it turned out that the company Humanos-3D (The Colombia chapter of the 
Seattle based E-nable) was based in the flat above the cafe. I crossed paths with the owner at the
time and explained what I was working on, and he let me start printing my arms as an inpromtu
intern with the company! (Big thanks to Adam for being such a rad human)


## Design considerations

The arm needed to be low cost, both for my own sanity as an unemployed traveller, and for whoever 
this could be donated to some day. This means it's entirely 3D printed from PLA, and has very simple
electronics and actuation on board.

The user can flex the elbow, causing the fingers to close on the hand. The fingers actuate in a 
mechanically resistive-adaptive manner, meaning that the force is automatically distributed across 
all fingers allowing a user to grasp anything from a cup, to a tennis ball, to a door handle. 

There is also some very naive vibration based haptics included in the arm. There are force sensors
(strain resistors) embedded into each finger tip. As these experience force applied, they vibrate a
small coin motor (similar to whats in a smartphone) with pulses that vary in speed and intensity
proportional to the force in the finger tips.