# Simplicits: Mesh-Free, Geometry-Agnostic, Elastic Simulation

Vismay Modi, Nicholas Sharp, Or Perel, Shinjiro Sueda, David I. W. Levin

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
\newcommand{\qq}{\mathbf q}
\newcommand{\uu}{\mathbf u}
\newcommand{\ww}{\mathbf w}
\newcommand{\xx}{\mathbf x}
\newcommand{\yy}{\mathbf y}
\newcommand{\zz}{\mathbf z}
\newcommand{\aalpha}{\boldsymbol \alpha}
\newcommand{\bbeta}{\boldsymbol \beta}
\newcommand{\ppsi}{\boldsymbol \psi}
\newcommand{\pphi}{\boldsymbol \phi}
\newcommand{\ttheta}{\boldsymbol \theta}
\newcommand{\PPhi}{\boldsymbol \Phi}
\DeclareMathOperator*{argmin}{argmin}
\newcommand{\abs}[1]{\left| #1 \right|}
\newcommand{\norm}[1]{\left\lVert #1 \right\rVert}
\newcommand{\partials}[2]{\frac{\partial #1}{\partial #2}}
\newcommand{\d}{\, \mathrm{d}}
$$

## Method

Input is rest state geometry with an inside-outside function $\Phi: \R^3 \to \R$ where $\Phi(\xx) = 1$ inside the object, $\Phi(\xx) = 0$ outside, and the boundary may be blurry

### Implicit time integration

Let $\phi$ be the deformation map, so $\xx = \phi(\XX, \zz(t))$ is the world space location of the deformed shape, assuming $\zz$ are some reduced degrees of freedom of the shape.

If $\phi$ is linear wrt $\zz$, then implicit time integration can be discretized as:
$$
\zz_{t+1} = \argmin_\zz \frac{1}{2} \norm{\zz - \tilde \zz_t}_\MM^2 + h^2 E_\text{pot}(\zz)
$$

- $\tilde \zz_t$ is the first-order predictor for $\zz$, which I think is $\zz_t + h \dot \zz_t$
- $\MM$ is the mass matrix, so the first term is the kinetic energy
- $E_\text{pot}$ is the potential energy

### Degrees of freedom

Deformation map is parameterized using linear blend skinning:
$$
\phi(\XX, \zz) = \XX + \sum_j^n \WW_j(\XX) \ZZ_j \begin{pmatrix} \XX \\ 1 \end{pmatrix}
$$

- $n$ is number of skinning handles
- $\WW_j : \R^3 \to \R$ is shape function for handle $j$
- $\ZZ_j \in \R^{3 \times 4}$ is the skinning handle

then we use $\zz = \mathrm{flat}(\ZZ) \in \R^{12n}$, aka the reduced coordinates are the skinning handles data.

$\WW : \R^3 \to \R^n$ is all of the shape functions, and it is represented as a neural field

### Meshless integration

Mass matrix and potential energy are integrated quantities:
$$
\MM = \int_\Omega \rho \JJ(\XX)^\top \JJ(\XX) \d \Omega
$$
where $\JJ = \nabla_\zz \phi$ is the deformation Jacobian. This is constant since $\phi$ is linear, so $\MM$ is constant as well.
$$
E_\text{pot} = \int_\Omega \Psi(\phi(\XX)) \d \Omega
$$
where $\Psi$ is the strain energy density function.

These integrals are computed using Monte Carlo

### Neural skinning field loss

To learn $\WW$