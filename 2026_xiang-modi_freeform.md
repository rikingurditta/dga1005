# FreeForm: Reduced-Order Deformable Simulation from Particle-Based Skinning Eigenmodes

Donglai Xiang, Vismay Modi, Rishit Dagli, Ty Trusty, Gilles Daviet, Anka He Chen, Nicholas Sharp, David I.W. Levin

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

## Intro

- Simplicits enabled ROM on mesh-free geometry
- Simplicits has 2 issues
  - requires training neural field on every input object before simulation
  - has low accuracy
    - possibly due to difficulty optimizing variational formulation
- RKPM = Reproducing Kernel Particle Method
  - mesh-free
  - "makes it possible to obtain a set of optimal skinning eigenmodes through eigenanalysis, which is more accurate and significantly faster than other comparable subspace generation techniques"

### Contributions

- mesh-free reduced-order elastodynamics using RKPM skinning eigenmodes
- simple Hessian of neo-Hookean energy

## Methodology

$\XX$ is point in object space, $\xx$ is in world space, $\phi$ is deformation map that maps between them

$$
\xx = \phi(\XX, \zz)
$$

where $\zz \in \R^n$ are degrees of freedom that control motion of object
