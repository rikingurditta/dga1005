# Shape Space Spectra

Yue Chang, Otman Benchekroun, Maurizio M. Chiaramonte, Peter Yichen Chen, Eitan Grinspun

$$
\newcommand{\ff}{\mathbf f}
\newcommand{\gg}{\mathbf g}
\newcommand{\qq}{\mathbf q}
\newcommand{\uu}{\mathbf u}
\newcommand{\xx}{\mathbf x}
\newcommand{\yy}{\mathbf y}
\newcommand{\zz}{\mathbf z}
\newcommand{\bphi}{\boldsymbol \phi}
\newcommand{\Loss}{\mathcal L}
\DeclareMathOperator*{argmin}{argmin}
\newcommand{\parens}[1]{\left( #1 \right)}
\newcommand{\curlies}[1]{\left\{ #1 \right\}}
\newcommand{\abs}[1]{\left\vert #1 \right\vert}
\newcommand{\norm}[1]{\left\lVert #1 \right\rVert}
\newcommand{\dbyd}[2]{\frac{\mathrm{d} #1}{\mathrm{d} #2}}
\newcommand{\partials}[2]{\frac{\partial #1}{\partial #2}}
\newcommand{\inv}[1]{#1^{-1}}
\newcommand{\d}{\,\mathrm{d}}
$$

---

## Summary

Eigenanalysis method for continuously parameterized shape families

---

## Eigenanalysis of a single shape

### Variational perspective

Suppose $\Omega \subseteq \R^n$ is compact with piecewise-smooth boundary $\partial \Omega$

We are concerned with the eigenfunctions of the Laplace operator $\Delta$ on the space of functions on $\Omega$.

Let $u: \Omega \to \R$ be a (sufficiently) smooth function, then:

$$
\Delta u = \nabla \cdot \nabla u = \sum_i \partials{^2 u}{x_i^2}
$$

$\phi_1(\xx) \in \mathcal U$ is the dominant eigenfunction if it minimizes the Dirichlet energy;

$$
E_D[\phi] = \frac{1}{2} \int_\Omega \abs{\nabla \phi} \d \Omega
$$

where $\mathcal U = \curlies{ f \in L^2(\Omega) : \norm{f}_2 = 1 }$. Using this space, we find the associated eigenvalue $\lambda_1 = E_D[\phi_1]$.

$\phi_2$ (with eigenvalue $\lambda_2 \geq \lambda_1$) minimizes $E_D$ but on the orthogonal complement $\mathrm{span}\{\phi_1\}^\perp \subseteq \mathcal U$, and similarly for the next-subdominant eigenfunctions, i.e.

$$
\phi_i \text{ minimizes } E_D \text{ on } C_i = \mathrm{span}\{\phi_1, ..., \phi_{i-1}\}^\perp
$$

with $\lambda_i \geq \lambda_{i-1}$.

All of these functions satisfy the natural boundary condition:

$$
\partials{\phi_i}{\mathbf n} = 0 \text{ on } \partial \Omega
$$

### Implementation with neural fields

$$
\phi_i = \mathcal P_i \circ \overline \phi_i
$$

- $\overline \phi_i : \Omega \to \R$ is a neural field (MLP)
- $\mathcal P_i : L_2 \to \mathcal U \cap C_i$ projects field to satisfy constraints

Neural fields are trained using $\Loss = E_D$, using stochastic cubature to estimate the integral:

$$
\tilde \Loss = \tilde E_D[\phi] = \sum_{\xx \in \mathcal X} \abs{\nabla \phi(\xx)}^2
$$

$\mathcal P_i$ is computed using Gram-Schmidt to orthogonalize, then dividing by norm to normalize. (Both steps use stochastic cubature to estimate norms.)

#### Elasticity

Work above has been for Laplace operator, but can apply it to elasticity by simply replacing Dirichlet energy with elastic energy:

$$
E_e[\bphi] = \frac{1}{2} \int_\Omega \mu \abs{\nabla \bphi + \nabla \bphi^\top}^2_F + \frac{\lambda}{2} \mathrm{Tr}^2 \parens{\nabla \bphi + \nabla \bphi^\top} \d\Omega
$$

## Eigenanalysis over shape space

Let $\mathcal D$ be shape space, then the domain is $\curlies{ \Omega^\gg : \gg \in \mathcal D }$ aka the geometries are parameterized by the shape space. Then we can consider eigenfunctions $\phi^\gg_i$ also parameterized by shape space

### Variational perspective

For single shape, $\phi_1$ minimizes $E_D[\phi]$. Could we generalize this by minimizing over shape space?

i.e.,

$$
\phi_1 = \argmin \int_\mathcal D E_D[\phi^\gg] \d \gg
$$

and then $\phi_2$ is the minimizer over $\mathrm{span}\{\phi_1\}^\perp$, etc

This causes weirdness across shape space and in training, so we don't do it. The core problem is the ordered nature of this way of doing things, so instead we jointly consider the eigenfunctions:

$$
\argmin_{\phi_1, \cdots, \phi_k} \sum_i \int_\mathcal D E_D[\phi_i^\gg] \d \gg \quad \text{(subject to "orthogonality")}
$$

### Implementation with neural fields

- Loss is computed using cubature
- uses "gradient causal filtering" to optimize orthogonality so that eigenfunctions aren't "pushing back" on each other
