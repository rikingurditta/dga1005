# LiCROM: Linear-Subspace Continuous Reduced Order Modeling with Neural Fields

Yue Chang, Peter Yichen Chen, Zhecheng Wang, Maurizio M. Chiaramonte, Kevin Carlberg, Eitan Grinspun

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
$$

## Discrete vs continuous

Typical reduced order modelling setup for a mesh:

$\uu(\XX, t) \in \R^{3n}$ is displacement map, and
$$
\uu = \overline \UU \qq
$$
Where $\qq$ is reduced model, aka reduced degrees of freedom, and $\overline \UU$ maps reduced coordinates to displacements of each vertex in $\R^{3n}$. i.e., $\overline \UU$ is tied up with the discretization of the mesh, where each row corresponds to a discretized vertex's coordinate.

Instead we can use a continuous map $\WW(\XX)$ that weights the reduced model DOFs to each point in the domain:
$$
\uu(\XX, t) = \WW(\XX) \qq(t)
$$
Now since the reduced model is discretization-independent, it can support multiple discretizations of the same geometry (e.g. different LOD meshes, or shapes that can get cut), or multiple geometries.

## Discretization-blind subspace learning

Input to training is point clouds $\tilde \XX$ and $\tilde \uu$, where subscript denotes point in point cloud and superscript denotes time step, so training set is $\{(\tilde \XX^1, \tilde \uu^1), \cdots, (\tilde \XX^m, \tilde \uu^m)\}$. Point clouds are unordered (so $\tilde \XX^j_i$ and $\tilde \XX^{j+1}_i$ don't need to correspond to the same point) and can be of different sizes (so $\abs{\tilde \XX^j}$ doesn't need to be equal to $\abs{\tilde \XX^{j+1}}$)

Need to find a projection $P: (\tilde \XX^j, \tilde \uu^j) \mapsto \qq^j$ which maps deformations into a subspace, and a basis $\WW$ such that
$$
\WW(\tilde \XX_i^j) P(\tilde \XX^j, \uu^j) \approx \uu_i^j
$$
This is similar to typical ROM, but uses $\WW$ whose domain is positions, rather than a matrix $\UU$ that acts on vertex indices.

The paper uses a PointNet encoder for $P$ and a neural implicit field for $\WW$

## Training

$\tilde n$ points are used for each observation point cloud $(\tilde \XX^j, \tilde \uu^j)$, and these are further subsampled to $\tilde{\tilde n}$ because PointNets are expensive for large sets of points. Subsampling operator is denoted $S$

Uses $L_2$ loss
$$
\mathcal L = \sum_j^m \sum_i^{\tilde n} \norm{ \WW(\XX_i) P(S((\tilde \XX^j, \tilde \uu^j))) - \uu_i^j }_2
$$

## Implicit dynamics

