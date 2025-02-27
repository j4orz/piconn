# Software 2.0 Math Reference

*A famous colleague once sent an actually very well-written paper he was quite proud of to a famous complexity theorist. His answer: “I can’t find a theorem in the paper. I have no idea what this paper is about.*

**Contents**
1. [Models](#models)
    - [Linear Models](#linear-models)
    - [Non-linear Parametric Models](#non-linear-parametric-models)
    - [System 1 Learning](#system-1-sequence-learning-as-autoregression)
    - [System 2 Search](#system-2-search)
    - [Non-linear Non-Parametric Models](#non-linear-non-parametric-models)
2. [Optimization](#optimization)
    - [SGD](#sgd)
    - [Adam](#momentum-adam)
    - [Shampoo](#preconditioining-shampoo)
3. [Generalization](#generalization)

# 1. Models

## Linear Models

**Classification**: recovers discriminative classifier $p(Y=c|\mathbf{X}=\mathbf{x}; \boldsymbol{\theta})$ where $Y \overset{\text{iid}}{\sim} Ber(p)$. First we start off with binary classification so $\mathcal{Y}=\{0,1\}$. The logistic regression (better thought of as sigmoidal classification) assumption is to take a linear hypothesis $h_{\boldsymbol{\theta}}(\mathbf{x})$ defined as so:

$$
\begin{align*}
h_{\boldsymbol{\theta}}&: \mathbb{R}^{d} \to [0,1] \\
h(\mathbf{x};\mathbf{\boldsymbol{\theta}})&:= \sigma(\boldsymbol{\theta}^{\top}\mathbf{x}) = \frac{1}{1+\text{exp}({-\boldsymbol{\theta}^{\top}\mathbf{x}})}
\end{align*}
$$

which typechecks since $\boldsymbol{\theta}^{\top} \in \mathbb{R}^{1,d}$ (so $\boldsymbol{\theta}^{\top}: \mathbb{R}^{d} \to \mathbb{R}$) and $\sigma: \mathbb{R} \to [0,1]$, and interpret it as $p$, where the final map $\sigma: \mathbb{R} \to [0,1]$ takes log odds $\frac{p}{1-p}$. That is,

$$
\begin{align*}
p(Y=1|X=x;\boldsymbol{\theta}) &:= \sigma(\boldsymbol{\theta}^{\top}\boldsymbol{x}) \tag{logistic regression assumption} \\
\implies p(Y=0|X=x;\boldsymbol{\theta}) &= 1- \sigma(\theta^{\top}\mathbf{x}) \tag{total law of prob} \\
\end{align*}
$$

which we can formulate as a continuous and differentiable probability density function $p(Y=y|X=x) = [\sigma(\theta^{\top}\mathbf{x})]^y[1-\sigma(\theta^{\top}\mathbf{x})]^{1-y}$. Two things to note:

1. Do not confuse the 0/1 in $\mathcal{Y}=\{0,1\}$ with the 0/1 in $h_{\theta}: \mathbb{R}^{d} \to [0,1]$. The former is the sample space mapped by random variable $Y:\Omega \to \mathbb{N}$, whereas the latter is the probability measure assigned to that event conditioned on $X$. We could change the label encodings to be $\mathcal{Y}=\{-1,1\}$. Also, multiclass classification has $\mathcal{Y}=\{0,1,...,k\}$ with the same type $h_{\theta}: \mathbb{R}^{d} \to [0,1]$.
2. Do not confuse parameter $p$ with parameters $\boldsymbol{\theta}$. It's helpful to conceptualize the former decomposing into the latter. $;$ is used to denote the notion of "parameterized by", which is not a 1-1 substitution.
3. We prepend vector $\mathbb{x} \in \mathbb{R}^d$ with $\mathbb{x}_0 := 1$, $\mathbb{x} \in \mathbb{R}^{d+1}$ so that $\boldsymbol{\theta}^{\top}\mathbf{x}$ has a bias term.


Then, taking the maximum likelihood estimate (MLE) of the negative log-likelihood yields:

$$
\begin{align*}

\hat{\mathbf{\theta}} &\in \underset{\boldsymbol{\theta} \in \boldsymbol{\Theta}}{\mathop{\text{argmin}}} -\log \mathcal{L}(\boldsymbol{\theta}) \\
 &\in \underset{\boldsymbol{\theta}}{\mathop{\text{argmin}}} -\log \prod_{i=1}^n p(y^{(i)}|\mathbf{x}^{(i)};\boldsymbol{\theta}) \tag{conditional prob after Y iid} \\
 &\in \underset{\boldsymbol{\theta}}{\mathop{\text{argmin}}} -\log \prod_{i=1}^n [\sigma(\boldsymbol{\theta}^{\top}\mathbf{x}^{(i)})]^{y^{(i)}}[1-\sigma(\boldsymbol{\theta}^{\top}\mathbf{x}^{(i)})]^{1-y^{(i)}} \tag{logistic regression assumption}\\
 &\in \underset{\boldsymbol{\theta}}{\mathop{\text{argmin}}} -\sum_{i=1}^n y^{(i)}\log \sigma(\boldsymbol{\theta}^{\top}\mathbf{x}^{(i)}) + (1-y^{(i)})\log (1-\sigma(\boldsymbol{\theta}^{\top}\mathbf{x}^{(i)})) \tag{log laws} \\
 &\in \underset{\boldsymbol{\theta}}{\mathop{\text{argmin}}} -\sum_{i=1}^n y^{(i)}\log \hat{y}^{(i)} + (1-y^{(i)})\log (1-\hat{y}^{(i)}) \tag{$\hat{y}^{(i)} = \sigma(\boldsymbol{\theta}^{\top}\mathbf{x}^{(i)})$}\\
&\in \underset{\boldsymbol{\theta}}{\mathop{\text{argmin}}} \sum_{i=1}^n \mathbb{H}_{ce}(y^{(i)}, \hat{y}^{(i)}) \tag{binary cross entropy}\\

\end{align*}
$$

which is the cross entropy between the data and predictions where
1. $\hat{p} := \hat{\boldsymbol{\theta}}$ (the estimators are 1-1)
2. $\mathcal{L}$ is used to denote likelihood, whereas the negative log likelihood $-log\mathcal{L}$ is the entire loss function.
3. $\in$ is used rather than $=$ to denote the potential existence of multiple optima, even though in this case, the solution is unique since the log-likelihood function is convex. Finding *the* optimum was more important back when the field was dominated by statistical learning methods, but, this has became less important over time with the empirical success of non-linear models such as deep neural networks.

Massaging the negative log likelihood into its cross entropy form is nice because it provides secondary motivation for maximum likelihood estimation from an information theoretical perspective. What the loss function effectively computes is the distance between the empirical distribution and the predicted distribution.

TODO: KL-divergence. exponential family?

While $\mathop{\text{argmin}}$ is usually implemented algorithmically (as opposed to numerically or symbolically) via automatic differentiation, we will save autograd for deep neural networks, and for now, derive the gradient manually via symbolic term rewriting. 

$$
\begin{align*}
\nabla_{\boldsymbol{\theta}}\text{NLL}(\boldsymbol{\theta}) &= \nabla_{\boldsymbol{\theta}}[-\sum_{i=1}^n y^{(i)}\log \hat{y}^{(i)} + (1-y^{(i)})\log (1-\hat{y}^{(i)})] \tag{$\hat{y}^{(i)} = \sigma(\boldsymbol{\theta}^{\top}\mathbf{x}^{(i)})$} \\
&= -\sum_{i=1}^n \nabla_{\boldsymbol{\theta}}[y^{(i)}\log \hat{y}^{(i)} + (1-y^{(i)})\log (1-\hat{y}^{(i)})] \tag{$\nabla$ is linear} \\
&= -\sum_{i=1}^n \nabla_{\boldsymbol{\theta}}[y^{(i)}\log \hat{y}^{(i)} + (1-y^{(i)})\log (1-\hat{y}^{(i)})] \tag{} \\

\end{align*}
$$

so $\theta^{t+1} := \theta^{t} - \eta \nabla_{\theta} \text{NLL}(\theta)$.

TODO: manually derive gradient.






general def of gradient. linearizatin. direction. inner product. points in direction of steepest descent.

Finally, because logistic regression is selecting a stochastic map by producing parameters for $\mathcal{Y}$'s distribution, sampling a point estimate from the distribution (confusingly called inference in machine learning literature) is done by evaluating $\hat{y} := \underset{y}{\mathop{\text{argmax}}}[p(y|\mathbf{x}; \boldsymbol{\theta})]$ Since we have a batch of inputs from data $\{\mathbf{x}^{(i)}, \mathbf{y}^{(i)}\}_{i=1}^n$, instead of sequentially sampling the model (distribution) for each input $\mathbf{x}^{(i)}$, parallel sampling is possible with a design matrix $\mathbf{X} \in \mathbb{R}^{n \times d}$:

$$
\mathbf{X} = \begin{bmatrix}
\mathbf{x}^{(1)}_{1} & \mathbf{x}^{(1)}_{2} & \cdots & \mathbf{x}^{(1)}_{d} \\
\mathbf{x}^{(2)}_{1} & \mathbf{x}^{(2)}_{2} & \cdots & \mathbf{x}^{(2)}_{d} \\
\vdots & \vdots & \vdots & \vdots \\
\mathbf{x}^{(n)}_{1} & \mathbf{x}^{(n)}_{2} & \cdots & \mathbf{x}^{(n)}_{d}
\end{bmatrix}
$$

and evaluate the samples for all inputs with a single matrix multiply $\mathbf{y} = \mathbf{X}\boldsymbol{\theta}$ which swaps the order of data and parameters since the shape of design matrix is $\mathbb{R}^{n \times d}$ rather than $\mathbb{R}^{d \times n}$.



For multi-class classification, multinouilli, softmax is the generalization of the sigmoid.

**Regression**: recovers discriminative model $p(Y=y|\mathbf{X}=\mathbf{x}; \theta)$ where $Y \overset{\text{iid}}{\sim} Nor(\mu, \sigma^2).$ The linear regression assumption is to take a linear hypothesis $h_{\theta}(\mathbf{x})$ defined as

$$
\begin{align*}
h_{\theta}&: \mathbb{R}^{d} \to \mathbb{R} \\
h_{\theta}(\mathbf{x})&:= \theta^{\top}\mathbf{x}
\end{align*}
$$

**Generalized linear models**

## Non-linear Parametric Models

**Neural Networks (NN)**: To motivate the functions that neural networks implement, the biological inspiration is ignored in favor of the mathematical specification. The previous family of functions that we considered as hypotheses were linear models, so the obvious idea is to come up with a non-linear function class $h(\mathbf{x}; \mathbf{w}, \mathbf{b}) := W\phi(\mathbf{x}) + \mathbf{b}$. Network design started out with manual feature engineering of $\phi: \mathbb{R}^{d_i} \to \mathbb{R}^{d_{i+1}}$, and evolved into deep learning: the construction of networks with  end-to-end representation learning of all feature extractors $\phi$.

This is done by recursively composing the non-linear functions with the aim of "lifting" the *representation* of the data into more complex "motifs". A loose analogy is to conceptualize a neural network with multiple compositions (layers) as a function which parses the representation of the data [(Olah 2015)](https://colah.github.io/posts/2015-09-NN-Types-FP):

$$
\begin{align*}
f&: \mathbb{R}^{d_0} \to \mathbb{R}^{d_L} \\
f(\mathbf{x}; \mathbf{w}) &:= W_L \circ (\phi \circ W_{L-1}) \circ \cdots \circ (\phi \circ W_1) \circ \mathbf{x}
\end{align*}
$$

where each linear + non-linear function composition is a *hidden layer*:

$$
\begin{align*}
h^{(1)}&:= \phi(W^{(1)}\mathbf{x} + \mathbf{b}^{(1)}) \\
h^{(2)}&:= \phi(W^{(2)}h^{(1)} + \mathbf{b}^{(2)}) \\
h^{(3)}&:= \phi(W^{(3)}h^{(2)} + \mathbf{b}^{(3)}) \\
\vdots \\
h^{(L)}&:= W^{(L)}h^{(L-1)} + \mathbf{b}^{(L)}

\end{align*}
$$

These networks are optimized with gradient descent, but the gradient is derived with algorithmic methods rather than symbolic or numeric — the latter two methods are infeasible in practice with respect to the optimization of deep neural networks. Autodifferentiation applies the chain rule via dynamic programming on a graph $G=\{V,E\}$ where the vertices are operations and the edges are compositional derivatives. Graphs are the primary representations for compiler construction — the difference between a graph for a C program and a PyTorch program is that the latter does not have loads/stores, or branches, which is amenable to polyhedral compilation.

*Forward Mode Differentiation*

*Reverse Mode Differentiation*

While the optimization and generalization of other non-linear models such as kernel methods and gaussian processes are formally well-understood (with functional analysis and bayesian probability, respectively), the primary method of inquiry for neural networks have been empiricism. There are many interesting problems that are open for the theoretician, but for now we proceed in this document with the understanding that the state of deep learning is more similar to alchemy than it is to chemistry.

With that said, the domain of modelling we are interested in is language, which has converged onto autoregressive modeling. Following [(Sutton 2019)](http://www.incompleteideas.net/IncIdeas/BitterLesson.html), system 1 like behavior is covered with autoregressive sequence learning, and system 2 like behavior is covered with a search of thoughts. Each advance in network architecture will be covered in order to understand the network design that went into making high dimensional distribution estimation tractable.






## System 1 Sequence Learning as Autoregression

Any sequence — whether text for language, pixels for vision, or waves for audio — can be modelled probabilistically with $(\Omega, \mathcal{F}, \mathbb{P})$ where

$$
p(X_1=x^{(1)}, \ldots, X_n=x^{(n)}; \boldsymbol{\theta}) = \prod_{i=1}^{n} p(\mathbf{x}^{(i)}|\mathbf{x}^{(< i)})
$$

is the n-dimensional joint probability distribution expressed with conditionals via factorization of the chain rule. We can also represent any joint distribution as a probabilistic graphical model which is encoded as a DAG $G=\{V, E\}$ where vertices represent random variables, and edges represent conditional distributions — this is called a *bayesian network*. Since the graph is encoding a high dimensional joint distribution, each vertex is a graph itself (called a probability table) in order to capture all conditional dependencies. Unfortunately, the parameter sizes of these joint probability distributions (and their graphical representations) grow exponentially with respect to $n$. This is known as the "curse of dimensionality".

For instance, if $|\Omega|=1e5$, then capturing the distribution of a sequence with context length 100 requires parameterizing a model with $(1e5)^{100}$ possible combinations, which is already higher than the number of atoms in the observable universe. Following LeCun's cake philosophy, these high dimensional distributions fall under the regime of unsupervised learning, since it's intractable to

1. provide an exponential amount of ground truth labels let alone
2. the compute needed to train that many parameters.

The next four sections will cover advances in network architecture to *fight* against the curse of dimensionality's exponential blow up. The order of advances is presented in a post-hoc logical ordering, even though historically recurrence was the predominant design choice for natural language processing before attention.

1. Sparsification of conditionals
2. Parameterization conditionals
3. Causally mask conditionals
4. Infinite context window






### 4 advances in network design contra curse of dimensionality
*1. Sparsify conditionals with bayesian network*: instead of conditioning on all the previous events (tokens), bayesian networks condition on a select few — namely, a few parents. This effectively adds causal assumptions as an inductive bias since the model does not need to learn those causal relations which can be great if the assumptions hold. However, with respect to large corpora such as wikipedia, common crawl, etc, these assumptions do not hold and end up limiting expressivity.

$$
p(X_1=x_1, \ldots, X_n=x_n; \boldsymbol{\theta}) = \prod_{i=1}^{n} p(\mathbf{x}_{i}|\mathbf{x}_{parents(i)})
$$



*2. Parameterize conditionals with neural networks*
learning a distributed representation for words allows each
training sentence to inform the model about an exponential number of semantically neighboring sentences.

When modeling continuous variables, we obtain generalization more easily (e.g. with smooth classes of functions like multi-layer neural networks or
Gaussian mixture models) because the function to be learned can be expected to have some local smoothness properties. 

discrete space, most observed objects are almost maximally far from each other in hamming distance

p(w_n|w_i<n) approx p(w_n|w_;i=n-L, i<n)
with ngrams, What happens when a new combination of n words appears
that was not seen in the training corpus?

Obviously there is much more information in the sequence that
immediately precedes the word to predict than just the identity of the previous couple of words

First, it is not taking into account contexts farther than 1 or 2 words,1
second it is not taking into account the “similarity” between words.


approximation of joint distribution
 The probabilistic graphical model can be simulated with a neural network, as shown in
 connections maximized. but no exp blow up because we use parameters in R^d to represent conditionals. no exp blow up with discrete table counting.

bayesian network with connections maximized. fnn which satisfies chain rule. expressive, but not enough parameter sharing for efficient training.

autoencoder + mask = autoregressive model
 
 
 1. [(Bengio, Bengio 1999)](https://papers.nips.cc/paper_files/paper/1999/file/e6384711491713d29bc63fc5eeb5ba4f-Paper.pdf)
 2. [(Larochelle, Murray 2011)](https://proceedings.mlr.press/v15/larochelle11a/larochelle11a.pdf)
 3. [(Germain et al. 2015)](https://arxiv.org/abs/1502.03509)
 - order agnostic
 - connectivity agnostic
 (more robust)








*3. Causal masks with neural networks*
- still parameterizing conditionals 
+ parameter sharing across conditionals
+ add coordinate coding to individualize conditionals
any buts?  finite context window? quadratic scaling?

Masking
1. convolutional (position) [(Oord et al. 2016)](https://arxiv.org/abs/1609.03499)
    - problem: limited receptive field (hard to capture long-range deps)
2. attention (context) [(Vaswani et. al 2017)](https://arxiv.org/abs/1706.03762)
    - unlimited receptive field
    - O(1) param scaling wrt data dim






*4. Infinite look back*
rnn (mikolov et al. 2010)
expressive, but
- not as amenable to parallelization
- gradients explode/vanish
- hard to truly have signal propagate from long history
- if you think about it, the horizontal connections aren't actually that more expressive. handwavy...

## System 2 Search:

- see the emergent behavior https://x.com/nrehiew_/status/1881792577122586869






## Non-linear Non-Parametric Models
**Kernel Methods**

**Gaussian Processes**

# 2. Optimization

**SGD**
The general definition of the derivative...important for understanding different optimizers that use momentum and preconditioning techniques. They're just using gradients with different notions of distance (inner product). Speedrun gpt2 training runs.

**Momentum: Adam**
**Preconditioining: Shampoo**

# 3. Generalization
<!-- https://x.com/cloneofsimo/status/1876047503830995167
https://x.com/srush_nlp/status/1838940002123727078 -->