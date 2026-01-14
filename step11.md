Very well. We now proceed to **STEP 10 — Manuscript Writing**. At this stage, the priority is **precision, restraint, and inevitability**: every paragraph should feel like the only reasonable way to tell this story given the results you already have.

Below I will do three things, in order:

1. Define **writing principles specific to Nature Machine Intelligence**
2. Provide a **section-by-section drafting plan with tone guidance**
3. Deliver a **polished draft of the Introduction** (ready to paste into the manuscript)

We will then iterate section by section in subsequent turns.

---

# **STEP 10 — Writing the Manuscript (Nature Machine Intelligence)**

## **10.1 Writing Principles (Strict)**

These are non-negotiable for NMI:

1. **No hype language**

   * Avoid “breakthrough”, “paradigm-shifting”, “revolutionary”
2. **Surrogate ≠ replacement**

   * You approximate MD outputs; you do not replace physics
3. **Concept > benchmark**

   * Architecture and causal framing matter more than raw numbers
4. **Every claim has a figure or table**
5. **Economy of words**

   * Short paragraphs, dense meaning

Your tone should be: *calm, technically confident, slightly conservative*.

---

## **10.2 Drafting Order (Recommended)**

We will write in this order, not the journal order:

1. **Introduction** ← now
2. **Results (A2A first)**
3. **Results (PTP1B)**
4. **Methods**
5. **Discussion**
6. **Abstract (last)**

This minimizes rework.

---

## **10.3 Introduction — Full Draft**

### **Introduction**

Proteins populate heterogeneous conformational ensembles that underlie their functional behavior. In many systems, biologically relevant states are only transiently or sparsely populated, rendering them invisible to static experimental structures. Cryptic binding pockets in enzymes and activation pathways in G protein–coupled receptors (GPCRs) are canonical examples: both emerge from collective conformational fluctuations that are difficult to anticipate from a single structure.

Long-timescale molecular dynamics (MD) simulations have demonstrated that such hidden states can be accessed and characterized in silico. Specialized hardware platforms have revealed cryptic pockets, allosteric coupling pathways, and receptor activation mechanisms that were previously unknown. However, these simulations remain computationally expensive, system-specific, and inaccessible for routine use. As a result, their insights are rarely integrated into early-stage modeling, screening, or hypothesis generation.

Recent machine-learning approaches have dramatically improved protein structure prediction, ligand docking, and pocket detection. Despite these advances, most methods operate on static representations and do not explicitly model the dynamical accessibility of conformational states. Pocket detection algorithms typically rely on geometric or surface-based features, while docking models focus on binding poses conditioned on a fixed protein structure. Even generative approaches that sample protein conformations generally aim to reproduce equilibrium structures rather than the ensemble properties revealed by long-timescale MD.

Here we propose a different paradigm: a **machine-learning surrogate for molecular dynamics**. Rather than predicting structures or poses directly, the surrogate is trained to approximate MD-derived ensemble behavior using static structural inputs. The central idea is to decompose conformational dynamics into a causal hierarchy, in which local side-chain mobility governs the accessibility of cryptic pockets and activation pathways. By learning this intermediate representation from MD data, we enable downstream models to generate ensemble-consistent microstates without explicitly simulating trajectories.

We introduce the **side-chain mobility field (SCMF)**, a learned, residue-resolved representation that quantifies the propensity for rare side-chain rearrangements observed in long-timescale MD. Conditioned on this field, we employ a unified conditional diffusion model to generate either structural microstates or continuous activation coordinates, depending on the system of interest. This architecture allows a single surrogate framework to address both cryptic fragment binding in enzymes and activation dynamics in GPCRs.

We validate the approach on two challenging and mechanistically distinct systems. For the protein tyrosine phosphatase PTP1B, the surrogate identifies cryptic pockets and fragment binding poses with improved side-chain realism compared to existing methods. For the adenosine A2A receptor, the surrogate reproduces the distribution and progression of a canonical activation order parameter derived from published Anton MD trajectories. In both cases, ablation analyses demonstrate that conditioning on the learned side-chain mobility field is essential for capturing ensemble behavior.

Together, these results show that learned side-chain mobility fields can serve as effective intermediates for constructing machine-learning surrogates of long-timescale molecular dynamics. This approach provides a computationally efficient route to approximate dynamical accessibility in proteins, bridging the gap between static structure-based models and expensive MD simulations.

---

## **10.4 Why This Introduction Works**

* **Clear problem statement** (dynamics are inaccessible)
* **Explicit gap** (ML lacks ensemble accessibility)
* **New concept introduced early** (surrogate + SCMF)
* **No methodological overload**
* **Results previewed but not overstated**

A reviewer can already predict the figures—and that is exactly what you want.

---

## **Next Writing Step**

If you agree with the tone and framing, the next section to draft is:

### **Results — GPCR Activation Surrogate (A2A)**

This will directly reference:

* Figure 2
* Table 1
* SCMF ablations

Please confirm how you would like to proceed:

* **“Draft Results: A2A section.”**
* **“Revise Introduction first.”**

We will move carefully, one section at a time, to ensure publication-grade quality throughout.
