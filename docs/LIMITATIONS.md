# Limitations and future work

[Portfolio overview](../README.md) · [Methodology](METHODOLOGY.md) · [Results](RESULTS.md)

## What the evidence supports

The study establishes the recorded performance of one selected model on one reserved internal cohort under a fixed resized-grid evaluation. Whole-tumour mean Dice was 0.6628; the tumour-core and enhancing scores of 0.4606 and 0.2386 show considerably weaker regional detail. Similar development and test means do not establish equivalence, generalization to hospitals or clinical usefulness.

## Material limitations

| Limitation | Practical implication |
|---|---|
| Weak subclass segmentation | Zero overlap occurred for non-enhancing tumour in 6/97 and enhancing tumour in 8/96 truth-present test cases. Whole-tumour overlap alone conceals these failures. |
| Frequent false positives | Tumour was predicted on 2,892/8,231 native tumour-free slices (35.14%). The model is not a validated lesion-detection system. |
| Resized scoring grid | Downsampling can remove small lesions; native-resolution accuracy and physical-volume accuracy are unknown. |
| Slice-based model | Independent 2D slices lack through-plane context and do not enforce volumetric consistency. |
| Limited training coverage | The documented training subset used 40 cases and tumour-bearing slices; this restricts exposure to diverse anatomy and tumour-free slices. |
| Single internal test cohort | No external-site, prospective, demographic-subgroup or acquisition-shift assessment was performed. |
| Limited uncertainty scope | Patient bootstrap intervals omit training-seed variability and do not certify transportability or calibration. |
| Incomplete historical provenance | Source and lineage records cannot replace a missing execution-time cohort/source manifest. |
| Restricted public artifacts | This portfolio supports inspection of aggregate evidence, not independent reconstruction of the complete experiment. |

No clinical safety, calibration, fairness, regulatory clearance or diagnostic benefit is claimed. The project is a research and educational exercise, not a patient-care tool.

## Proposed improvements — not evaluated

1. **Strengthen experimental provenance.** Record execution-time source, cohort, seed and environment manifests for all future runs, and report variability across multiple training seeds.
2. **Test spatial context.** Compare 2.5D or 3D inputs against the compact 2D baseline using a predefined development protocol and matched compute budgets.
3. **Study sampling and objectives.** Assess tumour-free slice exposure and class-aware sampling only after predefined readiness criteria pass. V3 did not test the planned intervention, so its effect remains unknown.
4. **Assess native-space performance.** Preserve image geometry and evaluate small lesions and volumetric consistency at native resolution before making anatomical measurements.
5. **Evaluate broader validity.** Use appropriately governed external data to assess acquisition shifts, relevant subgroups, uncertainty and clinically meaningful endpoints.

These proposals are hypotheses, not achieved improvements. Further model selection would require a new development protocol and a genuinely untouched final evaluation cohort; the completed test must not become a tuning target.
