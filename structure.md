
allostery-surrogate/
│
├─ README.md
├─ environment-gpu.yml
├─ bootstrap-gcp.sh
│
├─ src/
│   ├─ main.py
│   ├─ scmfm/
│   │    ├─ model.py
│   │    ├─ train.py
│   │    ├─ utils.py
│   ├─ diffusion/
│   │    ├─ conditional_diffusion.py
│   │    ├─ se3_layers.py
│   │    ├─ pose_rotamer_sampling.py
│   ├─ refine/
│   │    ├─ openmm_refine.py
│   ├─ baseline/
│   │    ├─ deepsurf_infer.py
│   │    ├─ fpocket_infer.py
│   ├─ evaluation/
│   │    ├─ rmsd_eval.py
│   │    ├─ chi_angle_eval.py
│   │    ├─ runtime_eval.py
│
└─ data/
    ├─ protein/
    ├─ fragments/
    └─ results/
