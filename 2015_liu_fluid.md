# Model-Reduced Variational Fluid Simulation

Beibei Liu, Gemma Mason, Julian Hodgson, Yiying Tong, Mathieu Desbrun

$$
\newcommand{\ff}{\mathbf f}
\newcommand{\xx}{\mathbf x}
\newcommand{\zz}{\mathbf z}
\newcommand{\Loss}{\mathcal L}
\DeclareMathOperator*{argmin}{argmin}
\newcommand{\abs}[1]{\left\vert #1 \right\vert}
\newcommand{\norm}[1]{\left\lVert #1 \right\rVert}
\newcommand{\partials}[2]{\frac{\partial #1}{\partial #2}}
\newcommand{\inv}[1]{#1^{-1}}
\newcommand{\d}{\, \mathrm{d}}
$$

---

# Summary



---

## Recap of Variational Eulerian Integration

A continuous function $f(\xx)$ is discretized into a vector $\ff$, where $f_i$ is the average (integrated) value of $f$ on grid cell $i$

Then the set of possible flows $\phi_t$ can be discretized using a functional map (Koopman operator) to model (?) $(f \circ \inv \phi_t)(\xx)$

i.e., there is some matrix $q$ so that $q\ff$ computes $\ff$ advected through the flow $\phi$

The constant function should not change under advection, so we know that $q\mathbf 1 = \mathbf 1$, which tells us that $q$ is a signed stochastic matrix, i.e. each row of $q$ sums to $1$.

Incompressibility also implies preservation of volume, which implies that for $\xx, \yy$, we need $q\xx \cdot q\yy = \xx \cdot \yy$, which means that $q$ is orthogonal, i.e. $q^\top = \inv q$

so $q \in G$, the Lie group of orthogonal signed-stochastic matrices

### Eulerian Lie algebra

$G$ parameterizes the space of possible "positions" of the fluid, as any element of $G$ could represent a way the fluid evolved from its initial position

This is a Lagrangian perspective because it describes how "particles" of the fluid move around

The Eulerian perspective comes from the Lie algebra $\mathfrak g$ associated with $G$, where $A \in \mathfrak g$ can be written as $\dot q \circ \inv q$ for some $q \in G$. We know that $A^\top = -A$ and $A \mathbf 1 = \mathbf 0$, and corresponds to the Lie derivative $L_v$ of the continuous velocity field $v = \dot \phi \circ \inv \phi$

Thus $A \ff$ approximates $v \cdot \nabla f$, and $A_{ij}$ represents flux across cell $i$ to cell $j$ (if they are adjacent)

### Non-holonomic constraint

In typical simulation, we use CFL to make sure we don't simulate fluid "skipping" through cells, to try to make sure we are modelling flux between adjacent cells. To do that in this setting, we restrict our view to the set of matrices $\mathcal S = \{A_{ij} = 0 \text{ if } i, j \text{ don't share a cell face} \}$. This also gives us sparsity and makes our simulations correspond to traditional MAC fluid sim.

However, $\mathcal S$ is not closed under Lie bracket!

### Creating a variational numerical method

The Lagrangian is

$$
\Loss_\text{Euler} = \frac{1}{2} \langle A, A \rangle \approx \frac{1}{2} \int v^2 \d x
$$

## Model-reduced Variational Integrator

### Spectral bases

The 3-form Laplacian $\Delta_3$ has eigenfunctions $\Phi_i$ with eigenvalues $-\mu_i^2$:
$$
\Delta_3 \Phi_i = -\mu_i^2 \Phi_i
$$
Then we can assemble the eigenfunctions with smallest eigenvalues into a reduced, low-frequency basis
$$
\{ \Phi_0, \cdots, \Phi_{M_3} \}
$$
Similarly, for the 2-form Laplacian we have
$$
\Delta_2 \Psi_i = -\kappa_i^2 \Psi_i
$$
with a low-frequency basis
$$
\{ \Psi_0, \cdots, \Psi_{M_2} \}
$$
Some $\Psi$ are not div-free, and these ones can be identified as gradient fields $\Psi_i = \frac{\nabla \Phi_j}{\mu_j}$
