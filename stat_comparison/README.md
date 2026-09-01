# Statistical comparison

`M3_yearly.ipynb` uses [statsforecast](https://github.com/Nixtla/statsforecast)
to run statistical baselines on the M3 yearly series in `../m3_yearly_validation`:

- AutoTheta
- AutoETS
- AutoARIMA
- Seasonal Naive
- Naive

It reports naive MASE, seasonal MASE, and SMAPE, using the same metric code
(`eval_metrics.py`) and timing method as `train_and_inference/inference.py`, so
the results can be compared with GRAIN directly.

## Setup

```bash
pip install statsforecast
```

The notebook imports `eval_metrics` from `train_and_inference/`. Before running
it, either copy `train_and_inference/eval_metrics.py` into this folder or add
`../train_and_inference` to `sys.path`.

Each model writes its results to its own folder, for example
`./autoETS/m3_yearly`.
