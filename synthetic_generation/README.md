# Synthetic time series generation

Generates synthetic time series with a Bayesian Local Global Trend (BLGT) model
fitted to benchmark forecasting datasets (M3) from the
[Monash Time Series Forecasting Archive](https://forecastingdata.com/).

## Requirements

R with the following packages:

```r
install.packages(c("Rlgt", "forecast", "tidyverse", "moments", "ggplot2"))
```

## Files

| File | Purpose |
| --- | --- |
| `load_cif_tsf.R` | Reads `.tsf` files, rebuilds timestamps, and splits series into train and test sets. |
| `blgt_2.Rmd` | Fits BLGT models and generates the synthetic series. |
| `m3_yearly_dataset.tsf` | M3 yearly data from the Monash archive. |

## Usage

1. **Get the data.** The M3 yearly `.tsf` file is already included. It comes from
   [Zenodo record 4656222](https://zenodo.org/records/4656222). Other frequencies
   (monthly, quarterly, hourly, daily, weekly) have their own Zenodo records in
   the same archive.

2. **Load the data.** In R, from this folder:

   ```r
   source("load_cif_tsf.R")
   ```

   This reads `m3_yearly_dataset.tsf` into `m3_yearly_df` and also writes it to
   `m3_yearly.csv`.

3. **Generate the synthetic series.** In the same R session, so that
   `m3_yearly_df` is still loaded, run the chunks of `blgt_2.Rmd` in RStudio.

## Output

A csv for us to convert to prior. It contains 250 synthetic series for each generated series, they are all in length of 200. Resulting in 645*250 = 161,250 series.

```
m3_yearly_200.csv
```

To train on it, move the CSV into `convert_prior/` and follow
[`convert_prior/README.md`](../convert_prior/README.md).

If you don't want to run the generator, you can download the converted prior
instead. The link is in the [root README](../README.md#the-prior).
