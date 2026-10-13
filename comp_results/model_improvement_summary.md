# Model Improvement Summary

## Evaluation population

The paired comparison files contain 3,601 clips. This is 131 fewer than the
3,732 rows in the baseline metric files. The comparison population contains:

| Category | Clips | Share |
|---|---:|---:|
| Near-linear | 1,928 | 53.5% |
| Non-rigid | 1,171 | 32.5% |
| Occlusion | 322 | 8.9% |
| Rotation | 180 | 5.0% |

All treatment results are paired with the corresponding model baseline. Higher
PSNR and SSIM are better; lower LPIPS is better. Confidence intervals are 95%
bootstrap confidence intervals for the mean paired improvement.

## Overall improvement from baseline

| Model | Treatment | PSNR improvement | SSIM improvement | LPIPS reduction | Improved clips: PSNR / SSIM / LPIPS |
|---|---|---:|---:|---:|---:|
| RIFE | Generic | +0.1427 dB [+0.1358, +0.1499] | +0.001256 | 0.000045 | 93.9% / 98.1% / 66.6% |
| RIFE | Rotation-targeted | +0.0322 dB [+0.0293, +0.0351] | +0.000227 | 0.000150 | 66.9% / 76.4% / 90.1% |
| FILM | Generic | +0.0099 dB [+0.0070, +0.0127] | +0.000063 | 0.000070 | 57.4% / 60.5% / 78.3% |
| FILM | Rotation-targeted | +0.0070 dB [+0.0045, +0.0095] | +0.000054 | 0.000108 | 55.4% / 58.5% / 87.6% |
| IFRNet | Generic | +0.0204 dB [+0.0191, +0.0217] | +0.000298 | 0.000122 | 70.6% / 91.2% / 79.8% |
| IFRNet | Rotation-targeted | -0.0003 dB [-0.0013, +0.0007] | +0.000013 | 0.000062 | 48.2% / 55.8% / 88.0% |

## Rotation-targeted treatment versus generic treatment on rotation clips

The table compares the rotation-targeted result directly with the generic
treatment on the 180 clips labelled `rotation`.

| Model | PSNR difference | SSIM difference | LPIPS difference | Interpretation |
|---|---:|---:|---:|---|
| RIFE | -0.0582 dB | -0.000822 | +0.000106 | Generic is better on all three metrics. |
| FILM | -0.0026 dB | -0.000004 | -0.000036 | Targeted improves LPIPS only; PSNR and SSIM are effectively unchanged/slightly lower. |
| IFRNet | -0.0113 dB | -0.000228 | +0.000112 | Generic is better on all three metrics. |

## Category-level generic-treatment highlights

| Model | Strongest PSNR gain | Largest LPIPS reduction |
|---|---|---|
| RIFE | Near-linear: +0.1757 dB | Rotation: 0.000333 |
| FILM | Near-linear: +0.0108 dB | Rotation: 0.000095 |
| IFRNet | Near-linear: +0.0254 dB | Rotation: 0.000194 |

## Runtime

| Model | Mean inference time | Median | 95th percentile | Frames per second |
|---|---:|---:|---:|---:|
| RIFE | 9.30 ms | 9.06 ms | 10.38 ms | 107.5 |
| IFRNet | 11.03 ms | 10.75 ms | 13.17 ms | 90.7 |
| FILM | 173.63 ms | 168.64 ms | 170.45 ms | 5.8 |

## Conclusions

- RIFE generic gives the largest PSNR and SSIM improvement while retaining the
  fastest runtime.
- IFRNet generic gives consistent improvement across metrics and the lowest
  LPIPS among the generic-treatment outputs.
- FILM begins with the highest baseline PSNR and SSIM, but it is much slower
  and its treatment improvements are small.
- The rotation-targeted approach does not show a reliable rotation-specific
  advantage for RIFE or IFRNet. For FILM, it is only preferable when LPIPS is
  the sole priority.
