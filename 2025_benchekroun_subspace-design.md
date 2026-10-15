# Force-Dual Modes: Subspace Design from Stochastic Forces

Otman Benchekroun, Eitan Grinspun, Maurizio Chiaramonte, Philip Allen Etter
$$
\newcommand{\ff}{\mathbf f}
\newcommand{\nn}{\mathbf n}
\newcommand{\rr}{\mathbf r}
\newcommand{\uu}{\mathbf u}
\newcommand{\vv}{\mathbf v}
\newcommand{\xx}{\mathbf x}
\newcommand{\zz}{\mathbf z}

\newcommand{\BB}{\mathbf B}
\newcommand{\HH}{\mathbf H}
\newcommand{\MM}{\mathbf M}

\newcommand{\bSigma}{\boldsymbol \Sigma}
\newcommand{\bLambda}{\boldsymbol \Lambda}

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

Constructs a linear simulation subspace taking into account arbitrary forces that could be applied to the object during simulation.

---

## Method

Want to build a subspace matrix $\BB$ that satisfies
$$
(\inv \HH \bSigma_F^\top \bSigma_F \inv \HH) \BB = \MM \BB \bLambda
$$

- $\HH$ is Hessian of energy
- $\bSigma$ is covariance of force distribution
- $\MM$ is mass matrix
- $\bLambda$ is eigenvalue matrix (diagonal)

### Statistical model reduction

Suppose we want to simulate
$$
U = \Loss (F)
$$

- $U$ is some quantity of interest (e.g. object displacements)
- $F$ is system input
- $\Loss$ is some complicated relationship between them

In the probabilistic setting, suppose $U$ and $F$ are random variables

We can do model reduction if we assume that $U$ has an underlying low-dimensional structure, e.g. if $U$ is linear/affine then
$$
U = \BB Z + \boldsymbol \mu_U
$$
($\boldsymbol \mu_U$ is just some mean term)

Given $\BB$, the optimal reconstruction of $Z$ (in the $\MM$-norm) is
$$
Z = \BB^\dagger (U - \boldsymbol \mu_U)
$$
where $\BB^\dagger = \inv{(\BB^\top \MM \BB)} \BB^\top \MM^\top$ (pseudoinverse of $\BB$)

The goal of model reduction is to find $\BB$ so that $Z$ and $\BB$ can model $U$ well, i.e. to minimize
$$
\min_\BB \mathbb E \norm{U - \BB \BB^\dagger U}_\MM^2
$$
This problem is underdetermined unless we also constrain $\BB^\top \MM \BB = \mathbf I$

Then, using the covariance matrix $\bSigma_U$, we know from appendix A that the solution is $\BB = $ the top $m$ eigenvectors of $\bSigma_U \MM$, i.e. $\BB$ satisfies
$$
\inv \bSigma_U \BB = \MM \BB \inv \bLambda
$$
