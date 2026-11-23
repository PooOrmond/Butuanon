# Butuanon to English Training Results
## Five seed T4 experiment summary
Prepared 23 November 2026

## Executive summary

This report consolidates the completed runs for seeds 42, 123, 456, 789, and 1011. The main evaluation task is Butuanon to English. The full fine-tuned baseline achieved the highest mean test scores. Back-translation improved LoRA results modestly, but the combined model did not exceed the baseline. Histories, metrics, and checkpoints are present for all five seeds.

## Mean test scores across seeds

Scores are mean ± sample standard deviation over five independent training seeds. Each main-model score covers the same frozen 1,149-example test set.

| Configuration | BLEU | METEOR | BERTScore F1 | Runs |
|---|---:|---:|---:|---:|
| Baseline mBART | 70.024 ± 0.133 | 0.8478 ± 0.0030 | 0.9143 ± 0.0015 | 5 |
| mBART with LoRA | 63.702 ± 0.362 | 0.7955 ± 0.0018 | 0.8791 ± 0.0015 | 5 |
| LoRA with back-translation | 64.272 ± 0.343 | 0.8056 ± 0.0045 | 0.8880 ± 0.0030 | 5 |

Baseline leads all three mean metrics. Relative to LoRA alone, back-translation raises the mean by about 0.570 BLEU, 0.0101 METEOR, and 0.0089 BERTScore F1, while remaining below baseline.

## Data and frozen splits

- Original parallel rows: 12,156.
- Removed empty or missing pairs: 0.
- Removed normalized duplicate pairs: 4,495.
- Removed by mBART token-length rule: 0.
- Retained pairs: 7,661.
- Frozen split: 5,363 train / 1,149 validation / 1,149 test; split seed 42.
- The audit reports grouping rows that share a normalized English or Butuanon sentence, with no shared sentence across partitions.

## Training configuration

- Checkpoint: `facebook/mbart-large-50-many-to-many-mmt` with the custom `bto_XX` language token.
- Main translation direction: Butuanon → English. The English → Butuanon reverse model is an auxiliary back-translation model.
- Hardware: Tesla T4, 14.56 GiB reported capacity; FP16; AdamW; micro-batch 8 × gradient accumulation 4 = effective batch 32; activation checkpointing off.
- Maximum 50 epochs; early-stopping patience 5; source and target token limits 128; weight decay 0.01; label smoothing 0.1; dropout 0.3; gradient clipping 1.0.
- Baseline learning rate 3e-5. LoRA learning rate 1e-4; main LoRA rank 8 / alpha 16. Reverse LoRA rank 32 / alpha 64.
- Software: Python 3.13.16, PyTorch 2.14.1+cu130, Transformers 4.57.6, PEFT 0.21.2, SacreBLEU 2.6.0, SciPy 1.16.3.

## Per-seed main model results

Best validation loss is the lowest recorded validation loss; epoch counts show how long training ran. Test metrics are for Butuanon → English.

### Baseline mBART

| Seed | Epochs | Best validation loss (epoch) | BLEU | METEOR | BERTScore F1 | Train minutes | Peak GPU MiB |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 42 | 12 | 2.1114 (epoch 7) | 69.976 | 0.8499 | 0.9160 | 32.6 | 12294 |
| 123 | 11 | 2.1039 (epoch 6) | 70.067 | 0.8451 | 0.9129 | 30.0 | 12294 |
| 456 | 11 | 2.1027 (epoch 6) | 70.046 | 0.8445 | 0.9125 | 29.4 | 12294 |
| 789 | 12 | 2.1058 (epoch 7) | 70.197 | 0.8514 | 0.9154 | 31.9 | 12294 |
| 1011 | 11 | 2.1136 (epoch 6) | 69.834 | 0.8479 | 0.9148 | 29.5 | 12294 |

### mBART with LoRA

| Seed | Epochs | Best validation loss (epoch) | BLEU | METEOR | BERTScore F1 | Train minutes | Peak GPU MiB |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 42 | 50 | 2.2962 (epoch 50) | 64.049 | 0.7961 | 0.8805 | 123.7 | 5431 |
| 123 | 50 | 2.2910 (epoch 48) | 63.868 | 0.7960 | 0.8780 | 123.7 | 5431 |
| 456 | 50 | 2.2968 (epoch 49) | 63.782 | 0.7967 | 0.8808 | 119.4 | 5431 |
| 789 | 50 | 2.2976 (epoch 45) | 63.719 | 0.7924 | 0.8774 | 119.8 | 5431 |
| 1011 | 50 | 2.2982 (epoch 48) | 63.094 | 0.7962 | 0.8786 | 120.4 | 5431 |

### LoRA with back-translation

| Seed | Epochs | Best validation loss (epoch) | BLEU | METEOR | BERTScore F1 | Train minutes | Peak GPU MiB |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 42 | 50 | 2.2788 (epoch 50) | 63.690 | 0.8108 | 0.8909 | 287.6 | 5431 |
| 123 | 50 | 2.2799 (epoch 46) | 64.535 | 0.8068 | 0.8904 | 273.0 | 5431 |
| 456 | 50 | 2.2844 (epoch 49) | 64.351 | 0.8081 | 0.8887 | 278.8 | 5431 |
| 789 | 50 | 2.2882 (epoch 50) | 64.274 | 0.7993 | 0.8843 | 279.0 | 5431 |
| 1011 | 50 | 2.2801 (epoch 50) | 64.511 | 0.8029 | 0.8855 | 284.6 | 5431 |

Baseline stopped after 11–12 epochs because validation loss stopped improving; its best validation loss was reached at epoch 6 or 7. LoRA and combined runs reached the 50-epoch cap.

## Reverse auxiliary model

| Seed | Epochs | Best validation loss (epoch) | Train minutes | Peak GPU MiB |
|---:|---:|---:|---:|---:|
| 42 | 50 | 2.5926 (epoch 48) | 129.2 | 6162 |
| 123 | 49 | 2.6062 (epoch 44) | 122.4 | 6162 |
| 456 | 50 | 2.5920 (epoch 48) | 122.4 | 6162 |
| 789 | 50 | 2.5921 (epoch 45) | 123.1 | 6162 |
| 1011 | 42 | 2.6045 (epoch 37) | 105.0 | 6162 |

The reverse model has no BLEU, METEOR, BERTScore, or test predictions in this experiment because it was used to generate synthetic training data, not evaluated as a main translation system.

## Back-translation details

Each seed generated candidates for 10,000 monolingual English inputs using temperature 0.8 and five candidates per input. Filtering used confidence ≥0.6, length ratio 0.5–2.0, and near-duplicate threshold 0.9. Validation and test sets remained unchanged.

| Seed | Accepted synthetic pairs | Reverse training min | Generation min | Combined training min | Full BT pipeline min |
|---:|---:|---:|---:|---:|---:|
| 42 | 6,935 | 129.2 | 168.3 | 287.6 | 585.1 |
| 123 | 6,948 | 122.4 | 161.0 | 273.0 | 556.4 |
| 456 | 7,075 | 122.4 | 160.2 | 278.8 | 561.5 |
| 789 | 6,996 | 123.1 | 160.5 | 279.0 | 562.7 |
| 1011 | 7,030 | 105.0 | 165.9 | 284.6 | 555.6 |

Mean accepted synthetic pairs: 6,996.8. Mean full back-translation pipeline time: 564.3 minutes per seed, combining reverse training, generation, and combined-model training.

## Mean compute use

| Configuration | Mean minutes per epoch | Mean training minutes | Mean peak GPU MiB | Trainable parameters |
|---|---:|---:|---:|---:|
| Baseline mBART | 2.69 | 30.7 | 12,294 | 610,880,512 |
| mBART with LoRA | 2.43 | 121.4 | 5,431 | 2,360,320 |
| LoRA with back-translation | 5.61 | 280.6 | 5,431 | 2,360,320 |

Training-only time sums completed training epochs and excludes validation, checkpoint writes, and abandoned partial epochs. Full pipeline time for back-translation is reported separately.

## Paired bootstrap results for translation

Step 9 used 1,000 paired test-sentence bootstrap resamples per seed and Holm correction within each metric and seed. There are 15 rows per metric: three model pairs × five seeds. The table shows mean per-seed A−B metric differences, how often A scored higher, and the number of seeds with corrected p < .05.

| Metric | Model A | Model B | Mean A−B | A higher | Significant seeds |
|---|---|---|---:|---:|---:|
| BLEU | Baseline mBART | mBART with LoRA | +6.3218 | 5/5 | 5/5 |
| BLEU | Baseline mBART | LoRA with back-translation | +5.7519 | 5/5 | 5/5 |
| BLEU | mBART with LoRA | LoRA with back-translation | -0.5699 | 1/5 | 1/5 |
| METEOR | Baseline mBART | mBART with LoRA | +0.0523 | 5/5 | 5/5 |
| METEOR | Baseline mBART | LoRA with back-translation | +0.0422 | 5/5 | 5/5 |
| METEOR | mBART with LoRA | LoRA with back-translation | -0.0101 | 0/5 | 3/5 |
| BERTScore F1 | Baseline mBART | mBART with LoRA | +0.0353 | 5/5 | 5/5 |
| BERTScore F1 | Baseline mBART | LoRA with back-translation | +0.0263 | 5/5 | 5/5 |
| BERTScore F1 | mBART with LoRA | LoRA with back-translation | -0.0089 | 0/5 | 5/5 |

These bootstrap tests estimate uncertainty over sampled test sentences for each trained seed. They are not a substitute for an across-training-seed statistical test and should not be described as 15 independent training runs.

## Matched-seed resource comparisons

Step 10 used paired tests over five matched seeds with Holm adjustment within each resource metric. Results are exploratory with n=5.

| Measure | Model A | Model B | Mean A−B | Holm-adjusted p | Significant < .05 |
|---|---|---|---:|---:|:---:|
| Minutes per epoch | Baseline mBART | mBART with LoRA | +0.265 | 4.85e-06 | Yes |
| Minutes per epoch | Baseline mBART | LoRA with back-translation | -2.920 | 1.46e-06 | Yes |
| Minutes per epoch | mBART with LoRA | LoRA with back-translation | -3.185 | 1.46e-06 | Yes |
| Peak GPU memory (MiB) | Baseline mBART | mBART with LoRA | +6863.754 | 1.68e-25 | Yes |
| Peak GPU memory (MiB) | Baseline mBART | LoRA with back-translation | +6863.752 | 9.56e-28 | Yes |
| Peak GPU memory (MiB) | mBART with LoRA | LoRA with back-translation | -0.003 | 0.406 | No |

Baseline peak allocation was about 6,864 MiB higher than LoRA or combined. LoRA and combined peak memory were effectively the same; that pair was not significant (Holm p=0.406). The extremely stable memory figures are measured allocator peaks and should be interpreted as descriptive hardware measurements, not broad claims about memory variability.

## Thesis reporting notes and file map

- Report the observed score ranking accurately: baseline highest; back-translation modestly improves LoRA but remains below baseline.
- Describe the reverse model as an English-to-Butuanon auxiliary generator. Its blank test metrics were not generated and are not missing Drive data.
- Keep the method description aligned with the run: the notebook used the pretrained mBART tokenizer plus `bto_XX` and fixed temperature 0.8; it did not train a new 32k BPE tokenizer or perform a temperature sweep.
- `runs/<model>/seed_<seed>/trainer_history.json`: epoch-level training and validation loss.
- `runs/<model>/seed_<seed>/metrics.json`: per-run settings, best validation loss, test metrics, and timing.
- `runs/<model>/seed_<seed>/epoch_timing.json`: per-epoch elapsed time and GPU allocation.
- `runs/baseline`, `runs/lora`, and `runs/combined`: test predictions for the 1,149 held-out examples.
- `runs/synthetic/seed_<seed>/`: accepted synthetic pairs and generation audit.
- `reports/all_runs.csv`, `summary_by_model.csv`, `translation_bootstrap_by_seed.csv`, `resource_significance.csv`, and `research_question_coverage.json`: the source reports consolidated here.

Evaluation used SacreBLEU with 13a tokenization and exponential smoothing, METEOR, and BERTScore F1 with `bert-base-uncased`. The test set was held out from training and validation-based early stopping.
