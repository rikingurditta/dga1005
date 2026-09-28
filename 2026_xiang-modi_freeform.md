# FreeForm: Reduced-Order Deformable Simulation from Particle-Based Skinning Eigenmodes

Donglai Xiang, Vismay Modi, Rishit Dagli, Ty Trusty, Gilles Daviet, Anka He Chen, Nicholas Sharp, David I.W. Levin

$$
\newcommand{\CC}{\mathbf C}
\newcommand{\JJ}{\mathbf J}
\newcommand{\KK}{\mathbf K}
\newcommand{\MM}{\mathbf M}
\newcommand{\PP}{\mathbf P}
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

One common formulation is DOFs are affine transformations, i.e. $\zz = \{\ZZ_j \in \R^{3 \times 4}\}_{j=1}^m$ with associated skinning weights $\{ \WW^j : \R^3 \to \R^m \}$:
$$
\xx = \phi(\XX, \zz) = \XX + \sum_j \WW^j(\XX) \ZZ_j \overline{\XX}
$$
Usually there is a training stage that finds the $\WW^j$ and a simulation stage that evolves $\zz$ in time. The standard simulation timestep is:
$$
\zz_{t+1} = \argmin_\zz \mathrm{\mathbf{Ir}}(\zz, \zz_t) + E_\text{pot}(\zz) + E_\text{ext}(\zz)
$$

### Simplicits

Simplicits proposes $\WW^j$ can be mesh-free by being trained as neural fields

### RKPM

This paper proposes RKPM as opposed to neural skinning weights

RKPM represents a vector-valued function $\uu : \Omega \to \R^d$ by

- node values $\cc = \{ \cc_k \in \R^d \}_{k=1}^K$
- weighted by reproducing kernels $\{ \phi_k : \Omega \to \R \}_{k=1}^K$
  - each centred at a position $\{ \pp_k \in \Omega \}_{k=1}^K$

Kernels are similar to Radial Basis Functions but have correction terms:
$$
\phi_k(\XX) = \varphi_k(\XX) \PP(\pp_k)^\top \CC(\XX)
$$
$\PP(\XX) = (1, x, y, z)$ and $\CC : \Omega \to \R^{\dim P}$
