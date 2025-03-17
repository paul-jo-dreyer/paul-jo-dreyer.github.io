---
layout: post
title: "Outlet Testing Robot"
author: "Paul Dreyer"
categories: documentation
tags: [documentation]
image: monks_odb.png
---

## Background

This is a robot which autonomously explores a room / corridor and tests each outlet for functionality. The use case here is
demonstrating an ability for a robot to self-dock for recharging in an arbitrary room without the need for a custom charging base.

## Contributions

I was the mechanical / electrical lead for this project. 

On the mechanical side, I designed and fabricated the following:
1. Translational mechanum wheel systems
2. Vertical gantry system
3. Tool head system

On the electrical side, I designed:
1. Power circuits
2. Motor drive circuits
3. Sensor IO

I also developed our main PCB which allows us to get rid of all jumper wires in the main body, as well as create a compact way to connect
our compute, sensors, and motors into a single board.

<div style="text-align: center;">
  <img src="{{ 'assets/img/monks_odb.png' | relative_url }}" 
       alt="photo not loaded"
       style="width: 50%; max-width: 500px; height: auto;">
</div>



###


## Design considerations

Comet is born from a dream to have a cost-feasible autonomous robot in the home. To achieve this dream, every decision is built on the basis of:

1. Accesible manufacturing techniques for most DIY makers
2. Widely sourcable component selection

## Project roadmap

## Web-app

## Team members
