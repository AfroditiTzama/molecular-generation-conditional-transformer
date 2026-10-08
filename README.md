# Conditional Molecular Generation with a SELFIES Transformer

**Undergraduate thesis project | Generative AI · Cheminformatics · Drug Discovery**

An end-to-end computational workflow for generating novel molecular structures with a **logP-conditioned autoregressive Transformer**, assessing their physicochemical properties, selecting diverse candidates through **Pareto optimization and MaxMin selection**, and evaluating a shortlist with **molecular docking against VEGFR2 (PDB: 3BE2)**.

> **Research scope:** This project explores molecular generation and computational prioritization. Generated structures and docking scores are **not** evidence of biological activity, safety, or therapeutic efficacy.

## Overview

The model generates molecules as **SELFIES** sequences while conditioning on a user-specified **logP** value (a measure of lipophilicity). The workflow combines sequence modeling, chemical validity checks, multi-objective filtering, diversity-based selection, and a structure-based docking assessment.

**Pipeline:**

`ZINC + GuacaMol → preprocessing → SELFIES → conditional Transformer → generation → chemical filtering → Pareto front → MaxMin diversity → VEGFR2 docking`

## Model architecture

| Component | Configuration |
|---|---|
| Molecular representation | SELFIES |
| Generation objective | Autoregressive next-token prediction |
| Conditioning variable | Target logP, projected into embedding space |
| Embedding dimension | 256 |
| Transformer blocks | 4 |
| Attention heads | 8 |
| Feed-forward hidden dimension | 2,048 |
| Dropout | 0.1 |
| Optimizer | AdamW |
| Training objective | Cross-entropy with padding ignored and label smoothing |
| Trainable parameters | Approximately 5.41 million |

The target logP is mapped through a small neural network and incorporated into the token representations. A causal attention mask prevents the model from attending to future tokens during generation.

## Reported research results

The following results are reported from the thesis experiments; they have **not been independently reproduced from a clean environment** for this repository.

| Metric | Reported result |
|---|---:|
| Molecules after dataset cleaning | 1,520,282 |
| Held-out test token accuracy | 81.53% |
| Held-out test cross-entropy loss | 1.2698 |
| Generation attempts | 150,000 |
| Chemically valid generated molecules | 149,995 (99.997%) |
| Unique generated molecules | 147,726 |
| Novel molecules vs. training set | 145,063 |
| Target-vs-generated logP MAE | 0.3795 |
| Target-vs-generated logP RMSE | 0.5491 |
| Target-vs-generated logP Pearson correlation | 0.9711 |

**Interpretation:** Token accuracy measures next-token prediction, not molecular validity. Novelty is measured relative to the training set. Pearson correlation indicates a strong linear relationship; it is not a percentage accuracy measure.

## Candidate filtering and selection

Generated molecules were assessed using computational descriptors and drug-likeness heuristics, including:

- **Lipinski's Rule of Five**
- **Topological Polar Surface Area (TPSA)**
- **Rotatable bonds**
- **Synthetic Accessibility Score (SAS)**
- **Quantitative Estimate of Drug-likeness (QED)**
- **Deviation from the requested logP**

A multi-objective **Pareto analysis** identified **63 non-dominated candidates** based on competing criteria including higher QED, lower SAS, and lower absolute logP deviation. **MaxMin selection** using **Morgan fingerprints** and **Tanimoto similarity** then prioritized **20 structurally diverse molecules** for docking.

These filters are prioritization heuristics, not proof of drug-likeness or synthesizability.

## Molecular docking: VEGFR2

The shortlisted candidates were evaluated against **VEGFR2**, using the experimental structure **PDB 3BE2** and **SMINA** for docking. The co-crystallized ligand **RAJ** was used as a reference in a redocking validation step.

The reported redocking analysis included a pose with **RMSD ≈ 0.70 Å** relative to the crystallographic reference; importantly, **this was not the top-scoring pose**. Candidate docking scores were used for computational comparison only and should not be interpreted as experimental binding affinities.

## Repository structure

```text
molecular-generation-conditional-transformer/
├── notebooks/
│   ├── molecular_generation.ipynb  # Training, generation, evaluation, selection
│   └── docking_3BE2.ipynb          # VEGFR2/3BE2 docking workflow
├── data/
│   └── README.md                   # Data availability notes
├── requirements.txt
├── .gitignore
└── README.md
```

## Getting started

Clone the repository and create an isolated Python environment:

```bash
git clone https://github.com/AfroditiTzama/molecular-generation-conditional-transformer.git
cd molecular-generation-conditional-transformer
python -m venv .venv
```

Activate it on macOS/Linux:

```bash
source .venv/bin/activate
```

Or on Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Install the Python packages listed in the repository:

```bash
python -m pip install -r requirements.txt
jupyter notebook
```

**Important reproducibility notes:**

- The notebooks are research notebooks and are **not yet packaged as a one-command reproducible pipeline**.
- Original molecular datasets, trained weights/checkpoints, and intermediate outputs are **not bundled**. Inspect the notebooks for expected filenames and paths before running.
- Docking requires additional external software, including **SMINA** and potentially **Open Babel/PyMOL**, depending on the notebook steps and environment.
- Some notebook cells may rely on absolute paths or platform-specific commands; these must be adapted locally.
- Dependency versions have not been fully pinned or validated against a fresh installation.

## Tools and technologies

**Python · PyTorch · RDKit · SELFIES · NumPy · pandas · scikit-learn · Matplotlib · Jupyter · SMINA · Meeko**

## Limitations and future directions

The generator is conditioned on **logP**, not directly on VEGFR2 activity. Computational filters and docking provide a way to prioritize candidates, but experimental synthesis and biological testing are necessary to assess actual therapeutic potential. Future extensions could include multi-property conditioning, scaffold-based evaluation, ADMET prediction, retrosynthesis feasibility, and experimental validation.

## Project status

**Academic research implementation.** The original notebooks are provided for transparency and portfolio review. A fully reproducible public release would additionally require documented dataset acquisition, validated environments, checkpoint availability, and an end-to-end execution test.

## Copyright & Usage

**© 2026 Afroditi Tzama. All Rights Reserved.**

This project was developed as part of my undergraduate thesis.

The repository is publicly accessible for portfolio presentation, academic reference, and research transparency.

**Public availability does not grant permission to reuse, modify, redistribute, or commercially exploit the original source code.**

If you wish to use any part of this implementation, please contact me for prior written permission.

See the [LICENSE](LICENSE) file for further information.
