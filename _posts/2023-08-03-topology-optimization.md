---
title: "A Brief Introduction to Topology Optimization"
date: 2023-08-03T16:30:00+09:00
category: FEA
tag: Tutorials
---
> Keywords: Applied mathematics.
> A short introduction to topology optimization.

## Overview
Structural optimization, in its simplest form, is achieved through size and shape optimization. Here, design parameters such as feature sizes or boundary shapes are parameterized to formulate the optimization problem. However, these methods have a significant limitation: they do not allow changes in the design topology. To address this issue, topology optimization (TO), is also known as the [homogenization approach](http://www.sciencedirect.com/science/article/pii/0045782588900862). Topology optimization can be formulated as gradient-based methods such as the density model (famous [top99 code](https://doi.org/10.1007/s001580050176) and [the optimized 88 lines code](https://doi.org/10.1007/s00158-010-0594-7)), [level set method](https://link.springer.com/article/10.1007/s00158-009-0430-0), phase field, topology derivative, and others for a review [here](https://doi.org/10.1007/s00158-013-0978-6).

In this post, we will focus on density-based topology optimization, which emerged as a general shape optimization approach in the 1980s and quickly developed into a popular optimization tool applicable to any physical system that can be described by partial differential equations (PDEs). This includes structural mechanics on the design of [metamaterials](https://onlinelibrary.wiley.com/doi/abs/10.1002/adma.201502485), [thermal and mass transport](https://www.aeromech.usyd.edu.au/WCSMO2015/papers/1264_paper.pdf), [fluid flow](https://www.sciencedirect.com/science/article/pii/S0017931017310037), and [wave propagation](https://opg.optica.org/oe/abstract.cfm?uri=oe-22-7-8525). Prof. Sigmund has given excellent seminars on [thermo-fluidic problems](https://www.youtube.com/watch?v=HH9RBQVzSZg) and [general applications of TO](https://www.youtube.com/watch?v=JVRs80HDvvM).

## Optimization workflow
The optimization workflow of TO consists of the following steps:
1.  Propose an initial design, which is often a design where materials are uniformly distributed in the design domain.
2.  Evaluate the objective function of the design using finite element analysis (FEA).
3.  Evaluate the sensitivity of objectives and constraints with respect to the density variation of each element using FEA and associated adjoint solutions.
4.  Update the densities of all elements based on the sensitivity analysis using the Method of Moving Asymptotes (MMA).
Repeat steps 1 to 4 until convergence criteria are met or a maximum number of iterations is reached.

### Formulation
Topology optimization is typically formulated in a predetermined design domain, $$\Omega$$, with a given set of loading and boundary conditions. The design domain is discretized into finite elements, where the material distribution is described by an element-wise constant density vector $$\rho$$. Within this domain, the algorithm finds the material distribution that optimizes a given objective function $$f$$. Density at each element can take values between 1 and 0, representing the presence and absence of material, respectively. Intermediate density values are permitted, which makes the optimization variable continuous, allowing gradient calculation to be done. The material properties are weighted by the element density and raised to the power of $$p$$, a penalization factor based on the [Simple Isotropic Material with Penalization (SIMP) method](https://doi.org/10.1007/BF01742754). This approach yields discrete designs with only 0 and 1 densities. The value of $$p$$ is typically 3, depending on the Poisson's ratio of the material.

### Filers and projections
Density filtering based on a [Helmholtz filter](https://onlinelibrary.wiley.com/doi/abs/10.1002/nme.3072) is used to prevent the well-known numerical instability of [checkerboarding](https://doi.org/10.1007/BF01743693) and mesh dependence. An overview of numerical instabilities can be found [here](https://doi.org/10.1007/BF01214002). Checkerboarding refers to optimized designs with alternating solid and void elements. Initially, it was believed to represent an optimal microstructure, but later studies revealed that checkerboarding is a numerical instability caused by stiffness misrepresentation due to low-order finite elements. To eliminate this issue, density filters or sensitivity filters have been employed to average across neighboring elements and avoid checkerboard patterns.

Density filtering is implemented by calculating the elemental density as a weighted average of all neighboring elements within a radius of $$r_{min}$$ at each iteration:

$$\tilde{\rho}_j-r_{min}^2(\frac{\partial^2\tilde{\rho}_j}{\partial x^2}+\frac{\partial^2\tilde{\rho}_j}{\partial y^2})=\rho_j$$

where $$\rho_i$$ and $$\tilde{\rho}_i$$ are the original and filtered density of element $$i$$, respectively.

Since the averaging of the density filter introduces a transition zone between solid and void regions that is not physically meaningful, a [Heaviside projection](https://onlinelibrary.wiley.com/doi/abs/10.1002/nme.1064) function is further implemented to eliminate intermediate densities. The projected density is given by:

$$\bar{\tilde{\rho}}_i=\frac{tanh(\beta \eta)+tanh(\beta(\tilde{\rho}_i-\eta))}{tanh(\beta \eta)+tanh(\beta(1-\eta))}$$

where $$\bar{\tilde{\rho}}_i$$, $$\beta$$, and $$\eta$$ represent the projected density, the projection slope, and the projection threshold that determines at which density the projection function applies.

### Robust formulation
To ensure the robustness of the design against one-node connections and fabrication errors such as under-etch and over-etch, the objective function is evaluated three times for eroded, intermediate, and diluted designs given by $$\eta=0.3,0.5,0.7$$ during each iteration. Subsequently, the MMA solver will automatically optimize the performance of the worst design among the three. The robust formulation imposes the minimum length scale specified by filter radius $$r_{min}$$ such that one-node connections that are common in stress/displacement maximization problems are eliminated. Note also that one-node connection problems can also be solved by introducing [von Mises stress constraints](https://link.springer.com/article/10.1007/s00158-015-1279-z).

### Continuation for ensuring global optimum
Local minima refer to the phenomenon where the optimized design gets trapped at suboptimality due to its dependence on various numerical parameters, such as the number of elements, filter parameters, move limits, and the geometry of design domains. This problem can be addressed, empirically, by [parameter continuation](https://doi.org/10.1007/978-94-011-1804-0_14). Continuation is commonly done by slowly ramping up the penalization $$p$$ and the projection slope $$\beta$$ For example, $$\beta$$ can start with 1 and double in every tens of iterations. A final value of 16 is typically sufficient to ensure a solid-void solution.

## Conclusion
Topology optimization is a powerful tool for designing structures with optimal performance. The use of density filtering and projection methods ensures the generation of clear, manufacturable designs. The implementation of robust formulation ensures the manufacturability of optimized designs. The continuation ensures the well-posedness of the optimization problem. With these building blocks, topology optimization can be applied to a wide range of physical design problems that require optimization in geometries.
