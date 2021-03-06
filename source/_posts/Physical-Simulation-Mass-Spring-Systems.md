---
title: 'Physical Simulation: Mass-Spring Systems'
date: 2020-10-05 19:58:29
tags: 
- Physical Simulation
mathjax: true
---

This a learning note from the [class](https://forum.taichi.graphics/t/topic/272) given by Yuanming Hu about **Physical Simulation**



## Introduction

**Two Views of Continuums**

There are two basic methods for solving fluid equations

- Lagrangian View: "What are my position and velocity?" 

  \- Method based on particles

  ![lagrange view](/images/lagrange.gif)

- Eulerian View: "What is the material velocity passing by?" 

  \- Method based on grid
  
  ![eulerian view](/images/eulerian.gif)



## Mass-Spring Systems

Mass Spring Systems are a physical simulation based on Lagrangian view, an object are seen as  . Mass-spring systems are good for  

**Hooke's Law**
$$
\textbf{f}_{ij} = -k(||\textbf{x}_i - \textbf{x}_j||_2-l_{ij})(\widehat{\textbf{x}_i-\textbf{x}_j})
\\
\ 
\\
\textbf{f}_i = \sum_{j}^{j \ne i}{\textbf{f}_{ij}}
$$
**Newton's second law of motion**
$$
\frac{\partial \textbf{v}_i}{\partial t} = \frac{1}{m}\textbf{f}_i
\\
\ 
\\
\frac{\partial \textbf{x}_i}{\partial t} = \textbf{v}_i
$$



## Time integration

**Time integration - Explicit**

(1) Forward Euler (explicit)
$$
\textbf{v}_{t+1} = \textbf{v}_t + \Delta t \frac{\textbf{f}_t}{m}
\\
\textbf{x}_{t+1} = \textbf{x}_t + \Delta t \textbf{v}_t
$$
(2) Semi implicit Euler (aka. symplectic Euler, explicit)
$$
\textbf{v}_{t+1} = \textbf{v}_t + \Delta t \frac{\textbf{f}_t}{m}
\\
\textbf{x}_{t+1} = \textbf{x}_t + \Delta t \textbf{v}_{t+1}
$$


| Pros                        | Cons                    |
| --------------------------- | ----------------------- |
| Future depends only on past | Easy to explode         |
| Easy to implement           | Bad for stiff materials |

Explosion limit:
$$
\Delta t \le c \sqrt{\frac{m}{k}} (c \sim 1)
$$



**Time integration - Implicit**

Backward Euler (often with Newton's method, implicit)


$$
\textbf{x}_{t+1} = \textbf{x}_t + \Delta t \textbf{v}_{t+1}
\\
\textbf{v}_{t+1} = \textbf{v}_t + \Delta t \textbf{M}^{-1}\textbf{f}(\textbf{x}_{t+1})
$$

Estimate $\textbf{x}_{t+1}$:
$$
\textbf{v}_{t+1} = \textbf{v}_t + \Delta t \textbf{M}^{-1}\textbf{f}(\textbf{x}_{t} + \Delta t \textbf{v}_{t+1})
$$
Linearize (one step of Newton's method):
$$
\textbf{v}_{t+1} = \textbf{v}_t + \Delta t \textbf{M}^{-1}	\left[\textbf{f}(\textbf{x}_{t}) + \frac{\partial \textbf{f}}{\partial \textbf{x}} (\textbf{x}_{t}) \Delta t \textbf{v}_{t+1}	\right]
$$
Clean up:
$$
\left[\textbf{I} - \Delta t^2 \textbf{M}^{-1} \frac{\partial \textbf{f}}{\partial \textbf{x}} (\textbf{x}_{t}) 	\right] \textbf{v}_{t+1} = \textbf{v}_t + \Delta t \textbf{M}^{-1}\textbf{f}(\textbf{x}_{t})
$$
How to solve it

- Jacobi/Gauss-Seidel iterations (easy to implement)
- Conjugate gradients 



**Unify explicit and implicit integrators**
$$
\left[\textbf{I} - \beta \Delta t^2 \textbf{M}^{-1} \frac{\partial \textbf{f}}{\partial \textbf{x}} (\textbf{x}_{t})   \right] \textbf{v}_{t+1} = \textbf{v}_t + \Delta t \textbf{M}^{-1}\textbf{f}(\textbf{x}_{t})
$$

- $\beta = 0$: forward/semi-implicit Euler (explicit)
- $\beta = 1/2$: middle-point (implicit)
- $\beta = 1$: backward Euler (implicit)



**Solve faster**

- Sparse matrices
- Conjugate gradients
- Preconditioning



## Code

Based on Mass-Spring System, I wrote a simple rope simulator using OpenGL. I choose semi implicit time integration as it's the most intuitive method without revolving other math libs dependency. 

Source repo is [here](https://github.com/suncio/OpenGL-Learning)

![Rope Simulator](/images/RopeSimulator.gif)



