---
layout: post
title: "The St. Petersburg Paradox, Re-Run Over Time"
date: 2025-06-28 18:19:00-0700
description: "How an old paradox reveals a deep flaw in classical economics and points the way to a more realistic model of human decision-making."
tags: ["ergodicity", "utility theory", "decision theory"]
categories: ["mathematics", "economics", "computer science"]
related_posts: false
published: true
pretty_table: true
---

I was recently trying to explain the St. Petersburg paradox to my uncle from memory during a walk in the park. The setup is a deceptively simple coin-tossing game.

> A fair coin is flipped until it lands heads for the first time. The payout is determined by the number of flips it takes. If heads appears on the 1st toss, you win $2. If on the 2nd, $4. If on the 3rd, $8, and so on, with the payout doubling each time.

**The question is: What is a fair price to pay to play this game?**

The mathematical answer for the expected payoff is famously, and confusingly, infinite. The calculation sums the probability of each outcome multiplied by its payout:

$$\mathbb{E}[X] = \left(\frac{1}{2}\right)\cdot \$2 + \left(\frac{1}{4}\right)\cdot \$4 + \left(\frac{1}{8}\right)\cdot \$8 + \dots = \sum_{n=1}^{\infty} \frac{1}{2^n} \cdot 2^n = \$1 + \$1 + \$1 + \dots = \infty$$

A purely "rational" actor, according to classical theory, should be willing to pay any finite price to play. Yet, intuitively, no one would. Most people would only offer a few dollars. As I explained this, I found myself questioning the standard explanation.

## The Classical "Solution" and Its Context

The classic solution, proposed by [Daniel Bernoulli in 1738](https://psych.fullerton.edu/mbirnbaum/psych466/articles/bernoulli_econometrica.pdf), is that people don't value money linearly. The "utility" of an extra dollar diminishes as you get wealthier. This **diminishing marginal utility**, often modeled with a logarithmic function, makes the *expected utility* of the game finite, justifying a small price.

Utility theory gives a coherent normative answer and can be derived axiomatically from [Kelly-growth](https://www.princeton.edu/~wbialek/rome/refs/kelly_56.pdf) or isoelastic preference arguments. It remains the benchmark in modern finance. However, it is not the only coherent answer. My intuition suggested a different approach: 

> People subconsciously know they have **finite wealth**. 

To have a real shot at the astronomical prizes that create the infinite expectation (provided the casino has unbounded credit, which is itself unrealistic), you'd need to survive a long string of losses. Most people know their starting stake would be gone long before that lucky streak arrived.

## A Simulation and the Risk Landscape

To test this intuition, I wrote a simulation. The crucial rule, which defines how real people must play, is that **winnings cannot be used to fund more plays**. You arrive with a fixed stake, and when that money is gone, your game is over. This "Stake-Based Model" directly tests the idea that your starting capital is the most important constraint.

The goal was to create a landscape of risk, showing how a player's chance of success changes across different starting stakes and entry prices.

<details>
<summary>Click to see the simulation code</summary>

<pre><code class="language-python">import random
import numpy as np

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
    print("## Simulation Results: Chance of Breaking Even or Profiting\n")
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

Running this code produces a table that tells a clear story:

## Simulation Results: Chance of Breaking Even or Profiting

| Starting Stake | Price = $6 | Price = $8 | Price = $10 | Price = $13 |
|:---|:---|:---|:---|:---|
| **$64** | 49.9% | 31.2% | 21.6% | 14.5% |
| **$256** | 73.7% | 45.9% | 31.0% | 19.1% |
| **$1,024** | 95.6% | 67.4% | 44.0% | 26.1% |
| **$8,192** | 100.0% | 98.0% | 75.6% | 41.1% |

This table vividly confirms the initial intuition. Reading across any row, the chance of profiting plummets as the price increases. Reading down any column, a larger stake dramatically improves a player's odds at a fixed price. The game is clearly not the same for everyone.

## The World of Ergodicity Economics

This line of thinking led me to the work of physicist [**Ole Peters**](https://arxiv.org/abs/1011.4404) and the field of **ergodicity economics**. It turns out there was a formal name for my intuition. The problem wasn't psychology; it was that economists were using the wrong kind of average.

1.  **Ensemble Average:** This is the standard "expected value" ($$\mathbb{E}[X]$$). It's the average outcome if a million people played the game in parallel universes *at the same time*. This is what gives the result of infinity.

2.  **Time Average:** This is the average outcome for *one person* playing the game over and over through time. This is what we actually experience in life.

For many systems, these averages are the same (a property called ergodicity). But for a game like St. Petersburg, they are wildly different. Ergodicity economics argues that a rational person should optimize for the time average. This means maximizing the long-term **growth rate** of their wealth, which, for multiplicative processes, is equivalent to maximizing the expected change in the logarithm of wealth.

### The Exact Growth-Neutral Price

A "fair" price `c` is one that makes the game "growth-neutral." The expected change in your log-wealth should be zero. For a player starting with total wealth `W` who **reinvests their winnings** (where `c ≤ W` is required), we can express this with the following exact equation:

$$\mathbb{E}[\Delta \log(W)] = \sum_{n=1}^{\infty} p_n \cdot \log(W - c + \text{payout}_n) - \log(W) = 0$$

For our game, where the probability $$p_n = 1/2^n$$ and the payout is $$2^n$$ (assuming $1 as our monetary unit for the first head), this becomes:

$$\sum_{n=1}^{\infty} \frac{1}{2^n} \log(W - c + 2^n) = \log(W)$$

This equation is the formal, complete expression for the fair price in a compounding game. However, it cannot be solved algebraically to isolate `c`.

### Approximate Rule

To find a simple, usable rule, we can derive an approximation. The key insight is that the fair price `c` should be related to the number of coin flips required to get a payout that is on the same order of magnitude as your entire wealth, `W`. Let's say this "crossover" happens after `k` flips, making the payout `2^k`. The central assumption for the approximation is that at the fair price, this transformative payout should be roughly equal to your wealth:

$$W \approx 2^k$$

This powerful relationship can be solved for `k` by taking the base-2 logarithm of both sides:

$$\log_2(W) \approx \log_2(2^k) \implies \log_2(W) \approx k$$

However, solving the exact time-average equation numerically reveals that a more accurate approximation for the fair price `c` a person with wealth `W` should pay is:

$$c^*(W) \approx \log_2(W) + 1$$

The additional +1 term comes from the leading-order expansion of the exact equation and shrinks only logarithmically with wealth. This single equation resolves the paradox cleanly. The price is not infinite; it is a finite number that depends directly on a player's wealth.

## Empirical Validation

The `log₂(W) + 1` formula gives us the fair price for the theoretical compounding game. But what is the true fair price for the games we simulated? We created code to find the exact price that gives a 50/50 chance of success for two different scenarios:

1.  **Stake-Based Model:** Our realistic simulation where winnings are kept separate.

2.  **Total Ruin Model:** An even riskier version where running out of stake means you forfeit all winnings.

By comparing the fair prices for these models, we can precisely measure the "risk premium" associated with different rules.

<details>
<summary>Click to see the price-finding code</summary>

<pre><code class="language-python">import random
import math

def simulate_stake_play(start_stake, entry_price):
    stake = float(start_stake)
    winnings = 0.0
    
    while stake >= entry_price:
        stake -= entry_price
        flips = 1
        while random.random() > 0.5:
            flips += 1
        winnings += 2**flips
    
    return winnings + stake

def simulate_total_ruin_play(start_stake, entry_price):
    stake = float(start_stake)
    winnings = 0.0
    
    while stake >= entry_price:
        stake -= entry_price
        flips = 1
        while random.random() > 0.5:
            flips += 1
        winnings += 2**flips
    
    # Total ruin: lose everything if you don't break even
    return winnings + stake if winnings >= start_stake else 0.0

def find_fair_price(game_func, wealth, target_success=0.5, tolerance=0.001):
    low = max(0.01, math.log2(wealth) - 3)
    high = math.log2(wealth)
    
    for _ in range(30):
        price = (low + high) / 2
        if price <= 0:
            low = 0.01
            continue
            
        outcomes = [game_func(wealth, price) for _ in range(10000)]
        success_rate = sum(1 for x in outcomes if x >= wealth) / len(outcomes)
        
        if abs(success_rate - target_success) < tolerance:
            return price
        elif success_rate > target_success:
            low = price
        else:
            high = price
    
    return (low + high) / 2

def compare_fair_prices():
    wealth_levels = [64, 256, 1024, 8192]
    
    print("| Starting Stake | Exact Compounding Price | Fair Price (Stake-Based) | Fair Price (Total Ruin) |")
    print("|:---|:---|:---|:---|")
    
    for w in wealth_levels:
        exact = math.log2(w) + 1
        stake_price = find_fair_price(simulate_stake_play, w)
        ruin_price = find_fair_price(simulate_total_ruin_play, w)
        
        print(f"| **${w:,}** | ${exact:.2f} | ${stake_price:.2f} | ${ruin_price:.2f} |")

if __name__ == "__main__":
    compare_fair_prices()
</code></pre>
</details>

## Final Results: The Risk Premium Quantified

| Starting Stake | Exact Compounding Price | Fair Price (Stake-Based) | Fair Price (Total Ruin) |
|:---|:---|:---|:---|
| **$64** | $7.00 | $5.97 | $5.82 |
| **$256** | $9.00 | $7.63 | $7.53 |
| **$1,024** | $11.00 | $9.39 | $9.31 |
| **$8,192** | $14.00 | $11.96 | $11.96 |

The results clearly show the hierarchy of risk. The exact compounding price (where winnings can be reinvested) represents the theoretical upper bound for a growth-neutral game. As game rules become riskier, fair prices decrease accordingly.

Because compounding reduces risk by allowing reinvestment of winnings, the exact compounding price acts as an **upper bound** above both the stake-based and total ruin prices. Our stake-based model, being more realistic but riskier than full compounding, commands a lower price. The brutal total ruin model requires the lowest price of all.

We can now quantify the risk premium precisely. For a player with $256:

* The premium for separating stake from winnings is `$9.00 - $7.63 = $1.37`.
* The additional premium for facing total ruin is `$7.63 - $7.53 = $0.10`.

## The Shrinking Risk Premium

A careful examination of the results reveals a fascinating pattern. The risk premium between the stake-based and total ruin models shrinks as wealth increases:

* **$64**: $0.15 premium (2.5% of stake price)
* **$256**: $0.10 premium (1.3% of stake price) 
* **$1,024**: $0.08 premium (0.9% of stake price)
* **$8,192**: Essentially zero premium

This convergence is not due to the fat-tailed nature of the payouts, but rather to a fundamental principle from probability theory: **gambler's ruin**.

The total ruin penalty only applies in a very specific scenario: you must go bust (your stake runs out) *and* your cumulative winnings must still be less than your initial wealth. As your wealth increases relative to the entry price, the probability of this specific event shrinks dramatically according to [classical gambler's ruin theory](https://en.wikipedia.org/wiki/Gambler%27s_ruin).

The probability of ruin before hitting a significant win scales roughly as `c/W` (entry price divided by wealth):

* **$64 wealth, $6 price**: ~9.4% chance of the penalty scenario
* **$8,192 wealth, $12 price**: ~0.1% chance of the penalty scenario

As this probability approaches zero, the "insurance value" of keeping winnings separate from stake becomes negligible. Both models converge to essentially the same expected outcome because the scenario that differentiates them almost never happens.

This insight reveals an important distinction in the mathematics:
* **Fat tails** explain why the `log₂(W)` pricing formula works and why wealth matters for fair pricing
* **Gambler's ruin probability** explains why different rule variants converge at high wealth levels

The convergence demonstrates that at sufficient wealth levels, the specific mechanics of how losses are handled become less important than simply having enough capital to survive until the inevitable large payout arrives.

## Conclusion

The paradox dissolves once we specify the averaging procedure. The ensemble average is blind to the realities of time, sequence, and survival that every real person faces.

You don't need a psychological theory of "utility" to explain why people don't pay much to play. You just need to model the game they are actually playing, one with a finite stake and a real risk of ruin. The value of the game is not a universal constant but a dynamic function of your personal circumstances and the specific rules under which you must operate, a conclusion that ergodicity economics formalizes and our simulation empirically verifies.