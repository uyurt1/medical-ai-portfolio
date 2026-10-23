# Methodology and experimental decisions

[Portfolio overview](../README.md) · [Results](RESULTS.md) · [Limitations](LIMITATIONS.md)

## Data and separation of evidence

The research used Medical Segmentation Decathlon (MSD) Task01_BrainTumour, a BraTS-derived dataset. The 484 labelled cases were partitioned by patient into **290 training, 97 development-validation and 97 reserved-test cases**, with disjoint identifiers. The 97-case internal test partition is distinct from MSD's separate 266-case official test set. No official challenge submission or external validation is claimed.

The documented Experiment 02 training subset used the first 40 training cases and up to 12 tumour-bearing axial slices per case. Historical checkpoint selection used eight development cases. Subsequent full-volume development assessment covered all 97 development cases, informed the final selection, and is therefore development evidence rather than an unbiased final estimate. Both completed development and reserved-test evaluations covered 15,035 slices each.

The historical checkpoint does not contain a runtime cohort/source manifest. The training description below comes from retained source and records, not a recovered execution manifest. Present-day hashes identify artifacts but cannot retrospectively prove every historical execution detail.

## Preprocessing and labels

Input channels are ordered FLAIR, T1-weighted, contrast-enhanced T1-weighted (T1Gd), then T2-weighted. Processing uses axial axis 2 in native array orientation. Each slice and modality is clipped at its first and ninety-ninth intensity percentiles, then scaled by the percentile range with a small numerical stabilizer (10⁻⁸).

Images are resized to 128 × 128 using bilinear interpolation with unaligned corners; masks use nearest-neighbour interpolation. Reported overlap is measured on the **128 × 128 × Z resized grid**, not native anatomy. Small target regions may disappear during resizing. No physical tumour-volume accuracy is established.

| Label | Dataset meaning | Regional membership |
|---|---|---|
| 0 | Background | None |
| 1 | Edema | Whole tumour |
| 2 | Non-enhancing tumour | Whole tumour; tumour core |
| 3 | Enhancing tumour | Whole tumour; tumour core; enhancing tumour |

These are the project's dataset label mappings. Enhancing tumour and ET refer to the same endpoint.

## Compact U-Net architecture

![Compact U-Net architecture and tensor sizes](../assets/unet-architecture.svg)

| Stage | Spatial size | Output channels |
|---|---:|---:|
| Input | 128 × 128 | 4 |
| Encoder 1 | 128 × 128 | 16 |
| Encoder 2 | 64 × 64 | 32 |
| Encoder 3 | 32 × 32 | 64 |
| Bottleneck | 16 × 16 | 128 |
| Decoder 3 | 32 × 32 | 64 |
| Decoder 2 | 64 × 64 | 32 |
| Decoder 1 | 128 × 128 | 16 |
| Output logits | 128 × 128 | 4 |

Each encoder, bottleneck and decoder block contains two padded 3 × 3 convolutions with ReLU. A 2 × 2 max pool downsamples between encoder stages and into the bottleneck. Decoder stages use 2 × 2 transposed convolutions at stride 2 and concatenate the matching encoder features before their convolution block. A final 1 × 1 convolution produces four logits; argmax produces the label map. The model has **482,228 parameters**, without batch normalization or dropout.

The design uses the U-Net architecture concept cited below. This schematic was drawn for this portfolio from the project's architecture; no published figure or external implementation is distributed.

## Training and model selection

The retained Experiment 02 source describes four continuation epochs from a baseline checkpoint, Adam with learning rate 0.0001, batch size 4 and seed 42. The objective combines weighted cross-entropy (class weights 0.15, 2, 4 and 5) with mean foreground soft Dice. These settings describe historical methodology; they are not a complete executable recipe for reconstructing the reported model.

Experiment 02 had stronger completed full-volume development evidence than Experiment 05, which suffered tumour-subclass collapse. V3 was a later experimental branch and did not produce the model used for the final independent test.

## V3: readiness control failed

The bounded eight-patient control completed 960 updates. Shared-positive **training** Dice changed as follows:

| Class | Before | After | Change |
|---|---:|---:|---:|
| Edema | 0.4650 | 0.6179 | +0.1528 |
| Non-enhancing | 0.4844 | 0.5270 | +0.0426 |
| Enhancing | 0.3872 | 0.6156 | +0.2284 |

Changes use the saved full-precision differences; displayed before/after values are rounded independently. These are training diagnostics on a small control cohort, not development or independent-test performance. Improved aggregate overlap did not satisfy readiness: non-enhancing improvement was below 0.05, only four of eight cases preserved that class within tolerance, and three cases had zero enhancing overlap.

The planned intervention was not run under the stop-on-failure rule. There is no paired treatment comparison and no basis for concluding that negative-slice sampling is ineffective. The pilot also does not establish annotation error or a unique optimization failure. Experiment 02 was retained using development evidence before independent-test evaluation.

## Frozen independent evaluation

One evaluation covered every axial slice from all 97 reserved patients, including tumour-free slices. The model, split, preprocessing and statistics were frozen before execution. Inference used CPU float32, batch size 2, seed 42, deterministic algorithms, four intra-operation threads and one inter-operation thread. Predictions used argmax with no ensemble, test-time augmentation or postprocessing.

Patient Dice, pooled confusion counts and prespecified uncertainty summaries were checked against saved outputs. No test inference was repeated for portfolio preparation and no test-driven optimization was performed. See [results](RESULTS.md) for definitions and uncertainty limits.

## What can be reproduced from this portfolio?

The public CSVs are sufficient to redraw the aggregate charts and verify displayed rounding, confusion-derived pooled scores and reported count ratios. They do not contain individual measurements needed to reconstruct bootstrap intervals. Exact training, inference and cohort verification require excluded private code, model artifacts, identifiers and data. Cross-platform numerical equivalence and independent retraining have not been established.

The private release checks recorded 81 passing core synthetic tests in a fresh environment on the same macOS arm64 host and 35 passing archived pilot tests. These checks support software validation, not clinical validity or independent replication of the scientific result.

## References and dataset access

- Antonelli et al. (2022), *The Medical Segmentation Decathlon*, Nature Communications 13, 4128. [Paper](https://doi.org/10.1038/s41467-022-30695-9).
- [MSD project and access information](https://medicaldecathlon.com/) and [download page](https://medicaldecathlon.com/dataaws/). Provider materials identify [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/); obtain data directly and review the current access and licence terms. No dataset material is redistributed here.
- Ronneberger, Fischer and Brox (2015), *U-Net: Convolutional Networks for Biomedical Image Segmentation*. [Paper](https://arxiv.org/abs/1505.04597).

Names of technologies identify tools used in the project; they do not imply affiliation or endorsement. No upstream software, paper figure or logo is bundled.
