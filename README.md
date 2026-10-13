<div align="center">

# Motion-Type-Aware Video Frame Interpolation

### Diagnose how frozen VFI models fail — then test a lightweight residual correction.

[![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](#)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.x-EE4C2C?logo=pytorch&logoColor=white)](#)
[![TensorFlow](https://img.shields.io/badge/TensorFlow-2.13-FF6F00?logo=tensorflow&logoColor=white)](#)
[![Models](https://img.shields.io/badge/Baselines-RIFE%20%7C%20FILM%20%7C%20IFRNet-7C3AED)](#)

<img src="/assets/vfi-pipeline.svg" alt="Animated motion-aware VFI pipeline" width="100%" />

</div>

## What this project asks

Most video-frame-interpolation (VFI) evaluations collapse all clips into one average score. That can hide an important question: **which motion condition is each model worst at?**

This project evaluates frozen, pretrained RIFE, FILM, and IFRNet models across four interpretable motion categories, then compares a generic residual correction module with a rotation-targeted counterpart.

> This is a diagnosis-to-correction study, not a new VFI backbone.

## At a glance

| Component | Approach |
|---|---|
| Input | Vimeo-90K frame triplets: `I₀`, ground-truth middle frame, `I₁` |
| Motion diagnosis | Frozen RAFT descriptors + calibrated motion classifier |
| Categories | Near-linear, rotation/curved, occlusion-heavy, non-rigid |
| Frozen baselines | RIFE, FILM, IFRNet |
| Metrics | PSNR ↑, SSIM ↑, LPIPS ↓ |
| Correction | 9-channel residual CNN with learned confidence mask |
| Evaluation set | 3,601 paired clips |

## Pipeline

```text
I₀ + I₁ ──► RAFT descriptors ──► motion category
  │
  └──────► frozen VFI baseline ──► Iᵦ ──► residual correction ──► Iᶜ
                                                │
                                        Iᶜ = Iᵦ + M ⊙ R
```

The correction network receives `[I₀, I₁, Iᵦ]`, predicts a residual image `R` and a one-channel confidence mask `M`, then applies the correction selectively. Ground truth is used only for training losses and evaluation — never as an inference input.

## Results

### Dataset composition

| Motion category | Clips | Share |
|---|---:|---:|
| Near-linear | 1,928 | 53.5% |
| Non-rigid | 1,171 | 32.5% |
| Occlusion | 322 | 8.9% |
| Rotation | 180 | 5.0% |

### Mean change from the frozen baseline

Positive PSNR/SSIM changes and positive LPIPS reduction are improvements.

| Model | Correction | Δ PSNR (dB) | Δ SSIM | LPIPS reduction |
|---|---|---:|---:|---:|
| RIFE | Generic | **+0.1427** | **+0.001256** | 0.000045 |
| RIFE | Rotation-targeted | +0.0322 | +0.000227 | 0.000150 |
| FILM | Generic | +0.0099 | +0.000063 | 0.000070 |
| FILM | Rotation-targeted | +0.0070 | +0.000054 | **0.000108** |
| IFRNet | Generic | +0.0204 | +0.000298 | **0.000122** |
| IFRNet | Rotation-targeted | −0.0003 | +0.000013 | 0.000062 |

### Inference time for frozen baselines

| Model | Mean latency | FPS |
|---|---:|---:|
| RIFE | **9.30 ms** | **107.5** |
| IFRNet | 11.03 ms | 90.7 |
| FILM | 173.63 ms | 5.8 |

### Takeaways

- RIFE’s generic correction delivers the largest average PSNR and SSIM gain.
- FILM starts with strong image quality but is substantially slower.
- Rotation-targeted correction does **not** consistently outperform generic correction on the 180 rotation clips. That negative result is intentional and part of the study’s evidence.
- The current targeted correction does not meet the pre-defined +0.5 dB targeted-PSNR success criterion.

For full paired-result tables and confidence intervals, see [the improvement summary](notebooks/comp_results/model_improvement_summary.md).

## Repository guide

| File | Purpose |
|---|---|
| [VFI project paper](/VFI.pdf) | Full methodology, equations, experimental protocol, and discussion |
| [RAFT diagnosis notebook](notebooks/raft-vfi.ipynb) | Descriptor extraction, calibration, final category classification, and diagnostic scoring |
| [RIFE notebook](notebooks/rife-bp-cnn.ipynb) | Frozen RIFE baseline and correction experiments |
| [FILM notebook](notebooks/film-vfi-3-10-env.ipynb) | Frozen FILM baseline and correction experiments |
| [IFRNet notebook](notebooks/ifrnet-vfi.ipynb) | Frozen IFRNet baseline and residual-correction implementation |
| [Comparison outputs](notebooks/comp_results) | Per-clip metric CSVs and result summary |

The IFRNet notebook has a preserved pre-alignment snapshot, [`ifrnet-vfi-before-residual-alignment.ipynb`](notebooks/ifrnet-vfi-before-residual-alignment.ipynb), retained for provenance.

## Reproducing the workflow

The notebooks were run in Kaggle with GPU acceleration and use dataset/checkpoint paths configured for that environment.

1. Add the required Vimeo-90K triplets, pretrained VFI checkpoints, RAFT source/checkpoint, split manifest, and saved descriptor/metric CSVs as Kaggle inputs.
2. Run [`raft-vfi.ipynb`](notebooks/raft-vfi.ipynb), following its **final 170-clip calibration** reading guide.
3. Run the desired baseline notebook. Each correction stage keeps the VFI backbone frozen and trains only the lightweight correction CNN.
4. Evaluate on the fixed 3,601-clip test set with PSNR, SSIM, and LPIPS.

> **Note on historical outputs:** notebooks intentionally retain execution outputs and earlier exploratory cells for auditability. The RAFT notebook explicitly marks the authoritative final calibration path. The current IFRNet correction architecture is aligned with RIFE/FILM; rerunning it requires checkpoints trained for that architecture.

## Citation

If you build on this repository, please cite the accompanying project report:

```bibtex
@report{motion_aware_vfi,
  title  = {Motion-Type-Aware Diagnostic Benchmarking and Targeted Correction of Video Frame Interpolation Models},
  year   = {2026}
}
```

---

<div align="center">
  <sub>Built around interpretable failure analysis for video frame interpolation.</sub>
</div>
