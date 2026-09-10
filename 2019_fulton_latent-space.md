# Latent-space Dynamics for Reduced Deformable Simulation

Lawson Fulton, Vismay Modi, David Duvenaud, David I. W. Levin and Alec Jacobson

$$
\newcommand{\JJ}{\mathbf J}
\newcommand{\MM}{\mathbf M}
\newcommand{\UU}{\mathbf U}
\newcommand{\bb}{\mathbf b}
\newcommand{\ff}{\mathbf f}
\newcommand{\qq}{\mathbf q}
\newcommand{\uu}{\mathbf u}
\newcommand{\xx}{\mathbf x}
\newcommand{\zz}{\mathbf z}
\newcommand{\ppsi}{\boldsymbol \psi}
\newcommand{\pphi}{\boldsymbol \phi}
\newcommand{\ttheta}{\boldsymbol \theta}
\DeclareMathOperator*{argmin}{argmin}
\newcommand{\norm}[1]{\left\lVert #1 \right\rVert}
\newcommand{\partials}[2]{\frac{\partial #1}{\partial #2}}
\newcommand{\inv}[1]{#1^{-1}}
$$

---

# Summary

Typical ROM models $\uu = \UU \qq$, so the actual motions $\uu$ are calculated as a linear function $\UU$ of the reduced DoF $\qq$. To encode nonlinear motions, we can replace $\UU$ with a neural deformation function $\ppsi$. This is learned as an autoencoder, where $\uu \approx \ppsi(\inv \ppsi(\uu))$. Some tricks are developed to learn a good $\ppsi$ and take its derivatives.

---

## Background: Linear reduced model

Let $\xx_0 \in \R^{3n}$ be rest pose of tet mesh and $\uu \in \R^{3n}$ be some deformation from rest. Then elasticity dynamics are governed by Newton's 2nd law:
$$
\MM \ddot \uu = \ff_\text{int} (\uu) + \ff_\text{ext}
$$
We reduce the model by writing $\uu = \UU \qq$ where $\qq \in \R^k$ is a vector of $k$ degrees of freedom and $\UU \in \R^{3n \times k}$ is a matrix that translates between them, i.e.
$$
\UU = \begin{pmatrix} | &  & | \\ \bb_1 & \cdots & \bb_k \\ | & & | \end{pmatrix}
$$
where $\{\bb_1, ..., \bb_k\}$ form a basis for some subspace $W$ of possible deformations.

Now the equations of motion are
$$
\tilde \MM \ddot \qq = \tilde \ff_\text{int} (\qq) + \UU^\top \ff_\text{ext}
$$
where $\tilde \MM = \UU^\top \MM \UU$ and $\tilde \ff_\text{int}(\qq) = \UU^\top \ff_\text{int} (\UU \qq)$

### Subspace construction

We can take a bunch of sample deformations $\uu_1, ..., \uu_N$ and perform PCA on them to find an orthogonal basis for their spanning set. Then we can take the first $k$ PCA vectors as our $\bb_1, ..., \bb_k$, as their span best represents the range of deformations

### Reduced model forces

Follow [[AKJ08]](https://www.cs.cornell.edu/~djames/papers/cubature08.pdf) to use cubature to precompute internal forces for reduced dimensions - i.e., instead of doing expensive full computation $\UU^\top \ff_\text{int} (\UU \qq)$ every time, precompute $\{\mathbf w_i\}$ and $\{\tilde \ff^i_\text{int}\}$ to quickly approximate
$$
\tilde \ff_\text{int} (\qq) \approx \sum_i \mathbf w_i \tilde \ff_\text{int}^i(\qq)
$$
(in special cases, can also use StVK energy [[BJ05]](https://publications.ri.cmu.edu/storage/publications/pub_files/pub4/barbic_jernej_2005_1/barbic_jernej_2005_1.pdf) for quick exact reduced force)

## Nonlinear reduced model

Instead of the linear parametrization $\uu = \UU \qq$, we can use an autoencoder to learn a nonlinear parametrization $\uu = \ppsi(\zz)$

We define an encoder $\overline \ppsi : \R^{3n} \to \R^r$ and a decoder $\ppsi : \R^r \to \R^{3n}$ such that
$$
\uu \approx \ppsi (\overline \ppsi ( \uu ) )
$$

### Outer layer is PCA

First, as in the linear method, perform PCA on the training data to find $\UU$ whose columns are the first $k$ PCA vectors. We can use this as the outermost layer of our autoencoder system:
$$
\ppsi(\zz) = \UU \pphi(\zz)
$$
Then $\pphi$ is the strictly nonlinear part of the network. It is a neural network with 2 hidden layers of size 100, and ELU activation:
$$
\mathrm{ELU}(x) = \begin{cases}
e^x - 1 & \text{if } x < 0 \\
x & \text{if } x \geq 0 \end{cases}
$$
(this is a smoothed version of RELU)

### Training

Model weights are computed by minimizing a loss:
$$
\hat \ttheta = \argmin_\ttheta \sum_{i=1}^N \norm{\uu_i - \ppsi_\ttheta(\overline \ppsi_\ttheta(\uu_i))}_2^2
$$
i.e., the encoder and decoder are learned by trying to use them to reproduce the sampled deformations.

We chose that the end layer is $\UU$, a linear layer. We could adjust it further during optimization, but in practice this does not help much, so instead we can take it for granted and factor it into optimization:
$$
\hat \ttheta = \argmin_\ttheta \sum_{i=1}^N \norm{ \UU^\top \uu_i - \pphi_\ttheta(\overline \pphi_\ttheta(\UU^\top \uu_i)) }_2^2
$$
(each $\UU^\top \uu_i$ is precomputed to speed up training)

## Reduced model dynamics

Directly using the autoencoder in place of the linear reduced model is problematic because evaluating the higher order derivatives would be very costly

### Variational implicit integrator

We can formulate implicit time integration as a variational problem:
$$
\uu_{n+1} = \argmin_\uu \frac{1}{2h^2} \norm{\uu - (\uu_n + h \dot \uu_n)}_\MM^2 + V(\uu)
$$
where $V$ is the elastic potential energy.

The backward difference approximation is used for the time derivative:
$$
\dot \uu_n \approx \frac{1}{h}(\uu_n - \uu_{n-1})
$$
We use the decoder parametrization $\uu = \ppsi(\zz)$ for our optimization:
$$
\zz_{n+1} = \argmin_\zz \frac{1}{2h^2} \norm{\ppsi(\zz) - (2\uu_n - \uu_{n-1})}_\MM^2 + V(\ppsi(\zz))
$$
This still requires computing a large matrix product $\uu^\top \MM \uu$, so we can speed it up by factoring out $\UU$. If $\qq = \pphi(\zz)$ then $\uu = \UU \qq$, so we can rewrite our optimization as
$$
\zz_{n+1} = \argmin_\zz \frac{1}{2h^2} \norm{\pphi(\zz) - (2\qq_n - \qq_{n-1})}^2_{\tilde \MM} + V(\ppsi(\zz))
$$
or,
$$
E(\zz, \qq_n, \qq_{n-1}) = \frac{1}{2h^2} \norm{\pphi(\zz) - (2\qq_n - \qq_{n-1})}^2_{\tilde \MM} + V(\ppsi(\zz)) \\
\zz_{n+1} = \argmin_\zz E(\zz, \qq_n, \qq_{n-1})
$$

### Optimization

Using Newton's method to minimize $E$ would require higher order derivatives of $\ppsi$ so it is infeasible. Instead we use L-BFGS.

The gradient is
$$
\partials{E}{\zz} = \frac{1}{h^2} \JJ_\zz^\top \tilde \MM (\pphi(\zz) - (2\qq_n - \qq_{n-1})) - \JJ_\zz^\top \UU^\top \ff_\text{int} (\ppsi(\zz))
$$
where $\displaystyle \JJ_\zz = \partials{\pphi}{\zz}$. $\JJ_\zz$ is costly to construct, but only its action on other vectors is needed, which can be computed using forward-mode autodiff.

L-BFGS uses gradients to estimate the Hessian. We can warm-start this estimation as:
$$
\tilde{\mathbf H} = \JJ_{\zz_n}^\top \tilde {\mathbf K}_0 \JJ_{\zz_n}
$$
where $\tilde{\mathbf K}_0 = \UU^\top \mathbf K_0 \UU$, and $\displaystyle \mathbf K_0 = \partials{^2 V(\mathbf 0)}{\uu^2}$ is the stiffness matrix at the rest state.

### Cubature

...

