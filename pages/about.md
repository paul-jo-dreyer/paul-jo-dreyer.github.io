---
layout: page
title: About
permalink: /about
---

<div style="text-align: center;">
  <img src="{{ '/assets/img/me_round.png' | relative_url }}" 
       alt="photo not loaded"
       style="width: 50%; max-width: 150px; height: auto;">
</div>

## Background

I'm in my final semester of a 5th year masters in Mechanical Engineering at Carnegie Mellon University in Pittsburgh, PA. I primarily focus in robot systems design and motion control systems (GNC), including optimal control theory and deep RL based control. 

## Education

M.S. Mechanical Engineering - Carnegie Mellon (Expected graduation May 2025)
B.S. Mechanical Engineering - Carnegie Mellon 

## Experiences

<div style="display: flex; align-items: flex-start; padding: 16px; width: 100%;">

  <!-- Image Container -->
  <div style="flex: 0 0 auto; margin-right: 20px;">
    <img src="{{ '/assets/img/dirac_logo.png' | relative_url }}" 
         alt="photo not loaded"
         style="width: 100px; height: auto;">
  </div>

  <!-- Text Container -->
  <div style="flex: 1;">
    <h2 style="margin-top: 0; padding: 16px;">Dirac Inc. - Backend Engineer (2024)</h2>
    <p>
      Dirac is a startup in NYC/LA that is creating a product to automate work instructions for manufacturing purposes.
      During my internship here, I developed a trajectory optimization algorithm that would create minimal collision 
      paths for general 3D objects to travel during the assembly of a system. I also developed an custom signed distance
      field (SDF) solver that would generate data structs for a given 3D object at a desired resolution.
    </p>
    <p><b>Language:</b> Python</p>
  </div>

</div>

<div style="display: flex; align-items: flex-start; padding: 16px; width: 100%;">

  <!-- Image Container -->
  <div style="flex: 0 0 auto; margin-right: 20px;">
    <img src="{{ '/assets/img/Ford_logo.png' | relative_url }}" 
         alt="photo not loaded"
         style="width: 100px; height: auto;">
  </div>

  <!-- Text Container -->
  <div style="flex: 1;">
    <h2 style="margin-top: 0;">Ford Motors - Computer Vision Research Engineer (2023)</h2>
    <p>
      During the manufacturing process of a vehicle, it's critical to inspect the installation of 
      components at various points along the line in order to catch errors early and minimize the 
      lost costs of correcting assembly errors. Ford is exploring the use of deep neural nets to 
      capture images of their vehicles, detect sub assemblies, and classify if the installation has
      been successful. A huge bottle neck in this is the need to capture and label hundreds of 
      images in order to train a model to accomplish this detection/classification task. During my 
      internship at Ford, I developed a python tool which enables users to load in a CAD model of 
      the target assembly, and generate a fully labeled synthetic image data set for training a model to 
      complete this task, eliminating the need for a human to complete the arduous capture / label 
      process. I found that through sufficient domain randomization, it was possible to train 
      a model on 100% synnthetic images to achieve >90% accuracy when deployed in the real world.
      Critically, this now allows engineering teams at ford to have a model pre-trained, ready to 
      deploy as soon as a line change occurs, as opposed to deploying the assembly line change followed
      by image collections, training, and testing tasks.
    </p>
    <p><b>Language:</b> Python</p>
  </div>

</div>


<div style="display: flex; align-items: flex-start; padding: 16px; width: 100%;">

  <!-- Image Container -->
  <div style="flex: 0 0 auto; margin-right: 20px;">
    <img src="{{ '/assets/img/biorobotics.png' | relative_url }}" 
         alt="photo not loaded"
         style="width: 100px; height: auto;">
  </div>

  <!-- Text Container -->
  <div style="flex: 1;">
    <h2 style="margin-top: 0;">CMU Biorobotics Lab - Research Assistant (2021-2022)</h2>
    <p>
    During my time with the Biorobotics lab, I worked on a research team developing a robot to 
    achieve pipe-in-pipe trenchless repair for the decaying natural gas piping infrastructure around
    the US. I designed and tested a radial nozzel that would extrude an exotic resin in a pipe shape
    creating a smaller pipe within the old infrastructure. This nozzle was design to adapt its 
    exterior geometry to acomodate stress induced ellipsoidal fallicies in the host pipe geometry.
    </p>
  </div>

</div>

<div style="display: flex; align-items: flex-start; padding: 16px; width: 100%;">

  <!-- Image Container -->
  <div style="flex: 0 0 auto; margin-right: 20px;">
    <img src="{{ '/assets/img/MAC_lab_logo.png' | relative_url }}" 
         alt="photo not loaded"
         style="width: 100px; height: auto;">
  </div>

  <!-- Text Container -->
  <div style="flex: 1;">
    <h2 style="margin-top: 0;">CU Boulder Matter Assembly Lab - Research Assistant (2020)</h2>
    <p>
      During my time with the Matter Assembly Lab, I developed a novel print-in-place pneumatic 
      flex sensor for a soft robotic finger. This allowed for pose estimation of the actuator 
      without the introduction of rigid sensing components in the actuator itself, preserving the 
      desireable features of a soft actuated finger
    </p>
  </div>

</div>


<div style="display: flex; align-items: flex-start; padding: 16px; width: 100%;">

  <!-- Image Container -->
  <div style="flex: 0 0 auto; margin-right: 20px;">
    <img src="{{ '/assets/img/Air_Force_Logo.png' | relative_url }}" 
         alt="photo not loaded"
         style="width: 100px; height: auto;">
  </div>

  <!-- Text Container -->
  <div style="flex: 1;">
    <h2 style="margin-top: 0;">United States Air Force - Communications (2014-2020)</h2>
    <p>
    In the Air force I worked as a communications technician. My responsiibilities included 
    operating and maintaining SATCOM systems, as well as deploying tactical HF communication systems
    in various regions and terrains around the world, from mountains to forrest / jungle terrain. I 
    completed a 6 month deployment to southern Turkey in support of mission Enduring Freedom. I
    honorably seperated from the Air Force at the completion of my 6-year enlistment as a Staff 
    Sergeant (E-5).
    </p>
  </div>

</div>
