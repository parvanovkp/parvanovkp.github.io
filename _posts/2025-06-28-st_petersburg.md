---
layout: post
title: "The St. Petersburg Paradox, Re-Run Over Time"
date: 2025-06-28 18:19:00-0700
description: "What finite wealth and repeated play do to an infinitely valuable game, and why the fair price depends on what you mean by fair."
tags: ["ergodicity", "utility theory", "decision theory"]
categories: ["mathematics", "economics", "computer science"]
related_posts: false
published: true
pretty_table: true
---

I was recently trying to explain the St. Petersburg paradox to my uncle from memory during a walk in the park. The setup is a deceptively simple coin-tossing game.

> A fair coin is flipped until it lands heads for the first time. If heads appears on the 1st toss, you win $2. If on the 2nd, $4. If on the 3rd, $8, and so on, with the payout doubling each time.

**What is a fair price to pay to play this game?**

The expected payoff is famously, and confusingly, infinite. Each outcome contributes its probability times its payout, and every one of those terms is worth exactly one dollar.

$$\mathbb{E}[X] = \left(\frac{1}{2}\right)\cdot \$2 + \left(\frac{1}{4}\right)\cdot \$4 + \left(\frac{1}{8}\right)\cdot \$8 + \dots = \sum_{n=1}^{\infty} \frac{1}{2^n} \cdot 2^n = \$1 + \$1 + \$1 + \dots = \infty$$

A purely "rational" actor, according to classical theory, should pay any finite price to play. Nobody would. Most people offer a few dollars, and as I explained this, I found myself questioning the standard explanation.

## The Classical "Solution"

The classic solution, proposed by [Daniel Bernoulli in 1738](https://psych.fullerton.edu/mbirnbaum/psych466/articles/bernoulli_econometrica.pdf), is that people don't value money linearly. An extra dollar matters less the wealthier you are. Model this **diminishing marginal utility** with a logarithm and the *expected utility* of the game becomes finite, which justifies a small price.

Expected utility rests on the von Neumann-Morgenstern axioms, and [Kelly-growth](https://www.princeton.edu/~wbialek/rome/refs/kelly_56.pdf) arguments single out the logarithm in particular. It remains the benchmark in modern finance. My intuition pointed somewhere else, though.

> People subconsciously know they have **finite wealth**.

The astronomical prizes behind the infinite expectation only arrive after long strings of losses (and only if the casino has unbounded credit, which is unrealistic in itself). Most people know their stake would be gone long before that lucky streak showed up.

## A Simulation of Finite Stakes

To test this, I wrote a simulation with one crucial rule. **Winnings cannot fund more plays.** You arrive with a fixed stake, and when it is gone, your game is over. Call this the stake-based model. How does a player's chance of breaking even change across starting stakes and entry prices?

<details>
<summary>Click to see the simulation code</summary>

<pre><code class="language-python">import random

def simulate_stake_play(start_stake, entry_price):
    stake = float(start_stake)
    net_winnings = 0.0
    
    while stake >= entry_price:
        stake -= entry_price
        
        # Flip until heads
        flips = 1
        while random.random() > 0.5:
            flips += 1
        
        payout = 2**flips
        net_winnings += payout
    
    return net_winnings + stake

def generate_results_table():
    entry_prices = [6, 8, 10, 13]
    wealth_levels = [64, 256, 1024, 8192]
    n_sims = 50_000

    header = "| Starting Stake | " + " | ".join([f"Price = ${p}" for p in entry_prices]) + " |"
    separator = "|:---" + "|:---" * len(entry_prices) + "|"
    print("## Chance of Breaking Even or Profiting\n")
    print(header)
    print(separator)

    for wealth in wealth_levels:
        row = [f"**${wealth:,}**"]
        
        for price in entry_prices:
            if wealth < price:
                row.append("N/A")
                continue

            outcomes = [simulate_stake_play(wealth, price) for _ in range(n_sims)]
            success_rate = sum(1 for outcome in outcomes if outcome >= wealth) / n_sims
            row.append(f"{success_rate * 100:.1f}%")

        print("| " + " | ".join(row) + " |")

if __name__ == "__main__":
    generate_results_table()
</code></pre>
</details>

## Chance of Breaking Even or Profiting

| Starting Stake | Price = $6 | Price = $8 | Price = $10 | Price = $13 |
|:---|:---|:---|:---|:---|
| **$64** | 49.9% | 31.2% | 21.6% | 14.5% |
| **$256** | 73.7% | 45.9% | 31.0% | 19.1% |
| **$1,024** | 95.6% | 67.4% | 44.0% | 26.1% |
| **$8,192** | 100.0% | 98.0% | 75.6% | 41.1% |

An exact calculation, which convolves the payout distribution instead of sampling it, agrees with every cell to within half a percentage point. Read across a row and the odds collapse as the price rises. Read down a column and a bigger stake buys far better odds at the same price. The game is not the same for everyone.

## Ergodicity Economics

This line of thinking led me to the physicist [**Ole Peters**](https://arxiv.org/abs/1011.4404) and the field of **ergodicity economics**, which gave my intuition a name. On this view the problem is not psychology. It is the kind of average being used.

The **ensemble average** is the standard expected value $$\mathbb{E}[X]$$, the average payout per player as more and more people play in parallel. Here it never settles. It creeps upward without bound, yet so slowly that a crowd of a million players would typically average only about $23 each. The infinity lives in outcomes too rare for any real crowd to see.

The **time average** follows *one person's* wealth as they play again and again. That is the average we actually live through.

When the two coincide, the system is called ergodic. Here they don't. Ergodicity economics argues that a rational person should maximize the long-term **growth rate** of their wealth, which for multiplicative processes means maximizing the expected change in log-wealth.

Notice what this does and doesn't change. The resulting condition is mathematically identical to Bernoulli's logarithmic utility, as Peters himself says in his abstract. What differs is the justification. The logarithm now comes from the dynamics of repeated play rather than from an assumption about how people feel about money.

That difference can be tested. If the logarithm comes from the dynamics, risk attitudes should shift when the dynamics shift. A [2021 lab study by Meder and colleagues](https://doi.org/10.1371/journal.pcbi.1009217) found risk aversion rising under multiplicative dynamics, roughly as the time-optimal model predicts. One study proves little on its own. Still, it is a genuine test.

There is a caveat, too. The framework matches the growth measure to the dynamics, using the logarithm when wealth compounds and the plain average when gains simply add up. In the stake-based game, gains add up. Applied literally there, the framework hands back the plain average, which never settles, so in that game the finite stake is what keeps the price finite. Peters' logarithm needs the extra assumption that the gamble sits inside a wealth process that compounds. For most people that is reasonable, but it remains an assumption.

### The Exact Growth-Neutral Price

A price `c` is "growth-neutral" if it leaves expected log-wealth unchanged. For a player with total wealth `W` who **reinvests their winnings** (with `c ≤ W`), the condition reads

$$\mathbb{E}[\Delta \log(W)] = \sum_{n=1}^{\infty} p_n \cdot \log(W - c + \text{payout}_n) - \log(W) = 0$$

With $$p_n = 1/2^n$$ and a payout of $$2^n$$ (taking $1 as the unit for the first head), this becomes

$$\sum_{n=1}^{\infty} \frac{1}{2^n} \log(W - c + 2^n) = \log(W)$$

There is no closed form for `c`. Solving it numerically is easy, though, and for large wealth a simple approximation emerges.

### Approximate Rule

Think about the toss `k` at which the payout first matches your wealth, so `2^k ≈ W` and `k ≈ log₂(W)`. Outcomes well below this crossover barely move your wealth. Each adds about $$\frac{1}{2^n} \cdot \frac{2^n}{W} = \frac{1}{W}$$ to the expected log-growth, which is the same dollar per outcome that makes the classical expectation diverge, now scaled by your wealth. There are about $$\log_2 W$$ of them. Outcomes past the crossover multiply your wealth, but they are rare enough to add only a bounded amount, and the entry price costs about $$c/W$$. Balance the books and you get

$$c^*(W) \approx \log_2(W) + \frac{1}{\ln 2} - \frac{1}{2} \approx \log_2(W) + 0.943$$

In other words, the fair price is roughly the expected payout over the outcomes that pay less than your wealth, one dollar each, plus a constant for the rest.

The constant comes out exactly. For $$c \ll W$$ the price enters as $$-c/W$$, so $$c^*$$ is close to

$$S(W) = W \sum_{n=1}^{\infty} \frac{1}{2^n} \ln\left(1 + \frac{2^n}{W}\right) = \sum_{n=1}^{\infty} g(n - \log_2 W), \qquad g(x) = 2^{-x}\ln(1 + 2^x).$$

Since $$g$$ tends to 1 for small outcomes and to 0 for large ones, the Euler-Maclaurin formula gives one unit per small outcome, a boundary correction of $$-\tfrac{1}{2}$$, and the integral $$\int_{-\infty}^{\infty} \big(g(x) - \mathbf{1}[x \le 0]\big)\,dx = \tfrac{1}{\ln 2}$$. (Substitute $$u = 2^x$$ and the remaining integral is exactly 1.) The leftover error is periodic in $$\log_2 W$$ and of order $$10^{-12}$$.

The approximation drops terms of order $$c^2/W$$. At $64 it gives $6.94 against an exact $7.21, and by $8,192 the two agree within a cent. My earlier rule of thumb, `log₂(W) + 1`, stays within about 20 cents over this range, but the offset is not really 1. It falls from 1.21 at $64 toward 0.943.

Either way, the price is finite and depends directly on wealth.

## Three Prices, Two Yardsticks

So what is the fair price for the stake-based game we simulated? An earlier version of this post fell into a trap here. "Fair" needs a definition, and the formula and the simulation suggest different ones. The growth-neutral equation uses a **log-growth yardstick**, under which expected log-wealth stays put. The simulation suggests a **median yardstick**, the price that gives you even odds of leaving with at least what you came with.

The median price turns out to be easy to compute exactly. A stake `W` at price `c` buys `N = ⌊W/c⌋` games and leaves `W − Nc` in change. You walk away with at least `W` precisely when your total payout reaches `Nc`, which means **your average payout per game has to cover the ticket price**. The change drops out entirely. Instead of searching by simulation, we can convolve the payout distribution `N` times and read off the answer.

<details>
<summary>Click to see the price code</summary>

<pre><code class="language-python">import math
import numpy as np
from scipy.optimize import brentq


def growth_neutral_price(W):
    """Price c with E[log(W - c + payout)] = log(W) for one play (payout 2^n w.p. 2^-n)."""
    n = np.arange(1, 200)
    excess_log_growth = lambda c: np.sum(0.5**n * np.log1p((2.0**n - c) / W))
    return brentq(excess_log_growth, 0.0, W)


def median_fair_price(W):
    """Exact largest price c at which a stake-based player with stake W
    has at least a 50% chance of walking away with >= W.

    With N = floor(W / c) games, walking away with >= W  <=>  total payout S_N >= N * c,
    so we only need the distribution of S_N (capped at W, which loses nothing here)."""
    payouts = []                          # (payout, probability); payouts >= W lumped at W
    n = 1
    while 2**n < W:
        payouts.append((2**n, 0.5**n))
        n += 1
    payouts.append((W, 0.5**(n - 1)))

    dist = np.zeros(W + 1)
    dist[0] = 1.0                         # distribution of min(S_N, W), starting at N = 0
    best_price, best_games, games = 0.0, 0, 0
    while True:
        games += 1
        if W / games <= best_price:       # c <= W / N, so no larger price is possible
            return best_price, best_games
        new = np.zeros(W + 1)
        for v, p in payouts:              # add one more game
            new[v:] += p * dist[:W + 1 - v]
            new[W] += p * dist[W + 1 - v:].sum()
        dist = new
        tail = np.cumsum(dist[::-1])[::-1]          # tail[k] = P(S_N >= k)
        m = np.nonzero(tail >= 0.5)[0].max()        # largest m with P(S_N >= m) >= 1/2
        price = m / games
        if price > W / (games + 1) and price > best_price:   # consistent with N = floor(W/c)
            best_price, best_games = price, games


def log_fair_stake_price(W, n_paths=200_000, step=0.005, seed=0):
    """Largest price c (on a grid) with E[log(final wealth)] >= log(W) in the stake-based
    game, where final wealth = S_N + leftover and N = floor(W / c). Monte Carlo."""
    rng = np.random.default_rng(seed)
    games_needed = {}
    for c in np.arange(math.log2(W) - 2.5, math.log2(W) + 2.0, step):
        games = int(W // c)
        games_needed.setdefault(games, []).append((c, W - games * c))
    total = np.zeros(n_paths)
    best = 0.0
    for games in range(1, max(games_needed) + 1):
        total += 2.0 ** rng.geometric(0.5, n_paths)   # play one more game on every path
        for c, leftover in games_needed.get(games, []):
            if np.mean(np.log(total + leftover)) >= math.log(W):
                best = max(best, c)
    return best


if __name__ == "__main__":
    print("| Starting Stake | Growth-neutral, compounding (exact) "
          "| Log yardstick, stake-based (simulated) | Median yardstick, stake-based (exact) |")
    print("|:---|:---|:---|:---|")
    for W in [64, 256, 1024, 8192]:
        median_price, _ = median_fair_price(W)
        print(f"| **${W:,}** | ${growth_neutral_price(W):.2f} "
              f"| ${log_fair_stake_price(W):.2f} | ${median_price:.2f} |")

    print()
    print("| Starting Stake | Games N | Median price | Median price − log₂N | Gap to growth-neutral |")
    print("|:---|:---|:---|:---|:---|")
    for W in [64, 256, 1024, 8192, 65536]:
        price, games = median_fair_price(W)
        print(f"| **${W:,}** | {games:,} | ${price:.2f} | {price - math.log2(games):.2f} "
              f"| ${growth_neutral_price(W) - price:.2f} |")
</code></pre>
</details>

| Starting Stake | Growth-neutral, compounding (exact) | Log yardstick, stake-based (simulated) | Median yardstick, stake-based (exact) |
|:---|:---|:---|:---|
| **$64** | $7.21 | $7.02 | $5.82 |
| **$256** | $9.06 | $8.85 | $7.64 |
| **$1,024** | $10.99 | $10.78 | $9.34 |
| **$8,192** | $13.95 | $13.81 | $11.99 |

The simulated column uses 200,000 paths and is accurate to about 3 cents. The other two are exact.

Under the same log-growth yardstick, the stake-based rules cost only 14 to 21 cents relative to compounding. Switching yardsticks costs more than a dollar at every stake. Why so much? The median only asks whether you broke even and ignores how far a jackpot carries you past that point. The log yardstick credits jackpots, damped by the logarithm, and with payouts this lopsided that difference is large.

## Why the Gap Widens with Wealth

The median price follows a law of its own. Feller (1945, *Annals of Mathematical Statistics*) proved that the total payout from `N` St. Petersburg games satisfies $$S_N/(N \log_2 N) \to 1$$ in probability, and [Martin-Löf (1985)](https://resolve.cambridge.org/core/journals/journal-of-applied-probability/article/abs/limit-theorem-which-clarifies-the-petersburg-paradox/38C6FA462D5E0F9CBBACC25F8C0A8ED4) later described the limiting distribution of these sums. The typical average payout per game grows like `log₂N`. A stake-based player gets `N = W/c` games, so the median price should be `log₂N` plus a constant.

| Starting Stake | Games N | Median price | Median price − log₂N | Gap to growth-neutral |
|:---|:---|:---|:---|:---|
| **$64** | 11 | $5.82 | 2.36 | $1.39 |
| **$256** | 33 | $7.64 | 2.59 | $1.42 |
| **$1,024** | 109 | $9.34 | 2.57 | $1.65 |
| **$8,192** | 683 | $11.99 | 2.57 | $1.96 |
| **$65,536** | 4,454 | $14.71 | 2.59 | $2.23 |

Past the smallest stake, the constant sits near 2.6. At a stake of about $1 million, with roughly 57,000 games, it is still 2.60. Substituting `N = W/c` gives

$$c_{\text{median}} \approx \log_2 W - \log_2 \log_2 W + 2.6$$

which grows more slowly than the growth-neutral price $$\log_2 W + 0.943$$. So the gap widens with wealth instead of shrinking, roughly like $$\log_2 \log_2 W$$, and reaches $2.54 at about $1 million.

The same law explains the million-player crowd from earlier. With `N = 1,000,000`, `log₂N` is about 20, and adding 2.6 gives roughly $22.5, close to the $23 median I got from simulating 400 such crowds.

## Conclusion

The paradox dissolves once you pin down two things, how the game is played and what "fair" means. Model a player with finite wealth playing through time, and the price turns finite and personal, about `log₂(W) + 0.94` dollars by the growth-neutral yardstick.

Does that make utility theory unnecessary? That is a question of interpretation, not arithmetic. The growth-neutral condition is Bernoulli's equation, and ergodicity economics changes the reason for the logarithm rather than the price it produces. Economists have pushed back on the stronger claims ([Doctor, Wakker & Wang, 2020](https://doi.org/10.1038/s41567-020-01106-x)), although Peters' reading makes a testable prediction that Bernoulli's does not.

The simulation adds one more lesson. A player who just wants even odds should pay less still, and that discount grows with wealth. The value of the game depends on your circumstances, the rules you play under, and the yardstick you choose.

---

*Revision note (October 2026).* An earlier version labeled `log₂(W) + 1` as exact and treated the gap between a log-growth price and a median price as a cost of the game rules. It also had a "total ruin" variant whose apparent premium came from leftover change, so I removed it.
