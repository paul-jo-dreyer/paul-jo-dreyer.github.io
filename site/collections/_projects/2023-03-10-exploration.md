---
date: 2023-03-10 05:20:35 +0300
title: Robot Exploration
subtitle: Algorithms
image: '/images/exploration/mapping.png'
---

# Background
I took a class called "Mobile Robot Algorithms" in college, where we got some exposure to a ton of different core algorithms, everything from mapping and localization, to dynamics and feedback control. As part of my final project, I developed this myopic 3D grid-based exploration simulation in Python.

![Gif](/images/exploration/exploration.GIF){: width="1200" height="900"}


# How It Works
What you're seeing here is a toy example of a robot (the blue sphere), exploring an environment while avoiding obstacles (dark grey voxels). The robot is equipped with an omnidirectional lidar sensor (blue lines) which it uses to sense obstacles, and update the occupancy map.

Exploration of the environment is encourage through a myopic mutual information based planner - meaning that at each time step, the robot computes the entropy of the map, and moves to the nearest voxel which would lead to the largest reduction in total map entropy. i.e. "Where should I move to learn the most new information about the map -> obstacles and free regions?". Myopic just means that it's only considering a single time step into the future. 


