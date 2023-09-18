---
title: "Microfluidic channel modelling"
date: 2023-09-09T11:30:00+09:00
category: FEA
tag: Projects
header:
  teaser: /assets/images/channel/teaser.jpg
---

## Introduction
The applications of microfluidic channels.
So it is important to be able to model the channel deformation and exert control over the device behavior in these various applications.

## Main Challenges
Since we are modelling a soft and shallow channel under fluid flow, the fluid pressure will significantly alter the channel geometry, which in turns alter the fluid behavior. The highly coupled problem requires constant update of mesh to accomodate the constantly changing geometry and account for the interaction between fluid and structural deformation. A theoretical account can be found in this article (https://pubs.rsc.org/en/content/articlelanding/2006/lc/b513524a), based on which I contribute to the simulation work done in this article (https://onlinelibrary.wiley.com/doi/abs/10.1002/advs.202204310).

Here, I will present the implementation of such a problem in COMSOL Multiphysics.

## Model Setup
### Geometry and Boundary Conditions
The model consists of a rigid substrate and a deformable polymeric cover with a shallow channel. In the inlet, a constant flow rate is imposed.

The estimated Reynolds number is xx, which is well below the turbulence threshold, so laminar flow is used to model the fluid flow.

### Moving Mesh
A moving mesh is set based on a COMSOL tutorial ()

### Post-Processing and Analysis
An animation.

The channel profile.

The pressure profile.

## Conclusion