# ExTurb: Extreme Value Gradient Statistics for Underwater Turbidity Estimation

Code, per-image results and reproduction scripts for the manuscript

> **Extreme Value Gradient Statistics for Underwater Turbidity Estimation and Their Validation in Real Water**
> Md Hasibuzzaman and Chan-Yun Yang
> Dept. of Electrical Engineering, College of Electrical Engineering and Computer Science, National Taipei University, New Taipei City, Taiwan
> Submitted to *IEEE Journal of Oceanic Engineering*, 2026.

## Abstract

Few underwater vehicles carry a turbidity meter, yet nearly every vehicle carries a camera. We study turbidity cues computed from the extreme values of locally normalised image gradients and summarised by a closed-form generalised extreme value fit. The resulting contrast index is exactly invariant to a common camera gain. Under stated assumptions it separates into scene, forward-scatter and veiling-light terms, and the ratio of the fitted locations at two scales gives the width of blurred edges. This inversion fails for about half of natural images, which are then flagged as invalid. On 1315 tank images and 1080 held-out fish images with turbidity measured in nephelometric turbidity units, the training-free index follows turbidity closely when only the water changes (Spearman rank correlation 8.10 × 10⁻¹) but not when the scene changes. With three calibrated coefficients the cues reach 8.36 × 10⁻¹ on the tank data and 5.38 × 10⁻¹ on an untouched hold-out split, among the leading methods but below a pair of simple contrast statistics, and they give the lowest median turbidity error when transferred from the tank to the fish images. A Type A uncertainty evaluation attributes 92% of the error variance to the scene and agrees with distribution-free prediction intervals. Upper percentiles of the same gradients outperform the fitted location, so the cues are best used on underwater vehicles as coarse indicators of water and image quality. Attenuation estimates are validated only in simulation.

**Keywords:** Underwater imaging; Turbidity; Extreme value theory; Autonomous underwater vehicles; Measurement uncertainty

---

## 1. Guide for reviewers

The results can be checked at three levels. Each level needs more data and time than the one before.

| Level | What it reproduces | Needs | Time (one CPU) |
|---|---|---|---|
| **A** | All real-data results: calibrated comparison, hold-out split, transfer, feature families, learning curves, uncertainty budget, conformal intervals | Only this repository (per-image scores are included) | about 3 minutes |
| **B** | Per-image scores on the real datasets, computed from the images | TUB and MFT images (public, Section 4) | a few hours |
| **C** | Synthetic benchmarks, theory checks, ablation, robustness, range and AUV experiments | Clear-scene image sets (public, Section 4) | about one day |

Run every command from the repository folder.

## 2. Installation

Python 3.10 or newer is required. No GPU is needed; PyTorch is used only for the supervised CNN baseline.

```bash
git clone https://github.com/hasib1233333/Extreme-Value-Gradient-Statistics-for-Underwater-.git
cd Extreme-Value-Gradient-Statistics-for-Underwater-
python -m venv venv && source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## 3. Running ExTurb on one image

```python
from skimage import io
from compute import frozen_params
from exturb import exturb

params, lam = frozen_params()              # frozen parameters from exturb_params.json
img = io.imread("frame.png") / 255.0       # RGB image scaled to [0, 1]
out = exturb(img, params)
print(out["T_c"], out["rho"], out["sigma_b"], out["sigma_valid"])
```

Always pass `params`: the defaults in `ExTurbParams` are development values, not the frozen ones used in the paper.

| Output key | Meaning |
|---|---|
| `T_c` | Contrast index; larger means more turbid. Training-free and invariant to a common camera gain |
| `rho` | Probe ratio of the GEV locations at the two scales |
| `sigma_b` | Edge-blur width, reported only when the inversion is valid |
| `sigma_valid` | 1 if the blur inversion has a solution, 0 if the width is not recoverable |
| `gev_scale`, `xi` | Scale and shape of the GEV fit to the block maxima |
| `mu1`, `mu2` | GEV locations at the fine and coarse probe scales |

`exturb_params.json` holds the frozen parameters: long side N_w = 512 px, probe scales s1 = 1.5 px and s2 = 3.0 px, block size B = 32 px, Weber scale s_w = 8 px, regulariser epsilon = 10⁻³ and weight lambda = 0.5. They were chosen on synthetic dead-leaves images only and were not tuned on any real dataset.

**Turbidity in NTU (ExTurb-C).** `frozen_calibration.json` holds the three coefficients `coef = [a1, a2, a0]` fitted on MFT Train (key `mft`, model `ExTurb-C`):

```
ln NTU = a1 * T_c + a2 * ln(rho) + a0
```

The key `q` is the 90% split-conformal half-width in ln NTU, so the interval is NTU × exp(±q). The coefficients apply to the MFT camera and turbidity range; recalibrate them for a new camera, site or range.

## 4. Data

All data are public. Level A needs no download.

| Dataset | Used for | Source |
|---|---|---|
| TUB | 1315 tank images with measured NTU | `huggingface.co/datasets/vapaau/TUB` |
| MFT | Fish images with measured NTU (Train 4320, Test 540, Eva 540) | IEEE Dataport, doi:10.21227/gd5d-ns10; mirror `huggingface.co/datasets/mieraehh/5_Malaysian_Fish_Species_in_Turbid_Water` |
| RUIE (UIQS, UCCS) | Real underwater photographs | `github.com/dlut-dimt/Realworld-Underwater-Image-Enhancement-RUIE-Benchmark` |
| U45 | Real underwater photographs | `github.com/IPNUISTlegal/underwater-test-dataset-U45-` |
| EUVP | Paired distorted and reference images | `github.com/xahidbuffon/FUnIE-GAN` (`data/test`) |
| BSDS500 | Clear scenes for rendering | `github.com/BIDS/BSDS500` |
| Kodak24, McMaster | Clear scenes for rendering | test sets in `github.com/cszn/FFDNet` |

Place downloads under `data/`. `dl.py` fetches the Hugging Face datasets.

## 5. Level A: real-data results from the included scores

```bash
python real_calib.py real_data      # ~75 s  calibrated comparison and significance tests
python real_ref.py real_data        # ~5 s   scene-referenced estimates and conformal intervals
python eva_study.py real_data       # ~40 s  untouched MFT Eva split and feature families
python transfer_study.py            # ~1 s   calibration transfer and coverage after transfer
python ext_gev.py                   # ~7 s   full-GEV model and learning curves
python metrology.py                 # <1 s   Type A uncertainty budget
```

| Script | Manuscript item | Output files |
|---|---|---|
| `real_calib.py` | Tables XIII and XIV | `real_data/calib/real_metrics.csv`, `real_significance.csv`, `P1_pred.csv`, `P2_pred.csv` |
| `real_ref.py` | Fig. 7 intervals, scene-referenced results | `real_data/calib/tub_scene_referenced.csv`, `tub_ref_pred.csv`, `conformal.json` |
| `eva_study.py` | Tables X and XV | `real_data/revision/eva_frozen.csv`, `eva_sig.csv`, `eva_sig_transfer.csv`, `feature_families.csv`, `feature_families_sig.csv` |
| `transfer_study.py` | Table XI | `real_data/revision/transfer_study.csv`, `transfer_summary.csv`, `transfer_coverage_by_category.json` |
| `ext_gev.py` | Fig. 8 | `real_data/revision/learning_curves.csv`, `learning_curves_summary.csv`, `ext_gev.json` |
| `metrology.py` | Table XII | `real_data/metrology.json` |

Each script overwrites its output files, so the regenerated numbers can be compared directly with the copies in this repository (`git diff` shows any change). Bootstrap resampling uses fixed seeds.

**Hold-out protocol.** `frozen_calibration.json` was written by `freeze_calibration.py` before the MFT Eva split was processed, and it records the SHA-256 hash of every input file. Do not rerun `freeze_calibration.py` when checking the results, because it replaces the frozen file.

## 6. Level B: scores from the real images

```bash
python run_real_ntu.py --tub_dir data/tub                        # NTU read from av_turbidity.csv
python run_real_ntu.py --mft_dir data/mft --mft_split Train      # NTU read from Multimodal Metadata.xlsx
python run_real_ntu.py --mft_dir data/mft --mft_split Test
python run_real_ntu.py --mft_dir data/mft --mft_split Eva
python extract_features.py                                       # gradient statistics for Table XV
```

Rename the MFT outputs to `MFT_train_scores.csv`, `MFT_test_scores.csv` and `MFT_eva_scores.csv` in `real_data/`, then run Level A again.

## 7. Level C: synthetic experiments

```bash
python tune_syndl.py              # parameter selection on dead-leaves images
python export_syndl.py
python build_sets.py bsds         # BSDS-UW renders
python build_sets.py km           # KM-UW renders
python build_sets.py robust       # Robust-UW renders
python build_sets.py range        # Range-UW renders
./pipeline.sh                     # scores of all methods, CNN baseline, theory checks, ablation, sensitivity, exposure
python syn_revision.py            # PSF approximation, edge types, camera pipeline, GEV ablation, range intervals
python sim_rho.py                 # factorial simulation of the probe ratio
python validity_study.py          # validity of the blur inversion
python rev4.py                    # information carried by the GEV parameters
./run_plume.sh                    # AUV plume survey (fixed and adaptive altitude) and final analysis
```

| Script | Manuscript item |
|---|---|
| `theory_exp.py` | Fig. 3 |
| `validity_study.py` | Table V |
| `sim_rho.py` | Fig. 4 |
| `syn_revision.py` | Fig. 5, Tables VI and VII, range intervals |
| `compute.py`, `analysis.py`, `analysis2.py` | Tables VIII and IX, Figs. 6, 9 and 10 |
| `rev4.py` | Table XVI |
| `exposure_test.py` | Table XVII |
| `ablation.py` | Table XVIII, Fig. S1 |
| `plume_sim.py` | Fig. 11 |
| `cnn_baseline.py` | Supervised CNN baseline rows |
| `real_figs.py` | Fig. 7 |

Generated tables and number macros are written to `paper/`.

## 8. Repository layout

| Path | Contents |
|---|---|
| `exturb.py` | The estimator (Algorithm 1) |
| `exturb_params.json` | Frozen estimator parameters |
| `frozen_calibration.json` | Frozen ExTurb-C coefficients and conformal quantile |
| `render.py`, `deadleaves.py`, `datasets.py`, `build_sets.py` | Physically based renderer and benchmark construction |
| `baselines.py` | UCIQE, UIQM, DCP, UDCP, dynamic range, RMS contrast, Tenengrad, Laplacian variance, entropy, PCD |
| `stats_eval.py` | SROCC, bootstrap intervals, Holm correction, Friedman test |
| `rangemodel.py` | Population model of T_c against optical depth and its inversion |
| `real_data/` | Per-image scores of every method on TUB and MFT, features, calibration outputs |
| `real_data/revision/` | Results of the validity, transfer, feature-family, learning-curve, identifiability and uncertainty analyses |
| `paper_figures/` | Figures of the manuscript and the supplementary material |
| `paper_results/` | Every number reported in the manuscript, as generated LaTeX macros |
| `results/`, `bench/`, `data/`, `paper/` | Working folders used by the scripts |

| Manuscript figure | File in `paper_figures/` |
|---|---|
| Figs. 1 and 2 | `overview.pdf`, `flow.pdf` |
| Figs. 3, 4 and 5 | `theory.pdf`, `sim_rho.pdf`, `psf_error.pdf` |
| Figs. 6, 7 and 8 | `scatter_bsds.pdf`, `real_ntu.pdf`, `learning_curves.pdf` |
| Figs. 9, 10 and 11 | `robust.pdf`, `range.pdf`, `plume.pdf` |
| Figs. S1 and S2 | `sensitivity.pdf`, `qualitative.jpg` |

## 9. Known limitations

* NTU is not the beam attenuation of the image model; attenuation estimates are validated only in simulation.
* The ExTurb-C coefficients and intervals hold for the camera and turbidity range they were fitted on, and lose coverage after a change of dataset.
* The blur inversion has no solution for many natural images; such images are flagged, not assigned a width.

## Citation

```bibtex
@article{hasibuzzaman2026exturb,
  author  = {Hasibuzzaman, Md and Yang, Chan-Yun},
  title   = {Extreme Value Gradient Statistics for Underwater Turbidity Estimation and Their Validation in Real Water},
  journal = {IEEE Journal of Oceanic Engineering},
  note    = {Submitted},
  year    = {2026}
}
```

## Contact

Md Hasibuzzaman (corresponding author), hsbuzzaman@gmail.com
Chan-Yun Yang, cyyang@mail.ntpu.edu.tw, cyyang.research@gmail.com
