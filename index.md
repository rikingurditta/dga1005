$$
\newcommand{\bb}{\mathbf b}
\newcommand{\rr}{\mathbf r}
\newcommand{\qq}{\mathbf q}
\newcommand{\uu}{\mathbf u}

\newcommand{\MM}{\mathbf M}
\newcommand{\UU}{\mathbf U}

\newcommand{\bpsi}{\boldsymbol \psi}
$$

# DGA 1005 notes

| Title                                                                                                                           | Authors               | Year | Notes |
| :------------------------------------------------------------------------------------------------------------------------------ | :-------------------- | :--- | :---- |
| [Model Reduction for Real-time Fluids](2006_treuille_fluids)                                                                    | Treuille et al.       | 2006 |       |
| [Model-Reduced Variational Fluid Simulation](2015_liu_fluid)                                                                    | Liu et al.            | 2015 |       |
| [Latent-space Dynamics for Reduced Deformable Simulation](2019_fulton_latent-space)                                             | Fulton et al.         | 2019 |       |
| [Subspace Neural Physics: Fast Data-Driven Interactive Simulation](2019_holden_subspace-neural)                                 | Holden et al.         | 2019 |       |
| [Deep Fluids: A Generative Network for Parameterized Fluid  Simulations](2019_kim_deep-fluids)                                  | Kim et al. 2019       | 2019 |       |
| [LiCROM: Linear-Subspace Continuous Reduced Order Modeling with Neural Fields](2023_chang_LiCROM.md)                            | Kim et al.            | 2023 |       |
| [PolyStokes: A Polynomial Model Reduction Method for Viscous Fluid Simulation](2023_panuelos_polystokes)                        |                       |      |       |
| [Accelerate Neural Subspace-Based Reduced-Order Solver of Deformable Simulation by Lipschitz Optimization](2024_lyu_accelerate) | Lyu et al.            | 2024 |       |
| [Simplicits: Mesh-Free, Geometry-Agnostic, Elastic Simulation](2024_modi_simplicits)                                            | Modi et al.           | 2024 |       |
| [Neural Implicit Reduced Fluid Simulation](2024_tao_NIRFS)                                                                      | Tao et al.            | 2024 |       |
| [Fast Subspace Fluid Simulation with a Temporally-Aware Basis](2025_chen_subspace-fluid)                                        | Chen et al.           | 2025 |       |
| [Shape Space Spectra](2025_chang_shape-space-spectra)                                                                           | Chang et al.          | 2025 |       |
| [FreeForm: Reduced-Order Deformable Simulation from Particle-Based Skinning Eigenmodes](2026_xiang-modi_freeform)               | Xiang and Modi et al. | 2026 | TODO  |

## todo

- address unfinished notes (with "TODO" in list)
- add missing summaries
- look at and compare results for various methods

## other papers

| Title                                 | Authors     | Year | Notes |
| :------------------------------------ | :---------- | :--- | :---- |
| [Vertex Block Descent](2024_chen_VBD) | Chen et al. | 2024 |       |

## summary of reading

### reduced order modelling

for a system dealing with a PDE in $\uu \in \R^n$, aka with $n$ degrees of freedom, we want to find a reduced representation $\qq \in \R^m$ with $m \ll n$, and reformulate the dynamics of our PDE in terms of $\qq$, so that we can instead simulate $\qq$.

### classical model reduction

classical model reduction typically uses PCA on a dataset of valid states to determine linear degrees of freedom

so we use PCA to find an orthogonal basis $\UU = \begin{pmatrix} \bb_1 & \cdots & \bb_m \end{pmatrix}$ so that we can approximate $\uu \approx \UU \qq$

then equations of motion are computed using various methods.

- for elasticity, we can compute inertia with a modified mass matrix $\hat \MM = \UU^\top \MM \UU$ and compute potential energy using existing energy computation $E(\UU \qq)$. using these we can take a typical variational implicit time step.
	- [Fulton et al. 2019](2019_fulton_latent-space) explain this in background, but maybe should find a primary source
- for fluids, each basis vector $\bb_i$ satisfies constraints, so fluid reconstructed from this basis will also satisfy constraints. diffusion operator can be projected to subspace $\UU^\top \mathbf D \UU$, and  advection operator can be linear combination of basis advections
	- TODO: read on changing constraints

[Holden et al. 2019](2019_holden_subspace-neural) uses PCA to reduce model then uses neural network to advance it through time rather than variational time integration

### neural model reduction

instead of using PCA to find a linear basis, each reduced degree of freedom in $\qq$ can represent a nonlinear motion as represented by a neural network

this is first done in graphics by [Fulton et al. 2019](2019_fulton_latent-space), where subspace is learned by autoencoder since this is essentially a dimensionality reduction task: we find decoder $\bpsi$ and encoder $\overline \bpsi$ so that $\qq = \overline \bpsi$ and $\bpsi(\qq)$ represents $\uu$ well, i.e. to minimize $\norm{\uu - \bpsi(\overline \bpsi(\uu))}^2$.
- since PCA is an already-known method to find most significant variations in $\uu$, we use PCA for the outermost (or innermost) layer of $\bpsi$ (or $\overline \bpsi$). but using neural networks for $\bpsi, \overline \bpsi$ still gives us good nonlinearity to represent nonlinear degrees of motion.
- use cubature + L-BFGS optimization to let us compute variational time integration
	- to speed up this integration computation, use [Lyu et al. 2024](2024_lyu_accelerate) to find equivalent $\bpsi$ which induces smaller Lipschitz constant for energy computation

LiCROM 