Excellent. **STEP 7** transitions us from system design to **actual execution on infrastructure**, with an emphasis on automation, robustness, and zero local-state assumptions. This step is written so that you (or a reviewer, or a future you) can reproduce results from a clean cloud environment without improvisation.

Everything below is **operationally complete**.

---

# **STEP 7 — Execution on GCP (T4 GPU)**

## **7.1 Objectives of Step 7**

By the end of this step, you will have:

* a reproducible **GCP VM setup**
* a **fully functional GPU environment** using `environment-gpu.yml`
* a **single-command workflow** to run any experiment
* logs, checkpoints, and results written deterministically
* no dependence on local (Windows) execution beyond code editing

---

## **7.2 GCP VM Specification (Locked)**

### **Recommended VM Configuration**

* **Machine type:** `n1-standard-8`
* **GPU:** NVIDIA **T4** (1×)
* **OS:** Ubuntu 20.04 LTS
* **Boot disk:** 100 GB SSD (minimum)
* **Region:** any with T4 availability

This configuration is:

* sufficient for SCMF + diffusion
* cost-efficient
* widely available
* stable for CUDA 11.8

---

## **7.3 One-Time VM Setup (Fully Scriptable)**

### **7.3.1 System Packages**

On the VM:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y \
    build-essential \
    git \
    wget \
    curl \
    tmux \
    htop
```

No GUI, no extras.

---

### **7.3.2 NVIDIA Driver & CUDA**

Use GCP’s **recommended driver install**:

```bash
sudo apt install -y ubuntu-drivers-common
sudo ubuntu-drivers autoinstall
sudo reboot
```

After reboot:

```bash
nvidia-smi
```

You should see the T4 recognized.

**Do not install CUDA manually** — PyTorch + conda will manage it.

---

## **7.4 Repository Setup**

### **7.4.1 Clone Repository**

```bash
git clone <your-repo-url> cryptic-md-surrogate
cd cryptic-md-surrogate
```

No submodules, no large files tracked in Git.

---

## **7.5 Environment Creation (Mambaforge, Locked)**

### **7.5.1 Install Mambaforge**

```bash
wget https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-Linux-x86_64.sh
bash Mambaforge-Linux-x86_64.sh
```

Accept defaults. Restart shell.

---

### **7.5.2 Create Environment**

```bash
mamba env create -f environment-gpu.yml
conda activate cryptic-md
```

Verify GPU access:

```bash
python - << EOF
import torch
print(torch.cuda.is_available())
print(torch.cuda.get_device_name(0))
EOF
```

This must return `True` and `Tesla T4`.

---

## **7.6 Data Acquisition (Automated, Step 2)**

From repo root:

```bash
python src/data/preprocessing.py --download_all
```

This will:

* fetch PDBs
* fetch Anton trajectories
* validate checksums
* populate `data/raw/`
* log provenance

**No manual downloads are permitted.**

---

## **7.7 Running Experiments (Single Command)**

### **A2A Activation Regression**

```bash
python src/pipeline/run_full_pipeline.py \
    --config experiments/a2a/activation_regression.yaml
```

### **PTP1B Cryptic Pocket Modeling**

```bash
python src/pipeline/run_full_pipeline.py \
    --config experiments/ptp1b/full_model.yaml
```

Each run will:

* create a unique results directory
* save trained models
* compute metrics
* generate figures
* write a `manifest.json`

You can safely disconnect and reattach via `tmux`.

---

## **7.8 Long-Running Job Safety**

### **Use tmux**

```bash
tmux new -s surrogate
```

Run experiments inside tmux.

Detach with `Ctrl+B`, then `D`.

Reattach:

```bash
tmux attach -t surrogate
```

This avoids accidental job termination.

---

## **7.9 Disk Layout (Expected)**

After a successful run:

```
results/
├── a2a/
│   └── a2a_activation_main/
│       ├── metrics.json
│       ├── predictions.csv
│       ├── manifest.json
│       └── models/
│
figures/
├── a2a/
│   ├── activation_scatter.png
│   ├── distribution_overlay.png
│   └── trajectory_overlay.png
```

Everything needed for the paper is here.

---

## **7.10 Failure Modes & Recovery**

* **VM interruption:** restart VM, re-run command (cached data + checkpoints)
* **OOM:** reduce batch size in YAML
* **CUDA mismatch:** recreate conda env from `environment-gpu.yml`
* **Corrupt data:** delete `data/raw/` and re-run download script

No state is irrecoverable.

---

## **7.11 What This Enables Next**

With Step 7 complete, you will have **real outputs**:

* trained SCMF
* trained conditional diffusion
* Anton-aligned predictions
* quantitative metrics
* paper-quality figures

From here, the project becomes about **scientific storytelling**, not infrastructure.

---

## **Next Step**

### **STEP 8 — Results Consolidation & Evaluation Tables**

* aggregate metrics across runs
* prepare comparison tables
* select representative figures
* sanity-check ensemble behavior

Please confirm:

> **“Proceed to Step 8.”**

You are now in execution territory, exactly where this project should be.
