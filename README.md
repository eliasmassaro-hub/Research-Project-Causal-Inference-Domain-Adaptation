# Causal Domain Adaptation — Transporting the ATE across domains

> **Status: work in progress.** Personally inotiated research project under the supervision of
Prof. Clausel (CRAN) and and Dr. A. Poinsot (Lead of Research, Ekimetrics).
> Code and report are still being written; the repository layout below is the target one.

Domain adaptation (DA) asks how a predictor trained on a source domain behaves on a shifted
target domain. Causal transportability (Pearl & Bareinboim) asks when a causal quantity
estimated in a source population is valid in another. This project studies the intersection:
**what does the causal structure of a shift tell us about which DA correction is legitimate?**

---

## 1. Problem statement

Two branches must not be confused:

| | Object | Nature of the problem |
|---|---|---|
| **Statistical transportability** | $P^*(y \mid z)$ | estimable given target labels — a *statistical* problem (classical DA) |
| **Causal transportability** | $P^*(y \mid do(x))$ | in general **not identifiable even with infinite target data** — an *identification* problem |

Standard DA lives in the first branch; the 2013 Bareinboim–Pearl machinery (do-calculus,
$s$-hedge, sID) lives in the second. This project works on the bridge between them, with a
focus on transporting the **Average Treatment Effect (ATE)** under three data regimes.

## 2. Working model

A Gaussian SCM with a continuous treatment, heterogeneous effect and an optional hidden parent:

```
C = μ + σ·N_C                                     (confounder, observed at source)
W = N_W                                           (latent parent, absent from the source model)
T = α·C + δ·W + σ_T·N_T                           (treatment / dose)
Y = b_C·C + T·τ(C) + γ·W + σ_Y·N_Y                (outcome)

τ(c) = τ₀ + τ₁·c + τ₂·c²                          (heterogeneous unit effect)
```

Two closed forms drive everything (both verified by Monte-Carlo, $3\cdot10^5$ points):

$$\mathrm{ATE} = \tau_0 + \tau_1\mu + \tau_2(\mu^2+\sigma^2)$$

$$\lambda \;=\; \beta_{\text{naive}} - \mathrm{ATE} \;=\; \frac{\alpha\sigma^2\big(b_C + \alpha\,\mathbb{E}[C\tau'(C)]\big) + \gamma\delta}{\alpha^2\sigma^2 + \delta^2 + \sigma_T^2}$$

where $\beta_{\text{naive}} = \mathrm{Cov}(T,Y)/\mathrm{Var}(T)$ and
$\mathbb{E}[C\tau'(C)] = \tau_1\mu + 2\tau_2(\mu^2+\sigma^2)$ (Stein's lemma).

**The structuring fact:** $\mathrm{ATE}$ depends only on $(\tau_0,\tau_1,\tau_2,\mu,\sigma)$,
while $\lambda$ depends on everything *except* $\tau_0$ and $\sigma_Y$. The two supports do not
overlap — hence a clean taxonomy of shifts.

## 3. Main results

### 3.1 Three transport regimes

| Regime | Critical assumption | Target data required | Target estimand |
|---|---|---|---|
| **(P)** population shift | $\tau$ invariant, overlap | unlabelled $C$ | density ratio $w = p^*/p$ |
| **(A)** additive mechanism shift | $(C,T)$ invariant, $\Delta\tau$ constant | $(T,Y)$, **no** $C$ | one scalar $\beta^*$ |
| **(G)** general mechanism shift | $(C,T)$ invariant, $g$ invariant | $(T,Y)$, **no** $C$ | full regression $\mathbb{E}^*[Y\mid T]$ |

- **(P)** — $\mathrm{ATE}(\mathcal{M}^*) = \mathbb{E}^*[\tau(C)] = \mathbb{E}[w(C)\tau(C)]$.
  The causal transport formula and the covariate-shift importance weighting of statistical
  learning are literally the same number read from two sides. The treatment-assignment
  mechanism may change arbitrarily.
- **(A)** — $\mathrm{ATE}(\mathcal{M}^*) = \mathrm{ATE}(\mathcal{M}) + (\beta^*-\beta)$.
  A constant shift has zero derivative, so the (non-identifiable) confounding bias $\lambda$
  cancels by differencing — difference-in-differences, transposed from time to domains.
  The confounder need not be measured in the target.
- **(G)** — $\mathrm{ATE}(\mathcal{M}^*) = \mathrm{ATE}(\mathcal{M}) + \mathbb{E}\big[\big(\mathbb{E}^*[Y\mid T]-\mathbb{E}[Y\mid T]\big)/T\big]$.
  No Gaussianity, no parametrisation, no constraint on $\Delta\tau$ — same data cost as (A),
  paid instead in estimation complexity (and instability near $T=0$).

No regime dominates: (P) buys freedom on the treatment side at the price of target access to
$C$; (A)/(G) buy the absence of $C$ at the price of invariance on the treatment side.

### 3.2 Partial identifiability

$(\mathrm{ATE}, \lambda)$ sees the mechanism shift $\Delta\tau$ **only through its projection
on $\mathrm{span}\{1, C, C^2\}$ in $L^2(P_C)$.** Any shift orthogonal to that 3-dimensional
subspace is strictly invisible from naive slopes. This is the bound that motivates moving from
the scalar $\beta^*$ to the full regression $\mathbb{E}^*[Y\mid T]$.

### 3.3 Taxonomy of shifts — which correction is legitimate?

| Perturbed parameter | $\Delta\mathrm{ATE}$ | $\Delta\lambda$ | do nothing | reweight $w=p^*/p$ | transport the slope |
|---|---|---|---|---|---|
| $\mu,\sigma$ (population) | ≠0 | ≠0 | ✗ | **✓** | ✗ |
| $\tau_0$ (additive) | ≠0 | 0 | ✗ | ✗ | **✓** |
| $\tau_1$ (slope) | ≠0 | $\propto\mu$ | ✗ | ✗ | ✓ iff $\mu=0$ |
| $\tau_2$ (curvature) | ≠0 | ≠0 | ✗ | ✗ | ✗ |
| $b_C, \alpha, \sigma_T$ | **0** | ≠0 | **✓** | ✓ | ✗ |
| $\sigma_Y$ | 0 | 0 | ✓ | ✓ | ✓ |
| hidden $W$ ($\gamma\delta \neq 0$) | 0 | ≠0 | **✓** | ✓ | ✗ |

Empirical biases (200 replications) match $-\Delta\mathrm{ATE}$ and $\Delta\lambda$ to $10^{-2}$.

**Three take-aways.**

1. *Transporting the naive slope is aggressive.* In 5 of 10 scenarios the ATE is already
   transportable as is, and the method **injects** bias (down to $-1.49$). Reweighting is
   conservative: $w \equiv 1$ when $P(C)$ does not move.
2. *The arrow $S \to Y$ of a selection diagram is too coarse.* It covers $\tau_0,\tau_1,\tau_2$
   and $b_C$, which have opposite consequences → ongoing work on **typed selection diagrams**
   (additive / modulator / off-effect arrows).
3. *Invariance of the estimand ≠ invariance of the statistic.* $\alpha,\sigma_T,b_C$ leave the
   ATE untouched but move $\beta_{\text{naive}}$. Any DA method calibrated on a marginal
   statistic (feature divergence, naive slope) may "correct" a shift that does not exist.

## 4. Open directions

- **The $P^*(X)$ gap.** Jalaldoust & Bareinboim (AAAI 2024) work in *domain generalisation*
  (zero target data); unsupervised DA gets $P^*(X)$ for free. A graphical criterion for
  "transportable given $P^*(X)$ but without $Y^*$" appears to be missing.
- **Diagram-structured optimal transport.** OT-DA aligns blindly with a uniform ground cost;
  the selection diagram says *which* coordinates are allowed to move. Structuring the OT cost
  with the diagram is, as far as we found, unexplored.
- **When alignment hurts.** A causal diagnosis of Zhao et al. (ICML 2019).
- Binary treatment; partially observed $C$ in the target (proxies); non-parametric shifts
  $\|\Delta\tau\| \le \varepsilon$ with worst-case bounds; target-label budget.

## 5. Repository layout (target)

```
.
├── report/                 # LaTeX source + compiled PDF of the report
│   ├── ate_transport_notes.tex
│   └── typed_selection_diagrams.tex
├── notebooks/
│   └── ATE_Transport.ipynb # SCM simulations, closed-form checks, shift taxonomy
├── src/
│   ├── scm.py              # sampling from the Gaussian SCM
│   ├── estimators.py       # naive slope, reweighting, slope transport, (G) estimator
│   └── experiments.py      # replication study over the shift taxonomy
├── figures/
├── requirements.txt
└── README.md
```

## 6. Getting started

```bash
git clone https://github.com/<user>/<repo>.git
cd <repo>
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter lab notebooks/ATE_Transport.ipynb
```

Python ≥ 3.10. Core dependencies: `numpy`, `scipy`, `pandas`, `statsmodels`, `networkx`
(d-separation checks), `matplotlib`.

## 7. References

**Causal transportability**

- Bareinboim & Pearl (2013). *A general algorithm for deciding transportability of experimental
  results.* Journal of Causal Inference.
- Jalaldoust & Bareinboim (2024). *Transportable Representations for Domain Generalization.* AAAI.
- Jalaldoust & Bareinboim (2025). *Partial Transportability.* arXiv:2503.23605.
- Subbaswamy, Schulam & Saria (2019). *Preventing failures due to dataset shift: learning
  predictive models that transport.* AISTATS. arXiv:1812.04597.
- Neal (2020). *Introduction to Causal Inference.*

**Causality → machine learning**

- Peters, Bühlmann & Meinshausen (2016). *Invariant causal prediction.* JRSS-B.
- Rojas-Carulla et al. (2018). *Invariant models for causal transfer learning.* JMLR.
- Magliacane et al. (2018). *Domain adaptation by using causal inference to predict invariant
  conditional distributions.* NeurIPS.
- Chen & Bühlmann (2021). *Domain adaptation under structural causal models.* JMLR.
- Zhao et al. (2019). *On learning invariant representations for domain adaptation.* ICML.

**Optimal-transport domain adaptation**

- Courty, Flamary, Habrard & Rakotomamonjy (2017). *Joint distribution optimal transportation
  for domain adaptation (JDOT).* NeurIPS.
- Damodaran et al. (2018). *DeepJDOT.* ECCV.
- Redko et al. (2020). *A survey on domain adaptation theory.* arXiv:2004.11829.
- Wang et al. (2021). *Generalizing to unseen domains: a survey on domain generalization.*
  arXiv:2103.02503.

## 8. Author

Elias Massaro — École des Mines de Nancy.
Research project on domain adaptation for causal models.

## License

MIT (to be confirmed).
