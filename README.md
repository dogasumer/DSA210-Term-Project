# Predicting A Materials Magnetic Moment From Compositional Featues
## 1. Project Motivation
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
## 5. Hypotheses
