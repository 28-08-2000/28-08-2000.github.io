# Understanding Rainbow: Combining Improvements in Deep Reinforcement Learning

In this we will try to understand the state-of-art Q-learning paper Rainbow DQN. Refer here for [Paper](https://arxiv.org/pdf/1710.02298.pdf).
Refer the implementation [here](https://github.com/higgsfield/RL-Adventure/blob/master/7.rainbow%20dqn.ipynb).

## Introduction

In recent time their has been many imporvements made in DQN algorithm, in this paper and it has been tried to find the best combination of these improvements to get better resuls.
First we will understand different progressions made in Q-learning. I have stated some of the achievements, that are used in rainbow-Q learning: -

1. Deep Q-Netowrks (DQN; Mnih et al. 2013) -> It uses a combination of Q-learning with CNN and experience replay. It provides ability to learn from images and giving human level performance in many Atari games.

2. Double DQN (2016)-> This algorithm fixes the overestimation bias of Q-learning by decoupling selection and evaluation of bootstrap action.

3. Prioritized Experience replay (2015) -> It replays samples of transitions more oftenly from which there is more to learn. This in turn increases the data efficiency of algorithm.

4. Dueling network architecture (2016) -> This architecture helps to generalize across actions by seperately representing state values and action advantages.

5. Noisy DQN (2017) -> This algorithm uses stochastic network layers for exploration.

6. Distributional Q-learning (2017) -> This algorithm learn a categorical distribution of discounted returns instead of estimating the mean.

7. Multi-Step learning (2016) -> As used in A3C, helps to propagate newly observed rewards faster to earlier visited-states and shifts the bias-variance trade-off.

## Background: -

As being Reinforcement learning algo, this also tries to train an agent to act in partially observable environment and tries to maximize the reward signal.

**Agents and environments:** 

This can be formulated as an Markov decision process (means current state depends on previous states). And this MDP is episodic.

MDP is formulized as tuple = (S, A, T, r, ??)

where,

Set of States, S

Set of actions, A

Stochastic Transition function, ```T(s, a, s') = P[S?????????=S'|S???=s, A???=a]```

Reward function, ```r(s, a) = E[R?????????|S??? = S, A??? = a]```

Discount factor, ?? ?? [0, 1]

for at any time step, ????? = ?? except at termination, ????? = 0.

For Action selection, we use policy ????, it defines probability distribution of given state over actions.

Using discounted return, G??? = ?????????0 ??? k=0 ??(k)t Rt+k+1

Discounted returns represents the dicounted sum of future rewards collected by the agent.

Dicount for a reward k steps in the future, ???????????t = ???????????????? ???????????
i.e, Product of discounts before t

Policy, ????
State, v????(s) = E????[Gt | St = s]
State action pair, q????(s, a) = E????[Gt | St = s, At = a]

**Deep RL and DQN:**

In reinforcement learning, we use the following parameters: -

policy, ????(s, a)

q values, q(s, a)

loss = (R????????? + ??????????? + max a' q?? (St+1, a') - q?? (St, At))??              --------------- (1)

In reinforcement learning, we try to minimize the loss using gradient descent. We backpropagate the loss to parameters ?? of online network (that is used to select actions).

## Extensions to DQN

DQN has been an important algorithm in Deep reinforcement learning, but it also has some limitations. I am explaining some of the most useful extensions to it: -

**Double Q-learning:**

It was proposed in 2010 by Van Hasselt, and this extension tries to fix the overestimation problem of DQN. Double Q-learning decouples the selection of the action from its evaluation.

Loss function used : -
loss = (R????????? + ????? + q?? (St+1, argmax a' q?? (St+1, a')) - q?? (St, At))??

Using this change we can reduce harmful overestimations.

**Prioritized replay:**

Samples transitions with probability Pt

``Pt ??? |Rt+1 + ??t+1 max a' q??(St+1, a') - q??(St, At)|??``

where, ?? = shape of distribution (hyperparameter)

This adds new transitions with max priority because their is more to learn from new transitions.

**Dueling network:**

We uses 2 streams of computations in this
a) Value stream
b) advantage stream

action values,
q??(s, a) = v???(f??(s)) + a?? (f??(s), a) - ???a' a??(f??(s), a') / N???c???????????????

where,
f?? = shared encoder

v??? = value stream

a?? = advantage stream

parameters, ?? = {??, n, ??}

**Multi-step learning:**

In this we define the truncated n-step return from a given state St

Rt????????? = k=0 ??? k=n-1 ??t????????? R???????????????                 --------(2)

loss = (R???????????? + ?????????????? + max a' q?? (St+n, a') - q?? (St, At))??

**Distributional RL:**

Support,
Zi = vmin  + (i - 1) Vmax - Vmin / Natoms - 1

where, i ?? {1, ...., Natoms}
z = vector with Natoms ?? N??? atoms

If

time = t

Probability mass on each atom i = P?????(St, At)

then

distribution, dt = (z, p??(St, At))

policy, ????*should match target distribution

target distribution ``d't = (Rt+1 + ??t+1z, P??(St+1, a*t+1)), D??????(??z d't||dt)``       -----------------(3)

Where, ??z = L2-projection of target distribution onto z,
greedy action w.r.t mean action values, a*t+1 = argmax a q??(S+1, a)

mean action values, q??(St, a) = z??? p?? (St, a)

**Noisy Nets:**

It proposes a noisy linear layer, which is combination of determinisitc and noisy stream.
It uses below equation in place of y = b + wx

y = (b + Wx) + (b????????????y ??? ????? + (W????????????y ??? ??w)x),     -------------(4)

where, ????? and ??w are random variables
??? = element wise product

## The Integrated Agent (or Rainbow)

??? Distributional RL ??? Multi-step learning

Using (2) and (3) or by combining distribution and multi-step learning we get,

dt????????? = (Rt????????? + ??t?????????z, p??(St+n, a*t+n))

and ``loss = D??????(??z dt????????? ||dt)``

??? Double Q-learning

We combine above result (of multi-step distribution loss) with double Q-learning.
By using *online network* for selecting bootstrap action a*t+n in state St+n and using*target network* for evaluating the bootstrap action.

??? Prioritized Replay

We prioritize the transitions by KL loss, since the algorithm is minimizing this.

``pt ??? D??????(??z dt????????? ||dt)??``

??? Dueling network

f??(s) = shared encoder
v??? [Natoms] = value stream
a?? [Natoms x Natoms] = advantage stream

Output at atom i and action a = a????? (f??(s), a)

return's ditrubtions:

P?????(s, a) = exp(v???n(??) + a?????(??, a) - a????????(s)) / ???j exp(v??n(??) + a????(??, a) - a???????(s))

where, ?? = f??(s)
a????????(s) = 1/ N???c??????????????? * ???a' a?????(??, a')

??? Noisy Nets

We replace the linear layers with noisy equivalent. In these noisy layers we use factorial Gaussian Noisy.
