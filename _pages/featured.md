---
title: "Beyond Trial-and-Error: A Scientific Approach to MEMS Sensor Design"
layout: splash
permalink: /featured/
date: 2023-10-06T11:48:41-04:00
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/to/anime.gif
  actions:
    - label: "Read More"
      url: "https://chaozhuang22.github.io/fea/to/"
  caption: "The optimization history of surface stress sensing MEMS sensor."
excerpt: "A Topology Optimization Study on Piezoresistive MEMS Sensors."
intro: 
  - excerpt: 'This site is used to publish notes about topics that I have learned and read.
  It also serves as a personal portfolio showcasing projects that I have beening working on.
  Check out [About](https://chaozhuang22.github.io/about/) page for more information.'
feature_row:
  - image_path: assets/images/to/teaser.jpg
    alt: "Topology Optimization of MEMS Sensor"
    image_caption: "Image courtesy of [Unsplash](https://unsplash.com/)"
    title: "Topology Optimization of MEMS Sensor: A COMSOL Tutorial"
    excerpt: "A guide for setting up a topology optimization model for MEMS sensor design."
    url: "https://chaozhuang22.github.io/fea/to-detailed/"
    btn_label: "Read More"
    btn_class: "btn--primary"
  - image_path: /assets/images/japanese/teaser.jpg
    alt: "From Zero to Fluent: My 1300-Hour Commitment to Learning Japanese"
    image_caption: "Image courtesy of [Unsplash](https://unsplash.com/)"
    title: "From Zero to Fluent: My 1300-Hour Commitment to Learning Japanese"
    excerpt: 'A review on my journey in learning Japanese from Duolingo to active/passive immersion.'
    url: "https://chaozhuang22.github.io/language/japanese/"
    btn_label: "Read More"
    btn_class: "btn--primary"
  - image_path: /assets/images/solidstatephysics.jpg
    alt: "Solid State Physics: Fundamentals and Modern Developments"
    title: "Solid State Physics: Fundamentals and Modern Developments"
    excerpt: 'A summarized study note introducing the fundamentals in solid state physics.'
    url: "https://chaozhuang22.github.io/physics/solid-state-physics/"
    btn_label: "Read More"
    btn_class: "btn--primary"

feature_row1:
  - image_path: assets/images/fea-tutorial/animation.gif
    alt: "Stiffness Fine-tuning in MEMS Sensors"
    title: "Stiffness Fine-tuning in MEMS Sensors"
    excerpt: "Nonlinear behavior in MEMS sensor is modeled using FEA in COMSOL Multiphysics. The stiffness of these sensors can be altered using a highly stressed thin film, pallowing access to zero stiffness and bistability."
    url: "https://chaozhuang22.github.io/fea/nonlinear-mss/"
    btn_label: "Read More"
    btn_class: "btn--primary"

feature_row2:
  - image_path: /assets/images/flow-channel/deformation.gif
    alt: "Microfluidic Fluid-Structure Interaction"
    title: "Microfluidic Fluid-Structure Interaction Modeling"
    excerpt: "The tutorial provides a detailed COMSOL setup for fluid-structure interaction modeling, simulating the deformation of a microfluidic channel under constant flow rate. The model can be extended to study channel deformation by different vapors, offering potential as a viscosity sensor and aiding in experimental flow measurement validation."
    url: "https://chaozhuang22.github.io/fea/flow-channel/"
    btn_label: "Read More"
    btn_class: "btn--primary"

feature_row3:
  - image_path: /assets/images/wrinkle/anime.gif
    title: "Nonlinear Mechanics in Wrinkle Formation"
    excerpt: "The formation mechanics of wrinkles in bilayer systems is explored through FEA using COMSOL Multiphysics. The presented basic model can serve as a base model for advanced simulations, potentially exploring wrinkles on curved surfaces and intricate wrinkle patterns."
    url: "https://chaozhuang22.github.io/fea/wrinkle/"
    btn_label: "Read More"
    btn_class: "btn--primary"
---

{% include feature_row id="intro" type="center" %}

{% include feature_row %}

{% include feature_row id="feature_row1" type="left" %}

{% include feature_row id="feature_row2" type="right" %}

{% include feature_row id="feature_row3" type="left" %}