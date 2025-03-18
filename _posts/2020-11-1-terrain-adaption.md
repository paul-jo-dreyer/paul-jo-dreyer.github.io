---
layout: post
title: "Terrain-Adaptive Contact Surfaces"
author: "Paul Dreyer"
categories: documentation
tags: [documentation]
image: terrain_adaption_wheel_cropped.JPG
---

## Background

I competed in the NASA Artemis Challenge in 2020. We were tasks to build a small <= 1ft^3 "lunar
rover" that was to autonomously explore a simulated lunar lava tube. The rover would need to 
traverse multiple terrain types, and above all else not get stuck. A common way to deal with loose
terrain is incorporating [grousers](https://en.wikipedia.org/wiki/Grouser) into the traction surface
. However, these cause large efficieny losses when traversing smoother terrain. To over come this,
I developed this wheel shown above, which allows for active control of grouser height from wheel
surface! A small stepper motor attaches to the back of each wheel, and drives a lead screw through
a hollow axel. this moves some linkages that cause the radial actuation of the grousers (black)
which actively change the contact geometry as the robot needs!
