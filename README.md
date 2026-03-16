# Cascading Agency in Clinical AI Delegation

**A Markov–Monte Carlo model of cascading agency failure in AI-augmented clinical decision-making**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)

---

## Overview

This repository contains the simulation code, model specification, and analysis pipeline for the paper:

> Mikkelsen, Y. (2026). *When delegation degrades: A Markov–Monte Carlo model of cascading agency failure in AI-augmented clinical decision-making.* [Manuscript in preparation]

The model extends the cognitive supply–demand (S/D) framework ([Mikkelsen, 2026a](https://doi.org/placeholder)) to capture what happens when clinicians delegate tasks to AI clinical decision support systems. It formalises three dynamics absent from existing AI safety and health informatics frameworks:

1. **Monitoring cost as a tax on bounded cognitive supply** — verifying AI output consumes the same finite resource (working memory, attention, executive function) as clinical reasoning itself.
2. **Dual-pathway monitoring degradation** — monitoring effort declines both under cognitive overload (effort–accuracy tradeoff) and through habituation to AI recommendations over time.
3. **Deskilling feedback loop** — sustained delegation erodes clinical expertise, which in turn degrades monitoring effectiveness, creating a self-reinforcing cycle that can produce irreversible delegation lock-in.

## The model

The extended framework modifies the base S/D ratio:

```
O(t) = f( S_eff(t) / D_eff(t) )

where:
  S_eff = S − C          (supply minus monitoring cost)
  D_eff = D − E + E'     (demand minus delegation benefit plus error propagation)
```

AI error propagation (E') is decomposed into five empirically grounded failure modes: hallucination, systematic bias, model drift, automation bias, and context mismatch. Each has distinct probability, detection difficulty, and downstream amplification characteristics.

Two recursive loops interact to produce cascading agency failure:
- **Loop 1 (PBC erosion):** Poor outcomes → reduced perceived behavioural control → reduced intention → reduced supply → worse outcomes
- **Loop 2 (Deskilling):** Delegation → reduced practice → skill erosion → reduced monitoring effectiveness → more undetected errors → worse outcomes

## Key findings

*[To be populated after simulation runs]*

## Repository structure

```
cascading-agency-model/
├── README.md                       # This file
├── LICENSE                         # MIT License
├── requirements.txt                # Python dependencies
├── pyproject.toml                  # Project metadata
│
├── src/                            # Source code
│   ├── __init__.py
│   ├── model.py                    # Core S_eff/D_eff dynamics and difference equations
│   ├── markov.py                   # 36-state Markov chain (R×Δ×K)
│   ├── simulation.py               # Monte Carlo engine (10,000 runs × 8 scenarios)
│   ├── sensitivity.py              # SALib integration for Sobol indices
│   ├── calibration.py              # Parameter distributions and evidence-based values
│   └── visualisation.py            # Publication-quality figures
│
├── configs/
│   └── scenarios.yaml              # 8 scenario definitions with parameter overrides
│
├── data/
│   ├── embedding_mrr.csv           # 294-condition MRR@10 data (from Mikkelsen 2026b)
│   └── calibration_sources.csv     # Parameter values with literature sources
│
├── results/                        # Simulation outputs (generated, not tracked)
│   ├── trajectories/
│   ├── sensitivity/
│   └── summary/
│
├── figures/                        # Publication figures (generated)
│
├── notebooks/
│   ├── 01_calibration_check.ipynb  # Verify parameter distributions against literature
│   ├── 02_single_run_debug.ipynb   # Step-through single simulation for validation
│   ├── 03_scenario_analysis.ipynb  # Main results and figure generation
│   └── 04_sensitivity.ipynb        # Sobol sensitivity analysis
│
├── paper/
│   ├── outline_v01.md              # Paper outline with section targets and citations
│   └── figures/                    # Final publication-ready figures
│
├── supplementary/
│   ├── model_specification_v04.md  # Full formal model specification
│   └── evidence_audit.md           # Colour-coded evidence assessment for all parameters
│
└── references/
    └── brynjolfsson_replication/   # Replication package from Brynjolfsson et al. (QJE, 2025)
```

## Parameter calibration

All model parameters are calibrated from published empirical evidence. Key sources:

| Parameter | Value | Source | Evidence status |
|-----------|-------|--------|-----------------|
| β (deskilling rate) | 0.07/month | Budzyń et al., *Lancet GastroHep*, 2025 (n=1,443) | Amber-Green |
| η (habituation rate) | 0.06/month | Brynjolfsson et al., *QJE*, 2025 (n=5,172) | Amber-Green |
| α (AI accuracy) | 0.09–0.88 | Mikkelsen, 2026b (294 conditions) | Green |
| p₄ (automation bias) | 7% | Rosbach et al., BVM 2025 (n=28) | Green |
| κ (error amplification) | Tri(1.4, 2.0, 5.5) | Bates, *JAMA*, 1997; Hug et al., 2012 | Amber-Green |

Full calibration table with distributions and evidence audit: see [`supplementary/model_specification_v04.md`](supplementary/model_specification_v04.md), Section 6.3.

## Simulation design

- **Step length:** 1 month (matches β, η calibration resolution and institutional KPI cycles)
- **Primary horizon:** 60 months (5 years) — aligned with AI deployment cycles and EHR contract lengths
- **Extended horizon:** 120 months (10 years) — captures career-span effects and lock-in persistence
- **Discount rate:** 3.5%/year (NICE/Norwegian HTA standard; sensitivity at 0% and 5%)
- **Monte Carlo runs:** 10,000 per scenario (convergence validated via MCSE diagnostics)
- **Warm-up period:** 6 months
- **Scenarios:** 8 (baseline, high-quality AI, variable AI, delegation under overload, lock-in test, institutional mandate, optimal monitoring, skill-dependent delegation)
- **Sensitivity analysis:** Sobol first-order and total-order indices via [SALib](https://github.com/SALib/SALib)

## Companion papers

This model builds on two research streams by the same author:

**Stream 1 — Cognitive supply–demand framework:**
- Mikkelsen, Y. (2026a). *A cognitive supply–demand framework for patient safety.* [Submitted]

**Stream 2 — Clinical embedding benchmarks:**
- Mikkelsen, Y. (2026b). *Embedding benchmark for clinical information retrieval.* [Submitted to JMIR Medical Informatics]
- Mikkelsen, Y. (2026c). *Layer-level embedding analysis for clinical text.* [Submitted to JAMIA]
- Mikkelsen, Y. (2026d). *clinical-embedding-fix: A Python library for clinical embedding correction.* [Submitted to SoftwareX]

## Installation

```bash
git clone https://github.com/[username]/cascading-agency-model.git
cd cascading-agency-model
pip install -r requirements.txt
```

### Dependencies

- Python ≥ 3.10
- NumPy
- SciPy
- SALib (sensitivity analysis)
- Matplotlib / Seaborn (visualisation)
- PyYAML (scenario configuration)
- Pandas (data handling)
- Jupyter (notebooks)

## Usage

### Run all scenarios

```bash
python -m src.simulation --config configs/scenarios.yaml --runs 10000 --output results/
```

### Run a single scenario

```bash
python -m src.simulation --scenario baseline --runs 10000 --output results/
```

### Sensitivity analysis

```bash
python -m src.sensitivity --config configs/scenarios.yaml --samples 2048 --output results/sensitivity/
```

### Generate figures

```bash
python -m src.visualisation --results results/ --output figures/
```

## Reproducibility

- All random seeds are fixed and documented
- The `results/` directory is not tracked by git; outputs are regenerated from code
- Simulation convergence is validated using Monte Carlo Standard Error (MCSE) diagnostics following Krijkamp et al. (*Med Decis Making*, 2018)
- Parameter distributions are specified in `src/calibration.py` with inline citations to source publications

## Citation

If you use this code or model in your work, please cite:

```bibtex
@article{mikkelsen2026delegation,
  title={When delegation degrades: A {Markov}--{Monte Carlo} model of cascading 
         agency failure in {AI}-augmented clinical decision-making},
  author={Mikkelsen, Yngve},
  journal={[Target journal]},
  year={2026},
  note={Manuscript in preparation}
}
```

## Key references

- Brynjolfsson, E., Li, D., & Raymond, L. (2025). Generative AI at work. *Quarterly Journal of Economics*, 140(2), 889–942. https://doi.org/10.1093/qje/qjae044
- Budzyń, K. et al. (2025). Endoscopist deskilling risk after exposure to artificial intelligence in colonoscopy. *Lancet Gastroenterology & Hepatology*, 10, 896–903.
- Berzin, T. M., & Topol, E. J. (2025). Preserving clinical skills in the age of AI assistance. *The Lancet*, 406(10513), 1719.
- Bainbridge, L. (1983). Ironies of automation. *Automatica*, 19(6), 775–779.
- Jabbour, S. et al. (2023). Measuring the impact of AI in the diagnosis of hospitalized patients. *JAMA*, 330(23), 2275–2284.
- Krijkamp, E. M. et al. (2018). Microsimulation modeling for health decision sciences using R. *Medical Decision Making*, 38(3), 400–422.

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

## Contact

Yngve Mikkelsen — [email/institution to be added]

---

*This repository is part of an open-science research programme. All code, model specifications, and analysis pipelines are shared to enable verification, replication, and extension.*
