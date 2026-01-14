Very good. **STEP 5** is where we convert architecture into **quantitative scientific claims**. Everything here is reviewer-facing: metrics, order parameters, and plots must be physically grounded, standard-compliant, and unambiguous.

As before: no training yet, no execution yet—**definitions, formulas, and code skeletons only**.

---

# **STEP 5 — Metrics, Order Parameters, and Plotting**

## **5.1 Guiding Principles (Locked)**

All evaluation must satisfy:

1. **Physically interpretable**
2. **Standard in the literature**
3. **Comparable to MD**
4. **Statistically defensible**
5. **Reproducible from code alone**

No bespoke metrics invented for convenience.

---

## **5.2 A2A GPCR — Activation Order Parameter (Primary Metric)**

### **Order Parameter (Locked Earlier)**

**TM6 outward displacement**, measured as:

[
d_{\text{TM6}} = \lVert \mathbf{r}*{\text{C}\alpha}(\text{R3.50}) - \mathbf{r}*{\text{C}\alpha}(\text{Y6.34}) \rVert
]

Properties:

* scalar
* continuous
* monotonic along activation
* widely used in Anton GPCR literature

This is the **only activation coordinate** used for regression.

---

### **Ground Truth Computation**

Computed from Anton trajectories:

* per-frame
* per-trajectory
* stored as time series
* cached to disk

**File**

```
src/evaluation/gpcr_metrics.py
```

```python
def compute_tm6_displacement(traj, topology):
    """
    Returns
    -------
    distances : np.ndarray, shape (n_frames,)
    """
```

Residue indices are mapped once and stored to avoid ambiguity.

---

## **5.3 A2A — Regression Evaluation Metrics**

### **Primary Metrics**

These are required for NMI-level rigor.

1. **Pearson correlation (r)**

   * linear agreement
2. **Spearman correlation (ρ)**

   * rank / monotonic agreement
3. **RMSE**

   * absolute accuracy

```python
def regression_metrics(y_true, y_pred):
    return {
        "pearson": ...,
        "spearman": ...,
        "rmse": ...
    }
```

---

### **Distributional Metrics (Critical for Ensemble Claim)**

Because diffusion predicts **distributions**, not points:

4. **Kolmogorov–Smirnov (KS) statistic**
5. **KL divergence** (symmetrized)

These compare **predicted vs Anton distributions** of TM6 displacement.

This directly supports the *ensemble surrogate* claim.

---

## **5.4 A2A — Visualization (Paper Figures)**

### **Figure Types**

1. **Predicted vs Anton scatter**

   * with regression line
2. **Trajectory overlay**

   * Anton vs surrogate (time-aligned or normalized)
3. **Distribution overlay**

   * KDE or histogram
4. **Activation landscape schematic**

   * optional (later)

**File**

```
src/common/visualization.py
```

All plots:

* deterministic
* labeled in physical units
* export-ready (PNG + PDF)

---

## **5.5 PTP1B — Cryptic Pocket Metrics**

### **5.5.1 Pocket Overlap (Primary)**

Measured at **residue level**.

[
J = \frac{|R_{\text{pred}} \cap R_{\text{MD}}|}{|R_{\text{pred}} \cup R_{\text{MD}}|}
]

Where:

* residues are pocket-lining
* defined by distance cutoff to pocket volume or fragment

This metric:

* is standard
* is interpretable
* compares directly to fpocket / DeepSurf

---

### **5.5.2 Fragment Pose RMSD**

For fragment-bound predictions:

* heavy-atom RMSD
* aligned on protein backbone
* thresholded for “hit” rates

This is identical to docking evaluation standards.

---

### **5.5.3 Side-Chain χ-Angle Agreement (Key Novel Metric)**

This is where SCMF earns its keep.

For pocket-lining residues:

* compute χ angles in predicted structures
* compare to MD / crystal ensemble
* score via circular distance

This metric demonstrates **side-chain realism**, not just geometry.

---

## **5.6 PTP1B — Metrics Code Skeleton**

**File**

```
src/evaluation/ptp1b_metrics.py
```

```python
def pocket_jaccard(pred_residues, ref_residues):
    ...

def fragment_rmsd(pred_coords, ref_coords):
    ...

def chi_angle_error(pred_chi, ref_chi):
    ...
```

All functions operate on **processed data**, not raw structures.

---

## **5.7 Runtime and Efficiency Metrics**

Required to justify “surrogate” framing.

We will report:

* wall-clock inference time
* GPU memory usage
* comparison vs short MD / Anton scale

These are measured automatically during inference runs.

---

## **5.8 Output Artifacts (Locked)**

All metrics saved as:

* `metrics.json`
* `metrics.csv` (if tabular)
* plots in `figures/`

No numbers appear in text without being traceable to a file.

---

## **5.9 What This Enables Scientifically**

After Step 5, we can claim:

* **Accuracy** (RMSE, RMSD)
* **Correlation** (Pearson/Spearman)
* **Ensemble fidelity** (KS/KL)
* **Physical plausibility** (χ angles)
* **Efficiency** (runtime)

This is exactly the evidence profile NMI expects.

---

## **Checkpoint Summary**

At the end of **Step 5**, we have:

* locked evaluation definitions
* order parameters aligned to Anton literature
* metrics for both systems
* plotting standards
* reviewer-proof methodology

Nothing ad hoc, nothing cosmetic.

---

## **Next Step**

### **STEP 6 — Full Pipeline Integration**

* wire SCMF → diffusion → metrics
* single command execution
* deterministic experiment YAMLs
* end-to-end reproducibility

Please confirm:

> **“Proceed to Step 6.”**

We are now past architecture and into system engineering.
