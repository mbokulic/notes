# K-armed bandits

Simplest form of reinforcement learning: the agent chooses between k actions and gets a reward based on the action it chose. The value of an action is the **expected reward** you are going to receive if you pick it. The goal is to maximize the expected reward and the problem is that the agent doesn't know this expectation nor the distribution of possible rewards so it has to learn them. The expression below is *not known*!

$q(a) = exp(R_t | A_t = a)$

## estimating the action values

Since we do not know the values of the different actions, we need to estimate them. The **greedy action** is the one with highest value based on the current estimates. If the agent picks that action, it is said to be **exploiting** (ie, exploiting its current knowledge). **Exploration** entails picking a non-greedy action, with the goal of learning more about it and possibly finding a better action that the current greedy one. Exploitation entails short-term gain, whereas exploration is for long-term.

Simplest option for estimating is to compute the **average reward** we got from the action, with a default value for when we don't have data. In the formula below we use $t - 1$ in the numerator and denominator because the estimate of the reward at time $t$ (ie, $Q_t$) is based on the results happening before t.

$Q_t(a) = \frac{\sum_{i=1}^{t-1}R_i}{t - 1}$

An incremental version of this rule looks like this, which is an example of a more general rule where $new = old + StepSize(reward - old)$, where $reward$ is the received reward at time $t$.

$Q_{t + 1} = Q_t + \frac{1}{n}(R_n - Q_t)$

Using a **constant step size** you would give less weight to older rewards. This solves the **non-stationary bandit problem**, where the reward distribution changes over time.

## solving the exploration / exploitation trade-off

The **epsilon-greedy** strategy involves taking a random action with probability $\epsilon$, and the greedy action with probability $1 - \epsilon$. This means that you perform "forced exploration" some cases. The problem with this strategy is that no matter how certain you are in what the best action is, you will still do some exploration. There are ways to mitigate this.

**Optimistic initial values** involves setting an overly optimistic default value. This would make you try out all actions to some degree, until you get data on them. This strategy only drives exploration at the beginning, making it bad for non-stationary problems.

**Upper-confidence bound** action selection involves estimating a confidence interval around the estimated action values. If we are uncertain about an action, we optimistically assume it is good and look at the higher bound. You pick the action with the *highest upper bound*. The formula below describes this (we pick the "argmax", ie maximum, upper bound). I do not recognize this confidence interval calculation. The parameter $c$ is the exploration parameter: the higher it is, the more "optimistic leeway" we give to the estimates.

$A_t = argmax[Q_t(a) + c(\sqrt{ \frac{ln(t)}{N_t(a)}  })]$

## contextual bandits (no clue yet)

Not sure what it involves, except using the contextual variables to some degree. Neither Ahmed knows much, besides suggesting that you have a separate action value estimates for different contexts (this can't work for continuous variables). Maybe check this video: https://vimeo.com/240429210.







