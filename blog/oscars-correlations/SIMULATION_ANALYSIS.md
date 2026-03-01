# Simulation Method Analysis: What Is This, Really?

This document analyzes the "gamma-ppf" simulation method used in `oscar_correlations.ipynb`, situates it among known statistical methods, explains what makes the gamma distribution special here, and considers simpler alternatives.

## Quick Recap of the Method

1. Draw one $u_f \sim \text{Uniform}(0,1)$ per **film** (shared across all its categories)
2. Transform: $\alpha_{f,c} = F^{-1}_{\text{Gamma}}(u_f;\; \text{shape}=k \cdot p_{f,c})$
3. Normalize within each category → Dirichlet-distributed adjusted probabilities
4. Sample one winner per category from the adjusted probabilities
5. The shared $u_f$ creates positive cross-category correlation

**Key property**: marginal win rates exactly equal the input market probabilities.

---

## The Closest Known Methods

### 1. Dirichlet-Multinomial Compound Model (Bayesian Statistics)

**Similarity: very high — the within-category mechanism is exactly this.**

The Dirichlet-Multinomial is one of the most standard models in Bayesian statistics. You place a Dirichlet prior on the category probabilities, then draw outcomes from a Multinomial. In our case, each category's adjusted probabilities follow:

$$\mathbf{q}_c \sim \text{Dirichlet}(k \cdot p_{1,c},\; k \cdot p_{2,c},\; \ldots,\; k \cdot p_{n,c})$$

and the winner is drawn from $\text{Multinomial}(1, \mathbf{q}_c)$.

This is used in:
- **Topic modeling** (Latent Dirichlet Allocation): each document's topic mixture is Dirichlet, words are Multinomial
- **Bayesian polling models**: voter preferences follow a Dirichlet, observed vote counts are Multinomial
- **Species abundance modeling** in ecology (Dirichlet-Multinomial for overdispersed count data)

**What the Oscar method adds**: coupling *across* categories via the shared $u_f$. A standard Dirichlet-Multinomial treats each category independently. The shared latent uniform is the novel structural ingredient.

### 2. One-Factor Copula Models (Quantitative Finance)

**Similarity: very high — the shared latent variable architecture is essentially the same.**

David Li's 2000 Gaussian copula model for CDO pricing works like this:
1. Each obligor (company) has a latent creditworthiness variable
2. These latents share a common factor: $X_i = \sqrt{\rho}\, Z + \sqrt{1-\rho}\, \epsilon_i$
3. Default occurs when $X_i$ falls below a threshold calibrated to match the marginal default probability
4. The shared factor $Z$ creates correlation across obligors

The Oscar method has the same architecture:
- **Shared factor**: $u_f$ (one per film, shared across categories) plays the role of the common market factor $Z$
- **Marginal calibration**: the gamma inverse CDF maps the shared uniform to a strength score calibrated so that the expected win rate matches the market probability — analogous to calibrating default thresholds to marginal PDs
- **Within-group independence**: within a single category, different films' draws are independent (different $u_f$ values), just as in a copula model, different obligors' idiosyncratic shocks are independent

**Key difference**: the Li copula uses a Gaussian copula (correlation via multivariate normal), while the Oscar method uses the gamma-Dirichlet mechanism. The Oscar method is arguably more elegant for this problem because the Dirichlet renormalization naturally produces a probability vector that sums to 1 — you don't need separate "who wins?" logic after computing latent scores.

The one-factor copula was widely used (and blamed) in mortgage-backed securities pricing in the lead-up to the 2008 financial crisis. The lesson there was that a single correlation parameter poorly captures tail dependence. In our setting, the single shared $u_f$ per film is a similar simplification — it assumes a film's "momentum" in Best Picture is perfectly correlated with its momentum in Best Director.

**Reference**: Li, D.X. (2000). "On Default Correlation: A Copula Function Approach." *Journal of Fixed Income*, 9(4), 43-54.

### 3. Plackett-Luce Model / Gamma Trick (Ranking & Choice Theory)

**Similarity: high — uses the same gamma → normalize → sample pipeline.**

The Plackett-Luce (PL) model is the standard model for rankings. To sample a ranking from PL with strength parameters $\pi_1, \ldots, \pi_k$:

1. Draw $G_i \sim \text{Exponential}(\pi_i)$ independently (equivalently $\text{Gamma}(1, 1/\pi_i)$)
2. Rank items by $G_i$ (lowest first)
3. Probability that item $i$ is ranked first = $\pi_i / \sum_j \pi_j$

There's a well-known generalization: if $G_i \sim \text{Gamma}(r, 1/\pi_i)$, the item with the largest $G_i$ is chosen with probability related to $\pi_i$, and when $r \to 0$, the winner probabilities approach $\pi_i / \sum \pi_j$.

Our method uses $\text{Gamma}(k \cdot p_{f,c}, 1)$ with shape parameters scaled by the concentration $k$. We then renormalize rather than taking the argmax. The renormalization is what gives us the exact Dirichlet, and the Dirichlet mean equals the input probabilities — this is cleaner than the PL argmax approach.

**Connection**: both methods exploit the gamma distribution's role as the "natural noise distribution" for categorical competitions. The Gumbel-max trick (used in modern ML for differentiable sampling) is the limiting case of the gamma trick. Yellott (1977) proved that the Gumbel family is the unique family of iid noise distributions consistent with Luce's Choice Axiom, establishing the deep equivalence between the gamma and Gumbel representations.

**References**:
- Yellott, J.I. (1977). "The relationship between Luce's choice axiom, Thurstone's theory of comparative judgment, and the double exponential distribution." *Journal of Mathematical Psychology*, 15(2), 109-144.
- Caron, F. & Doucet, A. (2012). "Efficient Bayesian Inference for Generalized Bradley-Terry Models." *Journal of Computational and Graphical Statistics*, 21(1), 174-196.

### 4. Bradley-Terry with Random Effects (Sports Analytics)

**Similarity: moderate — same conceptual motivation, different mechanism.**

In sports analytics, the Bradley-Terry model assigns each team a strength parameter and computes pairwise win probabilities as $P(i \text{ beats } j) = \pi_i / (\pi_i + \pi_j)$. To model "hot streaks" or "form," analysts add random effects:

$$\log \pi_i = \mu_i + z_i, \quad z_i \sim N(0, \sigma^2)$$

where $z_i$ is a shared random effect for team $i$ across all its matches in a tournament.

This is conceptually identical to our method: a shared latent draw per team/film that affects all its competitions. The difference is in the distribution used (log-normal vs. gamma) and how it connects to the within-competition probability model.

Bradley-Terry random effects models are used for:
- Chess (Elo + variance models like Glicko)
- Tennis tournament prediction
- FIFA World Cup simulations (e.g., FiveThirtyEight's models)
- eSports ranking

**Important**: these models generally do *not* exactly preserve marginal win probabilities after adding random effects, because the log-normal/normal perturbation introduces bias through Jensen's inequality. The gamma-Dirichlet method's exact marginal preservation is its key advantage.

### 5. Thurstone / Random Utility Models (Psychometrics & Discrete Choice)

**Similarity: moderate — same "add noise, pick the best" structure.**

In Thurstone's model of comparative judgment (1927):
1. Each item has a "perceived quality" that varies randomly: $V_i = \mu_i + \epsilon_i$
2. The decision-maker picks the item with the highest perceived quality
3. The distribution of $\epsilon$ determines the model:
   - $\epsilon \sim \text{Gumbel}$ → multinomial logit (McFadden, 1974)
   - $\epsilon \sim \text{Normal}$ → multinomial probit

Our method is a Thurstone-type model where:
- $\epsilon$ comes from a gamma distribution (via the shared $u_f$)
- Instead of argmax, we normalize and sample (which is softer — every nominee has a positive probability of winning, even in a single simulation)

The renormalize-and-sample approach is equivalent to the **Gumbel-softmax** / **concrete distribution** used in modern deep learning for differentiable categorical sampling (Jang et al., 2017; Maddison et al., 2017).

### 6. Election Forecasting Models (FiveThirtyEight, DDHQ)

**Similarity: very high — DDHQ uses nearly identical machinery.**

Election forecasting faces the exact same structural problem: simulate correlated outcomes (state-by-state results) while preserving marginal forecasts (each state's individual probability).

**Decision Desk HQ (DDHQ)** published a model in the *Harvard Data Science Review* (2022) that uses strikingly similar mathematics:
- **Gamma distributions** represent candidate strength at the county level
- Counties are **Dirichlet distributed** (via gamma normalization)
- A **Gaussian copula** induces cross-county correlation
- The model preserves marginal vote share estimates while generating realistic correlation

This is essentially the same recipe: gamma variates → Dirichlet normalization → copula-induced correlation through shared latent variables. The DDHQ model is more complex (Gaussian copula with a full correlation matrix vs. our one-factor shared-uniform structure), but the core mathematical mechanism is identical.

**FiveThirtyEight** uses a related but different approach: shared error components decomposed along multiple dimensions (national swing, regional factors, demographic factors). This is a multi-factor model rather than the one-factor-per-entity structure used here, but the conceptual motivation is the same.

**References**:
- DDHQ (2022). "Psephological Correlated Simulation Techniques With Decision Desk HQ." *Harvard Data Science Review*, 4(4).
- Silver, N. / FiveThirtyEight (2020). "How FiveThirtyEight's 2020 Presidential Forecast Works."

### 7. Same-Game Parlays (Sports Betting)

**Similarity: moderate — same structural problem, typically solved with Gaussian copulas.**

Sportsbooks face a closely related problem when pricing same-game parlays (SGPs): multiple correlated bets within a single game (e.g., "Player A scores 20+ AND Team A wins"). Each leg has a known marginal probability from the individual market, but legs are correlated. The joint probability must respect the marginals.

The industry standard is **Gaussian copulas** — structurally identical to the Li CDO model. The Oscar method solves the same problem but with the gamma-Dirichlet mechanism instead of a Gaussian copula.

### 8. Correlated Polya Urn / Exchangeable Partition Models (Probability Theory)

**Similarity: conceptual — different formalism, same flavor of outcome.**

In Bayesian nonparametrics, correlated random probability measures (e.g., correlated Dirichlet processes) generate coupled discrete distributions. The idea of "sharing a base measure" across multiple categories is analogous to sharing $u_f$ across categories. These appear in:
- Multi-task learning
- Hierarchical topic models
- Phylogenetic models

The connection is more abstract — our model is finite and parametric, while these are infinite-dimensional — but the motivation is the same: induce correlation across related discrete distributions.

---

## What Is Special About the Gamma?

The gamma distribution is the unique distribution (up to scale) with the property that **independent gamma variables, when normalized, produce a Dirichlet distribution whose expected value equals the normalized shape parameters**. This is the gamma-Dirichlet conjugacy, and it's what makes the whole method work.

### Why other distributions fail

| Distribution | What goes wrong |
|---|---|
| **Normal (additive)** | After adding $z \sim N(0,\sigma)$ to $p_{f,c}$ and renormalizing, $E[q_{f,c}] \neq p_{f,c}$. The nonlinearity of clipping (to avoid negatives) and renormalization creates systematic bias: favorites get boosted, underdogs get suppressed. This is exactly what the "naive" method demonstrates. |
| **Log-normal (multiplicative)** | If $\alpha_{f,c} = p_{f,c} \cdot e^{z_f}$ and you renormalize, the $e^z$ factor cancels in the ratio — you get back the original probabilities with *no* randomness. The correlation does nothing. You need different shape parameters per nominee to get any variation. |
| **Uniform** | Drawing $\alpha_{f,c} \sim \text{Uniform}(0, p_{f,c})$ and normalizing: $E[\alpha_{f,c}/\sum \alpha] \neq p_{f,c}$ in general. The uniform doesn't have the right moment structure. |
| **Beta** | The Beta distribution works for *two*-outcome categories (it's the 2D Dirichlet), but doesn't generalize to k-way competitions without introducing the Dirichlet, at which point you're back to gamma. |

### The mathematical property

If $X_i \sim \text{Gamma}(a_i, \theta)$ independently, then (for any $\theta > 0$):

$$\frac{X_i}{\sum_j X_j} \sim \text{Dirichlet}(\mathbf{a}), \quad E\left[\frac{X_i}{\sum_j X_j}\right] = \frac{a_i}{\sum_j a_j}$$

Setting $a_i = k \cdot p_{f,c}$ (with market probabilities summing to 1 within each category):

$$E[q_{f,c}] = \frac{k \cdot p_{f,c}}{\sum k \cdot p_{f',c}} = p_{f,c}$$

The $k$ cancels in the ratio, so **marginals are preserved for any $k > 0$**. And it holds for **any number of competitors** in the category. No other standard distribution family has this property.

### Role of $k$ (concentration parameter)

The parameter $k$ controls the **concentration** of the Dirichlet and therefore the **cross-category correlation strength**:

- **$k < 1$**: Dirichlet is dispersed — per-sim adjusted probabilities swing wildly from the base probs. The shared $u_f$ has a large effect, producing **stronger correlation** across categories. At the extreme ($k \to 0$), each sim puts nearly all weight on one nominee.
- **$k = 1$**: Default. The Dirichlet parameters equal the raw market probabilities. With $\sum p_{f,c} = 1.0$, the Dirichlet is already fairly dispersed.
- **$k > 1$**: Dirichlet concentrates near the base probability vector. The shared $u_f$ barely changes the adjusted probabilities, so correlation is **weaker**. At the extreme ($k \to \infty$), every sim uses the base probabilities — effectively independent.

Crucially, $k$ only affects the **joint distribution** across categories, not the within-category marginals. Within any single category, the winner distribution is $\text{Categorical}(p_1, \ldots, p_n)$ regardless of $k$.

---

## Is There a Simpler Approach?

### What "simpler" could mean

The gamma-ppf method is already quite simple — it's a ~30-line function. But the gamma inverse CDF may feel like a mysterious black box. Here are alternatives that are conceptually simpler:

### Alternative 1: Direct Dirichlet Sampling (Simplest Equivalent)

Since the entire gamma → normalize pipeline produces a Dirichlet, you can skip the gamma and sample the Dirichlet directly:

```python
from numpy.random import dirichlet

# Within each category, for each simulation:
q = dirichlet(probabilities)  # q[i] = adjusted probability for nominee i
winner = np.random.choice(nominees, p=q)
```

To add cross-category correlation, you need the shared $u_f$ trick — you can't just sample Dirichlets independently. But you could implement it as:

1. For each film, draw $u_f \sim \text{Uniform}(0,1)$
2. For each category, compute $\alpha_{f,c} = \text{Gamma.ppf}(u_f, k \cdot p_{f,c})$ and normalize

...which is exactly what the current method does. **There is no shortcut around the gamma step** if you want correlated Dirichlets via shared uniforms. The gamma is the *mechanism* by which the Dirichlet is constructed from shared latent variables.

### Alternative 2: Gaussian Copula + Categorical Inverse CDF

This is the "industry standard" approach from finance:

1. Draw $\mathbf{z}_f \sim N(0, 1)$ per film (shared factor)
2. For each (film, category), compute $u_{f,c} = \Phi(z_f)$ (probability integral transform)
3. Map $u_{f,c}$ to a categorical outcome using the inverse CDF of the discrete distribution

This preserves marginals by construction (it's a copula) and is conceptually clean. But it has drawbacks:
- You need to explicitly construct the inverse CDF for each category's multinomial
- The correlation structure is Gaussian (no tail dependence) — may not be appropriate
- It doesn't naturally produce a "probability vector" per simulation — each category is handled separately

**Verdict**: slightly more complex to implement for this problem, but well-understood and widely used.

### Alternative 3: Common Shock Model

The simplest possible correlation mechanism:

1. With probability $\rho$: draw a single $u \sim \text{Uniform}(0,1)$, use it for *all* categories (extreme positive correlation)
2. With probability $1-\rho$: draw independent $u_{f,c}$ per (film, category) (no correlation)

Within each category, use the $u$ to sample a winner via the categorical inverse CDF.

**Pros**: trivially simple, preserves marginals, $\rho$ directly controls correlation strength.
**Cons**: binary on/off correlation (all-or-nothing), no fine-grained control, doesn't naturally produce the "adjusted probability" intermediate that's useful for analysis.

### Alternative 4: Gumbel-Max with Shared Perturbation (Mathematically Equivalent)

By the Yellott/Thurstone/Luce equivalence, you can replace the gamma-ppf step with:

```python
# For each (film, category):
score = np.log(probability) + shared_gumbel_noise_per_film
# Then softmax across nominees in each category to get adjusted probabilities
```

This is mathematically equivalent to the gamma method (same distribution over winners) and avoids the `gamma.ppf` computation, which can have numerical issues for very small shape parameters. However, the gamma representation has the advantage of producing a proper Dirichlet distribution after normalization, making the probability interpretation more direct.

This is also closely related to the **Gumbel-Softmax** / **Concrete distribution** (Jang et al., 2017; Maddison et al., 2017) used in modern deep learning for differentiable categorical sampling.

### Alternative 5: Correlated Logistic-Normal (Most Flexible)

1. For each film, draw a shared shock $z_f \sim N(0, \sigma^2)$
2. For each (film, category), compute log-odds: $\ell_{f,c} = \log(p_{f,c} / (1 - p_{f,c})) + z_f$
3. Convert back to probabilities: $\tilde{p}_{f,c} = \text{sigmoid}(\ell_{f,c})$
4. Renormalize within category: $q_{f,c} = \tilde{p}_{f,c} / \sum \tilde{p}$
5. Sample winner from $q$

**Pros**: familiar to anyone who's worked with logistic regression, flexible correlation structure, $\sigma$ controls correlation strength.
**Cons**: **does not preserve marginals** — $E[\text{sigmoid}(\text{logit}(p) + z)] \neq p$ due to Jensen's inequality. You'd need to calibrate the base probabilities to compensate, adding complexity. The bias is smaller than the naive additive method but still present.

---

## Summary: Where This Method Sits

| Method | Preserves Marginals | Correlation Mechanism | Complexity | Used In |
|--------|--------------------|-----------------------|------------|---------|
| **Gamma-PPF (this method)** | Exact | Shared uniform → gamma → Dirichlet | Low | Oscar sims, election forecasting (DDHQ) |
| Dirichlet-Multinomial | Exact (within category) | None (independent categories) | Very low | Bayesian statistics, NLP, ecology |
| One-factor Gaussian copula | Exact | Shared normal factor | Medium | CDO pricing, credit risk, same-game parlays |
| Plackett-Luce / gamma trick | Approximate | Independent gammas → argmax | Low | Ranking, recommendation systems |
| Gumbel-max (shared perturbation) | Exact | Shared Gumbel + softmax | Low | ML (Gumbel-Softmax), equivalent to gamma-ppf |
| Bradley-Terry + random effects | Approximate | Shared log-normal | Medium | Sports analytics, chess ratings |
| Thurstone / multinomial probit | Approximate | Correlated normals | High | Psychometrics, discrete choice |
| Common shock | Exact | Binary mixture | Very low | Insurance, simple risk models |
| Logistic-normal | No (Jensen's inequality) | Shared normal on logit scale | Medium | Topic modeling, microbiome analysis |
| Naive additive | No | Shared normal, additive | Very low | Ad-hoc models (not recommended) |

### The Bottom Line

The gamma-ppf method is best understood as a **one-factor Dirichlet-Multinomial model**. It combines:
- A **within-category** mechanism from Bayesian statistics (Dirichlet-Multinomial)
- A **cross-category coupling** mechanism from quantitative finance (shared latent factor, as in copula models)
- A **noise distribution** from ranking theory (gamma, as in Plackett-Luce)

Each of these pieces is well-known. The combination is natural but not widely published as a named method — it exists in scattered forms across sports analytics blogs, Bayesian modeling papers, and the election forecasting community (where correlated categorical outcomes are a common problem).

The gamma distribution is special because it is the **unique** distribution whose normalized independent draws have expected values equal to the normalized shape parameters (the Dirichlet mean property). This is why it preserves marginals without any calibration — a property that no other standard distribution shares after the nonlinear normalization step.

There is no fundamentally simpler approach that achieves the same three goals simultaneously:
1. **Exact marginal preservation** (simulated win rates = market probabilities)
2. **Positive cross-category correlation** (a film's wins are correlated across categories)
3. **Per-simulation probability vectors** (useful for analysis and interpretability)

The Gaussian copula (Alternative 2) and common shock model (Alternative 3) achieve goals 1 and 2 but not 3. The logistic-normal (Alternative 5) achieves 2 and 3 but not 1. The Gumbel-max formulation (Alternative 4) achieves all three and is mathematically equivalent — it's the same method viewed through a different lens.

The gamma-ppf method is not novel in the sense that its components are all well-known. But the specific combination — shared uniforms → gamma inverse CDF → Dirichlet normalization → multinomial sampling, applied to multi-category competitions — is a natural and elegant construction that sits at the intersection of copula theory, Bayesian statistics, and ranking models. The closest published analogue is DDHQ's election forecasting model (2022), which uses the same gamma-Dirichlet core with a more complex (Gaussian) copula structure for the cross-entity correlation.

# Temp notes

The conjugacy is a red herring

  What you're actually exploiting is much simpler. It's two properties, and neither one is about conjugacy:

  Property 1 (the moment property): If $\mathbf{q} \sim \text{Dirichlet}(a_1, \ldots, a_n)$, then $E[q_i] = a_i / \sum a_j$. So if $a_i = k \cdot p_i$ where the $p_i$ sum to 1, then $E[q_i] = k \cdot p_i / \sum k \cdot p_j = p_i$. This holds for any $k > 0$.

  Property 2 (the construction property): If you draw $X_i \sim \text{Gamma}(a_i, \theta)$ independently and normalize, you get a Dirichlet. This is the only
  standard distribution with a "draw independently, then normalize" recipe that yields a known distribution with the right mean.

  That's the whole story. The gamma is special because of Property 2, and the Dirichlet is useful because of Property 1. Conjugacy to the multinomial is a     
  coincidence — a different nice property of the same distribution that happens not to matter here.

  How to think about it instead

  The right framing is: you need a way to add randomness to a probability vector while keeping the average equal to the original probabilities.

  Think of it as a two-step process per category:

  1. Perturb the probability vector $(p_1, \ldots, p_k)$ into a random probability vector $(q_1, \ldots, q_k)$
  2. Sample one winner from the perturbed vector

  For step 2 to give the right marginals, you need step 1 to satisfy $E[q_i] = p_i$. That's the constraint.

  There are many distributions over probability vectors (the simplex). But most of them either:
  - Don't have $E[q_i] = p_i$ after some perturbation + renormalization
  - Can't be constructed from independent per-nominee draws (which you need for the shared-$u$ correlation trick)

  The Dirichlet is the unique distribution that satisfies both. And the gamma is the mechanism that builds a Dirichlet from independent pieces — pieces that   
  you can then couple across categories via the shared $u_f$.
  Does it have a Bayesian interpretation?

  You can squint and see one, but it's a stretch:

  - Treat the market probabilities as a "prior" over who wins
  - Each simulation's $u_f$ represents an unobserved "state of the world" (film momentum)
  - The Dirichlet-perturbed vector is the "updated belief" given this latent state
  - Sampling a winner is the "posterior predictive"

  But this is retrofitting a Bayesian narrative onto something that's really just a moment-matching trick. The honest description is: the Dirichlet is a       
  convenient distribution over the simplex whose mean you can control, and the gamma gives you a way to construct it from independent components that you can  
  strategically correlate.