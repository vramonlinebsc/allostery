Understood — we will do **A1 + A2 + A3 + A4** in one structured package so you have a complete NMI-ready concept you can build, pitch, and execute.

I’ll deliver each component clearly and concretely:

* **A1 — Detailed Model Architecture Diagrams (described textually for now)**
* **A2 — Required Datasets + Benchmarks**
* **A3 — Phased Implementation Plan**
* **A4 — NMI-Style Abstract Draft**

If later you want **actual diagrams** (SVG/PDF) I can generate those too.

Let’s begin.

---

# **A1. Detailed Model Architecture (SCMF + Conditional Diffusion Surrogate)**

Below are the core components and how they link together.

---

## **I. Inputs**

**Protein Inputs:**

* Apo structure (e.g., PTP1B 2CM2)
* AlphaFold ensembles or NMR ensembles (optional)
* Rotamer libraries (e.g., Dunbrack)
* Secondary structure annotations

**Ligand Inputs:**

* Fragment SMILES → 3D conformer embeddings via RDKit

---

## **II. Modules (Novel Architecture)**

We define 3 main modules:

---

### **Module 1 — Side-Chain Mobility Field Predictor (SCMF)**

**Goal:** predict local side-chain mobility around pockets from static protein structure.

**Motivation:**
Cryptic pockets form due to **rare side-chain rearrangements**, not backbone movements.

**Architecture:**

* Input graph: Protein graph `G = (V, E)` with N residues
* Node features: `(AA type, rotamer stats, secondary structure, solvent exposure)`
* Edge features: `(distance, contact type)`
* Backbone coords used as geometric frame

**Model:**

* **SE(3) Transformer** or **E(n)-Transformer**
* Output: per-residue mobility field tensor `M ∈ ℝ^(N × K)`

  * where K encodes predicted χ-angle variance, rotamer entropy, and pocket opening likelihood

**Output interpretation:**

* **high M_i** → residue i likely to adopt alternative rotamers
* clusters of high M_i → **cryptic pocket region**

---

### **Module 2 — Pocket-Conditioned Diffusion Model (PCDM)**

**Goal:** jointly sample fragment pose + pocket rotamers conditioned on SCMF

**Architecture:**

* Base sampler: **DiffDock-like SE(3) diffusion**
* Conditioning:

  * Ligand pose conditioned on SCMF tensor + local geometry patch
  * Pocket conformation sampled via rotamer embeddings

**Forward diffusion:**

* Adds Gaussian noise in SE(3) space to poses + rotamer angles

**Reverse diffusion:**

* Denoises both:

  * **pose (R³ + SO(3))**
  * **rotamer vector (χ1, χ2, …)**

**Outputs:**

* Fragment pose distribution P(ligand | protein, SCMF)
* Rotamer distribution P(pocket | ligand, SCMF)

**Novelty:**

* Standard DiffDock only predicts **static pose**
* This predicts **pose + local side-chain dynamics**

This is where the **methodological contribution** sits.

---

### **Module 3 — Physics-Aware Lightweight Refinement**

**Goal:** ensure sampled pocket conformers are physically plausible

**Approach:**

* short OpenMM simulations (1–50 ns)
* local region only (radius e.g. 12 Å)
* apply minimal restraints to backbone only

**Outputs:**

* refined χ-angle distributions
* pocket open/closed microstates
* RMSD & contact maps

---

## **III. Outputs**

1. **Cryptic pocket map** with mobility fields
2. **Fragment pose ensemble**
3. **Rotamer ensemble**
4. **Side-chain χ1/χ2 distributions**
5. **Pocket volume trajectories**
6. **Compute time comparison (Anton vs ML surrogate)**

---

## **IV. Where the Novelty Is (Explicitly for NMI)**

* **ML surrogate for cryptic pocket formation** (non-trivial)
* **Joint generative model for poses + rotamers**
* **SE(3)-aware conditioning on mobility fields**
* **Hybrid ML–physics integration for plausibility**

This checks the NMI novelty box.

---

# **A2. Required Datasets + Benchmarks**

We need datasets for:

1. **Cryptic pocket ground truth**
2. **Fragment binding**
3. **Side-chain rearrangements**
4. **Computational benchmarks**

Here’s the curated list.

---

## **Dataset Category 1 — Cryptic Pocket Proteins**

Proteins with known cryptic sites:

| Protein           | PDB IDs           | Notes                    |
| ----------------- | ----------------- | ------------------------ |
| **PTP1B**         | 2CM2 + 8G65..8G6A | your main case           |
| **HSP90**         | 2BSM, 2BRC        | classic cryptic pocket   |
| **BRAF**          | various           | fragment pockets         |
| **P38α MAPK**     | 1WFC, 1A9U        | allosteric pockets       |
| **HIV Integrase** | multiple          | fragment cryptic binding |

This gives **generalization** beyond PTP1B.

---

## **Dataset Category 2 — Fragment Binding Data**

Sources:

* **PDBBind** (general ligand binding)
* **Fragalysis** (fragment screening datasets)
* **Diamond XChem** (fragment soaking experiments)

Key feature:

* many have **co-crystal fragments**
* perfect for RMSD evaluation

---

## **Dataset Category 3 — Side-Chain Rearrangement Data**

We need MD or ensembles:

* **Anton MD trajectories for PTP1B** (from uploaded paper)
* **GPCRmd**
* **DEEPMD datasets**
* **Dynameomics rotamer trajectories**

Extract χ1, χ2 from trajectories for validation.

---

## **Dataset Category 4 — Computational Benchmarks**

We compare compute to:

* **Anton MD**
* **OpenMM MD**
* **Classical docking**

Metrics:

**Pose metrics:**

* RMSD (heavy atom)
* Contact maps
* Interaction fingerprints

**Dynamics metrics:**

* rotamer χ-angle distributions
* pocket volume trajectories (PV(t))
* pocket opening probability P(open)

**Compute metrics:**

* wall time
* GPU hours
* $ cost (optional)

NMI loves this kind of **societal/compute impact scaling**.

---

# **A3. Phased Implementation Plan**

Feasible without wet lab, on your hardware.

---

## **Phase 0 — Data & Environment (1–2 weeks)**

* Set up GCP T4 instances
* Collect PTP1B + fragment PDBs
* Set up AlphaFold ensemble pipeline
* Prepare rotamer libraries

Deliverables:

* structured datasets
* reproducible conda/mamba env

---

## **Phase 1 — SCMF Module (2–4 weeks)**

Tasks:

* build protein graph encoder
* train SE(3) model to predict rotamer variability
* validate against MD χ-angle data

Deliverables:

* mobility field heatmaps
* rotamer entropy maps

---

## **Phase 2 — Pocket-Conditioned Diffusion (4–8 weeks)**

Tasks:

* modify DiffDock backbone for conditional SE(3) diffusion
* couple SCMF outputs as conditioning features
* implement joint rotamer/pose sampling

Deliverables:

* generative model that outputs ensembles
* pocket-aware pose clouds

---

## **Phase 3 — Physics Refinement (2–4 weeks)**

Tasks:

* short local MD refinement of ensembles
* pocket volume & χ-angle analysis

Deliverables:

* refined ensembles
* dynamic pocket stats

---

## **Phase 4 — Validation & Benchmarking (2–4 weeks)**

Tasks:

* compare to Anton MD
* compare to crystal fragments
* runtime benchmarking

Deliverables:

* RMSD tables
* χ-angle histograms
* compute cost plots
* NMI-ready figures

Total timeline: **~12–24 weeks**

---

# **A4. Nature Machine Intelligence-Style Abstract Draft**

Here’s a draft written in **NMI tone + structure**:

---

**Draft Abstract:**

> **Cryptic allosteric pockets** govern the selectivity of many therapeutic targets but typically require **microsecond molecular dynamics (MD)** simulations on specialized hardware to observe, due to **rare side-chain rearrangements** that are invisible to static structural models. Here we introduce a **hybrid SE(3)-equivariant generative framework** that serves as a **surrogate for cryptic pocket dynamics**, jointly predicting **fragment binding poses** and **side-chain conformational ensembles** from a single apo structure. Our approach combines a **side-chain mobility field predictor**, trained to identify residues likely to undergo rotameric transitions, with a **pocket-conditioned diffusion model** that samples ligand poses and pocket conformations in a shared SE(3) latent space. Applied to **PTP1B** and a panel of cryptic-pocket proteins, our model recapitulates **microsecond-timescale pocket opening events**, **fragment-binding orientations**, and **χ-angle distributions** observed in **Anton MD** and **crystal complexes**, while running in **hours on a single commodity GPU**. These results suggest that generative geometric learning can **approximate rare conformational substates** that traditionally require long-timescale MD, enabling **scalable cryptic pocket discovery** and **fragment prioritization** without access to supercomputing infrastructure.

---

If you want, I can now:

* Turn this into a **full NMI paper skeleton (sections + figures)**,
* Draft a **grant proposal**,
* Draft a **collaboration pitch**,
* Provide **detailed pseudocode + model equations**.

Just reply: **“paper skeleton”**, **“grant proposal”**, **“collab pitch”**, or **“pseudocode”**.
