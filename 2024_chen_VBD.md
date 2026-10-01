# Vertex Block Descent

Anka He Chen, Ziheng Liu, Yin Yang, Cem Yuksel
$$
\newcommand{\aa}{\mathbf a}
\newcommand{\ff}{\mathbf f}
\newcommand{\vv}{\mathbf v}
\newcommand{\xx}{\mathbf x}
\newcommand{\yy}{\mathbf y}
\newcommand{\zz}{\mathbf z}
\newcommand{\HH}{\mathbf H}
\newcommand{\MM}{\mathbf M}
\DeclareMathOperator*{argmin}{argmin}
\newcommand{\abs}[1]{\left\vert #1 \right\vert}
\newcommand{\norm}[1]{\left\lVert #1 \right\rVert}
\newcommand{\partials}[2]{\frac{\partial #1}{\partial #2}}
\newcommand{\inv}[1]{#1^{-1}}
\newcommand{\d}{\, \mathrm{d}}
$$

---

## Summary

---

## VBD for elastic bodies

### Global optimization

Suppose $N$ vertices, then simulation state is positions and velocities of each vertex: $(\xx^t, \vv^t) \in \R^{3N} \times \R^{3N}$

Then implicit Euler variational time step is:

$$
\xx^{t+1} = \argmin_\xx G(\xx); \quad \vv^{t+1} = \frac{1}{h} \left( \xx^{t+1} - \xx^t \right) \\
\text{where }
G(\xx) = \frac{1}{2h^2} \norm{\xx - \yy}^2_\MM + E(\xx)
$$

The first term is the *inertia potential*, i.e. potential from deviating from fully ineratial time step. $\yy = \xx^t + h\vv^t + h^2 \aa_\text{ext}$

Paper proposes coordinate-based optimization, i.e. optimizing each coordinate fixing the rest. We can define the *local variational energy* $G_i$:

$$
G_i(\xx) = \frac{m_i}{2h^2} \norm{\xx_i - \yy_i}^2 + \sum_{j \in \mathcal F_i} E_j(\xx)
$$

$\mathcal F_i$ is the set of force elements that refer to vertex $i$, so the sum term above covers the energy changes from the change in vertex $i$

Note that $G(\xx) \neq \sum_i G_i(\x)$, but if we reduce $G_i(\xx)$ by $\Delta G$, then $G(\xx)$ also reduces by $\Delta G$. So we can just iteratively optimize the $G_i$s to eventually optimize $G$!

### Local system solver

Local system only has 3 DoFs, so we can solve it with Newton's method using the Hessian:

$$
\HH_i \Delta \xx_i = \ff_i
$$

$\ff_i$ is the total force acting on vertex $i$:

$$
\begin{align*}
\ff_i &= -\partials{G_i(\xx)}{\xx_i} \\
&= -\frac{m_i}{h^2} (\xx_i - \yy_i) - \sum_{j \in \mathcal F_i} \partials{E_j(\xx)}{\xx_i}
\end{align*}
$$

$\HH_i$ is the Hessian of $G_i$:

$$
\begin{align*}
\HH_i &= \partials{^2G_i(\xx)}{\xx_i^2} \\
&= \frac{m_i}{h^2} \mathbf I + \sum_{j \in \mathcal F_i} \partials{^2 E_j(\xx)}{\xx_i^2}
\end{align*}
$$

Since system is only $3 \times 3$, we can just solve $\Delta \xx_i = \inv \HH_i \ff_i$ analytically

If $\abs{\det \HH_i} < \epsilon$ (for some chosen $\epsilon$) then this system is rank-deficient, so we just skip this vertex and come back to it in a future iteration. In the meantime we'll probably adjust the vertices around it, so it probably won't be rank-deficient when we come back.

#### Line search

Solving $\HH_i \Delta \xx_i = \ff_i$ gives us $\Delta \xx_i$, which is direction of descent for one Newton iteration, not actual solution. To make sure this actually reduces $G_i$, we can use it as the direction for line search. However in practice this is not necessary.

### Damping, constraints

can modify to add damping and constraints per-vertex :]

### Collisions

Collisions can also be handled per-vertex:

$$
E_c(\xx) = \frac{1}{2} k_c d^2
$$

where $d$ is the penetration depth: $d = \max \left( 0, (\xx_b - \xx_a) \cdot \mathbf{\hat n} \right)$ - $\xx_a, \xx_b$ are contact points and $\mathbf{\hat n}$ is the contact normal

Can run collision resolution every few iterations instead of every iteration, since it is expensive

### Friction

For collision $c$, the friction depends on the relative motion

$$
\delta \xx_c = (\xx_a - \xx_a^t) - (\xx_b - \xx_b^t)
$$

### Initialization

Since it is an iterative method, we need a first guess:

$$
\xx = \xx^t + h \vv^t + h^2 \tilde \aa
$$

This estimates inertia and acceleration, but has a special method for estimating acceleration:

First we compute acceleration across prev frame:

$$
\aa^t = \frac{1}{h} \left( \vv^t - \vv^{t-1} \right)
$$

Then we compute the component of $\aa^t$ along the direction of the external acceleration:

$$
a^t_\text{ext} = \aa^t \cdot \frac{\aa_\text{ext}}{\norm{\aa_\text{ext}}}
$$

Then we clamp $a^t_\text{ext}$ so that it doesn't actually amplify the external acceleration:

$$
\tilde \aa = \tilde a \aa_\text{ext} \text{ where } \tilde a = \mathrm{clamp} \left( \frac{a^t_\text{ext}}{\norm{\aa_\text{ext}}}, 0, 1 \right)
$$

### Parallelization

All the vertices touching the same force element must be dealt with sequentially (coloured differently), as they affect each other's energy computations. But otherwise we can parallelize vertex computations.

