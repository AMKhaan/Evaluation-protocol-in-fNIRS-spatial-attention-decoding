# Evaluation protocol in fNIRS spatial-attention decoding (ds004830)

Code and results for a paper about how the choice of train/test split, not the model, the
features or the data, drives the accuracy an fNIRS decoding study reports.

Holding the data, features and classifier fixed and changing only how samples are split into
training and test sets moves three-class accuracy on this dataset by **46.6 percentage
points**:

| protocol | what is split | accuracy | vs. protocol A |
|---|---|---|---|
| A | leave-one-subject-out | 32.72% | baseline |
| B | random split over trials, subjects pooled | 37.87% | +5.2 pp |
| C | random split over sliding windows | 43.25% | +10.5 pp |
| D | channel-as-sample, labels indexed by channel | **79.28%** | **+46.6 pp** |

Chance is 33.3%. Protocol D's 79% is not a measurement of attention: an oracle trained to
predict *which optode a window came from* scores 100% on the same samples. Protocols B–D are
instances of the leakage taxonomy in Kapoor & Narayanan (2023): illegitimate features [L2]
and train/test non-independence [L3.2].

## What the honest protocols give

Under protocol A, on the 907 behaviourally correct trials from 12 subjects:

| | within-subject 5-fold | leave-one-subject-out |
|---|---|---|
| best model | Logistic (L2) **39.91%** | Random Forest **35.94%** |
| label permutation (300) | p = 0.0000 | p = 0.0833 |
| subjects above chance | 10/12 | 8/12 |

Within-subject decoding is real but modest. Cross-subject transfer is not established: the
permutation test that respects the design gives p = 0.083, and a one-sample t-test across
subject means gives p = 0.028 on the *same numbers*. Which test you choose decides the
answer, which is the paper's point, turned on our own results.

Seven deep architectures (HemoNet, CNN+BiLSTM, Transformer, CNN+TemporalAttn, TCN,
InceptionTime, GRU+Attention) all land between 32.4% and 33.8% under leave-one-subject-out.
None beats the classical baseline.

## The trial-onset reconstruction

Worth stating separately, because it is the part most likely to be useful to someone who does
not care about our results: **ds004830 as distributed carries no usable trial timing.** The
BIDS `events.tsv` files are empty and the Homer stimulus matrix `s` is identically zero.
Onsets are reconstructed from the PsychToolbox logs as `t_onset(i) = startT + Trigger3(i)`.

The reconstruction is validated three ways: against a shuffled-onset control, by a global lag
sweep (the evoked response is sharpest at lag 0), and by reproducing Ning et al.'s published
per-subject behavioural accuracies; all twelve match exactly.

## Layout

```
Code/analysis/       analysis pipeline
Code/analysis/logs/  console output from the runs behind the reported numbers
manuscript/          figures
```

The dataset is **not** redistributed here. Download it from
[OpenNeuro ds004830](https://openneuro.org/datasets/ds004830) and point `DS004830_ROOT` at its
derivatives tree.

```bash
cd Code/analysis
python run_all.py --quick --root /path/to/ds004830/derivatives   # smoke test
python run_all.py         --root /path/to/ds004830/derivatives   # ~9 h on 8 cores
```

Requirements, per-stage runtimes, the flags, and the table mapping each paper element to the
script that produced it are in [`Code/analysis/README.md`](Code/analysis/README.md), including
a caution about `leakage_demo.py`, which deliberately reproduces the leaky protocol D and is
there to be measured, not reused.

## Licence

Released under the **MIT Licence** (see [`LICENSE`](LICENSE)). You may reuse, modify and
redistribute this code, including commercially, provided the copyright notice is retained. The
dataset itself is not covered by this licence; ds004830 carries its own terms on OpenNeuro.
