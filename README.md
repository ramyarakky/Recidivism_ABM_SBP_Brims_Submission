# Recidivism ABM — SBP-BRiMS 2026 Calibration

**Paper:** *An Agent-Based Model of Recidivism*  
**Conference:** SBP-BRiMS 2026 — Social, Cultural, and Behavioral Modeling  
**Authors:** Ramya Rakkiappan and Hamdi Kavak  
**Institution:** George Mason University  

---

## Overview

This repository contains the agent-based model, calibration scripts, validation outputs, and figures used in the SBP-BRiMS 2026 paper *An Agent-Based Model of Recidivism*.

The model represents post-release justice-system trajectories through four states: Trial, Prison, Supervision, and Free. It is calibrated under race- and gender-neutral assumptions to reproduce three empirical structures:

| Stage | Calibration structure | Empirical source | Parameters |
|---|---|---|---|
| 1 | National cumulative rearrest rates at 3, 6, and 9 years | Alper et al. (2018), BJS NCJ 250975 | `alpha`, `delta_s3`, `delta_s6` |
| 2 | PCRA risk-tier cumulative rearrest rates at 3, 6, and 9 years | Johnson (2023), *Federal Probation* 87(2) | `gamma` |
| 3 | Offense-specific cumulative rearrest rates at 3, 6, and 9 years | Alper et al. (2018), BJS NCJ 250975, Table 7 | `o_v`, `o_d`, `o_p`, `o_o` |

Previously calibrated parameters are held fixed when the next stage is evaluated. This sequential one-at-a-time grid-search design narrows the set of parameter configurations consistent with the selected empirical targets, while higher-order interactions remain outside the scope of the conference paper.

---

## SBP-BRiMS Model Configuration

| Configuration item | SBP-BRiMS value | Role |
|---|---:|---|
| Initial agents | 1,500 | Initial synthetic justice-system population |
| Warm-up period | 144 months | Establishes a stable pre-study population |
| Fixed monthly intake | 100 agents/month | Equal to 10% of the original 1,000-agent design; not compounded growth |
| Approximate end-of-warm-up population | 23,000 agents | Includes initial agents and fixed monthly inflow before exits |
| Study cohort | Approximately 5,000 eligible agents | Agents in Free or Supervision states retained after warm-up |
| Study period | 108 months | Matches the 9-year BJS follow-up |
| Outcome checks | Quarterly | Rearrest hazard evaluated every three months |
| Outcome definition | First rearrest | Absorbing measurement during the study period |
| Peer influence | Enabled | Applied during incarceration |
| Bias factor | 0.0 | Race- and gender-neutral baseline |
| Replications per candidate value | 10 | Repeated stochastic runs |
| Random seeds per replication | 10 | Common seed set |
| Simulations per candidate value | 100 | 10 replications × 10 seeds |

---

## Calibration Procedure

| Step | Procedure |
|---|---|
| 1 | Select one candidate value from the parameter's predefined sweep. |
| 2 | Run 10 replications across 10 random seeds, producing 100 simulations per candidate value. |
| 3 | Compute the stage-specific mean absolute error. |
| 4 | Select the candidate value with the minimum loss. |
| 5 | Lock the selected value before proceeding to the next parameter or stage. |

### Loss Functions

| Stage | Loss function | Number of target cells |
|---|---|---:|
| 1 | Mean absolute error across national 3-, 6-, and 9-year cumulative rearrest rates | 3 |
| 2 | Mean absolute error across 4 PCRA tiers × 3 follow-up windows | 12 |
| 3 | Mean absolute error across 4 offense groups × 3 follow-up windows | 12 |

---

# Stage 1 — National Rearrest Calibration

## Stage 1 Targets

| Follow-up window | BJS target | Calibration status | Source |
|---|---:|---|---|
| 1 year | 43.9% | Diagnostic only; not included in the loss | Alper et al. (2018) |
| 3 years | 68.4% | Calibration target | Alper et al. (2018), BJS NCJ 250975 |
| 6 years | 79.4% | Calibration target | Alper et al. (2018), BJS NCJ 250975 |
| 9 years | 83.4% | Calibration target | Alper et al. (2018), BJS NCJ 250975 |

## Stage 1 Parameters and Sweeps

| Symbol | Code parameter | Meaning | Baseline | Sweep specification | Selected value | Primary target |
|---|---|---|---:|---|---:|---|
| α | `Supervision_Monitoring_Intensity` | Overall supervision monitoring intensity | 1.000 | 1.00–1.20; 11 values; step 0.02 | **1.120** | 3-, 6-, and 9-year aggregate MAE |
| δ_s3 | `Supervision_Monitoring_Decay_After_3Y` | Supervision intensity multiplier during years 3–6 | 1.000 | 0.60–0.99; 13 values; step 0.0325 | **0.990** | 6-year aggregate error |
| δ_s6 | `Supervision_Monitoring_Decay_After_6Y` | Supervision intensity multiplier during years 6–9 | 1.000 | 0.20–0.70; 11 values; step 0.05 | **0.400** | 9-year aggregate error |

## Exact Stage 1 Sweep Values

| Symbol | Candidate values |
|---|---|
| α | `1.000, 1.020, 1.040, 1.060, 1.080, 1.100, 1.120, 1.140, 1.160, 1.180, 1.200` |
| δ_s3 | `0.6000, 0.6325, 0.6650, 0.6975, 0.7300, 0.7625, 0.7950, 0.8275, 0.8600, 0.8925, 0.9250, 0.9575, 0.9900` |
| δ_s6 | `0.200, 0.250, 0.300, 0.350, 0.400, 0.450, 0.500, 0.550, 0.600, 0.650, 0.700` |

## Fixed BJS-Anchored Desistance Parameters

| Symbol | Code parameter | Period | Search status | Fixed value | Derivation |
|---|---|---|---|---:|---|
| dr1 | `Risk_Effect_Decay_After_1Y` | Years 1–3 | Not swept | **0.524** | BJS quarterly-hazard decomposition |
| dr3 | `Risk_Effect_Decay_After_3Y` | Years 3–6 | Not swept | **0.500** | BJS quarterly-hazard decomposition |
| dr6 | `Risk_Effect_Decay_After_6Y` | Years 6–9 | Not swept | **0.508** | BJS quarterly-hazard decomposition |

## Stage 1 Aggregate Results

| Follow-up window | Uncalibrated model | Calibrated model | BJS target | Absolute error |
|---|---:|---:|---:|---:|
| 3 years | 65.8% | 70.2% | 68.4% | 1.8 pp |
| 6 years | 78.0% | 79.9% | 79.4% | 0.5 pp |
| 9 years | 80.8% | 81.5% | 83.4% | 1.9 pp |

---

# Stage 2 — PCRA Risk-Tier Calibration

The risk-contrast parameter scales the fixed log-odds contrast associated with each PCRA tier:

`gamma × c_tier`

It does **not** exponentiate the individual normalized risk score. When `gamma = 0`, the tier contrasts are removed. Increasing `gamma` widens the separation among tier-specific rearrest hazards.

## Stage 2 Targets

| PCRA tier | 3-year target | 6-year target | 9-year target | Source |
|---|---:|---:|---:|---|
| Low | 46.2% | 61.4% | 67.6% | Johnson (2023), *Federal Probation* 87(2), Table 6 |
| Low-Moderate | 72.0% | 84.3% | 88.8% | Johnson (2023), *Federal Probation* 87(2), Table 6 |
| Moderate | 84.5% | 92.1% | 94.6% | Johnson (2023), *Federal Probation* 87(2), Table 6 |
| High | 91.0% | 95.0% | 95.0% | Johnson (2023), *Federal Probation* 87(2), Table 6 |

## Stage 2 Parameter and Sweep

| Symbol | Code parameter | Meaning | Baseline | Sweep specification | Selected value | Loss |
|---|---|---|---:|---|---:|---|
| γ | `Risk_Contrast_Strength` | Multiplier on fixed PCRA tier log-odds contrasts | 0.000 | 0.75–1.50; 16 values; step 0.05 | **1.000** | MAE across 12 tier × window cells |

## Exact Stage 2 Sweep Values

| Symbol | Candidate values |
|---|---|
| γ | `0.75, 0.80, 0.85, 0.90, 0.95, 1.00, 1.05, 1.10, 1.15, 1.20, 1.25, 1.30, 1.35, 1.40, 1.45, 1.50` |

## Stage 2 Three-Year Results

| PCRA tier | Uncalibrated | Calibrated | Target | Calibrated minus target |
|---|---:|---:|---:|---:|
| Low | 65.5% | 52.3% | 46.2% | +6.1 pp |
| Low-Moderate | 66.0% | 69.4% | 72.0% | −2.6 pp |
| Moderate | 65.9% | 82.6% | 84.5% | −1.9 pp |
| High | 65.5% | 92.0% | 91.0% | +1.0 pp |

| Follow-up window | Mean absolute tier deviation before calibration | Mean absolute tier deviation after calibration |
|---|---:|---:|
| 3 years | 17.34 pp | 2.90 pp |
| 6 years | 19.50 pp | 9.00 pp |
| 9 years | 20.75 pp | 12.40 pp |

---

# Stage 3 — Offense-Type Calibration

Four offense-specific log-odds shifts are applied through `delta_off` after the Stage 1 and Stage 2 parameters are locked.

## Stage 3 Targets

| Offense group | 3-year target | 6-year target | 9-year target | Source |
|---|---:|---:|---:|---|
| Violent | 62.2% | 74.2% | 78.7% | Alper et al. (2018), BJS NCJ 250975, Table 7 |
| Drug | 68.6% | 79.8% | 83.8% | Alper et al. (2018), BJS NCJ 250975, Table 7 |
| Property | 75.0% | 84.4% | 87.8% | Alper et al. (2018), BJS NCJ 250975, Table 7 |
| Other/Public Order | 65.0% | 76.9% | 81.9% | Alper et al. (2018), BJS NCJ 250975, Table 7 |

## Stage 3 Parameters and Sweeps

| Symbol | Code parameter | Baseline | Sweep specification | Selected value | Loss |
|---|---|---:|---|---:|---|
| o_v | `offense_hazard_shift.Violent` | 0.000 | −0.40 to +0.05; 10 values; step 0.05 | **−0.300** | Offense MAE across 12 cells |
| o_d | `offense_hazard_shift.Drug` | 0.000 | −0.15 to +0.20; 8 values; step 0.05 | **+0.050** | Offense MAE across 12 cells |
| o_p | `offense_hazard_shift.Property` | 0.000 | +0.20 to +0.80; 13 values; step 0.05 | **+0.600** | Offense MAE across 12 cells |
| o_o | `offense_hazard_shift.Other(PublicOrder)` | 0.000 | −0.40 to +0.05; 10 values; step 0.05 | **−0.400** | Offense MAE across 12 cells |

## Exact Stage 3 Sweep Values

| Symbol | Candidate values |
|---|---|
| o_v | `-0.40, -0.35, -0.30, -0.25, -0.20, -0.15, -0.10, -0.05, 0.00, 0.05` |
| o_d | `-0.15, -0.10, -0.05, 0.00, 0.05, 0.10, 0.15, 0.20` |
| o_p | `0.20, 0.25, 0.30, 0.35, 0.40, 0.45, 0.50, 0.55, 0.60, 0.65, 0.70, 0.75, 0.80` |
| o_o | `-0.40, -0.35, -0.30, -0.25, -0.20, -0.15, -0.10, -0.05, 0.00, 0.05` |

## Stage 3 Calibrated Differences

| Offense group | 3-year difference | 6-year difference | 9-year difference |
|---|---:|---:|---:|
| Violent | +4.0 pp | −0.3 pp | −3.4 pp |
| Drug | +0.4 pp | −1.2 pp | −3.7 pp |
| Property | +0.0 pp | +0.7 pp | −1.0 pp |
| Other/Public Order | +0.6 pp | −0.6 pp | −3.6 pp |

---

# Calibration Summary

| Stage | Symbol | Parameter | Status | Sweep range or fixed value | Number of candidates | Selected value | Calibration objective |
|---|:---:|---|---|---|---:|---:|---|
| 1 | α | Supervision monitoring intensity | Estimated | 1.00–1.20 | 11 | **1.120** | National MAE at 3, 6, and 9 years |
| 1 | δ_s3 | Supervision decay after year 3 | Estimated | 0.60–0.99 | 13 | **0.990** | National aggregate MAE; 6-year primary |
| 1 | δ_s6 | Supervision decay after year 6 | Estimated | 0.20–0.70 | 11 | **0.400** | National aggregate MAE; 9-year primary |
| 1 | dr1 | Desistance ratio after year 1 | BJS-fixed | 0.524 | 1 | **0.524** | BJS hazard decomposition |
| 1 | dr3 | Desistance ratio after year 3 | BJS-fixed | 0.500 | 1 | **0.500** | BJS hazard decomposition |
| 1 | dr6 | Desistance ratio after year 6 | BJS-fixed | 0.508 | 1 | **0.508** | BJS hazard decomposition |
| 2 | γ | PCRA tier risk contrast | Estimated | 0.75–1.50 | 16 | **1.000** | PCRA-tier MAE across 12 cells |
| 3 | o_v | Violent offense shift | Estimated | −0.40 to +0.05 | 10 | **−0.300** | Offense MAE across 12 cells |
| 3 | o_d | Drug offense shift | Estimated | −0.15 to +0.20 | 8 | **+0.050** | Offense MAE across 12 cells |
| 3 | o_p | Property offense shift | Estimated | +0.20 to +0.80 | 13 | **+0.600** | Offense MAE across 12 cells |
| 3 | o_o | Other/Public Order shift | Estimated | −0.40 to +0.05 | 10 | **−0.400** | Offense MAE across 12 cells |

---

## Sensitivity and Robustness Experiments

These experiments validate the calibrated SBP-BRiMS model but are not additional calibration stages.

| Experiment | Design | Simulation count | Purpose |
|---|---|---:|---|
| OAT sensitivity analysis | 13 parameters × 9 perturbation levels × 100 simulations | 11,700 | Quantify local sensitivity under ±10% to ±40% perturbations |
| Stress testing | 19 scenarios × 200 simulations | 3,800 | Evaluate single-parameter perturbations, baseline behavior, and two joint extremes |

---

## Output Files

| File | Stage or purpose |
|---|---|
| `baseline.json` | Uncalibrated baseline results |
| `recommended_params.json` | Final calibrated parameter values |
| `sweep_alpha.csv` | Stage 1 α sweep |
| `sweep_smi_decay3y.csv` | Stage 1 δ_s3 sweep |
| `sweep_smi_decay6y.csv` | Stage 1 δ_s6 sweep |
| `sweep_gamma.csv` | Stage 2 γ sweep |
| `sweep_oshift_violent.csv` | Stage 3 violent-offense sweep |
| `sweep_oshift_drug.csv` | Stage 3 drug-offense sweep |
| `sweep_oshift_property.csv` | Stage 3 property-offense sweep |
| `sweep_oshift_pubord.csv` | Stage 3 public-order sweep |
| `CalibrationSummary.png` | Aggregate calibration figure |
| `equifinality_3windows.png` | PCRA tier calibration figure |
| `chart3_cumulative_by_offense.png` | Offense-specific calibration figure |
| `oat_tornado_3yr.png` | OAT sensitivity figure |

---

## Running the Calibration

| Command | Purpose |
|---|---|
| `python OAT_Calibrate_BJS_PCRA.py` | Run the complete calibration |
| `python OAT_Calibrate_BJS_PCRA.py --replot` | Regenerate figures from saved results |
| `python OAT_Calibrate_BJS_PCRA.py --rerun` | Ignore checkpoints and rerun simulations |
| `python OAT_Calibrate_BJS_PCRA.py --cores 8` | Use eight parallel workers |

---

## Dependencies

| Dependency | Requirement |
|---|---|
| Python | 3.9 or later |
| Mesa | Required |
| NumPy | Required |
| pandas | Required |
| Matplotlib | Required |
| tqdm | Required |
| psutil | Optional; used for automatic CPU-core detection |

```bash
pip install mesa numpy pandas matplotlib tqdm psutil
```

---

## References

| Reference | Use in calibration |
|---|---|
| Alper, M., Durose, M. R., and Markman, J. (2018). *2018 Update on Prisoner Recidivism: A 9-Year Follow-up Period (2005–2014).* BJS NCJ 250975. | National and offense-specific rearrest targets |
| Johnson, J. L. (2023). “Federal Post-Conviction Supervision Outcomes.” *Federal Probation*, 87(2), 20–28. | PCRA risk-tier targets |
| Petersilia, J. (2003). *When Prisoners Come Home.* Oxford University Press. | Supervision-decay rationale |
| Windrum, P., Fagiolo, G., and Moneta, A. (2007). “Empirical Validation of Agent-Based Models.” *Journal of Artificial Societies and Social Simulation*, 10(2), 8. | Sequential calibration and equifinality rationale |

---

## Citation

```text
Rakkiappan, R., and Kavak, H. (2026). An Agent-Based Model of Recidivism.
In Proceedings of SBP-BRiMS 2026.
```
