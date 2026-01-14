Understood — we’ll refocus cleanly on **the paper itself** and forget the side threads.

To make this productive and not overwhelming, here’s the **minimal structure we need to settle before writing anything**:

---

# **(1) What Paper Are We Writing?**

You already chose **Variant B**, so the paper is:

> **An ML surrogate for cryptic pocket dynamics + fragment binding**,
> **DeepSurf only as baseline**,
> **Anton MD as reference**,
> **no wet lab**,
> **target: Nature Machine Intelligence**.

Good — fixed and clear.

---

# **(2) Minimal Paper Outline (Specific to Variant B)**

Here is the **correct outline** for NMI-style computational methodology:

1. **Introduction**

   * why cryptic pockets matter
   * why MD is expensive
   * why ML surrogate is interesting

2. **Related Work**

   * cryptic pockets (MD/NMR context)
   * ML for docking (DiffDock etc.)
   * ML for protein dynamics (AlphaFold, SE(3), diffusion)

3. **Method Overview**

   * what the surrogate does (conceptually)
   * what inputs it needs
   * what outputs it generates

4. **Model Components**
   4.1 Side-chain mobility field (SCMF)
   4.2 Conditional diffusion for pose + rotamers
   4.3 Lightweight MD refinement (optional)

5. **Experimental Setup**

   * systems/Datasets (PTP1B, HSP90, etc.)
   * baselines (DeepSurf, fpocket, DiffDock, MD)
   * metrics

6. **Results**

   * pocket detection vs cryo/MD/crystal
   * pose accuracy vs co-crystal
   * rotamer dynamics vs MD
   * runtime vs Anton

7. **Discussion**

   * strengths / limits / biological insight
   * no wet-lab claims
   * future extensions

8. **Conclusion**

This is the format NMI expects.

---

# **(3) Before We Go Deeper — We Need ONE DECISION**

To write the paper, we need to lock in **one thing** first:

> **What system(s) will the paper analyze?**

Minimum requirement for credibility is **one main system + one secondary system**.

Given your interest, here are the realistic choices:

### **Primary System Options (Pick One):**

A) **PTP1B** (cryptic allosteric fragments; Anton data exists)
B) **HSP90** (classic cryptic pocket protein)
C) **P38α MAPK** (allosteric pockets known)
D) **BRD4** (fragment binding well-characterized)

For Nature Machine Intelligence, **PTP1B (A)** is ideal because:

* Anton MD results exist for direct comparison
* Cryptic pockets validated structurally
* DeepSurf has been tested on it
* Fragment co-crystals exist

### **Secondary System Options (Pick One):**

1. **HSP90**
2. **P38α**
3. **BRAF**
4. **BRD4**
5. **GPCR** (optional but harder)

Secondary system shows **generalization**, which reviewers always check.

---

# **(4) We Move One Step at a Time**

So the next step is very small.
Please reply with **just two letters/numbers**, like this:

```
Primary: A
Secondary: 1
```

Example valid responses:

```
Primary: A
Secondary: 2
```

or

```
Primary: B
Secondary: 4
```

Once you reply, I will:

✔ Lock those systems into the paper outline
✔ Map datasets & references
✔ Align evaluation metrics
✔ Prevent scope creep

Your turn — just reply with:

**Primary: _**
**Secondary: _**

