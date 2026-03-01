# How the Oscar Simulations Work

This document explains the correlated simulation method used to estimate how many Oscars each film will win, given per-category win probabilities from prediction markets (Kalshi).

## The Problem

Prediction markets give us the probability that a film wins each individual category (e.g., Conclave has a 30% chance of Best Picture, 25% chance of Best Director, etc.). But they don't directly tell us: **what's the probability that Conclave wins 3 or more Oscars total?**

You can't just multiply the per-category probabilities together, because Oscar wins are **correlated** — if a film wins Best Picture, it's more likely to also win Best Director, Best Editing, etc. There's a shared "momentum" or "it's that film's year" effect. A naive independent model would underestimate the chance of both sweeps and shutouts.

## The Gamma-PPF Method (Step by Step)

### 1. Start with market probabilities

For each nominee in each category, we have a Kalshi-derived probability. For example:

| Category | Film | Probability |
|----------|------|-------------|
| Best Picture | Conclave | 0.30 |
| Best Picture | The Brutalist | 0.25 |
| Best Director | Conclave | 0.20 |
| Best Director | The Brutalist | 0.35 |

### 2. Draw one random number per film (not per nominee)

For each simulation, draw a single uniform random number `u` between 0 and 1 for each **film**. This same `u` is shared across every category that film is nominated in.

This is the key insight: a film that draws a high `u` gets a boost in *all* its categories simultaneously. A film that draws a low `u` gets suppressed everywhere. This creates realistic cross-category correlation — the "momentum" effect.

### 3. Convert to adjusted strengths via the gamma distribution

For each (film, category) pair, convert the shared `u` into an adjusted "strength" score:

```
strength = gamma.ppf(u, shape=k * probability)
```

- `gamma.ppf` is the inverse CDF (percent-point function) of the gamma distribution
- The **shape parameter** is `k * probability`, where `k` is the Dirichlet concentration parameter
- `k` controls cross-category correlation strength:
  - `k < 1`: stronger correlation (dispersed Dirichlet — per-sim probabilities swing wildly, so the shared `u` dominates)
  - `k = 1`: default
  - `k > 1`: weaker correlation (concentrated Dirichlet — adjusted probabilities stay near base probs, shared `u` barely matters)
  - `k → ∞`: effectively independent (Dirichlet collapses to the base probability vector)

**Why gamma?** The gamma distribution has a critical property: after renormalization (step 4), the expected win rate for each nominee exactly matches the original market probability, **regardless of `k`**. This means `k` is a clean correlation knob — it only affects the joint distribution across categories, not the within-category marginals.

### 4. Renormalize within each category

The raw strengths don't sum to 1 within a category, so we renormalize:

```
adjusted_prob = strength / sum(strengths in this category)
```

Now each nominee has an adjusted probability for this particular simulation run.

### 5. Pick one winner per category

Sample one winner from each category using the adjusted probabilities. In a simulation where a film drew a high `u`, it will have elevated probabilities across all its categories — and is more likely to win multiple awards.

### 6. Repeat 10,000 times

Across all simulations, count how many times each film wins 0, 1, 2, 3, ... Oscars. This gives us a full win distribution.

## The Math

### Setup and notation

We have $C$ categories and $F$ films. Let $p_{f,c}$ denote the market probability that film $f$ wins category $c$ (zero if not nominated). Within each category, probabilities sum to 1:

$$\sum_{f} p_{f,c} = 1 \quad \text{for all } c$$

Our goal: construct a joint distribution over all category outcomes that (a) reproduces these marginal win probabilities exactly, and (b) introduces positive correlation across categories for the same film.

### Step 1: Shared uniform draws

For each simulation, draw one latent variable per film:

$$u_f \sim \text{Uniform}(0, 1), \quad \text{independently across films}$$

The same $u_f$ is reused in every category where film $f$ appears. This is the sole source of cross-category correlation.

### Step 2: Gamma inverse-CDF transform

Convert each $(f, c)$ pair's shared uniform draw into a "strength" score using the gamma distribution's inverse CDF (percent-point function):

$$\alpha_{f,c} = F_{\text{Gamma}}^{-1}\!\left(u_f;\; \text{shape} = k \cdot p_{f,c}\right)$$

where $k > 0$ is the Dirichlet concentration parameter. Equivalently, this is the same as drawing $\alpha_{f,c} \sim \text{Gamma}(k \cdot p_{f,c},\, 1)$ but with the randomness coming through $u_f$ rather than a fresh draw.

**Key observation**: within a single category, different nominees come from different films, so their $u_f$ values are independent. That means within one category, the $\alpha$ values are independent gamma random variables:

$$\alpha_{f,c} \sim \text{Gamma}(k \cdot p_{f,c},\, 1) \quad \text{independently across nominees in category } c$$

### Step 3: Renormalize → Dirichlet

Within each category, normalize the strengths to get adjusted probabilities:

$$q_{f,c} = \frac{\alpha_{f,c}}{\sum_{f' \in c} \alpha_{f',c}}$$

This is where the well-known **gamma–Dirichlet connection** comes in. If you have independent gamma random variables $X_i \sim \text{Gamma}(a_i, \theta)$, then the vector of ratios $(X_1/S, \ldots, X_n/S)$ where $S = \sum X_i$ follows a $\text{Dirichlet}(a_1, \ldots, a_n)$ distribution. Crucially, this holds regardless of the scale $\theta$ (it cancels out in the ratios).

So within each category, the adjusted probabilities follow:

$$(q_{f,c})_{f \in c} \sim \text{Dirichlet}(k \cdot p_{1,c},\; k \cdot p_{2,c},\; \ldots,\; k \cdot p_{n,c})$$

The parameter $k$ controls the concentration of this Dirichlet. Small $k$ produces wildly varying probability vectors across simulations; large $k$ keeps them close to the base probabilities $(p_{1,c}, \ldots, p_{n,c})$.

### Step 4: Why marginals are preserved

The Dirichlet distribution has the property:

$$\mathbb{E}[q_{f,c}] = \frac{k \cdot p_{f,c}}{\sum k \cdot p_{f',c}} = \frac{p_{f,c}}{\sum p_{f',c}} = p_{f,c}$$

The $k$ cancels in the ratio, so **marginals are preserved for any $k > 0$**.

since the market probabilities already sum to 1. We then sample one winner per category from the adjusted probabilities:

$$\text{winner}_c \sim \text{Multinomial}(1,\; q_{\cdot,c})$$

By the law of iterated expectations:

$$P(\text{film } f \text{ wins category } c) = \mathbb{E}[q_{f,c}] = p_{f,c}$$

So the simulation's marginal win rate for every nominee matches the market probability exactly, in expectation. This is the central guarantee: **we add correlation without distorting the inputs**.

### Step 5: Where correlation comes from

Across categories, the $q$ vectors are **not** independent — they're coupled through the shared $u_f$.

When $u_f$ is high (close to 1), $F_{\text{Gamma}}^{-1}(u_f;\, p_{f,c})$ is large for every category $c$, giving film $f$ an elevated $q_{f,c}$ everywhere. When $u_f$ is low, the film is suppressed everywhere. This produces positive correlation:

$$\text{Corr}\!\left(\mathbb{1}[f \text{ wins } c_1],\; \mathbb{1}[f \text{ wins } c_2]\right) > 0$$

The independent baseline uses a separate $u_{f,c} \sim \text{Uniform}(0,1)$ per (film, category), breaking this coupling. Within each category the math is identical — the same Dirichlet, the same marginals — but the cross-category correlation vanishes.

### Summary of the probabilistic model

| Component | Distribution | Drives |
|-----------|-------------|--------|
| $u_f$ | $\text{Uniform}(0,1)$, one per film | Cross-category correlation |
| $\alpha_{f,c}$ | $\text{Gamma}(k \cdot p_{f,c},\, 1)$ via inverse-CDF of $u_f$ | Strength score |
| $(q_{\cdot,c})$ | $\text{Dirichlet}(k \cdot p_{1,c}, \ldots, k \cdot p_{n,c})$ | Adjusted probabilities |
| $\text{winner}_c$ | $\text{Multinomial}(1,\; q_{\cdot,c})$ | Category outcome |

The full joint distribution over all 20 category winners is **not** Dirichlet in any simple sense — it's a mixture induced by the shared latent $u_f$ variables. But the construction guarantees that every marginal is correct (for any $k$) and the cross-category correlation is positive and realistic (controlled by $k$).

## Why This Beats Simpler Approaches

### vs. Independent simulation

Drawing a separate random number per (film, category) — rather than one shared number per film — removes all cross-category correlation. The independent model produces tighter distributions: it underestimates both the chance of winning zero awards and the chance of sweeping many.

### vs. Naive additive method

A simpler approach might add a random "momentum" shock directly to each probability (`adjusted = prob + noise`), then renormalize. This is intuitive but **breaks the marginal probabilities** — it systematically overestimates favorites and underestimates underdogs. The gamma-ppf method avoids this problem entirely; its marginals match the market inputs almost exactly.

## Validating the Results

The simulation is validated by running 50,000 simulations and comparing each nominee's simulated win rate against the original market probability. The gamma-ppf method achieves near-zero mean absolute error, confirming that the correlation structure is added without distorting the underlying probabilities.

## How to Read the Output

The key output is a table like:

| Film | E[wins] | P(0 wins) | P(3+ wins) |
|------|---------|-----------|------------|
| Conclave | 2.1 | 0.15 | 0.45 |

- **E[wins]**: Expected (average) number of Oscars across all simulations
- **P(0 wins)**: Probability of a total shutout
- **P(3+ wins)**: Probability of winning 3 or more — the "sweep" scenario

These distributions can then be compared against Manifold prediction markets that ask "how many Oscars will Film X win?" to find potential mispricings.



# Additional notes (Dylan)

Minimal TL;DR reminder.

- We can simulate a Dirichlet distribution using independent gamma draws (and normalizing so they sum to 1).
- The Dirichlet is the conjugate prior for the multinominal (which is what we actually draw winners from). 