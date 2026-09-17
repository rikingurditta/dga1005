# Fast Subspace Fluid Simulation with a Temporally-Aware Basis

Siyuan Chen, Yixin Chen, Jonathan Panuelos, Otman Benchekroun, Yue Chang, Eitan Grinspun, Zhecheng Wang

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

## Contributions

- introduce DMD: dynamic mode decomposition
  - fast control of fluid simulations
  - high accuracy, low time/memory cost
- spatiotemporal nature of DMD enables control of wave modes
- demonstrate DMD's versatility
  - frequency editing
  - time reversal
  - super resolution
  - simulation styling

## Koopman operator and DMD

Consider:

$$
\dbyd{\uu}{t} = \ff(\uu)
$$

i.e., $\ff : \R^N \to \R^N$ is not dependent on time

Let $n$ be the total spatial degrees of freedom. Consider "observables" $g_1, ..., g_n : \R^N \to \R^N$ with associated points $\xx_1, ..., \xx_n$, and define

$$
\gg(\uu(t))
= \begin{pmatrix} g_1(\uu(t)) \\ g_2(\uu(t)) \\ \vdots \\ g_n(\uu(t)) \end{pmatrix}
= \begin{pmatrix} \uu(\xx_1, t) \\ \uu(\xx_2, t) \\ \vdots \\ \uu(\xx_n, t) \end{pmatrix}
\in \R^{Nn}
$$

Suppose we take $T \leq n+1$ observations of $\gg(\uu)$ at different times, $\gg_1, ..., \gg_T$.

Then there exists an operator $\KK$ that applies the linearized time step:

$$
\gg_{k+1} = \KK \gg_k
$$

This is guaranteed to exist due to [mumbling]
