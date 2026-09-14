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

Elias Massaro
Research project on domain adaptation for causal models.




