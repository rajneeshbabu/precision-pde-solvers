# Effect of Floating-Point Precision on Finite-Difference PDE Solvers

[![GitHub Pages](https://img.shields.io/badge/🌐_Project_Page-GitHub_Pages-6366f1?style=for-the-badge)](https://rajneeshbabu.github.io/precision-pde-solvers/)
[![IISc](https://img.shields.io/badge/IISc-DS_289_NSDE-003580?style=for-the-badge)](https://cds.iisc.ac.in)
[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python)](https://python.org)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.0+-EE4C2C?style=for-the-badge&logo=pytorch)](https://pytorch.org)

**Course:** DS 289 / Numerical Solutions of Differential Equations (NSDE)  
**Institution:** Indian Institute of Science (IISc), Bengaluru  
**Project Page:** [rajneeshbabu.github.io/precision-pde-solvers](https://rajneeshbabu.github.io/precision-pde-solvers/)

| Name | SR No. |
|------|--------|
| Nishchith C. Bharadwaj | 26650 |
| Dhruv Yadav | 26641 |
| Rajneesh Babu | 26058 |

---

## Overview

This project investigates how **floating-point precision** (FP64 / FP32 / FP16) affects the accuracy, stability, conservation, and computational cost of four finite-difference numerical schemes applied to two canonical PDEs:

- **Linear advection-diffusion:** $u_t + c\,u_x = \nu\,u_{xx}$
- **Viscous Burgers' equation:** $u_t + \left(\frac{u^2}{2}\right)_x = \nu\,u_{xx}$

Each scheme is a **fully self-contained Jupyter notebook**. All four share an identical six-experiment catalogue and diagnostics pipeline so results are directly comparable across schemes.

---

## Schemes

| Scheme | Spatial Method | Flux | Time Integration | Order |
|--------|---------------|------|-----------------|-------|
| 1 | Upwind / Godunov | Godunov (exact Riemann) | Forward Euler | 1st / 1st |
| 2 | Lax–Friedrichs / Rusanov | Rusanov (local LxF) | Forward Euler | 1st / 1st |
| 3 | Lax–Wendroff / Richtmyer | Richtmyer predictor–corrector | Forward Euler | 2nd / 2nd |
| 4 | MUSCL + slope limiter | Rusanov | SSP-RK2 (Shu–Osher) | 2nd / 2nd |

---

## Experiments

All four schemes run the same six experiments:

| # | Name | Purpose | Grid | CFL |
|---|------|---------|------|-----|
| 01 | `baseline_regime_sweep` | Accuracy across Re = 10, 100, 1000 | nx=1024 | 0.80 |
| 02 | `ultra_high_re_shock_stress` | Shock sharpening at Re = 100,000 | nx=1024 | 0.80 |
| 03 | `underresolved_high_re_nx128` | Coarse grid — FP16 breakdown | nx=128 | 0.80 |
| 04 | `tiny_amplitude_quantization` | Tiny amplitude (1e-6) — precision floor | nx=1024 | 0.80 |
| 05 | `long_horizon_drift` | Conservation & TV drift over extended time | nx=1024 | 0.50 |
| 06 | `cfl_overdrive_failure` | Overdriven CFL = 1.2 — method failure | nx=256 | 1.20 |

Each experiment runs each solver three times — once per floating-point type:

| Precision | `torch.dtype` | Bits | Mantissa bits |
|-----------|--------------|------|---------------|
| FP64 | `torch.float64` | 64 | 52 |
| FP32 | `torch.float32` | 32 | 23 |
| FP16 | `torch.float16` | 16 | 10 |

---

## Key Findings

- **FP64** consistently achieves the lowest L2 / Linf errors across all schemes and experiments.
- **FP32** introduces small but measurable errors at high Reynolds numbers (Re ≥ 1000); adequate for most problems.
- **FP16** degrades significantly in high-Re and long-horizon experiments due to limited mantissa (10 bits). Catastrophic cancellation in tiny-amplitude waves reduces effective accuracy to near zero.
- **Scheme 4 (MUSCL + SSP-RK2)** maintains TVD behaviour even at FP32; costs ~3× more FLOPs than Scheme 1 but achieves substantially higher accuracy per grid point.
- At CFL = 0.80, all schemes are stable for FP64 and FP32. **FP16 loses stability** on under-resolved, high-Re grids (exp03: nx=128, Re=100,000).

---

## Repository Structure

```
precision-pde-solvers/
├── scheme1/
│   └── scheme1(Godunov+Forward_Euler).ipynb
├── scheme2/
│   └── scheme2(Lax-Friedrichs_Rusanov+Forward_Euler).ipynb
├── scheme3/
│   └── scheme3(Lax-Wendroff).ipynb
├── scheme4/
│   └── scheme4_MUSCL_Rusanov_SSPRK2.ipynb
├── report/
│   └── Effect_of_Precision_on_PDE_solvers.pdf
├── requirements.txt
├── README.md
└── index.html
```

---

## Installation

```bash
git clone https://github.com/rajneeshbabu/precision-pde-solvers.git
cd precision-pde-solvers
pip install -r requirements.txt
```

> **GPU (optional):** All solvers run on CPU by default (`prefer_cuda=False`). Set `prefer_cuda=True` in `ExperimentConfig` and install a CUDA-capable PyTorch build to use a GPU.

---

## Running the Notebooks

Each notebook is **fully self-contained** — no external modules needed. Run all cells **top to bottom**; the final cell executes the complete six-experiment catalogue automatically.

### Option 1 — Jupyter Notebook (classic)

```bash
# Scheme 1: Upwind / Godunov + Forward Euler
jupyter notebook "scheme1/scheme1(Godunov+Forward_Euler).ipynb"

# Scheme 2: Lax–Friedrichs / Rusanov + Forward Euler
jupyter notebook "scheme2/scheme2(Lax-Friedrichs_Rusanov+Forward_Euler).ipynb"

# Scheme 3: Lax–Wendroff / Richtmyer
jupyter notebook "scheme3/scheme3(Lax-Wendroff).ipynb"

# Scheme 4: MUSCL + Rusanov + SSP-RK2
jupyter notebook "scheme4/scheme4_MUSCL_Rusanov_SSPRK2.ipynb"
```

### Option 2 — JupyterLab

```bash
jupyter lab
```
Then open the desired notebook from the file browser in the left panel.

### Option 3 — Run all schemes from the command line

```bash
jupyter nbconvert --to notebook --execute --inplace \
    "scheme1/scheme1(Godunov+Forward_Euler).ipynb" \
    "scheme2/scheme2(Lax-Friedrichs_Rusanov+Forward_Euler).ipynb" \
    "scheme3/scheme3(Lax-Wendroff).ipynb" \
    "scheme4/scheme4_MUSCL_Rusanov_SSPRK2.ipynb"
```

### Expected output (last cell of each notebook)

```
================================================================================
Notebook: running all experiments (from build_experiments)
================================================================================
Running exp01_baseline_regime_sweep          ✓ Completed
Running exp02_ultra_high_re_shock_stress     ✓ Completed
Running exp03_underresolved_high_re_...      ✓ Completed
Running exp04_tiny_amplitude_quantization    ✓ Completed
Running exp05_long_horizon_drift_...         ✓ Completed
Running exp06_cfl_overdrive_failure          ✓ Completed
```

> `RuntimeWarning: invalid value encountered in reduce` from NumPy is **expected** for FP16 runs in high-Re regimes — not a code error.

---

## Output Structure

Running any notebook creates this layout **inside the scheme folder**:

```
scheme<N>/
├── all_experiments_summary.csv
├── failed_experiments.csv
├── exp01_baseline_regime_sweep/
│   ├── linear_profiles.png
│   ├── linear_error_bar.png
│   ├── burgers_profiles_re_sweep.png
│   ├── burgers_error_vs_re.png
│   ├── conservation_error.png
│   ├── total_variation.png
│   ├── runtime_comparison.png
│   └── table_*.png
├── exp02_ultra_high_re_shock_stress/
├── exp03_underresolved_high_re_breakdown_nx128/
├── exp04_tiny_amplitude_quantization_stress/
├── exp05_long_horizon_drift_accumulation/
└── exp06_cfl_overdrive_method_failure/
```

---

## References

1. LeVeque, R. J. — *Finite Volume Methods for Hyperbolic Problems*, CUP, 2002.
2. Toro, E. F. — *Riemann Solvers and Numerical Methods for Fluid Dynamics*, Springer, 2009.
3. Shu, C.-W. & Osher, S. — "Efficient implementation of essentially non-oscillatory shock-capturing schemes," *J. Comput. Phys.*, 77(2):439–471, 1988.
4. van Leer, B. — "Towards the ultimate conservative difference scheme V," *J. Comput. Phys.*, 32(1):101–136, 1979.

---

*© 2025 Rajneesh Babu · Nishchith C. Bharadwaj · Dhruv Yadav · IISc Bengaluru*
