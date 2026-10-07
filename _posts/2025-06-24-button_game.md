---
layout: post
title: "The Button Game: When Should You Stop for the Best Prize?"
date: 2025-06-24 12:00:00-0700
description: "An interesting interview question about optimal stopping."
tags: ["optimal stopping", "backward induction", "dynamic programming", "probability"]
categories: ["mathematics", "computer science", "data analysis"]
related_posts: false
published: true
pretty_table: true
---

## An Interview Puzzle

While preparing for an upcoming technical interview, I came across a probability puzzle that, at first glance, seems like a simple game of chance, but quickly reveals layers of strategic depth. The problem was as follows:

> An agent can press a button up to 10 times. Each press generates a random monetary value, drawn from a continuous uniform distribution on the interval `[0, $100,000]`. After each press, the agent can either stop and keep the current value, ending the game, or discard the value and continue to the next press. If the agent completes all 10 presses, they must accept the value from the 10th press.

**The question is: What is the optimal strategy to maximize the expected payoff?**

My initial intuition was to work backward from the end. But after solving it, I became more interested in the bigger picture: what kind of problem is this, and what mathematical principles guarantee that the solution is truly optimal?

## Zooming Out: The World of Optimal Stopping

This puzzle is not a one-off brain teaser; it's a classic example of a **finite-horizon optimal stopping problem**. This class of problems is fundamental in fields like finance, economics, and computer science, dealing with the challenge of choosing the perfect time to take a particular action to maximize a reward or minimize a cost.

To analyze such problems rigorously, we model them as [**Markov Decision Processes (MDPs)**](https://en.wikipedia.org/wiki/Markov_decision_process). An MDP is a mathematical framework for decision-making in situations where outcomes are partly random and partly under the control of a decision-maker. Our game fits this framework perfectly:

1.  **States:** The state at any point is the number of presses remaining, $$n$$, together with the value $$x$$ we just drew.
2.  **Actions:** In any state (where $$n > 1$$), our actions are to `stop` or `continue`.
3.  **The Markov Property:** Most importantly, the game is "memoryless." Because each press is independent of the others, the history of previously rejected values carries no information about future draws. The decision with $$n$$ presses left depends only on $$n$$ and the value $$x$$ in hand.

Recognizing the problem as an MDP is the first step, as it allows us to bring a powerful toolkit to bear on finding the solution: the theory of dynamic programming.

## Dynamic Programming and Bellman's Principle

The cornerstone for solving MDPs is **dynamic programming**, a method for breaking down a complex problem into a sequence of simpler, nested subproblems. The philosophical foundation of this method is [**Bellman's Principle of Optimality**](https://en.wikipedia.org/wiki/Bellman_equation#Bellman's_principle_of_optimality).

> **Bellman's Principle of Optimality** states that an optimal policy has the property that whatever the initial state and first decision are, the remaining decisions must constitute an optimal policy for the subproblem starting from the new state.

In simpler terms, a perfect overall plan is composed of a series of perfect sub-plans. To play the 10-press game optimally, our strategy for the remaining 9 presses (should we continue) must also be the optimal strategy for a 9-press game.

This principle is captured mathematically in the [**Bellman Equation**](https://en.wikipedia.org/wiki/Bellman_equation). For our type of optimal stopping problem, the equation takes the following form. If we let $$V_n$$ be the maximum expected payoff we can get with $$n$$ presses remaining, then:

$$
V_n = \mathbb{E}[\max(X_n, V_{n-1})]
$$

This elegant equation is the blueprint for our strategy. It says the value of being in a state with $$n$$ presses left is the expected value of the *best possible choice*: either the random value $$X_n$$ we just drew, or the value $$V_{n-1}$$ we expect to get if we discard the current draw and play optimally from the next state.

## Applying the Theory to Solve the Puzzle

With this theoretical framework, we can now solve the interview puzzle methodically using **backward induction**. For clarity, let's set $$C = 100,000$$.

**Step 1: Solve the Base Case ($$n=1$$)**
We start at the end. With only one press left, we must accept the outcome. The expected value is the mean of the uniform distribution:
$$
V_1 = \mathbb{E}[X] = \frac{C}{2} = \$50,000
$$

**Step 2: Derive the General Recurrence Relation**
For any stage $$n \ge 2$$, the decision rule is to stop if the draw $$x$$ is greater than the continuation value, $$V_{n-1}$$. We can find the value $$V_n$$ by calculating the expectation from the Bellman Equation:

$$
V_n = \mathbb{E}[\max(X, V_{n-1})] = \int_{0}^{C} \max(x, V_{n-1}) \frac{dx}{C}
$$

We split the integral at the threshold $$V_{n-1}$$, since the value of $$\max(x, V_{n-1})$$ changes at that point:

$$
V_n = \frac{1}{C} \left[ \int_{0}^{V_{n-1}} V_{n-1} \,dx + \int_{V_{n-1}}^{C} x \,dx \right]
$$

Now, we solve the integrals:

$$
V_n = \frac{1}{C} \left[ V_{n-1} [x]_0^{V_{n-1}} + \left[\frac{x^2}{2}\right]_{V_{n-1}}^C \right]
$$

$$
V_n = \frac{1}{C} \left[ V_{n-1}(V_{n-1} - 0) + \left(\frac{C^2}{2} - \frac{V_{n-1}^2}{2}\right) \right]
$$

$$
V_n = \frac{1}{C} \left[ V_{n-1}^2 + \frac{C^2 - V_{n-1}^2}{2} \right] = \frac{2V_{n-1}^2 + C^2 - V_{n-1}^2}{2C}
$$

This simplifies to the clean, general recurrence relation we can use for all steps:

$$
V_n = \frac{V_{n-1}^2 + C^2}{2C}
$$

Using this formula, we can now iterate backward from our known base case ($$V_1 = 50,000$$) to generate the complete optimal strategy. For instance:
- **For n=2:** $$V_2 = \frac{(50000)^2 + (100000)^2}{2 \cdot 100000} = \$62,500$$
- **For n=3:** $$V_3 = \frac{(62500)^2 + (100000)^2}{2 \cdot 100000} \approx \$69,531$$

Repeating this process for all 10 stages gives us the following policy:

| Presses Left (n) | Threshold to Stop ($$V_{n-1}$$) | Expected Payoff ($$V_n$$) |
| :--------------- | :-----------------------------  | :------------------------ |
| 10               | $84,982                         | $86,110                   |
| 9                | $83,645                         | $84,982                   |
| 8                | $82,030                         | $83,645                   |
| 7                | $80,038                         | $82,030                   |
| 6                | $77,508                         | $80,038                   |
| 5                | $74,173                         | $77,508                   |
| 4                | $69,531                         | $74,173                   |
| 3                | $62,500                         | $69,531                   |
| 2                | $50,000                         | $62,500                   |
| 1                | (Must accept draw)              | $50,000                   |

*Note: Dollar values are rounded to the nearest dollar.*

## Empirical Validation with Python Simulation

As a final check, a Monte Carlo simulation confirms that this strategy works in practice.

### Implementation
```python
import random

def calculate_thresholds(max_presses, C=100000):
    V = {}
    V[1] = C / 2
    for n in range(2, max_presses + 1):
        V_n_minus_1 = V[n - 1]
        V[n] = (V_n_minus_1**2 + C**2) / (2 * C)
    return V

def play_one_game(thresholds, max_presses, C=100000):
    for presses_left in range(max_presses, 0, -1):
        current_draw = random.uniform(0, C)
        if presses_left == 1:
            return current_draw
        threshold = thresholds[presses_left - 1]
        if current_draw >= threshold:
            return current_draw
    return 0 

def main():
    MAX_PRESSES = 10
    C = 100000
    NUM_SIMULATIONS = 1_000_000
    thresholds = calculate_thresholds(MAX_PRESSES, C)
    print(f"Theoretical Expected Payoff: ${thresholds[MAX_PRESSES]:,.2f}\n")
    total_payoff = sum(play_one_game(thresholds, MAX_PRESSES, C) for _ in range(NUM_SIMULATIONS))
    average_payoff = total_payoff / NUM_SIMULATIONS
    print(f"Simulated Average Payoff ({NUM_SIMULATIONS:,} games): ${average_payoff:,.2f}")

if __name__ == "__main__":
    main()
```

### Results

```text
Theoretical Expected Payoff: $86,109.82

Simulated Average Payoff (1,000,000 games): $86,112.80
```

The simulated average differs from the theoretical value by less than 3 dollars. With a million games, the Monte Carlo standard error is about 14 dollars, so the agreement is as close as we should expect.

## Why the Strategy is Optimal

This brings us to the decisive question: why is this threshold rule not merely good, but provably **optimal**? Because the game has a finite horizon, backward induction is itself the proof.

The argument runs by induction on the number of presses left:

- **Base case.** With one press left there is no decision to make, so $$V_1 = C/2$$ is the best achievable expected payoff.
- **Inductive step.** Suppose $$V_{n-1}$$ is the best expected payoff any strategy can achieve with $$n-1$$ presses left. With $$n$$ presses left and a draw $$x$$ in hand, a strategy can either stop and receive $$x$$, or continue.
  - Future draws are independent of everything seen so far, so continuing starts a fresh $$(n-1)$$-press game. By assumption, no strategy can expect more than $$V_{n-1}$$ in that game.
  - So from this point no strategy can expect more than $$\max(x, V_{n-1})$$, and the threshold rule achieves exactly that.
  - Averaging over $$x$$ shows that $$V_n = \mathbb{E}[\max(X, V_{n-1})]$$ is the best achievable value with $$n$$ presses left.

By induction, the rule "stop when the draw $$x \ge V_{n-1}$$" is optimal at every stage. No strategy, however elaborate, can beat the expected payoff $$V_{10} \approx 86{,}110$$. This is Bellman's Principle of Optimality at work. The best plan for the 10-press game is built from the best plans for every shorter game.

The same conclusion is a small instance of a general result from optimal stopping theory. The values $$\max(x, V_{n-1})$$ form the [*Snell envelope*](https://en.wikipedia.org/wiki/Snell_envelope) of the payoff process, which is the smallest supermartingale that dominates it. A classical theorem says that stopping the first time the payoff equals its Snell envelope is optimal. Here, that is exactly the first draw with $$x \ge V_{n-1}$$.

*Updated October 2026: this section previously appealed to the Bellman operator being a contraction with a unique fixed point. That argument is the tool for infinite-horizon, discounted problems, where there is no last step to start from. For a finite horizon like this one, backward induction from the last press is the proof.*