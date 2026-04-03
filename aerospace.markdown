---
layout: page
title: Aerospace
permalink: /aerospace/
image: /assets/images/Odysseus.webp
description: "Spacecraft guidance, navigation, and control — from lunar landing mission design to Kalman filters and trajectory optimization. I bootstrapped the mission design for Odysseus (IM-1), the first privately-owned lunar lander on the Moon."
---

In February 2024, Odysseus became the first privately-owned lunar lander to soft-land on the Moon. I bootstrapped its original mission design as the sole mission designer at Intuitive Machines during the proposal phase — reference trajectories, ground station selection, attitude timelines, sensor specs, and landing hazard analyses. I'm a subject matter expert on the [Navigation Doppler LIDAR](https://www.nasa.gov/directorates/stmd/impact-story-navigation-doppler-lidar/), the sensor Odysseus relied upon for its landing after its laser altimeters failed.

<img alt="Image of Earth from Odysseus, on its way to the Moon" src="/assets/images/Odysseus.webp" style="width: 50%" align="right" />

My core expertise is spacecraft navigation — state estimation under uncertainty using noisy sensor measurements. I've worked on lunar landers (Intuitive Machines NOVA-C, Moon Express MX-1, Masten XL-1), autonomous rendezvous with non-cooperative objects using LIDAR, optical navigation, space situational awareness, and orbit determination. My tools are extended Kalman filters and batch least squares estimators. I consider myself a mathematician first: the filter, not the flight software, is the interesting part.

The probabilistic reasoning at the heart of state estimation connects directly to my [AI agent research](/agentic/) — both are problems of inference under uncertainty. The mathematical frameworks transfer.

<img alt="Image of a water-damaged notebook containing equations related to orbits" src="/assets/images/wet_notebook.jpg" style="width: 50%" align="left" />

I have also worked in climate tech (automating industrial systems at Charm Industrial) and space policy (export controls and collective invention at Open Lunar Foundation). I believe firmly that we cannot solve our environmental problems without moving heavy industry into space, which requires a sustained presence on the Moon.

While I code very well in C++ and Python, I consider myself a mathematician first. If you want to see an example of numerical software of which I'm proud, I recommend looking at [Pyquat](https://github.com/translunar/pyquat), a Python 3 package for unit quaternions. In graduate school, I wrote the now-deprecated [NMatrix](https://github.com/SciRuby/nmatrix) for the [SciRuby Project](https://sciruby.com), which was at the time the Ruby equivalent of Numpy.

_I do not work on any weapons systems._

## Spacecraft GN&C Capabilities

### Navigation & State Estimation
Design and derive extended Kalman filters and batch least squares estimators with sensor measurement models. Conduct observability and linear covariance analyses to specify navigation requirements. Optical navigation and LIDAR-based relative navigation for non-cooperative objects. Select ground station locations for optimal orbit determination.

### Guidance & Trajectory
Design and optimize trajectories. Provide delta-v estimates and error budgets. Understand and implement guidance algorithms for planetary landing, including terrain-relative navigation.

### Control & Attitude
Size reaction control systems and analyze controllability. Design eigenaxis controllers and PID controllers with plant models. Determine sensor locations, mount angles, and mission attitude timelines. Design state machines for automation sequences.

### Sensors & Simulation
Spec out inertial measurement units. Derive and implement sensor models and simulators. Relative navigation using LIDAR and cameras. Design hardware-in-the-loop simulations.
