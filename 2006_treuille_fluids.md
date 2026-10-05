# Model Reduction for Real-time Fluids

Adrien Treuille, Andrew Lewis, Zoran Popović

$$
\newcommand{\ff}{\mathbf f}
\newcommand{\nn}{\mathbf n}
\newcommand{\rr}{\mathbf r}
\newcommand{\uu}{\mathbf u}
\newcommand{\vv}{\mathbf v}
\newcommand{\xx}{\mathbf x}
\newcommand{\zz}{\mathbf z}
\newcommand{\Loss}{\mathcal L}
\DeclareMathOperator*{argmin}{argmin}
\newcommand{\abs}[1]{\left\vert #1 \right\vert}
\newcommand{\norm}[1]{\left\lVert #1 \right\rVert}
\newcommand{\partials}[2]{\frac{\partial #1}{\partial #2}}
\newcommand{\inv}[1]{#1^{-1}}
\newcommand{\d}{\, \mathrm{d}}
$$

---

# Summary



---

## Model reduction overview

Suppose $\uu \in \R^n$ and $\rr \in \R^m$, $m < n$, so that $\rr$ is the reduced representation of $\uu$

We define a projection operator $P : \uu \mapsto \rr$ and an "inverse" $\inv P : \rr \mapsto \uu$

In simulations we are typically interested in the time evolution of $\uu$, i.e. $\dot \uu = F(\uu)$

Model reduction thus entails simulating dynamics of the reduced variable, i.e. $\dot \rr = \hat F(\rr)$

Consider the all-linear case:

- $\dot \uu = M \uu$
- $\uu = B \rr$

Then using the Galerkin projection, we get the differential equation

$$
\dot \rr = B^\top M B \rr
$$

Note that $B^\top M B$ can be precomputed!

## Model reduction of fluids

Suppose we have $U = [\uu_1, ..., \uu_n]$ as a basis for the expected interactions ($\uu_i$ are sampled from example simulations). $\uu_i$ all satisfy:

- incompressibility: $\nabla \cdot \uu_i = 0$
- free-slip boundary: $\uu_i \cdot \nn = 0$ at (static) solid walls

Then we want to find a low-dimensional orthogonal basis $B = [\hat \uu_1, ..., \hat \uu_m]$ with $m \ll n$ so that we minimize reconstruction error:

$$
\norm{ U - B B^\top U }_F^2
$$

We know that we can solve this by choosing $\hat \uu_i$ as the first $m$ eigenvectors of $U U^\top$, aka doing PCA on $U$

Suppose $C$ encodes either of the constraints (incompressibility or free-slip boundary), so that any vector $\uu$ that satisfies the boundary has $C\uu = 0$. Then clearly $CU = 0$. Since $\hat \uu_i$ is a principal component of $U$, we have

$$
\begin{align*}
\lambda_i C \hat \uu_i &= C(\lambda_i \hat \uu_i) \\
&= C(UU^\top \hat \uu_i) \\
&= (CU)(U^\top \hat \uu_i) \\
&= 0(U^\top \hat \uu_i) \\
&= \mathbf 0
\end{align*}
$$

So the basis for our reduced model always satisfies both constraints, so our reduced simulation automatically satisfies constraints.

## Simulation

For the full simulation,

$$
\dot \uu = -\uu \cdot \nabla u - \nu \nabla^2 \uu + \nabla p + \ff \quad \text{such that } \nabla \cdot \uu = 0
$$

We can split up each term using operator splitting a la [Stam 1999]

### Advection

We can write advection component-wise as:

$$
\dot u^x = -\vv \cdot \nabla u^x = -\nabla \cdot (\vv u^x)
$$

(and similarly for $y, z$ components) where $\vv$ is the fixed advection velocity field, i.e. the field we are advecting through

Combining all 3 components, we can transform this into a linear equation $\dot \uu = A_\vv \uu$

Breaking $\vv$ up into reduced basis elements so that $\vv \approx B \rr$ where $\rr = [r_1, ..., r_m]$, we can approximate this as

$$
\dot \uu = \left( r_1 A_{\hat \uu_1} + ... + r_m A_{\hat \uu_m} \right) \uu
$$

Projecting into the subspace to get our reduced dynamics, we get

$$
\begin{align*}
\dot \rr &= \left( r_1 B^\top A_{\hat \uu_1} B + ... + r_m B^\top A_{\hat \uu_m} B \right) \rr \\
&= \hat A \rr
\end{align*}
$$

This linear ODE can be solved using the matrix exponential. It takes time $\mathcal O(m^3)$ (aka time doesn't depend on $n$ at all!) , but $m$ is small

#### Energy preservation

Since $\xx^\top A_\vv \xx = 0$ for all $\xx$, we end up with $\dot E = 0$, aka we exactly preserve energy

### Viscosity

Normal viscosity is solving $\dot \uu = \nu D \uu$, where $D$ discretizes $\nabla^2$. We can instead solve $\dot \rr = B^\top D B \rr$, using the same method as advection. Since the system is small an implicit solve will be very quick

### Projection

This step is typically done to help us satisfy our constraints, but our reduced model already guarantees this, so we can skip it

## Boundary conditions (this part deals with moving boundaries)

TODO

