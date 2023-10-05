---
title: "Splash Page"
layout: splash
permalink: /splash-page/
date: 2023-10-05T11:48:41-04:00
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/to/anime.gif
  actions:
    - label: "Read More"
      url: "https://chaozhuang22.github.io/fea/to/"
  caption: "Beyond Trial-and-Error: A Scientific Approach to MEMS Sensor Design"
excerpt: "A Topology Optimization Study on Piezoresistive MEMS Sensors."
intro: 
  - excerpt: 'This site is used to publish notes about topics that I have learned and read, and it also serves as a personal portfolio showcasing projects that I have beening working on. If you are interested in knowing more, check out the following articles and the About page for more information.'
feature_row:
  - image_path: assets/images/fea-tutorial/anime.gif
    alt: "Stiffness Tuning Using Nonlinearity in MEMS Sensors: A COMSOL Tutorial"
    title: "Stiffness Tuning Using Nonlinearity in MEMS Sensors: A COMSOL Tutorial"
    excerpt: "A numerical demonstration on utilizing nonlinear mechanics to fabricate MEMS sensors with controlled stiffness and outstanding sensitivity."
    url: "https://chaozhuang22.github.io/fea/nonlinear-mss/"
    btn_label: "Read More"
    btn_class: "btn--primary"
  - image_path: /assets/images/flow-channel/anime.gif
    alt: "Microfluidic Fluid-Structure Interaction Modeling: A COMSOL Tutorial"
    title: "Microfluidic Fluid-Structure Interaction Modeling: A COMSOL Tutorial"
    excerpt: "A step-by-step setup to model fluid-structure interactions in microfluidic channel in COMSOL."
    url: "https://chaozhuang22.github.io/fea/flow-channel/"
    btn_label: "Read More"
    btn_class: "btn--primary"
  - image_path: /assets/images/wrinkle/anime.gif
    title: "Beneath the Surface: Understanding Wrinkle Formation"
    excerpt: "A finite element simulation on the non-linear mechanical analysis of wrinkle formation."
    url: "https://chaozhuang22.github.io/fea/wrinkle/"
    btn_label: "Read More"
    btn_class: "btn--primary"
feature_row2:
  - image_path: assets/images/to/teaser.jpg
    alt: "Topology Optimization of MEMS Sensor: A COMSOL Tutorial"
    image_caption: "Image courtesy of [Unsplash](https://unsplash.com/)"
    title: "Topology Optimization of MEMS Sensor: A COMSOL Tutorial"
    excerpt: "A in-depth setup guide for the topology optimization model for MEMS sensor optimization."
    url: "https://chaozhuang22.github.io/fea/to-detailed/"
    btn_label: "Read More"
    btn_class: "btn--primary"
feature_row3:
  - image_path: /assets/images/japanese/teaser.jpg
    alt: "From Zero to Fluent: My 1300-Hour Commitment to Learning Japanese"
    title: "From Zero to Fluent: My 1300-Hour Commitment to Learning Japanese"
    excerpt: 'A review on my journey of learning Japanese.'
    url: "https://chaozhuang22.github.io/language/japanese/"
    btn_label: "Read More"
    btn_class: "btn--primary"
---

{% include feature_row id="intro" type="center" %}

{% include feature_row %}

{% include feature_row id="feature_row2" type="left" %}

{% include feature_row id="feature_row3" type="right" %}