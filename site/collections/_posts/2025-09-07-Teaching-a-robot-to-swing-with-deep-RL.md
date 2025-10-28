---
title:  Teaching A Robot To Swing With Deep RL (Part 1)
date:   2025-09-07 15:01:35 +0300
image:  '/images/comet/comet_char_1.png'
description: Using deep RL to learn a control policy for a cable driven underactuated robot
tags:   [Robotics, Deep Reinforcement Learning]
---

# Introduction

I’ve been developing a cable driven parallel robot I call ‘Comet’. It’s intended to be an interactive, expressive robot that can zoom around my living room autonomously. Four motorized winches at the room’s corners reel cables in and out, letting the robot zip around anywhere in 3D. I think of Comet as a blend between WALL-E and Spider man, and it uses the same principle of actuation which allows the NFL Spider Cam system to move through out the stadium.

This is Part 1 in a 2-part series covering details for training an RL agent to control the robot’s motion.
Press enter or click to view image in full size
Comet — A cable driven parallel robot for the home. 3D printed body, 5-inch LCD screen, and two 2-DOF actuated arms for animation.


# Deep Reinforcement Learning For Control

Control Theory is a sub field of engineering that provides tools to analyze and “control” systems (or robots) so they can perform useful tasks. In this context, that means designing a controller that produces the motor actions to safely swing Comet wherever it needs to go.

Designing a feedback control system that governs how a robot moves through an environment is often incredibly challenging, and is often one of the fundamental bottlenecks preventing many mobile robots from integrating into everyday life.

Over the past decade, deep reinforcement learning (DRL) has made tremendous progress and has been used to control complex robots like humanoids. I’ll cover some basic jargon here, but feel free to skip ahead if you’d like.

When looking at using DRL from the highest level of abstraction, we have two primary decision tree choices to make;

1. Model-based vs. Model-free methods
2. On-policy vs. Off-policy algorithms

## Model-Based vs. Model-Free RL

These terms essentially differentiate between methods which do or do not utilize a dynamics model to predict how the robot will move through the environment.

Model-based Methods rely on a dynamics model of the system. This allows us to simulate the system and predict what will happen as time unfolds. Models can range from simple (e.g., Cart-Pole) to complex (e.g., fluid dynamics). These models are prerequisites for optimal control techniques like Iterative Linear Quadratic Regulator (iLQR) or Model Predictive Control (MPC). The trade-off is between model accuracy and compute time: more accuracy usually means more computation. This mismatch is known as the sim-to-real gap. Still, surprisingly simple models can work incredibly well, i.e. most quadrupeds are modeled as a floating brick with four forces at its corners. (check out the MIT paper)

Model-free Methods make no prior assumptions about the system and rely purely on data collected through interactions with the environment. This can be confusing at first: when simulating a robot in PyBullet or IsaacSim, you are already using a dynamics model. The simulator is the model. The sim-to-real gap here depends on how well the simulator represents friction, contacts, etc. Despite this, the approach is still considered model-free RL, because the network itself never accesses forward dynamics predictions, it only sees states and rewards.

For the Comet robot, I spent a long time trying to derive the full non-linear equations of motion which are fairly similar to a spherical pendulum. This is surprisingly hard to derive and harder yet to validate. It is entirely possible to use a simplified model here; for instance two orthogonal unpinned free pendulums. However, using a model free method avoids this pain entirely. In this context, I’m not worried about the nuances of friction interactions or accurately solving contact events in the environment, so the simulator should actually be more than enough to fully capture the most important dynamics of the robot. I will however use a dynamics model of the motors so accurately capture feasible responses to current inputs.

Before we move on, this might raise an interesting question:

*“If the simulator is a model, why don’t we use that for other techniques like model predictive control?”*

Turns out this has been done fairly recently by some folks at Google. Check out this (github repo or the paper) on utilizing Mujoco (a simulator) for model predictive control a.k.a MJPC. They’ve added helper functions to get the jacobians and hessians needed for the MPC algorithm.

## On-Policy vs. Off-Policy Algorithms

This distinction is essentially: Do we use the agent in it’s current state to collect new data, or re-use previously collected data from some other policy to train the agent.

On-Policy Methods:

The agent interacts with the environment — we measure how good or bad it’s actions were — we estimate a gradient to improve it — repeat.

Now, the actual details of how this is done can get a bit cumbersome, for the curious I’d encourage you to go start a conversation with an LLM of your choice to learn more about all of the algorithms we use to find the best weights and biases for improving the networks performance — but at the end of the day they are all attempting to estimate gradients on the weights and biases of the policy with respect to some reward function. On-Policy algorithms include Policy-Gradient (PG), Reinforce, Trust Region Policy Optimization (TRPO), and Proximal Policy Optimization (PPO).

The major drawback for using on-policy methods is that they require the network to continuously collect new data (samples), from the environment. In many cases this scale is in the millions of samples. Thus, these methods are called sample-inefficient. This is not a deal-breaker if you can collect these samples in simulation (i.e. very quickly), but if you are collecting data in the real world this can be infeasible (and dangerous).

Off-Policy Methods:

Off-policy methods learn from a replay buffer of past transitions generated by any behavior policy (The current actor, older actors, even MPC).

The defining feature is data reuse, which makes them far more sample-efficient than on-policy methods. Architecturally, many use an actor (policy) and one or two critics (Q-functions), plus slowly updated target copies for stability — here “target network” means an exponential-moving-average clone used to compute Bellman targets, not a separate teacher. Because training bootstraps from its own value estimates, off-policy RL can be harder to stabilize (distribution shift, overestimation), but modern tricks like replay, double Q-learning, target policy smoothing, and entropy regularization make it practical. Algorithms in the off-policy category include Deep Q Network (DQN), Twin Delayed Deep Deterministic Policy Gradient (TD3), and more.
Comet Training Strategy

I will use both on-policy and off-policy algorithms (both model-free) through what is know as imitation learning. While it could be possible to train a model from scratch using raw sensor data, it will be much more efficient to first train an expert policy with access to privileged data (i.e. noise-free & latency-free robot position, orientation, velocity etc.), then we can train a student policy to replicate the expert’s behavior when only given access to sensor measurements that the real robot will have access to.

Concretely, we will train our expert policy using PPO allowing it to explore the full environment and learn an ideal control policy (Expert), then we will distill this behavior into a student policy using TD3 (with other concepts such as DAgger and advantage-weighting) in Part 2.

# Learning The Ideal Control Policy (Expert)

Our expert policy should do one thing really well: when I ask for a certain velocity, make Comet move as closely as possible to that command. During training it gets perfect information from the simulator, you can think of “no sensor noise, no delay, everything works exactly as I expect”. I’m not aiming to train a controller that can be directly deployed on the robot just yet; I’m training a fast, safe policy that never lets the cables go slack, and respects velocity limits. The action and observation for the expert policy looks like this:

<div style="text-align: center;">
  <img src="/images/teaching_to_swing/decision_vars.webp" alt="Decision Vars" width="600">
</div>


## Reward Shaping

Reward shaping is the process of designing a function which indicates how ‘good’ your policy’s actions were given a specific simulation rollout. This takes some experimentation to get good at, and I encourage you to go find other RL papers to see how the define reward functions. The most important idea though is that you want your reward signal to be dense, meaning that the signal is present and changing with as much fidelity as possible.

The inverse is known as sparse reward signaling, and this is significantly harder for a network to learn from. Imagine we designed a video game where you only get a reward from finding the key at the end of a hallway, there’s no information to tell the network how ‘good’ the actions taken in between the start of an episode and eventually finding the key are. I’ll touch on reward signalling more as we go over each component next

The reward function for comet is defined below, where W indicates a weight, and gamma indicates a curriculum scaling factor. Weights are typically a static hyper parameter we choose, where as curriculum factors are typically changing over time, a great example of creating game-styled curriculum can be found in this paper. Note that I’m sure there are better objective functions out there for these tasks, but these are straight forward and worked for my needs.

<div style="text-align: center;">
  <img src="/images/teaching_to_swing/rew.webp" alt="Decision Vars" width="700">
</div>

**Velocity Tracking:** We need to reward the network for moving the robot how we want it to move. This blended term rewards velocities close to the target with an exponential on the mean squared error, and penalizes velocities far from the target with a 2-norm. I found that it’s important to keep penalties reasonable, as extremely large negative losses can result in the network learning to terminate the episode as fast as possible to maximize the cumulative return.

<div style="text-align: center;">
  <img src="/images/teaching_to_swing/vel_targ.webp" alt="Decision Vars" width="600">
</div>

<div style="text-align: center;">
  <img src="/images/teaching_to_swing/vel_targ_vis.gif" alt="Decision Vars" width="600">
</div>

**Action Limits:** We want to penalize the network for letting the cables go slack, as it’s harder to predict where they are in space, and it will result in jerky motion when tension is restored. We also want to make sure the network isn’t railing the motor limits, so here we can use a piecewise log function to create a smooth transition at the boundaries of the safe action space, and asymptotic behavior at the platform limits. Note that an action value of -1 means the cables have gone slack, and 1 means we have reached motor saturation. The following parameters allow us to shape the actual reward accordingly.

<div style="text-align: center;">
  <img src="/images/teaching_to_swing/action_lim.webp" alt="Decision Vars" width="600">
  <img src="/images/teaching_to_swing/log_plot.webp" alt="Decision Vars" width="600">
</div>




**Orientation:** Finally, we need to encourage the network to take actions which dampen any oscillatory behavior. Here, we can take the 2-norm of the angle of rotation about each axis.

**Angular Velocity:** We also want to encourage the network to minimize angular velocity of the robot through out the trajectory as well. Since we are aiming for the zero vector, we can repeat the same shape as the orientation objective utilizing the angular velocity components.

## Implementation Details

Training the model on my 16-core MSI Pulse GL66 laptop took roughly 90 minutes to converge to a stable state, requiring ~2.1 million samples and ~15,000 policy updates using PPO. This would be much faster using a GPU ready simulator instead of PyBullet (probably 5–10 minutes).

The robot learns to obey angular velocity limits fairly quickly, and survives for the maximum number of time steps in roughly 100,000 samples. The remaining samples allowed the network to improve target velocity tracking and mitigate robot oscillations.

Here’s a gif of the model performance. The renderings are quite basic, but the gist is the blue parts indicate the ceiling/floor dimensions, and the cables are always oriented towards the corners of the ceiling. The red lines indicate the amount of force on each cable, while the green vector indicates the target linear velocity, and the purple indicate the actual linear velocity. You can see the policy attempt to correct the initialized velocity and eventually matches the target velocity quite well.

<div style="text-align: center;">
  <img src="/images/comet/simulator_expert.gif" alt="Expert Simulation" width="600">
</div>

Simulation: I started in PyBullet. It’s approachable, but contact events are poorly handled, and CPU-only scaling slows down on-policy training. For higher throughput, consider MuJoCo or IsaacSim (GPU-parallel RL workflows).

Robot Model: I designed the robot in OnShape and converted to URDF with Onshape-To-Robot which works nicely for Bullet, but MuJoCo requires a different representation format all together (MJCF). It’s also important to note that the cables connect to a universal joint on the top of comet, which means that the robot is actually able to hang vertically even at the limits of the workspace, where as connecting them directly to the robot would result in non zero orientation equilibrium at the workspace limits.

### Gymnasium Environment:

This is essentially a standardized API defining:

- Actions: per-winch tension force inputs. max = 50 N
- Observations: Data from the environment.
- Rewards Function: Encourage velocity tracking, penalize instability.
- Truncation: Max steps, exceeding environment bounds.
- Resets/terminations: Cable slack, hard contacts, velocity limits.
- Stepping forward the simulator.

### Expert Policy Hyper Parameters:

— Simulation —

- Time step size: 0.005s
- Action Repeat: 4
- Max env steps: 150 (3.0s sim time per environment)

— Neural Network —

- MLP actor architecture: [256 x 256]
- Action noise log std init: -2.5
- Learning rate: 3e-4
- PPO discount factor: 0.995
- PPO generalized advantage estimator (GAE): 0.95
- PPO entropy coefficient: 0.001

— Training —

- N epochs: 5
- Mini batch size: 64
- N training iterations: 15
- N training environments: 14
- N steps per environment: 128
- N steps per iteration: 179,200
- N evaluation environments: 14
- N evaluation episodes (per evaluation period): 14
- Evaluation frequency: 1024

— Weights —

- Velocity objective: 10.0
- Action objective: 1.0
- Orientation objective: 1.0
- Angular velocity objective: 1.0

— Curriculum Factors—

- Velocity objective: [0.1, 0.5, 1.0, … , 1.0]
- Action objective: [1.0, … , 1.0]
- Orientation objective: [1.0, … , 1.0]
- Angular velocity objective: [1.0, … , 1.0]