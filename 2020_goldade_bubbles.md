# Constraint Bubbles and Affine Regions: Reduced Fluid Models for Efficient Immersed Bubbles and Flexible Spatial Coarsening

Ryan Goldade, Mridul Aanjaneya, Christopher Batty

$$
\newcommand{\ff}{\mathbf f}
\newcommand{\nn}{\mathbf n}
\newcommand{\rr}{\mathbf r}
\newcommand{\uu}{\mathbf u}
\newcommand{\vv}{\mathbf v}
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

## Constraint bubbles

$\Omega_A, \Omega_S, \Omega_L$ refer to air, solid, and liquid regions respectively, the components of $\Omega_A$ are bubbles (exterior air counts as a "bubble")

Each bubble should preservie its volume, which can can be expressed in terms of velocity as:
$$
\iint_{\partial \Omega_{A_i}} \uu_A \cdot \nn \d A = 0
$$
Separating into liquid and solid parts,
$$
\iint_{\Omega_L \cap \partial \Omega_{A_i}} \uu \cdot \nn \d A = 0
\qquad \text{and} \qquad
\iint_{\Omega_S \cap \partial \Omega_{A_i}} \uu_S \cdot \nn \d A = 0
$$
$B_i(\uu)$ (which is linear) denotes the contribution of the liquid surface to the $i$th bubble, and $b_{S_i}$ denotes the solid's contribution. We can collect the $B_i$ and $b_{S_i}$ into vectors $\mathbf B$ and $\mathbf b$. Note that no information inside the bubble is needed

Then PDE becomes
$$
\begin{align*}
\rho \partials{\uu}{t} &= -\nabla p - \partials{\mathbf B}{\uu}^\top \boldsymbol \lambda \\
\nabla \cdot \uu &= 0 \\
\mathbf B(\uu) &= -\mathbf b_S
\end{align*}
$$
($\boldsymbol \lambda$ is the vector of Lagrange multipliers)