Excellent. **STEP 6** is where the project becomes a **coherent, end-to-end scientific system** rather than a collection of components. The goal here is strict integration discipline: one command, one config, one pipeline, fully reproducible.

As before, we define **structure, contracts, and execution flow**, not performance tuning.

---

# **STEP 6 — Full Pipeline Integration**

## **6.1 Objective of Step 6**

Integrate all prior components into a **single deterministic pipeline** that:

* starts from raw downloaded data
* produces trained SCMF + diffusion models
* runs inference
* computes metrics
* emits paper-ready figures

All controlled by **YAML experiment files**, with **zero manual intervention**.

This step is essential for:

* reviewer reproducibility
* ablation clarity
* scaling experiments later

---

## **6.2 Pipeline Philosophy (Locked)**

1. **Configuration drives everything**
2. **No notebooks**
3. **One experiment = one folder**
4. **Intermediate artifacts are cached**
5. **Every result is traceable**

No hidden state, no implicit defaults.

---

## **6.3 Pipeline Entry Point**

### **Single Entry Script**

```
src/pipeline/run_full_pipeline.py
```

This script is the *only* executable used for experiments.

Invocation:

```bash
python src/pipeline/run_full_pipeline.py \
    --config experiments/a2a/activation_regression.yaml
```

or

```bash
python src/pipeline/run_full_pipeline.py \
    --config experiments/ptp1b/full_model.yaml
```

---

## **6.4 Experiment Configuration Schema**

Each YAML defines **everything**.

### **Example: A2A Activation**

```
experiments/a2a/activation_regression.yaml
```

```yaml
experiment:
  name: a2a_activation_main
  seed: 42
  system: a2a
  mode: a2a

data:
  raw_dir: data/raw/anton/a2a
  processed_dir: data/processed/a2a

scmf:
  enabled: true
  model:
    hidden_dim: 128
    n_layers: 4
  training:
    epochs: 50
    batch_size: 8
    lr: 1e-4

diffusion:
  enabled: true
  timesteps: 1000
  model:
    hidden_dim: 256
  training:
    epochs: 100
    batch_size: 16
    lr: 2e-4

evaluation:
  metrics:
    - pearson
    - spearman
    - rmse
    - ks
    - kl
  plots:
    - activation_scatter
    - trajectory_overlay
    - distribution_overlay

output:
  results_dir: results/a2a
  figures_dir: figures/a2a
```

Every field maps directly to code. Nothing implicit.

---

## **6.5 Pipeline Execution Stages**

The pipeline executes **strictly in order**:

### **Stage 1 — Environment & Seed Setup**

* set random seeds (Python / NumPy / Torch)
* log system info (GPU, CUDA, versions)

---

### **Stage 2 — Data Loading**

* load raw PDB / Anton trajectories
* load processed features if present
* otherwise trigger preprocessing (cached)

---

### **Stage 3 — SCMF Training**

* build residue graphs
* train SCMF
* save:

  * `scmf.pt`
  * `scmf_config.yaml`
  * training curves

If `scmf.enabled = false`, load pretrained SCMF.

---

### **Stage 4 — Diffusion Training**

* inject SCMF conditioning
* train conditional diffusion
* save:

  * `diffusion.pt`
  * `diffusion_config.yaml`

This stage is **system-agnostic**.

---

### **Stage 5 — Inference**

* sample microstates or activation values
* multiple stochastic samples per input
* store raw predictions

---

### **Stage 6 — Evaluation**

* compute all metrics (Step 5)
* save `metrics.json`
* generate figures

---

### **Stage 7 — Final Manifest**

A single file:

```
results/<experiment>/manifest.json
```

Contains:

* git commit hash
* config hash
* file checksums
* timestamps

This is extremely reviewer-friendly.

---

## **6.6 Pipeline Code Skeleton**

### **File**

```
src/pipeline/run_full_pipeline.py
```

Conceptual structure:

```python
def main(config_path):
    cfg = load_yaml(config_path)
    set_seeds(cfg.experiment.seed)

    data = load_data(cfg.data)

    scmf_model = train_or_load_scmf(cfg.scmf, data)
    diffusion_model = train_or_load_diffusion(cfg.diffusion, data, scmf_model)

    predictions = run_inference(diffusion_model, scmf_model, data, cfg)

    metrics = evaluate(predictions, data, cfg.evaluation)
    make_plots(predictions, metrics, cfg.output)

    write_manifest(cfg)
```

Every function is independently testable.

---

## **6.7 Determinism & Reproducibility Guarantees**

We explicitly enforce:

* fixed random seeds
* deterministic PyTorch flags
* frozen dependency versions
* logged configs

This is crucial for a single-author paper.

---

## **6.8 Ablations Are First-Class**

Because SCMF and diffusion are toggled independently, we can trivially run:

* diffusion without SCMF
* SCMF only (no diffusion)
* static-structure conditioning

These ablations will later become **Figure 3 / Table 1** material.

---

## **6.9 What We Still Do Not Do**

Still deferred:

* hyperparameter sweeps
* performance optimization
* OpenMM refinement
* scaling to many systems

Those come **after** the full pipeline is proven.

---

## **Checkpoint Summary**

At the end of **Step 6**, we have:

* a single, clean execution path
* YAML-driven experiments
* end-to-end automation
* built-in ablations
* manifest-level provenance

This is now a **real surrogate system**, not a proposal.

---

## **Next Step**

### **STEP 7 — Running on GCP (T4)**

* VM setup
* mamba environment creation
* disk layout
* launch commands
* failure-safe execution

Please confirm:

> **“Proceed to Step 7.”**

From this point onward, we transition from design to execution.
