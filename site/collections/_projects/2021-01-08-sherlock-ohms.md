---
date: 2021-01-08 06:20:35 +0300
title: Sherlock Ohms
subtitle: Robot Build
image: '/images/outlet_bot/sherlock_ohms_ISO_wide.png'
---

# Background

This is a robot which autonomously explores a room / corridor and tests each outlet for 
functionality. The use case here is demonstrating an ability for a robot to self-dock for recharging 
in an arbitrary room without the need for a custom charging base.


# Testing The Outlet

In order to traverse a flat room and test outlets, we need to be able to move a probe head around
the space with four degrees of freedom: X, Y, Z (height from ground), and Theta (the orientation 
angle). I chose to use a mecanum wheel setup as our translation system, as this allows for omni-
directional travel of the robot base in X, Y, and Theta, which means we can pose the robot in any 
position and orientation we want to without kinematic restrictions. Now, instead of needing a multi-
axis actuator to handle the variable outlet heights, we only need to account for Z which lead me to 
design a single degree of freedom actuator to move the probe head, changing its height from the 
ground.

# Sensing The Environment

In order to actually plug in and test an outlet, we need to be able to gain environmental
information for the robot to make decision on. Our primary sensor that enables perception is our
D435 camera, which is mounted to the tool head. The camera can simultaneously provide point cloud
data from its active stereo suite which allows us to gain an understanding of distances to objects
in the environment, as well as photos of the forward view of the robot. We can then inference a 
custom trained Yolo-V8 model on the photos the D435 captures to predict outlet locations.

![Tool Head](/images/outlet_bot/sherlock_ohms_tool_head.jpeg){: width="1200" height="900"}

The tool head is equipped with two conductive "pogo-pin" probes, which conduct the current from the 
outlet to our AC voltage sensor on board. The probes are mounted on a 'break-away' surface at the 
end of a load cell in the tool head, which allows us to sense the axial force applied to the
test probes. This enables the robot to sense if the probes are in contact with the outlet, and stops
advancing if a safe limit force is experienced. For additional saftey, the probes are able to 
'break-away' from the tool head, when the magnetic force holding them to the tool head is exceeded.
This is designed to allow them to shear from the robot in the event that our trajectory planning or
tracking experiences errors that could damage the probes.

Finally, the robot is also equipped with encoders on each drive motor, as well as an inertial
measurement unit (IMU) onboard for simultaneous localization and mapping (SLAM) closure. 

![Components](/images/outlet_bot/sherlock_ohms_top_open_hub.jpeg){: width="1200" height="900"}

# Mecanum Wheel Demo

<a href="https://youtube.com/shorts/h42TzjA3sZ4?si=z6tAcEfAXYWnJIHy" target="_blank" rel="noopener"><strong>(Link to YouTube Short)</strong></a>.
