# Deep Fluids: A Generative Network for Parameterized Fluid  Simulations

Byungsoo Kim, Vinicius C. Azevedo, Nils Thuerey, Theodore Kim, Markus Gross and Barbara Solenthaler

$$
\newcommand{\JJ}{\mathbf J}
\newcommand{\KK}{\mathbf K}
\newcommand{\MM}{\mathbf M}
\newcommand{\UU}{\mathbf U}
\newcommand{\VV}{\mathbf V}
\newcommand{\WW}{\mathbf W}
\newcommand{\XX}{\mathbf X}
\newcommand{\YY}{\mathbf Y}
\newcommand{\ZZ}{\mathbf Z}
\newcommand{\bb}{\mathbf b}
\newcommand{\cc}{\mathbf c}
\newcommand{\ff}{\mathbf f}
\newcommand{\gg}{\mathbf g}
\newcommand{\mm}{\mathbf m}
\newcommand{\pp}{\mathbf p}
\newcommand{\qq}{\mathbf q}
\newcommand{\uu}{\mathbf u}
\newcommand{\ww}{\mathbf w}
\newcommand{\xx}{\mathbf x}
\newcommand{\yy}{\mathbf y}
\newcommand{\zz}{\mathbf z}
\newcommand{\aalpha}{\boldsymbol \alpha}
\newcommand{\bbeta}{\boldsymbol \beta}
\newcommand{\ggamma}{\boldsymbol \gamma}
\newcommand{\ppsi}{\boldsymbol \psi}
\newcommand{\pphi}{\boldsymbol \phi}
\newcommand{\ttheta}{\boldsymbol \theta}
\newcommand{\PPhi}{\boldsymbol \Phi}
\DeclareMathOperator*{argmin}{argmin}
\newcommand{\abs}[1]{\left| #1 \right|}
\newcommand{\norm}[1]{\left\lVert #1 \right\rVert}
\newcommand{\d}{\, \mathrm{d}}
\newcommand{\dbyd}[2]{\frac{\d #1}{\d #2}}
\newcommand{\partials}[2]{\frac{\partial #1}{\partial #2}}
\DeclareMathOperator{SDF}{SDF}
$$

---

## Summary

---

## Generative model for fluids

Goal is to train CNN that can approximate velocity fields

- "CNNs organize the data manifold into shift-invariant feature maps"

Network input is $[\uu_\cc, \cc]$

- $\uu_c \in \R^{H \times W \times D \times V_\text{dim}}$ is velocity vector
- $\cc$ is simulation parameters, e.g. smoke source properties and frame time

Network output is a streamfunction\* $G(\cc)$, where $G : \R^n \to \R^{H \times W \times D \times G_\text{dim}}$. $G_\text{dim} = 1$ for 2d and $= 3$ for 3d, since $\mathbf v = \nabla \times G$

Let $\hat \uu_\cc = \nabla \times G(\cc)$

L1 loss is first used:

$$
L_G(\cc) = \norm{ \uu_\cc - \hat \uu_\cc }_1
$$

However, this does not guarantee smooth behaviour, so loss is modified to also match derivatives:

$$
L_G(\cc)
= \lambda_\uu \norm{ \uu_\cc - \hat \uu_\cc }_1
+ \lambda_{\nabla \uu} \norm{ \nabla \uu_\cc - \nabla \hat \uu_\cc }_1
$$

### \*Divergent case

Liquids have divergence at the free surface boundary, so training a streamfunction doesn't work. In this case $G$ is just velocity, so $\hat \uu_\cc = G(\cc)$ in this case instead of $\nabla \times G(\cc)$

### Implementation

uses a CNN, not really sure what it's doing with the Small Blocks and Big Blocks

## Extended parameterizations - encoder model

Network so far doesn't take into account history of simulation, so e.g. moving snoke source cannot be captured without inputting lots and lots of data as "parameters". Instead we make a separate time stepper.

Create an encoder $G^\dagger : \R^{H \times W \times D \times V_\text{dim}} \to \R^n$ and a time integrator $T : \R^{n+k} \to \R^{n-k}$. $G$ and $G^\dagger$ act kinda like a decoder-encoder pair in an autoencoder.

$G^\dagger : \mathbf v \mapsto \cc$ maps velocity fields into parameterizations $\cc = [\zz, \pp] \in \R^n$, where $\zz \in \R^{n-k}$ is unsupervised features in the latent space and $\pp \in \R^k$ is a "supervised parameterization to control specific attributes"

$G$ and $G^\dagger$ are jointly trained with loss:

$$
L_{AE}(\uu)
= \lambda_\uu \norm{ \uu_\cc - \hat \uu_\cc }_1
+ \lambda_{\nabla \uu} \norm{ \nabla \uu_\cc - \nabla \hat \uu_\cc }_1
+ \lambda_\pp \norm{ \pp - \hat \pp }_2^2
$$

(where $\hat \uu_\cc = \nabla \times G(G^\dagger(\uu))$ I THINK and $\hat \pp = \mathrm{proj}_\pp(G^\dagger(\uu))$)

### Latent space integration

$T : \xx_t \mapsto \Delta \zz_t$

-  $\xx_t = [\cc_t, \Delta \pp_t] \in \R^{n+k}$
  - $\cc_t$ is latent code
  - $\Delta \pp_t = \pp_{t+1} - \pp_t$ is difference between input parameters in time, so this helps transition $\cc_t$ to $\cc_{t+1}$
- new latent code is $\zz_{t+1} = \zz_t + T(\xx_t)$, so that $\cc_{t+1} = [\zz_{t+1}, \pp_{t+1}]$

Training loss uses a window of $w$ sequential codes:

$$
L_T(\xx_t, ..., \xx_{t+w-1}) = \frac{1}{2} \sum_{i=t}^{t+w-1} \norm{ \Delta \zz_i - T_i }_2^2
$$

where each $T_i$ is computed recursively

$T$ is modeled as a 3-layer MLP

### Algorithm

<img src="/Users/rikin/School/phd/dga 1005/img/2019_kim_deep-fluids_alg-and-arch.png" alt="image-20260916182956205" style="zoom: 33%;" />

