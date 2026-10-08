# PolyStokes: A Polynomial Model Reduction Method for Viscous Fluid Simulation

Jonathan Panuelos, Ryan Goldade, Eitan Grinspun, David Levin, Christopher Batty

$$
\newcommand{\uu}{\mathbf u}
\newcommand{\vv}{\mathbf v}
\newcommand{\xx}{\mathbf x}

\newcommand{\AA}{\mathbf A}
\newcommand{\DD}{\mathbf D}
\newcommand{\JJ}{\mathbf J}
\newcommand{\MM}{\mathbf M}

\newcommand{\GGG}{\mathcal G}
\newcommand{\HHH}{\mathcal H}

\newcommand{\d}{\, \mathrm{d}}

\newcommand{\inv}[1]{#1^{-1}}
\newcommand{\norm}[1]{\left\lVert #1 \right\rVert}
\newcommand{\partials}[2]{\frac{\partial #1}{\partial #2}}
$$

---
## Summary

---

## Methods

Implicit viscosity step with operator splitting

$$
\frac{\uu - \uu^*}{\Delta t} = \frac{\mu}{\rho} \nabla \cdot \left(\nabla \uu + (\nabla \uu^\top) \right)
$$

Equivalently, we can solve this variationally by minimizing $J[\uu]$:

$$
J[\uu] = \iiint_{\Omega_L} \left( \frac{\rho}{2} \norm{\uu - \uu^*}^2 + \Delta t \mu \norm{\frac{\nabla \uu + (\nabla \uu)^\top}{2}}_F^2 \right) \d V
$$

The fluid domain $\Omega_L$ is separated into uniform Cartesian regions $\Omega_C$ and reduced fluid regions $\Omega_R$, so $\Omega_L = \Omega_C \cup \Omega_R$. Their intersection is at the boundary of the reduced fluid $\Omega_C \cap \Omega_R = \partial \Omega_R (\subseteq \partial \Omega_C)$. The reduced region is only on the interior, not near the surface, so $\partial \Omega_L \subseteq \partial \Omega_C$.

The velocity field $\uu$ can also be split using indicator functions, i.e. $\uu_C = \uu I_{\Omega_C}$ and $\uu_R = \uu I_{\Omega_R}$. Now we can split up the integral:

$$
\begin{align*}
J[\uu_C, \uu_R] = \iiint_{\Omega_L} &\frac{\rho}{2} \norm{\uu_C - \uu_C^*}^2 + \frac{\rho}{2} \norm{\uu_R - \uu_R^*}^2 \\
&+ \rho (\uu_C - \uu_C^*) \cdot (\uu_R - \uu_R^*) \\
&+ \Delta t \mu \left( \norm{\frac{\nabla \uu_C + (\nabla \uu_C)^\top}{2}}^2_F + \norm{\frac{\nabla \uu_R + (\nabla \uu_R)^\top}{2}}^2_F \right) \\
&+ 2\Delta t \mu \left\langle \frac{\nabla \uu_C + (\nabla \uu_C)^\top}{2}, \frac{\nabla \uu_R + (\nabla \uu_R)^\top}{2} \right\rangle_F \d V
\end{align*}
$$

(aka corresponding norm terms and inner product terms coming from breaking $\uu$ into 2 pieces)

Minimizing this solves for implicit viscosity step for each subdomain and enforces $\uu_C = \uu_R$ on $\partial \Omega_R$, aka separate viscosity solves with strong two-way coupling between them

### Reduced model

Suppose linear reduced model is used, so $\uu_R = \JJ^\top \vv_R$ where $\vv_R$ are the reduced coordinates and $\JJ$ projects onto them. Then, the reduced model can be solved as:

$$
\begin{pmatrix} \AA_{11} & \AA_{12} \\ \AA_{12}^\top & \AA_{22} \end{pmatrix}
\begin{pmatrix} \uu_C \\ \vv_R \end{pmatrix}
=
\frac{1}{\Delta t} \begin{pmatrix} \MM_C W_F^u W_L^u \uu^* \\ \MM_R \vv_R^* \end{pmatrix}
$$

where:

$$
\begin{align*}
\AA_{11} &= \frac{1}{\Delta t} \MM_C W_F^u W_L^u + 2W_F^u \DD^\top \inv{(W_F^\tau)} \boldsymbol \mu W_L^\tau \DD W_F^u \\
\AA_{12} &= 2 W_F^u \DD^\top \boldsymbol \mu \DD \JJ^\top \\
\AA_{22} &= \frac{1}{\Delta t} \MM_R + 2 \JJ \DD^\top \boldsymbol \mu \DD \JJ^\top
\end{align*}
$$

($\DD$ is the discrete gradient operator, $W_L^a$ is what % volume around $a$ is in $\Omega_L$, and $W_F^a$ is what % volume around $a$ is in the fluid domain $\Omega_F$)

$\AA_{11}$ and $\AA_{22}$ represent fluid updates in uniform and reduced regions respectively, $\AA_{12}$ is coupling between them

#### Affine velocity fields

Follows reduced modelling from Goldade et al. 2020 (TODO)

$$
\uu_R(\xx) = \uu_\text{const} + \GGG(\xx - \xx_\text{COM})
$$
where $\GGG = \nabla \uu_R$ is the gradient 2-tensor, i.e.

$$
\GGG_{3D} = \begin{pmatrix}
\partials{u}{x} & \partials{u}{y} & \partials{u}{z} \\
\partials{v}{x} & \partials{v}{y} & \partials{v}{z} \\
\partials{w}{x} & \partials{w}{y} & \partials{w}{z}
\end{pmatrix}
$$

So $\mathrm{Tr}(\GGG) = \nabla \cdot \uu = 0$ (this allows us to model $\GGG$ with one fewer DOF, since we know $\partials{w}{z} = -\partials{u}{x}-\partials{v}{y}$)

#### Problems with the Affine Description

If we model reduced velocities as affine fields as above we get $\GGG = \DD \JJ^\top \vv_R$. This is constant, so taking another derivative i.e. the $\JJ \DD^\top \DD \JJ^\top$ term gives us $0$, which means we can't solve for viscous forces

#### Polynomial velocity fields

We can have a quadratic instead of affine model:

$$
\uu_R(\xx)
= \uu_\text{const}
+ \GGG(\xx - \xx_\text{COM})
+ \frac{1}{2} (\xx - \xx_\text{COM})^\top \HHH (\xx - \xx_\text{COM})
$$

where $\HHH$ is the velocity Hessian 3-tensor, i.e.

$$
\HHH_{i, j, k} = \partials{^2 u_i}{x_j \partial x_k}
$$