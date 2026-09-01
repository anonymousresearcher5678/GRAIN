# GRAIN

## Repository layout

| Folder | Contents |
| --- | --- |
| `synthetic_generation/` | R code that generates the synthetic prior with a Bayesian Local Global Trend (BLGT) model. |
| `convert_prior/` | Converts the generated CSV into the `.npy` / `.npz` files used for training. |
| `train_and_inference/` | Model, training, inference, and the GIFT-Eval wrapper. Includes the trained weights (`model.ckpt`). |
| `m3_yearly_validation/` | M3 yearly series, one `T{n}.csv` file per series. |
| `stat_comparison/` | Statistical baselines (ETS, ARIMA, Theta, Naive) on M3 yearly. |
| `gift-eval/` | GIFT-Eval notebook, GRAIN results, and comparison tables and plots. |

## Requirements

Python **3.10** (tested with 3.10.20).

```bash
conda create -n py310 python=3.10
conda activate py310
pip install -r environments.txt
```

## The prior

Get the prior in one of two ways:

- **Download it** from this [anonymous Google Drive folder](https://drive.google.com/drive/folders/1bJGhHVUo_fgY_Bj21H0HVkJNtCewk3GW?usp=sharing).
- **Generate it** with the pipeline in `synthetic_generation/`, then convert it with
  `convert_prior/convert_prior.py`:

  ```bash
  python convert_prior.py --csv "<path>/m3_yearly_200.csv" --out prior/
  ```

  You only need to regenerate it if the generator CSV changes.

Either way, place the files under `train_and_inference/prior/`:

```
train_and_inference/prior/m3_yearly_200.y.npy
train_and_inference/prior/m3_yearly_200.meta.npz
```

## Training

From `train_and_inference/`:

```bash
python train.py -c config.example.yaml
```

Each run writes to `runs/<model_save_name>.<timestamp>/`. The checkpoint at
`checkpoints/model.ckpt` in that folder is overwritten after every epoch. All
settings (data paths, horizon, optimisation, devices) are in
`config.example.yaml`.

## Inference

Trained weights are included at `train_and_inference/model.ckpt`. The M3 yearly
validation series are in `m3_yearly_validation/`.

`inference.py` scores a folder of `T{n}.csv` files (columns `date,OT`). Run from
`train_and_inference/`:

```bash
python inference.py \
    -w model.ckpt \
    -d ../m3_yearly_validation \
    -p 6 -n 1 \
    -o ./grain/m3_yearly
```

These are also the defaults, so `python inference.py` on its own does the same.
To score your own run, pass `-w runs/<run>/checkpoints/model.ckpt`.

| Flag | Meaning |
| --- | --- |
| `-w` | Checkpoint to load. |
| `-d` | Folder of `T{n}.csv` files. |
| `-p` | Horizon, 1–6. The head is 6 steps wide, so one checkpoint serves every horizon. |
| `-n` | Number of rolling origins. `1` forecasts only the final `-p` points. |
| `-o` | Output folder. |
| `--save-txt` | Write per-series `.txt` reports and `average_metrics.txt`. |
| `--save-plots` | Write per-series `.png` plots (median and 10–90% band). |
| `--limit N` | Score only the first N series (smoke test). |
| `--device` | Defaults to `cuda` when available, otherwise `cpu`. |

Run `python inference.py --help` for the full list.

## Statistical comparison

`stat_comparison/M3_yearly.ipynb` runs the statistical baselines on the same M3
yearly series. See [`stat_comparison/README.md`](stat_comparison/README.md).

## GIFT-Eval

1. Clone the [GIFT-Eval repository](https://github.com/SalesforceAIResearch/gift-eval)
   and follow its setup, including the `GIFT_EVAL` entry in `.env`.
2. Copy `gift-eval/GRAIN.ipynb` into GIFT-Eval's `notebooks/` folder.
3. In the notebook's configuration cell, point `CODE_DIR` and `WEIGHTS` at this
   repository's `train_and_inference/` folder and `train_and_inference/model.ckpt`.
4. Run the notebook. Results are written to GIFT-Eval's `results/GRAIN/`.
5. Run `gift-eval/results.ipynb`, then `gift-eval/plots.ipynb`, to compare GRAIN
   with the other GIFT-Eval baselines. Set `RESULTS_DIR` in `results.ipynb` (or
   the `GIFT_EVAL_RESULTS` environment variable) to GIFT-Eval's `results/`
   folder.

Only `m4_yearly` with `term="short"` is supported: the model forecasts at most 6
steps ahead.

## License

Apache 2.0. See [`LICENSE`](LICENSE).
