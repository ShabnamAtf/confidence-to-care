From Confidence to Care — Code & Artifact

Confidence-to-Care (C2C) is a logic-grounded framework that converts confidence signals from medical LLMs into auditable Care-0…Care-3 actions. It fuses token-level probabilities with robustness cues (semantic entropy, self-consistency, paraphrase agreement) and uses an executable Prolog policy to trigger escalation with a human-readable trace aligned to ESI/CTAS/NEWS2.

1) What this repository contains

Signals/metrics for uncertainty and robustness

sequence log-probability, arithmetic/geometric means, minimum-token probability

semantic entropy over paraphrases; variance from self-consistency sampling

inter-prompt agreement via Cohen’s κ

Local explanations (SHAP) for risk-evidence spans

Executable policy (Prolog): thresholds + duty-to-warn predicates → Care-0/1/2/3

Triage alignment: mapping to ESI, CTAS, NEWS2 bands

Reproducibility scripts: batch evaluation, calibration curves, coverage–risk, figures

Audit logs (JSONL/CSV): rule firings, cited spans, final care decision

Data: a small, synthetic demo bank for end-to-end runs is included. No PHI/PII.

2) Directory layout (proposed)

<details>
<summary><b>Repository structure</b></summary>

<div dir="ltr">
```text
.
├─ configs/
│   ├─ experiment.yml                # seeds, sampling K, paths, figure toggles
│   ├─ policy.yml                    # τ0, τ1, σ0, κmin, duty-to-warn toggles
│   ├─ triage_mapping.yml            # Care-0..3 → ESI/CTAS/NEWS2 bands
├─ data/
│   ├─ demo_bank.jsonl               # synthetic QA/triage items (+ paraphrases)
│   └─ red_flags.jsonl               # synthetic duty-to-warn cases
├─ signals/
│   ├─ extract_signals.py            # aggregates token-level probs → metrics
│   └─ explain_spans.py              # SHAP span extraction
├─ policy/
│   ├─ rules.pl                      # Prolog policy (thresholds, duty-to-warn)
│   └─ loader.pl                     # JSONL → Prolog facts
├─ scripts/
│   ├─ run_demo.py                   # one-click end-to-end demo
│   ├─ batch_eval.py                 # four care levels; writes metrics/plots
│   ├─ score_calibration.py          # ECE, Brier, NLL; reliability diagrams
│   ├─ make_figures.py               # calibration, radar, tables
│   └─ export_audit.py               # JSONL traces → readable CSV
├─ outputs/                          # created on first run
│   ├─ audit_traces.jsonl
│   ├─ condition_summary.csv
│   ├─ calib_curve.png
│   ├─ radar_perf.png
│   └─ coverage_risk.png
├─ requirements.txt
└─ README.md
```

</div>
</details>
```
3) Installation

Python ≥ 3.10 and SWI-Prolog (for the rules engine).

python -m venv .venv
# Linux/macOS
source .venv/bin/activate
# Windows PowerShell
# .\.venv\Scripts\Activate.ps1

pip install -r requirements.txt
# Install SWI-Prolog from https://www.swi-prolog.org/ (ensure `swipl` is on PATH)


The pipeline is model-agnostic: it accepts pre-computed token log-probabilities. Optional helpers for HuggingFace models can be added if you want to compute them locally.

4) Quick start (demo, 2–3 minutes)

Run the synthetic end-to-end demo (signals → explanations → policy → audit):

python scripts/run_demo.py \
  --config configs/experiment.yml \
  --policy configs/policy.yml \
  --triage configs/triage_mapping.yml \
  --data data/demo_bank.jsonl


Outputs

outputs/audit_traces.jsonl — per-case rules fired, spans cited, Care-0…3

outputs/condition_summary.csv — Acc/Prec/Rec/F1/AUROC + calibration

outputs/*.png — calibration curve, radar plot, coverage–risk

5) Reproduce paper figures & tables

Deterministic batch with fixed seeds:

# 1) Evaluate (includes semantic entropy, self-consistency, paraphrase κ)
python scripts/batch_eval.py --config configs/experiment.yml

# 2) Calibration & reliability diagrams (ECE, Brier, NLL)
python scripts/score_calibration.py --inputs outputs/audit_traces.jsonl

# 3) Figures (Calibration, Radar, Coverage–Risk) + Table exports
python scripts/make_figures.py --inputs outputs/audit_traces.jsonl


Expected (illustrative numbers from the manuscript demo):

Geometric-mean confidence best calibrated (ECE ≈ 0.21)

Overall performance: Acc 74.1%, Prec 76.4%, Rec 74.1%, F1 74.3%, AUROC 64.6%

Selective prediction at ~80% coverage → ~35% error reduction

Paraphrase agreement κ ≈ 0.62; duty-to-warn escalates ~7% to Care-3

6) Policy & thresholds (edit in configs/policy.yml)
# Confidence thresholds
tau0: 0.35         # low-confidence abstain boundary
tau1: 0.65         # confident acceptance boundary
sigma0: 0.45       # semantic entropy threshold
kappa_min: 0.55    # paraphrase agreement floor

# Duty-to-warn toggles (examples)
duty_to_warn:
  chest_pain_unstable: true
  neuro_deficit_acute: true
  resp_distress_spo2_low: true
  hypotension_shock: true


Care-0: mean confidence < τ0 OR semantic entropy > σ0

Care-1: τ0 ≤ conf < τ1 with no duty-to-warn

Care-2: high-risk spans (SHAP) OR κ < κmin

Care-3: any duty-to-warn predicate fired

Mapping to triage frameworks in configs/triage_mapping.yml mirrors Table 1 (ESI/CTAS/NEWS2).

7) Notes on signals & explanations

Confidence: sequence probability; arithmetic/geometric means; min-token probability

Robustness: semantic entropy across paraphrases; self-consistency variance across K samples

Agreement: Cohen’s κ across paraphrased prompts

Explanations: SHAP spans over risk lexicon; spans are cited in the policy trace

All signals are serialized to JSONL and loaded by policy/loader.pl for Prolog execution.

8) Reproducibility, ethics, and scope

Seeds fixed in configs/experiment.yml; results are deterministic on Python 3.10/3.11

Demo data are synthetic; no PHI/PII

This artifact is a research prototype and not a medical device. It is not intended for clinical use without regulatory clearance and institutional validation.

9) Citing

If you use this software, please cite:

Z. Atf, A. Mahjoubfar, P. R. Lewis. From Confidence to Care: Rule-Based Escalation for Trustworthy Clinical AI. (preprint/manuscript).
