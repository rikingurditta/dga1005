# Accelerate Neural Subspace-Based Reduced-Order Solver of Deformable Simulation by Lipschitz Optimization

Aoran Lyu, Shixian Zha, Chuhua Xian, Zhihao Cen, Hongmin Cai, Guoxin Fang

$$
\newcommand{\ff}{\mathbf f}
\newcommand{\gg}{\mathbf g}
\newcommand{\qq}{\mathbf q}
\newcommand{\uu}{\mathbf u}
\newcommand{\xx}{\mathbf x}
\newcommand{\yy}{\mathbf y}
\newcommand{\zz}{\mathbf z}
\newcommand{\MM}{\mathbf M}
\newcommand{\Fcal}{\mathcal F}
\newcommand{\Loss}{\mathcal L}
\DeclareMathOperator*{argmin}{argmin}
\newcommand{\parens}[1]{\left( #1 \right)}
\newcommand{\abs}[1]{\left\vert #1 \right\vert}
\newcommand{\norm}[1]{\left\lVert #1 \right\rVert}
\newcommand{\dbyd}[2]{\frac{\mathrm{d} #1}{\mathrm{d} #2}}
\newcommand{\partials}[2]{\frac{\partial #1}{\partial #2}}
\newcommand{\inv}[1]{#1^{-1}}

\newcommand{\Lip}{\mathrm{Lip}}
$$

---

## Summary

Lorem ipsum

---

## Neural Reduced Order Solver: Preliminary and Analysis

$\qq \in \R^n$ is DoF vector for the full simulation, e.g. vertex positions. We can evolve it in time using implicit Euler:

$$
\qq^{k+1} = \argmin_\qq \left( \frac{1}{2\Delta t^2} \norm{\qq - \overline \qq^{k+1}}^2_\MM + P(\qq) \right)
$$

- $\overline \qq^{k+1}$ is inertia-based guess of next timestep
- $\MM$ is mass matrix
- $P$ is potential energy

(standard stuff)

$\Omega \subseteq \R^r$ is an $r$-dimensional coordinate space, which maps to $\mathcal M$ which is the $r$-dimensional manifold of configurations. The mapping is $\ff : \Omega \to \mathcal M$. $\zz \in \Omega$ is a reduced coordinate. The point is for $r \ll n$ to greatly reduce the problem dimension.

Variational time integration with reduced coordinates now becomes:

$$
\zz^{k+1} = \argmin_\zz \underbrace{\left( \frac{1}{2\Delta t^2} \norm{\ff(\zz) - \overline \qq^{k+1}}^2_\MM + P(\ff(\zz)) \right)}_{E(\zz)}
$$

(aka replace $\qq = \ff(\zz)$)

### Learning neural subspaces

Trying to find an appropriate $\ff_\theta \in \Fcal = \{\ff_\theta : \theta\}$ by minimizing a loss $\Loss_C$

#### Supervised

With a dataset of samples, can train an autoencoder to learn the reduced representation:
$$
\theta^* = \argmin_\theta \frac{1}{\abs{\mathcal Q}} \sum_{\qq_i \in \mathcal Q} \norm{ \ff_\theta(\gg_\theta(\qq_i)) - \qq_i }^2_\MM
$$

- $\gg_\theta$ is encoder, $\ff_\theta$ is decoder, so $\ff_\theta : \Omega \to \mathcal M$ is our reduced coordinate mapping function
- $\mathcal Q$ is dataset of samples of a simulation

This is the method of [Fulton et al. 2019](2019_fulton_latent-space)

#### Unsupervised

This is the method of Sharp et al. 2023

#### Convergence speed

Convergence time for time integration is = $n_\text{iter} (C_\text{eval} + C_\text{dir})$

- $n_\text{iter}$ is number of iterations
- $C_\text{eval}$ is cost of evaluating $\Loss_C, \nabla \Loss_C, \mathbf H_{\Loss_C}$
- $C_\text{dir}$ is cost of finding descent direction

$C_\text{dir}$ is reduced with neural methods because dimensionality is reduced so much, $r \ll n$

$n_\text{iter}$ depends on the problem at hand - Newton's method converges quadratically

$$
\norm{ \mathbf e^{k+1} } \leq \parens{ \Lip \nabla^2_\zz E } \norm{ \nabla^2_\zz E(\zz^*)^{-1} } \norm{ \mathbf e^k }^2
$$

where $\mathbf e^k = \zz^k - \zz^*$ is the error of the $k^\text{th}$ guess vs the true solution $\zz^*$

So $n_\text{iter}$ scales by the Lipschitz constant $\Lip \nabla^2_\zz E$:

$$
\Lip \nabla^2_\zz E = \max_{\xx, \yy \in \Omega} \frac{\norm{ \nabla^2_\zz E(\xx) - \nabla^2_\zz E(\yy) }}{\norm{\xx - \yy}}
$$

Recall that $E$ depends on $\ff_\theta$, so $\Lip \nabla^2_\zz E$ does too

Method in paper tries, given $\theta^\text{init}$, to find $\theta^*$ which has the same mapping $\mathrm{Im}(\ff_{\theta^\text{init}}) = \mathrm{Im}(\ff_{\theta^*})$ but which has smaller $\Lip \nabla^2_\zz E$

### Loss with Lipschitz Optimization and Training Details

Use $\Loss_{LS}$ to denote paper's "Lipschitz loss"

#### Lipschitz loss

Directly optimizing Lipschitz constant is intractable because it is global, instead use statistics from a set of observations $\mathcal Z$
$$
\Loss_{LS} = \mathbb E_{\xx, \yy \sim \Pi_\theta(\zz)} \frac{\norm{ \nabla^2_\zz E(\xx) - \nabla^2_\zz E(\yy) }^2}{\norm{ \xx - \yy }^2}
$$

#### Training

Use original methods (i.e., Fulton et al. 2019 or Sharp et al. 2023) to find first guess $\theta^\text{init}$ (WHY FIND FIRST GUESS?). Then transform this guess by training on construction and Lipschitz losses together:
$$
\min_\theta \Loss_C(\theta) + \lambda_{LS} \Loss_{LS}(f_\theta)
$$
For supervised setting, $\mathcal Z$ is computed by using a subset of $\gg_\theta(\mathcal Q)$. For unsupervised setting $\mathcal Z = \mathcal N$ normal distribution

Inertia term adds noise to $\Loss_{LS}$, so paper computes it with $\nabla^2_\zz P$ instead of full $\nabla^2_\zz E$

#### Cubature

Training this directly requires a lot of memory, so paper uses the cubature method of Von Tycowicz et al. 2013

### Results

Big tradeoff between precomputation (training) time and time step time
