# TRENCH-IDS

**Transferable representation learning for continual heterogeneous graph-based intrusion detection.**

TRENCH-IDS is a reproducible continual-learning benchmark for network intrusion detection using heterogeneous graphs. The benchmark evaluates catastrophic forgetting, replay, parameter regularization, distillation, architectural depth, and cross-dataset generalization across six sequential tasks and ten attack classes.

## Research snapshot

The benchmark was frozen on 2026-09-07. Its canonical result is the three-seed mean for a three-layer residual heterogeneous GNN with experience replay:

| Metric | Result |
|---|---:|
| Final accuracy | **0.902 Â± 0.005** |
| Catastrophic forgetting | **0.081 Â± 0.008** |
| Seeds | 42, 1, 2 |

The same evaluation reported a Flat+Host replay baseline of **0.872 Â± 0.016** final accuracy and **0.105 Â± 0.017** forgetting. Results, ablations, and caveats are documented in [project-metrics.md](project-metrics.md) and [docs/results_final.md](docs/results_final.md).

These numbers should be read with the documented limitations: cross-dataset generalization remained poor, and several forward-compatibility mechanisms did not improve on replay.

## What the benchmark found

- A one-layer GNN left 40.9% of model parameters unreachable from the classification loss.
- Residual connections enabled deeper relation-specific message passing.
- Replay outperformed the evaluated EWC and distillation configurations.
- The corrected three-layer graph model beat the non-message-passing Flat+Host baseline across all three reported seeds.
- Cross-dataset evaluation exposed a severe generalization failure, documented rather than hidden.

## Benchmark design

Five NetFlow v2 datasets were evaluated; three contribute to the final task sequence:

| Dataset | Use |
|---|---|
| NF-ToN-IoT-v2 | Included |
| NF-CSE-CIC-IDS2018-v2 | Included |
| NF-BoT-IoT-v2 | Included for reconnaissance |
| NF-UNSW-NB15-v2 | Excluded after task-design revision |
| NF-UQ-NIDS-v2 | Excluded to avoid double counting the merged union |

The final benchmark contains six continual-learning tasks. Benign traffic is sampled separately for each task, and mini-graphs are constructed in capture order with relation-specific forward and reverse edges. See [datasets.md](datasets.md), [attack-class-counts.md](attack-class-counts.md), and [attack-similarity-matrix.md](attack-similarity-matrix.md).

## Reproducibility map

| Evidence | Location |
|---|---|
| Frozen metrics and ablations | [project-metrics.md](project-metrics.md) |
| Generated results report | [docs/results_final.md](docs/results_final.md) |
| Dataset and task design | [datasets.md](datasets.md) |
| Experiment configuration | [configs/](configs/) |
| Training and analysis code | [src/trench_ids/](src/trench_ids/) |
| Experiment runners | [scripts/](scripts/) and [run_training.py](run_training.py) |
| CPU test suite | [tests/](tests/) |
| Technical supplement | [TRENCH-IDS_Technical_Supplement_FINAL.tex](TRENCH-IDS_Technical_Supplement_FINAL.tex) |

## Installation

Python 3.10 is the documented environment. The test suite can run on CPU; reproducing the headline training runs is intended for a CUDA-capable GPU.

```bash
uv venv --python 3.10
uv pip install -e ".[dev]"
.venv/Scripts/python.exe -m pytest -q
```

On Linux or macOS, use `.venv/bin/python` instead.

## Reproduce the canonical baseline

The full reported baseline consists of three runs with seeds 42, 1, and 2:

```bash
trench-train \
  model.num_layers=3 \
  model.use_residual=true \
  replay.enabled=true \
  ewc.lambda_r=0 ewc.lambda_s=0 ewc.lambda_u=0 \
  train.warmup_epochs=1 \
  train.seed=42 \
  paths.out_dir=runs/gnn_3layer_residual_s42
```

Repeat with `train.seed=1` and `train.seed=2`, changing `paths.out_dir` accordingly. Generate the consolidated report with:

```bash
python -m trench_ids.cl.report --out docs/results_final.md
```

The published environment used a CUDA-enabled PyTorch build on an NVIDIA RTX 3050 Laptop GPU. CPU execution is supported but substantially slower.

## No-cap robustness study

The `configs/no-cap/` and `scripts/run_no_cap.py` paths define a separate robustness study that removes the main benchmarkâ€™s per-task attack-volume cap. It does **not** replace the frozen canonical baseline above.

```bash
conda run -n base python -m trench_ids.preprocess_no_cap \
  --config configs/no-cap/preprocess.yaml
conda run -n base python -m trench_ids.graphs \
  --config configs/no-cap/graph.yaml
conda run -n base python scripts/run_no_cap.py --phase train
```

The no-cap study has materially larger task sizes and different replay proportions. Compare it only within the assumptions recorded in its configuration and validation artifacts.

## Tests

```bash
pytest -q
```

The repositoryâ€™s recorded frozen state reports 373 CPU tests. Treat that as a project artifact rather than a guarantee about an unverified environment; run the suite locally before extending the benchmark.

## Status

The main benchmark is frozen at the existing `v1.0.0-experiments` tag. Further changes should preserve the recorded experimental evidence and clearly distinguish new studies from the canonical result.

