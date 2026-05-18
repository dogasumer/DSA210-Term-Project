# Predicting A Materials Magnetic Moment From Compositional Features
## 1. Project Motivation

Magnetic materials are at the core of modern technology-from data storage and electric motors to medical imaging devices like MRI machines. Designing new magnetic materials typically requires Density Functional Theory (DFT) calculations, which are computationally expensive and slow, making large-scale screening of candidate materials impractical.

This project investigates whether the magnetic moment of an inorganic compound can be predicted directly from its chemical composition using machine learning, without any DFT calculations.

## 2. Research Question
Can we predict the magnetic moment of an inorganic compound from its compositional and structural features?
## 3. Data
**Primary Dataset:** Materials Project database (~20,000 compounds)
- Collected using the `mp-api` Python client
- Sampled across four magnetic ordering types: FM, AFM, FiM, NM (~5,000 each)
- Features: chemical formula, nsites, density, bandgap, formation_energy_per_atom, total_magnetization_normalized_formula_units,  ordering, crystal_system, spacegroup

**Enrichment:** Compositional features generated using `matminer` and `pymatgen`
- ElementProperty featurizer with Magpie preset
- Generates 100+ features including statistics of electronegativity, atomic radius, d-electron count, and valence electron count across 
  constituent elements
## 4. EDA
**Dataset:** 12 base features + 12 Magpie compositional features across ~20,000 compounds balanced across FM, AFM, FiM, and NM ordering types.

### Magnetization Distribution by Ordering Type
Histograms of magnetization_per_fu were plotted for each ordering type with the top 1% of values removed to reduce outlier effects. FM and FiM materials show clearly non-zero magnetization distributions while NM materials are tightly concentrated around zero, confirming the validity of the sampling strategy.

### Spearman Correlation Heatmap
A Spearman correlation matrix was computed across the base numerical features (magnetization_per_fu, density, band_gap, nsites, n_elements). Spearman correlation was chosen over Pearson due to the heavily skewed nature of the magnetization distribution.

### Magnetic Element Fraction
A custom feature was engineered representing the fraction of atoms in each compound that belong to known magnetic elements (3d transition metals: Fe, Co, Ni, Mn, Cr, Cu, V and rare earth elements: Gd, Tb, Dy, Ho, Er, Tm, Nd, Sm, Eu, Yb, Ce, Pr). A scatter plot against magnetization reveals that FM and FiM materials tend to have higher magnetic element fractions compared to NM materials.

### Magpie Compositional Feature Distributions
Three key Magpie features were examined:

Mean Electronegativity- distribution across constituent elements
Mean NUnfilled- average unfilled electron shells
Mean NdUnfilled- average unfilled d-orbitals, directly tied to transition metal magnetism

Scatter plots of each feature against magnetization, colored by ordering type, provide a visual preview of their predictive relevance ahead of formal hypothesis testing.
## 5. Hypothesis Testing

Four statistical tests were run to confirm that the features we engineered actually relate to magnetic ordering and magnetization. All tests use α = 0.05 as the significance level. The feature distributions are heavily skewed so standard ANOVA and t-tests are notappropriate, non-parametric tests are used throughout.

### H1- Does the average number of unfilled electron orbitals differ across ordering types?

The average number of unfilled electron orbitals per atom is computed via the Magpie feature-engineering framework. Unfilled orbitals are a proxy for electrons available for magnetic exchange interactions, so we expect FM materials to differ from NM.

- **H0:** There is no significant difference in the distribution of unfilled orbitals across FM, AFM, FiM, and NM
- **H1:** There is a significant difference in the distribution of unfilled orbitals across at least one ordering type
- Result: H0 rejected (p = 2.08e-190)

### H2- Does the average number of unfilled d-orbitals differ across ordering types?

Transition-metal magnetism (such as Fe, Co, Ni, Mn) arises from partially filled 3d shells, so the number of unfilled d-orbitals is a physically motivated feature for distinguishing magnetic from non-magnetic materials and for predicting magnetic moment magnitude.

- **H0:** There is no significant difference in the distribution of unfilled d-orbitals across FM, AFM, FiM, and NM
- **H1:** There is a significant difference in the distribution of unfilled d-orbitals across at least one ordering type
- Result: H0 rejected (p ≈ 0, H = 2255.58)

### H3- Does band gap differ between magnetic and non-magnetic materials?

Many magnetic materials are metals with a band gap of zero, while non-magnetic materials include insulators and semiconductors with non-zero band gaps. If confirmed, band gap becomes a useful separating feature for the ML model.

- **H0:** There is no significant difference in band gap distribution between magnetic and non-magnetic materials
- **H1:** There is a significant difference in band gap distribution between magnetic and non-magnetic materials
- Result: H0 rejected (p = 5.58e-94)

### H4- Does the fraction of magnetic elements in a compound correlate with its magnetization?

Tests whether our engineered feature — the proportion of magnetic elements (Fe, Co, Ni, Mn, etc.) in a chemical formula, is a useful predictor of
magnetization.

- **H0:** No monotonic relationship between magnetic element fraction and magnetization
- **H1:** A significant positive monotonic relationship exists
- Result: H0 rejected (ρ = 0.348, p ~ 0)

## 6. Machine Learning
### Models
Three classifiers were trained to predict magnetic ordering type (FM / AFM / FiM / NM) from compositional features alone:
| Model | Validation Accuracy |
|---|---|
| Logistic Regression (Baseline) | 0.592 |
| Gradient Boosting (125 trees) | 0.696 |
| XGBoost (200 Trees) | 0.703 |
| Random Forest (300 trees) | 0.723 |

### Methodology:
- Train / Validation / Test split: 70% / 15% / 15% with stratification
- StandardScaler fit on training data only to prevent data leakage
- 5-fold cross-validation mean accuracy: 0.710
- Final test set accuracy: 0.684

### Per-Class Performance (Random Forest)

- NM: easiest to predict, distinct band gap and near-zero magnetic element fraction
- AFM: moderate, some confusion with FM and FiM
- FiM: moderate, frequently confused with FM
- FM: ardest, FM and FiM share the same magnetic transition metals (Fe, Co, Ni) and composition alone cannot capture spin alignment differences

### Feature Importance

| Feature | Rank | Importance | Hypothesis |
|---|---|---|---|
| mag_element_fraction | #1 | 0.103 | H4 |
| band_gap | #2 | 0.097 | H3 |
| MagpieData mean NdUnfilled | #4 | 0.077 | H2 |
| MagpieData mean NUnfilled | #12 | 0.047 | H1 |

### Conclusions

- Magnetic ordering type can be predicted from composition alone with 72.3% validation accuracy and 68.4% test accuracy.
-  mag_element_fraction and band_gap are the two strongest predictors, confirming H4 and H3
- FM vs FiM confusion is the main limitation. Both share the same magnetic transition metals and composition alone cannot capture spin alignment
- Results are stable across 5-fold cross-validation (mean 0.710), confirming reliable generalisation to unseen data
