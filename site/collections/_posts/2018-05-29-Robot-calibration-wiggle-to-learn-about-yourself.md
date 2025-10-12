---
title:  "Robot Calibration: Wiggle to Learn About Yourself"
date:   2025-09-15 18:05:55 +0300
image:  '/images/robustness/robustness.png'
tags:   [SysID, Model Calibration]
---
# Introduction

For most robots, having a decent dynamics model is a prerequisite for many critical tasks needed for the robot to operate properly. This includes allowing us to plan and execute motion trajectories, build a map of the environment and figure out where we are in that map (SLAM).

There are multiple ways of finding a model that represents your system well, and this field of study is known as System Identification (SysID). However, in many cases there are degrees of uncertainty in the actual parameters we choose within the model itself, i.e. for a legged robot; we may estimate the mass of a linkage or part, the true dimensions of the kinematic assembly, joint friction, sensor location, etc. We generally have two options for how to handle this uncertainty:

**(Option 1) Robust Control:** We could (and should) account for this uncertainty in our control design for the robot. In modern control, one example technique is called H-infinity controller synthesis. In deep learning based control we typically try to force the network to adapt to this uncertainty by randomizing parameters in our simulation, a.k.a domain randomization.

**(Option 2) Calibration:** Instead of just attempting to blanket cover this uncertainty with a robust controller, we can attempt to refine some of these parameters on the hardware, i.e. run a calibration sequence to reduce uncertainty.

However, If we can just design a controller that can be robust to this uncertainty, what’s the point in spending the time developing a calibration scheme?

## Robustness-Speed Tradeoff

There’s an interesting concept called the robustness-speed tradeoff; which says that gaining robustness gives up controller response speed, and gaining response speed gives up robustness. So we could design a controller that rails the system to its absolute limits for speed, but any epsilon difference between our model and the real world system can cause catastrophic instability, i.e. shake the robot to pieces or cause it to face plant.
Press enter or click to view image in full size

Essentially, by calibrating the system and improving our parameter estimates, we can reclaim some of that lost speed while being robust to any uncertainty that remains.

This article will introduce a neat algorithm which can be used for robot calibration; Input-Output Error Minimization.

*Making our robot wiggle around a bit can help us get better estimates of its model parameters, allowing us to push the hardware further.*


# Input-Output Error Minimization (IOEM)

The core idea behind this algorithm is using observable response data from some motion in the system to find the set of system parameters which minimize the error between the given inputs (control effort) and measured outputs (states). On a robot, this works as follows:

1. Send the same series of inputs (joint torques) to the real robot and the simulated robot model.
2. Measure the observable outputs of the real system, i.e. encoder velocities, IMU data, etc., and record the same outputs from our simulated robot model.
3. Solve a constrained optimization problem minimizing the error between simulation outputs and the real outputs, using your model parameters as decision variables.

# Case Study

Recently, I’ve been building a cable-driven parallel robot, which is driven around it’s environment by four winches letting cables connected to the robot in and out. The winches are installed in my living room, not a lab, so I have fairly poor estimates of exactly where these are in space. Additionally, I don’t really know where the robot is in space when I turn it on, as the encoders don’t store rotation that happens when the system is powered off.

![Expert Simulation](/images/comet/simulator_expert.gif){: width="1200" height="900"}
Cable drive parallel robot simulation. Front (left), Top (right).

This uncertainty in winch mounting location and robot pose essentially results in my controller applying forces to the robot in directions that aren’t quite what it expects, leading to sloppy motion. While we could use vision techniques such as April Tags, or PnP correspondence, Input-Output error minimization can help us out here without the need of relying on camera sensors.

Importantly, this robot in particular is not designed to make body contact with the environment, which means I don’t need to worry about having multiple models for different contact modes (a.k.a Hybrid dynamics).

## 2D Simplification

Let’s reduce the system down to 2D for simplicity in explaining this technique. The animation below shows the free pendulum system response from a step input on the winches, which wiggles around over time. In this context, we will aim to solve the coordinates for the robot’s initial position, and the location of the 2nd winch (black square on the right). We will also make the assumption that the cable’s do not stretch at all.

![Pendulum Step Response](/images/robustness/pendulum_anim.gif){: width="1200" height="900"}
2D Step response of the unfixed pendulum

## Setup

Given that we can set our origin arbitrarily at the first winch location, we are then trying to solve the following, relative to the first winch. Thus our decision variables in this are:

<div style="text-align: center;">
  <img src="/images/robustness/decision_vars.webp" alt="Decision Vars" width="400" mb=10>
</div>

<!-- ![Pendulum Step Response](/images/robustness/decision_vars.webp){: width="400" height="200"} -->

Notice that the bounds on this indicate that the robot initial position must be between the two winches in the x axis, and below the origin. The bounds on the winch location variables are simply chosen from my uncertainty w.r.t to the nominal winch mounting location at (2, 0).

### Defining the Residual Function
The residual for this algorithm is just a least squares on the system responses Y:

<div style="text-align: center; mb-10">
  <img src="/images/robustness/residual.webp" alt="Decision Vars" width="400" mb=10>
</div>


## Root Finding Trick for Safe Data Collection

We need to collect our measurements on the outputs of our real system, but it’s important to note that the reaction of my system to force inputs can be wildly different depending on where the robot is in space. Well, I’m going to use a trick here to make sure I’m at least in the ballpark before I get ready to send any commands to the real robot for the initial data collection sequence.

When my system is powered on, the motors find a torque where a non-zero tension force is sensed on cable. Then the motors attempt to hold their current position, meaning the robot can now hang freely with tension in all cables regardless of where it is in the room. Once it comes to rest, I can record these tension forces, and use them to solve a rough estimate of the initial position of the robot.

In plain terms, I’m solving a root-finding problem: finding a robot position where the cable tensions applied do not cause motion. Of course, with uncertain winch locations, I can’t find the perfect solution, but I can get close enough to use as a starting point. This has the form:

<div style="text-align: center;">
  <img src="/images/robustness/x_.webp" alt="Decision Vars" width="400" mb=10>
</div>

To solve this problem, we can minimize the quadratic cost:

<div style="text-align: center;">
  <img src="/images/robustness/J_.webp" alt="Decision Vars" width="400" mb=10>
</div>

subject to the equality constraint:

<div style="text-align: center;">
  <img src="/images/robustness/c_.webp" alt="Decision Vars" width="400" mb=10>
</div>

Additionally, we have inequality constraints to impose on the root finding decision variables. To account for these constraints, we can turn this constrained optimization problem into an unconstrained problem by forming a Karush-Kuhn-Tucker system (KKT), and solve the unconstrained KKT problem with the Newton-Raphson method.

## Real System Data Collection

Once we have a solution for the roots of this system, I can safely add a small positive perturbation to these tension forces which will make the robot move slightly while remaining safe, i.e. within our workspace bounds and observe very small body velocities. We can then send these commands over a short horizon (1–2s) and record the response of the system. On the real system, it turns out I can get away with just measuring the change in cable length from the winch encoders to get a high quality estimate of the pivot position.

For the toy 2D example, I’m adding Gaussian noise to my response measurements proportional to the amount of noise I expect on the real sensors.

## Solving the Model Parameters

Now that we have our real data, we can begin the core input-output minimization routine to refine out estimates of the uncertain variables. I used scipy.optimize.least_squares() selecting the trust region-reflective solver method to minimize the residual we defined earlier.

It turns out that in this example, this routine converges to near machine precision at 1e-10 on our residual function. The solution solves for the robot initial position, and winch location to within +/- 0.5 mm of the true solution even with measurement noise!

While the +/- 20 cm uncertainty we initially had on these parameters would lead to unusable feedback control and state estimation, sub-millimeter precision on these parameters is more than enough accuracy to meet my needs. Other issues in the system such as cable stretch, IMU noise, etc. will introduce error in a more impactful way than this remaining parameter uncertainty ever will.

# Final Thoughts

There’s many other techniques out there which can help you push hardware further depending on the task, i.e. iterative learning control. But for a general purpose calibration scheme which is simple to implement, this algorithm really shines. The hardest part of implementing something like this on a real world system will inevitably come down to your ability to observe the necessary responses on the real system, and ensuring your system remains in one contact mode for the full collection routine.

To see a full paper on some of the algorithm details, see here. To read about real world implementation on identifying parameters of an aerodynamic model see here.

System Identification is an incredible field for exploring, and I highly encourage you to look into it further even if it’s outside your wheel house. I constantly find that many of these techniques bring together a plethora of foundational skills in extremely elegant ways. Additionally, gaining back some controller speed can allow you to take more aggressive actions in the real world and push your hardware further, i.e. more back flips and barrel rolls.