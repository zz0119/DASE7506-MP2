# DASE7506 MP2 Selecting Reliable Mathematical Answers

Official complete-test selection accuracy: **83.5118% (390 / 467)**.
The exact official `accuracy` value is **83.5117773019272**.

- [Eight-page report PDF](MP2_Report.pdf)
- [Checkpoint bundle ZIP](model_bundle.zip)
- [Official test score JSON](test-score.json)
- [AI assistance and personal contribution](AI_DISCLOSURE.md)

## Install and restore the supplied data

The complete source, selected checkpoint, report and experiment evidence are
in `source_bundle.zip`. The separate `model_bundle.zip` contains only the
inference assets. Use Python 3.12. Clone this repository, enter it, and run:

```bash
unzip -oq source_bundle.zip
python3.12 -m venv code/.venv
code/.venv/bin/python -m pip install -r code/requirements-inference.txt
code/.venv/bin/python prepare_data.py
cd code
bash run.sh check_data.py
```

`prepare_data.py` downloads only the official supplied course data from pinned
upstream commit `bb801f4f2e7629d270df765ab12c333b2dc616bb` and checks every file
against the unchanged official package manifest. It downloads no model weights.
For offline setup, use `prepare_data.py --source-data /path/to/official/code/data`.
This setup transfer happens before evaluation; inference and scoring run offline.

## Evaluate the frozen checkpoint without training

From `code/`:

```bash
bash run.sh verify.py resources --checkpoint runs/final
bash run.sh verify.py measure --checkpoint runs/final --freeze runs/final-freeze.json --split test --output runs/reproduction-test
```

The wrapper runs the unchanged official predictor and independent scorer in
fresh processes with four evaluation threads. Alternatively:

```bash
bash run.sh predict.py --method submission.py --checkpoint runs/final --split test --output runs/reproduction-test/test-predictions.json
bash run.sh score_predictions.py --split test --predictions runs/reproduction-test/test-predictions.json --output runs/reproduction-test/test-score.json
```

The selected bundle totals **2,565,097 bytes (2.4463 MiB)**. Its reported full-test
prediction plus independent scoring time is **0.6941 seconds** locally on an
Apple M4 with 16 GB RAM. Local timing does not establish reference Xeon eligibility.
The resource limit is 16 MiB and 30 seconds on the four-thread reference CPU.

## Reproduce the selected training process

From `code/`:

```bash
.venv/bin/python -m pip install -r requirements-train.txt
MP2_TRAIN_THREADS=2 bash run.sh retrain_selected.py --manifest runs/selected-training.json --output runs/retrained
MP2_CHECKPOINT=runs/final bash run.sh -m unittest discover -s tests -v
```

The final recipe uses train-only normalization and labels, a 2228-to-256 shared
candidate encoder, question mean/max and same-answer context, and a 64-unit head.
It combines BCE with pairwise ranking (coefficient 0.5), easy-question weight 0.25,
dropout 0.25 and AdamW (learning rate 0.0007, decay 0.03). Seed 17 and epoch 10 are fixed.
Local retraining matched every saved array and the checkpoint SHA-256 exactly.
After source extraction, code and asset identities are recorded in `code/runs/final-freeze.json`.

## Experiment records

The report and `experiments/` describe experiments executed in this development
cycle. `code/runs/neural-experiments.csv` summarizes 148 neural training runs;
text/tree/PCA and random-feature searches are included separately. These records
are evidence, not additional required inference assets. Unselected weights are
not required to reproduce the frozen result. To rerun a search from code/, use a new output directory:

```bash
MP2_TRAIN_THREADS=2 bash run.sh train.py --output runs/reproduce-search --seeds 17 --epochs 70
```

After source extraction, recipe lists are in `experiments/*-recipes.json`. Full validation is 754/798
(94.4862%); difficult validation is 311/355 (87.6056%). The complete validation
bank has 443 questions with only one valid answer group, whereas test questions
have competing valid answers. Do not present validation accuracy as test accuracy.
Test scoring occurred only after model selection and source/asset freeze; the
result was not used for subsequent tuning. The final implementation passed 36 tests.

## Attribution

Data, the numeric parser and official evaluation tools come from
[xudongwu-0/DASE7506-MP2](https://github.com/xudongwu-0/DASE7506-MP2).
Retain `code/data/LICENSE.txt` and `code/data/NOTICE.txt`. The report and
`AI_DISCLOSURE.md` state the student contribution and substantive Codex assistance.
