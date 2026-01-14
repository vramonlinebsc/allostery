Understood. We will design **Step 2** so that **every dataset is fetched, verified, and organized via code**, with no manual downloads, clicks, or ad-hoc file handling. This is the correct choice for both reproducibility and reviewer confidence.

Below is **STEP 2 — fully specified and execution-ready**.

---

# **STEP 2 — Fully Automated Data Acquisition Plan**

## **Design Principles (Locked)**

All data acquisition must satisfy:

1. **100% scripted** (single command reproducibility)
2. **Publicly accessible sources only**
3. **Exact provenance traceable** to published work
4. **Deterministic file structure** (no ambiguity)
5. **Methods-section defensible**

No manual browser downloads, no Google Drive artifacts, no human intervention.

---

## **Overview of Data Sources**

| System   | Data Type          | Source                | Automation              |
| -------- | ------------------ | --------------------- | ----------------------- |
| PTP1B    | PDB structures     | RCSB PDB              | REST API                |
| PTP1B    | Fragment complexes | RCSB PDB              | REST API                |
| A2A      | Inactive structure | RCSB PDB              | REST API                |
| A2A      | Active structure   | RCSB PDB              | REST API                |
| A2A      | Anton trajectories | D. E. Shaw Research   | HTTP scripted fetch     |
| Metadata | Trajectory info    | Published supplements | Parsed programmatically |

---

## **2.1 Directory Targets (From Step 1)**

All downloads land in **`data/raw/` only**.

```
data/raw/
├── pdb/
│   ├── ptp1b/
│   └── a2a/
│
└── anton/
    └── a2a/
        ├── trajectories/
        └── metadata/
```

No other directory is allowed to write raw files.

---

## **2.2 Automated PDB Acquisition (PTP1B + A2A)**

### **Targets**

#### **PTP1B**

* Apo structures
* Fragment-bound cryptic pocket structures
* Multiple conformations where available

#### **A2A**

* Inactive: **3EML**
* Active: **5G53**

---

### **Implementation Strategy**

We use **RCSB PDB REST API** (stable, rate-safe, publication-standard).

**Script location**

```
src/data/pdb_loader.py
```

**Core functionality**

* Fetch PDB/mmCIF
* Validate checksum
* Normalize filenames
* Store metadata (resolution, method)

---

### **Example (Conceptual Skeleton)**

```python
def download_pdb(pdb_id, out_dir):
    url = f"https://files.rcsb.org/download/{pdb_id}.pdb"
    ...
```

The actual implementation will:

* retry on transient failures
* hash files
* write a `.json` metadata companion

---

## **2.3 PTP1B Fragment Dataset Definition**

We will **explicitly enumerate** PTP1B fragment-bound PDBs (from published cryptic pocket studies).

This list will be:

* version-controlled
* cited in Methods
* loaded programmatically

Example:

```
data/raw/pdb/ptp1b/pdb_list.json
```

```json
{
  "apo": ["1PTY", "2CM2"],
  "fragment_bound": ["4I8N", "4I8L", "4I8A"]
}
```

(Exact list finalized before coding; no ambiguity.)

---

## **2.4 Anton A2A Trajectory Acquisition**

### **Ground Truth Source (Locked)**

**Published Anton trajectories from D. E. Shaw Research**

These are distributed as **tarballs** with:

* trajectories
* topology files
* README metadata

---

### **Why This Is Acceptable**

* Publicly released
* Used extensively in GPCR literature
* Explicitly allowed for academic reuse
* Considered gold-standard MD

---

### **Automation Strategy**

**Script location**

```
src/data/anton_loader.py
```

**Mechanism**

* HTTP GET via Python (`requests`)
* Check file sizes
* Extract deterministically
* Validate expected frame counts

---

### **Expected Result**

```
data/raw/anton/a2a/trajectories/
├── a2a_activation_1.xtc
├── a2a_activation_2.xtc
└── topology.pdb
```

Metadata:

```
data/raw/anton/a2a/metadata/
├── source.txt
├── citation.bib
└── trajectory_map.json
```

This metadata will be used verbatim in **Methods**.

---

## **2.5 Reproducible Entry Point**

### **Single Command Execution**

From repo root:

```bash
python src/data/preprocessing.py --download_all
```

This will:

1. Download all PDBs
2. Download Anton trajectories
3. Validate integrity
4. Produce a summary log

No manual steps.

---

## **2.6 Integrity & Provenance Guarantees**

For every downloaded file we will store:

* SHA256 hash
* Source URL
* Date fetched
* Associated publication DOI

This ensures:

* reviewer trust
* internal reproducibility
* future re-runs unchanged

---

## **2.7 What Happens Next (Deterministic)**

Once **Step 2 completes**, we will immediately proceed to:

### **STEP 3 — SCMF Model Skeleton**

* Define side-chain entropy targets
* Feature representation
* Model interface (inputs/outputs)
* Loss functions aligned to MD physics

No architectural guessing, no scope creep.

---

## **No Clarifications Needed**

Your constraint (“everything automated”) is fully satisfied by this plan.

---

### **Next Confirmation**

Please confirm:

> “Proceed to Step 3.”

We will then begin implementing the **SCMF model skeleton**, grounded directly in the data acquired here.
