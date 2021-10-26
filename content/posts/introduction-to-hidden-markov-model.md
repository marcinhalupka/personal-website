---
title: "Introduction to Hidden Markov Model"
date: "2021-10-22"
draft: false
tags: []
---

The goal of this post and series is to present the foundational concept, usability, intuition of the algorithmic part and some basic axamples of Hidden Markov Models.
To understand the content, basic knowledge on probability should be sufficient.

# Why kind of problems I can solve with them?

The ones where we observe a sequential data but we suspect that behind the scenes there may be something going on that affects the sequence we are looking at.
Weather, text, speech and biological sequences could be a great candidates.

Let's look at a simple example with a dishonest casino. The game we are playing is to guess the outcone of a dice roll and the dice has 6 sides.
As we told, casino is not playing fair and is using two dices - one of them is fair and another one unfair. Unfair means that the probabilities of the outcomes are different than
`(1/6, 1/6, 1/6, 1/6, 1/6, 1/6)` - they are not uniform. We know the outcome of the dice roll (1 to 6), we know the sequence of the throws (obsevations) but we can't tell when the unfair die was used (hidden state).
Hidden Markov Model can give us the answer to this question.

# The intuition of Markov Models

Markov Models are used to model randomly changing systems, like weather patterns. In Markov Model all the states are visible/observable.

The key assumption of Markov Model is that **the future state/event depends only on current state/event** and not on any other states (`Markov Property`). If we consider weather pattern (sunny, rainy, cloudy)
then we can say tomorrow's weather will only depend on today's weather and not on the weather from previous n days.

Mathematically speaking, the probability of the `state` at time `t` will only depend on time step `t-1`, in other words $P(s(t)|s(t-1))$. This is known as `First Order Markov Model`.

If the probability of the `state` at time `t` depends on time step `t-1` and `t-2`, the order increases and we receive $P(s(t)|s(t-1), s(t-2))$ - `Second Order Markov Model`.
If we need to increase the dependency of past time events, the recipe for higher orders is clear.

Eventually, te idea is to model the joint probability of a sequence, such as the probability of $(s_1, s_2, s_3)$ where $s_1$, $s_2$ and $s_3$ happened one after another.
We can use the join and conditional probability rule and write it as:

$$\begin{eqnarray} 
    P(s_3,s_2,s_1) &=& P(s_3|s_2,s_1)P(s_2,s_1) \\\\\\
	&=& P(s_3|s_2,s_1)P(s_2|s_1)P(s_1) \\\\\\
	&=& P(s_3|s_2)P(s_2|s_1)P(s_1) 
\end{eqnarray}$$

The diagram for this simple Markov Model:

![Markov Model Diagram](/posts/images/1_1.png)

## Transition probabilities

The probability of one state changing to another state is defined as `transition probability`. So in case of having 3 states `(Sun, Cloud, Rain)`, there will be total 9 transition probabilities.
As you look at the diagram, all transition probabilities were defined. Transition probability generally are denoted as $a_{ij}$ which can be interpreted as the probability of the system to transition from
state `i` to state `j`.

$$
a_{ij} = P(s(t+1) = j | s(t) = i)
$$

![Markov Model Transition Probabilities](/posts/images/1_2.png)

For an example presented above, the transition probability fromk Sun to Cloud is defined as $a_{12}$. Note that the transition might happen to the same state also. If we have sun in two consecutive days
then the transition probability from sun to sun at the next time step will be $a_{11}$.

Generally, the transition probabilities are defined using a `(M x M)` matrix, known as **Transition Probability Matrix**. For our example, the Transition Probability Matrix would be defined as:

$$
A =  \begin{bmatrix}a_{11} & a_{12} & a_{13} \\\\\\ a_{21} & a_{22} & a_{23} \\\\\\ a_{31} & a_{32} & a_{33} \end{bmatrix}
$$

Nice property - when the system transitions to another state, the sum of all transition probabilities given the current state should be equal to 1, so for example $a_{11} + a_{12} + a_{13} = 1$.

In general:

$$
\sum_{j=1}^{M} a_{ij} = 1 \\;  \\; \\; \forall i
$$


## Initial probability distribution

The system has to start from one state. The initial state of Markov Model (when time step `t = 0`) is denoted as $\pi$ - N dimensional row vector.
All the probabilities must sum to 1, so $ \sum_{i=1}^{N} \pi_i = 1 \\; \\; \\;   \forall i$.

During implementation, we can assume the initial probability distribution is uniform. In our weather example, we could define initial state as
$ \pi = [ \frac{1}{3} \frac{1}{3} \frac{1}{3}]$.

In some cases we may have $\pi_i = 0$, if particular state can't be an initial one.

## Markov Chain

When the system is fully observable and autonomous, it's called a `Markov Chain`.
What we've learned so far is a great example.

We can conclude that Markov Chain consists of the following parameters:
* A set of `N states`
* A `transition probability` matrix A
* An `initial probability distribution` $\pi$

## Absorbing state

As the name suggests, when transition probabilities from one state to all the other except for itself are 0, then it's called an Absorbing State.
When the system enters into that state, it never leaves. 

# Hidden Markov Model

In Hidden Markov Model the state of the system will be hidden (unknown), however at every time step `t` the system in state `s(t)` will
emit an observable symbol `v(t)`.

The idea is presented on a graph below:

![Markov Model Transition Probabilities](/posts/images/1_3.png)

In our initial example of dishonest casino, the die rolled (fair or unfair) is hidden. However every time the die is rolled, we know the outcome
(number between 1 and 6) - this is the symbol we observe.

**Summary:**
* We can define a particular sequence of visible states/symbols as $V^T = \{ v(1), v(2) … v(T) \}$
* Then define model $\theta$, so for every state `s(t)` we have a probability of emitting a particular visible state $v_k(t)$
* Since we have access to the visible states only (`s(t)'s` are unobservable), a model is called a `Hidden Markov Model`


## Emission probability

We can redefine our previous example. Assume based on the weather of any day the mood of a person changes from happy to sad.
Also assume the person is at a remote place and we don't know how the weather is there. We can only know the mood of the person.
In this scenario, the weather is the `hidden state` in the model and the mood (happy/sad) is the `visible state`. So we can use the HMM to predict
the weather, knowing the mood of the person.

The graphical presentation of transition mechanism looks like that:

![Markov Model Transition Probabilities](/posts/images/1_4.png)

The observable symbols are $\{ v_1 , v_2 \}$ - from each hidden state one of them must be emitted.
The probability of emitting a symbol is known as `emission probability` - defined by us as $b_{jk}$. Mathematically,
it is the probability of emitting symbol `k` given state `j`:

$$
b_{jk} = p(v_k(t) | s_j(t) )
$$

Emission probabilities are also defined using `N x M` matrix, called `Emission Probability Matrix`:

$$
B = \begin{bmatrix} 
b_{11} &  b_{12} \\\\\\ 
b_{21} &  b_{22} \\\\\\
b_{31} &  b_{32} 
\end{bmatrix}
$$

The Emission Probabilities, same as Transition Probabilities, sum to 1.

$$
\sum_{k=1}^{C} b_{jk} = 1,  \\; \\; \\;   \forall j
$$

We have the attributes and properties of our Hidden Markov Model defined. Few more obstacles are waiting for us before reaching our ultimate goal for a new tool - prediction.

# Central issues with Hidden Markov Model


## Evaluation problems

We can define the model $\theta$ as following:

$$
\theta \rightarrow s, v, a_{ij},b_{jk}
$$

Given the model ($\theta$) and sequence of visible states ($V^T$), we need to determine the probability that a particular sequence
of visible states was generated from the model.

There could be many models $\{ \theta_1, \theta_2 … \theta_n \}$. We need to find $p(V^T | \theta_i)$ then use Bayes Rule to correctly
classify the sequence $V^T$.

$$
p(\theta | V^T  ) = \frac{p(V^T | \theta) p(\theta)}{p(V^T)}
$$


## Learning problems

In general HMM is unsupervised learning process - number of different visible symbol types is known (like happy/sad), but the number
of hidden states is not. We can always try out different options however this may result in significant amount of computation.
Picking the number of hidden states can be a more or less educated guess.

Once we have the high level structure (number of hidden and visible states) of the model, we need to estimate the transition ($a_{ij}$) and
emission ($b_{jk}$) probabilities using the training sequence of visible states - that's known as a learning problem.

We'll be using the evaluation problem to solve the learning problem, so understanding the former is necessary. The transition
and emission probabilities will be estimated with use of the `Expectation Maximization` algorithm. Entire learning problem is known as `Forward-Backward Algorithm`
or `Baum-Welch Algorithm`.

## Decoding problem

Once we have the estimates for transition ($a_{ij}$) and emission ($b_{jk}$) probabilities, we can use the model ($\theta$) to predict the
hidden states ($W^T$) which generated the visible sequence ($V^T$). The decoding problem is known as `Viterbi Algorithm`.

# Conclusion

The introduction gives some intuition about the Hidden Markov Models. To implement the algorithm from scratch, we have to go through
three problems defined above as the next steps.
