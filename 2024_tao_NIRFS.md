# Neural Implicit Reduced Fluid Simulation

Yuanyuan Tao, Iavan Puhachov, Derek Nowrouzezahrai, Paul Kry

$$
\newcommand{\JJ}{\mathbf J}
\newcommand{\MM}{\mathbf M}
\newcommand{\UU}{\mathbf U}
\newcommand{\VV}{\mathbf V}
\newcommand{\WW}{\mathbf W}
\newcommand{\XX}{\mathbf X}
\newcommand{\YY}{\mathbf Y}
\newcommand{\ZZ}{\mathbf Z}
\newcommand{\bb}{\mathbf b}
\newcommand{\ff}{\mathbf f}
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

## Model

- ICE - initial condition encoder
  - physics input
  - maps initial conditions to latent space
- DHNODE - damped Hamiltonian neural ODE
  - evolves shape dynamics in latent space
- INR decoder - implicit neural representation decoder
  - maps latent space to shape space

### Reduced representation

Fluid geometry is represented by $\SDF : \R^3 \to \R$ which provides signed distance to fluid surface

INR decoder is an MLP that learns SDF, so $D(\xx, \qq) = \SDF(\xx)$ subject to latent space $\qq$

#### Gaussian embedding

not really sure what this is at all tbh

#### Fourier features

$$
\xx'(\xx) = (\cos(2\pi\bb_1^\top \xx), \sin(2\pi\bb_1^\top \xx), \cdots, \cos(2\pi\bb_m^\top \xx), \sin(2\pi\bb_m^\top \xx))
$$

#### Importance sampling

bruh

### Latent DHNODE

Typical formulation is PDE that describes local dynamics, instead will use neural ODE that describes global dynamics.

ODE is in $\zz(t) = (\qq(t), \pp(t))$, where $\qq$ is latent geometry and $\pp$ is its momentum. It is not based on Navier-Stokes, but instead on the fluid geometry

$$
\begin{align*}
\dbyd{\qq}{t} &= \inv \mm \pp \\
\dbyd{\pp}{t} &= -\ggamma \inv \mm \pp - \partials{V(\qq)}{\qq}
\end{align*}
$$

- $\mm$ is diagonal mass matrix
- $\ggamma$ is diagonal damping matrix
- $V$ is potential energy (using MLP)

Even though these simplifications (Cartesian coordinates, diagonal mass, linear/diagonal damping) might not be correct for fluids, the learned result looks good enough!

## Training

ICE, DHNODE, INR decoder all trained jointly

bruh
