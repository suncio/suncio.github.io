---
title: 'Real Time Collision Detection: Intro'
date: 2020-10-16 23:00:12
tags:
- Collision Detection
mathjax: true
---

A learning note while reading *Real-Time Collision Detection* by Christer Ericson

This is a personal blog so I'll skip some of the common knowledge like general 3D math or linear algebra. This intro will list some of the key concepts used in **collision detection** that I haven't heard of or find important enough to re-memorize again.

## Polygons 

​	A *polygon* is a closed figure with *n* sides, defined by an ordered set of three or more points in the plane in such a way that each point is connected to the next (and the last to the first) with a line segment. 

- **Simple**: A polygon is *simple* if no two nonconsecutive edges have a point in common. A simple polygon partitions the plane into two disjoint parts: the *interior* (the bounded area covered by the polygon) and the *exterior* (the unbounded area outside the polygon).

- **Convex/Concave vertex**: A vertex is a convex vertex if the interior angle is less than or equal to 180 degrees. If the angle is larger than 180 degrees, it is instead called a concave (or reflex) vertex.

  ![Convex vertex and Concave vertex](/images/ConvexConcave.png)

- **Convex/Concave polygon**: A polygon *P* is a convex polygon if all line segments between any two points of *P* lie fully inside *P*. A polygon that is not convex is called a concave polygon. A polygon with one or more concave vertices is necessarily concave, but a polygon with only convex vertices is not always convex.

- **Convex hull**: A convex point set $S$ is a set of points wherein the line segment between any two points in $S$ is also in $S$. Given a point set $S$, the convex hull of $S$, denoted $CH(S)$, is the smallest convex point set fully containing $S$. $CH(S)$ can also be described as the intersection of all convex point sets containing $S$. 

  ![Convex Hull](/images/ConvexHull.png)

- **Affine hull**: Related to the convex hull is the affine hull, $AH(S)$. The affine hull is the lowest dimensional hyperplane that contains all points of $S$. That is, if $S$ contains just one point, $AH(S)$ is the point; if $S$ contains two points, $AH(S)$ is the line through them; if $S$ contains three noncollinear points, $AH(S)$ is the plane determined by them; and if $S$ contains four (or more) non co-planar points, $AH(S)$ is all of $\mathbb{R}^3$.

### Test Polygon Convexity



## Polyhedra



## Voronoi Diagram

The Voronoi diagram is named after Georgy Voronoy, and is also called a **Voronoi tessellation**, a **Voronoi decomposition**, a **Voronoi partition**, or a **Dirichlet tessellation**. Voronoi cells are also known as **Thiessen polygons**.
$$
R_{k}=\{x\in X\mid d(x,P_{k})\leq d(x,P_{j})\;{\text{for all}}\;j\neq k\}
$$


## Minkowski Sum and Difference

### Minkowski Sum

​	Let $A$ and $B$ be two point sets, and let $\mathbf{a}$ and $\mathbf{b}$ be the position vectors corresponding to pairs of points in $A$ and $B$. The Minkowski sum, $A \oplus B$, is then defined as the set
$$
A \oplus B=\{\mathbf {a} + \mathbf {b} \,|\,\mathbf {a} \in A,\ \mathbf {b} \in B\}
$$
where $\mathbf {a} + \mathbf {b}$ is the vector sum of the position vectors $\mathbf{a}$ and $\mathbf{b}$. 

![Minkowski Sum Example](/images/MinkowskiSum.png)

### Minkowski Difference

​	The Minkowski difference of two point sets $A$ and $B$ is defined analogously to the Minkowski sum:
$$
\begin{equation}
\begin{aligned}
A \ominus B &= \{\mathbf {a} - \mathbf {b} \,|\,\mathbf {a} \in A,\ \mathbf {b} \in B\}\\
&= A \oplus (-B) 
\end{aligned}
\end{equation}
$$

​	The Minkowski difference is important from a collision detection perspective because two point sets $A$ and $B$ collide (that is, have one or more points in common) **if and only if** their Minkowski difference $C (C = A \ominus B)$ contains the origin:

![Minkowski Diff Example](/images/MinkowskiDiff.png)

​	In fact, it is possible to establish an even stronger result: computing the minimum distance between A and B is equivalent to computing the minimum distance between C and the origin. This fact is utilized in the GJK algorithm:
$$
\begin{equation}
\begin{aligned}
distance(A, B) 
&= min\left\{ \| \mathbf{a} - \mathbf{b} \| \,:\,\mathbf {a} \in A,\, \mathbf {b} \in B\ \right\}\\
&= min\left\{ \| \mathbf{c} \| \,:\,\mathbf {c} \in A \ominus B \right\}
\end{aligned}
\end{equation}
$$






