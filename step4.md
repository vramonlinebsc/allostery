Excellent. We now proceed to **STEP 4**, which completes the **core surrogate architecture**. This step must be handled carefully: the diffusion model is where novelty can easily be diluted if the design looks like “DiffDock with extra features.” We will explicitly avoid that.

This will be a **clean conditional diffusion skeleton**, shared across both systems but **used differently**, which is exactly what *Nature Machine Intelligence* expects.

No training, no hyperparameter tuning yet—only **architecture, contracts, and scientific intent**.

---

# **STEP 4 — Conditional Diffusion Model Skeleton**

## **4.1 Scientific Role of the Diffusion Component**

The diffusion model is **not** the surrogate by itself.

Its role is:

> **To sample or regress protein microstates conditioned on a learned side-chain mobility field (SCMF), approximating the ensemble accessibility revealed by long-timescale MD.**

Key distinction reviewers will notice:

* MD → samples trajectories
* Your model → samples *microstates consistent with MD-derived entropy structure*

That is a **surrogate**, not a shortcut.

---

## **4.2 Why Diffusion (and Not Direct Regression)**

Diffusion is justified here because:

* cryptic pockets and GPCR activation are **ensemble phenomena**
* MD does not produce a single answer, but a distribution
* diffusion naturally models conditional distributions

This is **conceptually aligned** with NMI’s ensemble framing.

---

## **4.3 Unified Conditional Diffusion, Dual Usage**

We will use **one diffusion backbone** with **two output modes**:

| System | Diffusion Output                                  | Interpretation                      |
| ------ | ------------------------------------------------- | ----------------------------------- |
| PTP1B  | Atomic coordinates (fragment pose / pocket atoms) | Conditional structure sampling      |
| A2A    | Continuous scalar (TM6 displacement)              | Conditional activation distribution |

Same conditioning, same noise process, different heads.

This unification is important for novelty.

---

## **4.4 Conditioning Signal (Locked)**

The **only conditioning signal** from the protein is:

> **SCMF field over residues**

No raw MD, no time information.

Conditioning channels:

* per-residue SCMF values
* mapped to atomic or residue representations
* injected at every diffusion timestep

This enforces causal hierarchy:
**structure → entropy → dynamics**

---

## **4.5 Diffusion Formalism (Minimal, Defensible)**

We use a standard **denoising diffusion probabilistic model (DDPM)**.

### **Forward Process**

[
x_t = \sqrt{\alpha_t} x_0 + \sqrt{1 - \alpha_t} \epsilon
]

### **Reverse Process**

[
p_\theta(x_{t-1} \mid x_t, \text{SCMF})
]

No tricks, no custom noise schedules yet.

Reviewers trust standard diffusion math.

---

## **4.6 Input / Output Definitions**

### **Common Inputs**

* Noisy state ( x_t )
* Diffusion timestep ( t )
* SCMF conditioning tensor
* Optional static structure embedding

---

### **PTP1B Mode**

**State ( x ):**

* fragment atomic coordinates
* optionally nearby side chains

**Output:**

* denoised coordinates

---

### **A2A Mode**

**State ( x ):**

* scalar activation value (TM6 displacement)

**Output:**

* denoised scalar

This may feel unconventional, but it is **explicitly allowed**: diffusion over low-dimensional continuous variables is well established.

---

## **4.7 Architecture Choice (Locked)**

### **Backbone**

* Transformer-based conditional denoiser
* Shared trunk
* Task-specific heads

This matches SCMF’s representation style and simplifies integration.

---

## **4.8 Model Interface (Non-Negotiable Contract)**

```python
class ConditionalDiffusionModel(nn.Module):
    def forward(
        self,
        x_t,
        t,
        conditioning,
        mode
    ):
        """
        Parameters
        ----------
        x_t : torch.Tensor
            Noisy state at timestep t
        t : torch.Tensor
            Diffusion timestep
        conditioning : torch.Tensor
            SCMF-derived conditioning signal
        mode : str
            'ptp1b' or 'a2a'

        Returns
        -------
        eps_pred : torch.Tensor
            Predicted noise
        """
```

The `mode` switch is deliberate and explicit—this avoids code duplication and supports ablations.

---

## **4.9 Skeleton Code**

### **File**

```
src/diffusion/model.py
```

```python
import torch
import torch.nn as nn

class ConditionalDiffusionModel(nn.Module):
    def __init__(self, x_dim, cond_dim, hidden_dim=256):
        super().__init__()

        self.x_embed = nn.Linear(x_dim, hidden_dim)
        self.t_embed = nn.Embedding(1000, hidden_dim)
        self.cond_embed = nn.Linear(cond_dim, hidden_dim)

        self.trunk = nn.Sequential(
            nn.LayerNorm(hidden_dim),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
        )

        self.heads = nn.ModuleDict({
            "ptp1b": nn.Linear(hidden_dim, x_dim),
            "a2a": nn.Linear(hidden_dim, x_dim)
        })

    def forward(self, x_t, t, conditioning, mode):
        h = self.x_embed(x_t)
        h = h + self.t_embed(t)
        h = h + self.cond_embed(conditioning)

        h = self.trunk(h)
        return self.heads[mode](h)
```

This is intentionally **plain**. We are not competing on architectural cleverness.

---

## **4.10 Loss Functions (Per Mode)**

### **PTP1B**

* Standard diffusion MSE on noise
* RMSD evaluated post-sampling (not a training loss)

### **A2A**

* Diffusion noise MSE
* Evaluation via correlation, RMSE, distributional metrics

Training loss is identical across modes.

This simplifies Methods substantially.

---

## **4.11 How This Differs from Existing Work (Reviewer Framing)**

You will explicitly state:

* We do **not** diffuse full protein conformations
* We condition on **learned entropy fields**, not static structures alone
* We unify structural and scalar dynamics under one surrogate

This is not DiffDock, not AlphaFold, not PocketMiner.

---

## **4.12 What We Still Do *Not* Do**

Still deferred:

* SCMF ↔ diffusion wiring code
* sampling loops
* OpenMM refinement
* hyperparameter tuning

We remain skeletal and controlled.

---

## **Checkpoint Summary**

At the end of **Step 4**, we have:

* a unified conditional diffusion framework
* explicit SCMF conditioning
* dual-use capability (PTP1B + A2A)
* reviewer-defensible simplicity
* a clean bridge between entropy and dynamics

This completes the **core surrogate architecture**.

---

## **Next Step**

### **STEP 5 — Metrics, Order Parameters, and Plotting**

* TM6 displacement computation
* pocket overlap metrics
* fragment RMSD
* distributional tests
* publication-ready plots

Please confirm:

> **“Proceed to Step 5.”**

We are exactly on track for a strong NMI submission.
