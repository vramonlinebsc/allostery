Excellent. We now move into **STEP 3**, which is the **scientific core** of the project. I will be deliberately precise here, because decisions made at this stage determine whether reviewers perceive this as a *true surrogate for MD* or merely a predictive model.

No downloads, no training yet. This step produces a **clean, defensible SCMF model skeleton** that is:

* physically interpretable
* MD-grounded
* ablation-ready
* easy to extend
* clearly novel relative to DeepSurf

---

# **STEP 3 — SCMF (Side-Chain Mobility Field) Model Skeleton**

## **3.1 Scientific Purpose of SCMF (Re-stated Precisely)**

The **SCMF** is **not** a pocket detector.

It is a **learned surrogate for side-chain conformational entropy and rearrangement propensity**, approximating what long-timescale MD reveals about:

* rare χ-angle transitions
* rotamer interconversion likelihoods
* side-chain gating events that enable cryptic pockets

Formally:

> **SCMF predicts a spatially indexed field over residues / atoms representing the probability of observing rare side-chain rearrangements under equilibrium or biased dynamics.**

This is the *causal precursor* to cryptic pocket formation.

That causal framing is critical for NMI.

---

## **3.2 What SCMF Predicts (Target Definition)**

### **Prediction Target (Primary)**

For each residue ( i ):

[
\text{SCMF}*i = \mathbb{E}*{t \in \text{MD}} \left[ \mathbb{I}(\Delta \chi_i(t) > \theta) \right]
]

Where:

* ( \chi_i ) are side-chain dihedrals
* ( \theta ) is a rotamer transition threshold (e.g. 60°)
* Expectation is taken over Anton trajectories (or long unbiased MD for PTP1B)

Interpretation:

* 0 → rigid side chain
* 1 → highly mobile / rearranging side chain

This is **MD-derived**, **continuous**, and **physically meaningful**.

---

### **Optional Secondary Targets (Later Ablations)**

Not implemented now, but enabled by design:

* χ-angle variance
* Shannon entropy of rotamer bins
* transition rate constants (log-scaled)

We start with **binary transition frequency** because it is:

* robust
* low-noise
* reviewer-friendly

---

## **3.3 Inputs to SCMF (Static Only)**

SCMF **must not** see MD trajectories at inference time.

Inputs are limited to:

### **Structural Inputs**

* Single static structure (apo or bound)
* Atomic coordinates
* Element types
* Residue identity

### **Local Geometry**

* Interatomic distances (within cutoff)
* Local frame features (bond vectors)
* Solvent exposure (optional later)

No time-series, no ensembles.

This enforces the surrogate claim.

---

## **3.4 Representation Choice (Locked)**

We will use a **Residue-Centric Graph Neural Network**.

### **Graph Definition**

* **Nodes**: residues
* **Edges**: residue–residue if any heavy atoms < 6.0 Å
* **Node features**:

  * one-hot residue type
  * backbone dihedrals (φ, ψ)
  * Cα coordinates (relative frame)
* **Edge features**:

  * minimum heavy-atom distance
  * relative orientation vectors

This representation:

* is invariant to rotation/translation
* scales well
* aligns with protein ML standards
* is easy to condition diffusion on later

---

## **3.5 SCMF Model Interface (Contract)**

This interface is **non-negotiable**, because downstream diffusion will depend on it.

```python
class SCMFModel(torch.nn.Module):
    def forward(self, graph):
        """
        Parameters
        ----------
        graph : ProteinGraph
            Residue-level graph representation of a protein structure

        Returns
        -------
        scmf : torch.Tensor, shape (N_residues,)
            Continuous side-chain mobility scores in [0, 1]
        """
```

No fragment info, no ligand info. Pure protein physics proxy.

---

## **3.6 Loss Function (MD-Aligned)**

### **Primary Loss**

Binary cross-entropy on MD-derived labels:

[
\mathcal{L}_{\text{SCMF}} =
\text{BCE}(\hat{p}_i, p_i^{\text{MD}})
]

Where:

* ( p_i^{\text{MD}} ) = empirical transition frequency
* ( \hat{p}_i ) = SCMF output

This is statistically well-posed and interpretable.

---

### **Regularization (Light, Optional)**

* smoothness regularization across neighboring residues
* prevents salt-and-pepper artifacts

Not required in skeleton.

---

## **3.7 Code Skeleton (Minimal, Correct)**

### **File**

```
src/scmf/model.py
```

```python
import torch
import torch.nn as nn

class SCMFModel(nn.Module):
    def __init__(self, node_dim, edge_dim, hidden_dim=128, n_layers=4):
        super().__init__()
        self.node_embed = nn.Linear(node_dim, hidden_dim)
        self.edge_embed = nn.Linear(edge_dim, hidden_dim)

        self.layers = nn.ModuleList([
            nn.TransformerEncoderLayer(
                d_model=hidden_dim,
                nhead=8,
                batch_first=True
            )
            for _ in range(n_layers)
        ])

        self.readout = nn.Sequential(
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, 1),
            nn.Sigmoid()
        )

    def forward(self, node_features, edge_features, mask=None):
        x = self.node_embed(node_features)

        for layer in self.layers:
            x = layer(x)

        scmf = self.readout(x).squeeze(-1)
        return scmf
```

This is intentionally **simple and replaceable**. Reviewers value clarity over cleverness.

---

## **3.8 Training Script Skeleton**

### **File**

```
src/scmf/train.py
```

Responsibilities:

* load processed MD-derived labels
* build residue graphs
* train SCMF only
* save model + config hash

No diffusion involved yet.

---

## **3.9 How SCMF Connects to the Whole Architecture**

This is important for framing later.

* SCMF output becomes a **conditioning field**
* Diffusion model will learn:

  * *“Given this side-chain mobility landscape, what microstates are likely?”*
* This mimics MD’s causal flow:
  **entropy → pocket → binding / activation**

That causal chain is your novelty.

---

## **3.10 What We Do *Not* Do Yet**

Explicitly deferred:

* diffusion implementation
* fragment conditioning
* GPCR activation regression
* OpenMM refinement

This keeps scope controlled and credible.

---

## **Checkpoint Summary**

At the end of **Step 3**, we have:

* a physically grounded SCMF definition
* a clean ML interface
* a training-ready skeleton
* a clear role in the full surrogate
* a strong Methods narrative

No assumptions violated. No automation broken.

---

## **Next Step**

### **STEP 4 — Conditional Diffusion Skeleton**

* how SCMF conditions generation
* dual use: PTP1B (poses) and A2A (activation scalar)
* shared architecture, different heads

Please confirm:

> **“Proceed to Step 4.”**

We remain fully deterministic and incremental.
