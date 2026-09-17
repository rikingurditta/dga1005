# Subspace Neural Physics: Fast Data-Driven Interactive Simulation

Daniel Holden, Bang Chi Duong, Sayantan Datta, Derek Nowrouzezahrai

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
\newcommand{\norm}[1]{\left\lVert #1 \right\rVert}
\newcommand{\partials}[2]{\frac{\partial #1}{\partial #2}}
$$

![image-20260906152023649](/Users/rikin/Library/Application Support/typora-user-images/image-20260906152023649.png)

---

# Summary

A ROM is used to compress the motions of an input simulation, and the nonlinear dynamics of this simulation (including interactions) are learned using an MLP. A novel training method is proposed to enable learning of coherent trajectories across time.

---

## Training data

Training data is $10^5$ to $10^6$ frames of animations simulated by some other method

## Training

### Parameterization

For a simulation object mesh with $c$ vertices, each frame of training data is represented by a vector $\xx \in \R^{3c}$. Then vectors are concatenated into training matrix $\XX = \begin{pmatrix} \xx_1 & \cdots & \xx_n \end{pmatrix} \in \R^{3c \times n}$.

We can do the same for external objects, but use degrees of freedom instead of vertices. E.g. if we have two frictionless spheres, we can use $\yy = (y_1, y_2)$ as state for each frame since we have $e = 2$ degrees of freedom. We end up with another matrix $\YY \in \R^{e \times n}$.

Let $\xx_\mu$ and $\yy_\mu$ be the means of $\xx_i$ and $\yy_i$ respectively across all $n$ animation frames

We can then do PCA on $\XX$ to get subspace representations $\ZZ = \UU(\XX - \xx_\mu) \in \R^{u \times n}$ and $\WW = \VV (\YY - \yy_\mu) \in \R^{v \times n}$ where $u$ (typically = 256) is the size of the subspace aka the number of PCA vectors used, so $\UU \in \R^{u \times 3c}$. We use $v = e$ for the external objects and so $\VV \in \R^{v \times e} = \R^{e \times e}$ since we don't want to compress our external objects.

So each frame of subspace data is $\zz_t$ or $\ww_t$ respectively

### Initial model

Because of inertia, absent external forces we imagine that

$$
\zz_t = \aalpha \odot \zz_{t-1} + \bbeta \odot (\zz_{t-1} - \zz_{t-2})
$$

(where $\odot$ is component-wise multiplication and $\aalpha, \bbeta$ are some parameter vectors)

We can find $\aalpha$ and $\bbeta$ using linear least squares, and use them to calculate a first guess $\overline \zz_t$.

### Extended model

Since $\overline \zz_t$ does not take into account external objects, a neural network $\PPhi$ is used to figure out the residual effects to guess $\zz_t$:

$$
\zz_t = \overline \zz_t + \PPhi \left( \overline \zz_t, \zz_{t-1}, \ww_t \right)
$$

The paper uses 10 layer network with ReLU activation

### Training

Instead of training globally, train by predicting and correcting trajectories over small windows of frames (e.g. 16 or 32). Predict all motion across frames (+ noise) then correct, then move to next window. This leads to stable long term predictions and avoids training instability that comes from predicting large changes.
