---
date: 2021-01-05 05:20:35 +0300
title: Winch Actuator
subtitle: Hardware Development
image: '/images/actuator/CA1_lid_pop.png'
---

# Background
I developed this actuator as part of a larger robotics project for the "Comet" robot build <a href="/project/comet-robot" target="_blank" rel="noopener"><strong>(Read about it here)</strong></a>. 

The Comet robot was designed to move around its environment suspended by four actuated cables fixed to it's head. It sounds strange, but this turns out to give the robot some really cool properties, for example - it moves extremely quietly, it has an incredibly long battery life, and it won't 'trip' over any obstacles in the room.

However, since this isn't a run of the mill wheeled robot or a quad rotor vehicle etc, there weren't any readily available winch actuators on the market that could meet my needs, so I set out to build my own!

# Design Requirements

This actuator began with five fundamental design requirements:

- **Force Capability**: To lift the robot to a reasonable height, it must pull with at least 110N of force
- **Force Feedback**: The actuator must sense the instantaneous force being applied on the cable for dynamics observability and saftey monitoring
- **Length Estimation**: The actuator must be able to estimate the length of the unspooled cable in order to estimate the kinematic configuration
- **Wireless Communication**: The robot will communicate with each actuator wirelessly, thus it must have a robust, low-latency wireless capability
- **Cost**: Bom cost should be below 200$/Unit



# Building The Hardware

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%;">
  <iframe 
    src="https://www.youtube.com/embed/9CA9UobdyqM" 
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;" 
    frameborder="0" 
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
    allowfullscreen>
  </iframe>
</div>


