# Multimodal AI for Predicting the Compressive Strength of Cementitious Materials

Can we predict the compressive strength **fc (MPa)** of concrete from its microstructure (SEM/BSE images) and its composition (mix formulation)? This repository presents my M2 internship project, built entirely from data extracted from the scientific literature.

**M2 internship** · L2MGC Laboratory, CY Cergy Paris Université · April – July 2026
**Master 2 Applied Mathematics (MApI3)**, Université de Toulouse III Paul Sabatier

## Pipeline Architecture

![Pipeline architecture](architecture_pipeline.png)

*388 articles → anchored extraction → physical validation → 2 datasets → image ↔ formulation association → modeling*

## 1. Data Pipeline ("CLEAN")

Guiding principle: **no invented values**. A value enters the dataset only if it is written in the source article, and it stays traceable.

- **Corpus**: 388 articles collected via the CrossRef API, filtered automatically (keywords, ≥ 2 micrographs, formulation, fc in MPa) then manually, converted from PDF to Markdown
- **Extraction**: one JSON per article, with a source column for traceability
- **Physical validation**: out-of-range values are left empty, with a log; audit of 469 flagged rows (mostly false positives; 38 ages completed, 2 articles corrected)
- **Datasets**:

| Dataset | Content | Rows | Columns |
|---|---|---|---|
| `formulations_CLEAN` | mix × age: dosages, W/C, W/B, curing, strengths, porosity | 2,297 | 43 |
| `images_CLEAN` | SEM/BSE images with caption and association | 483 | 33 |

## 2. The Multimodal Bottleneck: Image ↔ Mix Association

One article can describe up to 61 mixes, and the link to an image exists only in the figure caption. I built a **caption-matching** algorithm (exact name match, then keyword dominance + discriminance, with 5 safeguards) that refuses to associate at the slightest doubt.

Result: reliable image–mix associations went from **76 to 141 images (+86%)**, each manually reviewed, with zero inconsistencies in cross-checking.

## 3. Image Features

Multi-Otsu segmentation into 3 phases (pores, C-S-H hydrates, anhydrous grains), plus texture descriptors (entropy, gradient, contrast, GLCM) and pore count/size. Estimated pore fraction vs. measured porosity: r = 0.52 (31 images).

## 4. Exploratory Analysis

The data reproduce known civil-engineering laws: fc decreases with porosity (r = −0.62) and with the water/binder ratio (r = −0.40), and increases with age (Abrams' law, hydration).

## 5. Models and Results

Six approaches were compared: MLP, Random Forest, XGBoost (tabular); ResNet18 with frozen ImageNet weights (image only); deep fusion (CNN ⊕ MLP); frozen ResNet → PCA-16 → XGBoost (multimodal).

**Tabular models** (test set n = 201, never seen during training):

| Model | MAE (MPa) | RMSE (MPa) | R² |
|---|---|---|---|
| Baseline (mean) | 23.8 | 35.1 | −0.03 |
| MLP | 13.8 | 24.0 | 0.52 |
| Random Forest | 11.7 | 20.0 | 0.66 |
| **XGBoost** | **9.8** | **17.0** | **0.76** |

**Multimodal models** (small test set, n = 8):

| Model | MAE (MPa) | RMSE (MPa) | R² |
|---|---|---|---|
| Baseline (mean) | 38.6 | 42.5 | −0.18 |
| Image only (ResNet18) | 30.9 | 38.6 | 0.03 |
| Multimodal (deep fusion) | 23.5 | 28.3 | 0.48 |
| Multimodal XGBoost | 19.5 | 21.9 | 0.69 |
| *Ablation: formulation only* | *7.7* | *15.9* | *0.83* |

**Key finding**: with the same samples, the formulation alone (R² = 0.83) outperforms formulation + image (R² = 0.69). The image does not yet bring a measurable gain: the bottleneck is the **number of reliable images** (only 40 with a known fc).

## 6. Explainability (XAI)

Grad-CAM maps and t-SNE projections of the latent space show that the network relies on the **material texture** (dense clusters, pores), not on image borders or annotations, which supports the physical validity of the learned features.

## Limitations and Next Steps

- Small multimodal dataset (141 images, 40 with fc), so conclusions on the image's contribution remain limited
- Publication bias, varied magnifications, heterogeneous fc test protocols
- Next: densify the multimodal dataset (main lever), repeat train/test splits for a more robust evaluation, inject physical descriptors into tree models, and eventually use real numerical EDS data

## Tech Stack

Python · pandas · NumPy · scikit-learn · XGBoost · PyTorch · OpenCV · BeautifulSoup · Selenium · CrossRef API · Git · Linux

## Confidentiality

This repository intentionally does **not** include the source code, datasets or trained models. The work was carried out within the L2MGC laboratory and is subject to research confidentiality.

## About Me

**Moussa Diallo**: junior Data Scientist / Machine Learning Engineer, M.Sc. in Applied Mathematics (Université de Toulouse III Paul Sabatier, 2026). Looking for a CDI or CDD in Data Science / ML in France (Toulouse or Île-de-France). Available immediately.

📧 diallomoussa052001@gmail.com · 🔗 [GitHub](https://github.com/dialloalassane052001-ui)
