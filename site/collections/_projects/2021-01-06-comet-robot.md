---
date: 2021-01-06 05:20:35 +0300
title: Comet
subtitle: Robot Build
image: '/images/comet/comet_char_1.png'
---
# Background
Comet is my main squeeze, I've been working on this robot for about a year and a half from the first
idea, and have been pouring as much love as possible into this little guy. 

Comet is a suspension actuated robot (AKA Cable Driven Parallel Robot), meaning it swings around the
room suspended by four actuated cables. It's entire purpose is to be an interactive companion bot,
and is really just a platform for me to learn and implement a ton of different skills during its
development. 

# Why Suspension?
Yes, it sounds strange - why would I want a robot swinging around the room near
my head? Well, it turns out that while this does present some really unique challenges - this style
of locomotion also offers some really cool benefits too. Unlike noisy rotor-based drones - Comet can 
move almost silently around the room, and because the locomotive energy is located off-body, the 
robot can "hover" or hangout in a position almost indefinetly - so no need to worry about short 
battery life here. Additionally, being a pseudo-aerial vehicle makes the actual locomotion task
much easier to plan around, and we can bypass many of the challenges that ground based vehicles face!

# What can it do?

Comet is equiped with a 5" round LCD screen, as well as a forward facing camera. This let's it
deploy a GUI for the user, where you can move a cusor around the screen using hand-tracking from the
onboard camera. I can pull up a youtube video while I;m on the couch, or see what my cat's up to 
while I'm out! LOL

![App UI](/images/comet/comet_front_apps.PNG){: width="1200" height="900"}

# How does it move?

Comet is semi-autnomous; meaning it first identifies a person in the room, and if you raise your hand
it will swing over to you. However, there are a suprising number of "Devil-in-the-details" to make
this work:
- The winch mounting locations are highly uncertain -> dinematic uncertainty
- Cables should not every go slack -> dynamics constraints
- Real cable stretch -> unmodeled dynamics

All of these things present challeneges to controlling the robot motion. I did a deep dive
on training a deep neural network to handle these challenges which you can read more about in this
article <a href="/blog/teaching-a-robot-to-swing-with-deep-rl" target="_blank" rel="noopener">
<strong>(Here)</strong></a>.

At a systems level, Comet is wirelessly connected to four custom winch actuators around the room.
Each of these actuators constantly report the estimated spooled cable length, as well as a measurment
of the tension forces on each cable. Comet then reports back desired tension commands to each actuator.