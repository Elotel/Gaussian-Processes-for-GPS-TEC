# Driver-Informed Gaussian Processes for GPS-TEC: ADIS vs BJCO, 23 June 2015 Storm

Probabilistic modelling of storm-time vertical total electron content (VTEC) at two African GNSS stations, **ADIS** (Addis Ababa, Ethiopia) and **BJCO** (Cotonou, Benin), using Gaussian processes (GPs) and solar-wind drivers from NASA OMNI (SYM-H, IMF Bz, E<sub>y</sub>).

This repository extends my undergraduate dissertation, *Comparing the Ionospheric Effects of the 23rd June 2015 Geomagnetic Storm Event on Addis Ababa and Benin Ionospheric Stations* (E. A. Ezekwueme and D. C. Obiegbuna, University of Nigeria, Nsukka). The dissertation compared quiet and disturbed TEC and described the storm drivers. The notebook adds **uncertainty quantification** and a **driver-informed prediction**, evaluated on held-out data.

> **Status: method demonstration.** The notebook runs on **synthetic data** with arbitrary planted responses so that it is reproducible and testable against a known truth. Its outputs are **not scientific results**. Real-data analysis requires the input files described below.

## Background

The dissertation processed dual-frequency GNSS (RINEX) data with the GOPI GPS-TEC software (Seemala and Valladares, 2011) to obtain VTEC at ADIS and BJCO, and defined the storm-time change as ΔTEC = DTEC − QTEC (disturbed minus quiet-day TEC). It used the Kyoto Dst index and OMNI solar-wind data (IMF Bz, solar-wind speed, proton density, and the interplanetary electric field E<sub>y</sub> = −B<sub>z</sub>·V<sub>sw</sub>·10⁻³). It reported a Dst minimum of about −204 nT near 04 UT on 23 June 2015, with IMF Bz turning strongly southward.

## What the notebook does

1. **Quiet-time baseline.** A GP with a ~24 h quasi-periodic kernel is trained on quiet-time VTEC only, giving a baseline with uncertainty.
2. **Storm-time ΔTEC and z-scores.** ΔTEC = observed − baseline, with a 95% band and flags where |z| > 2.
3. **Driver-informed model.** A second GP (ARD kernel) maps lagged −SYM-H (or E<sub>y</sub>) and a local-daylight factor to ΔTEC. The lag is selected on the training segment only.
4. **Two-station comparison.** Best lag, peak ΔTEC in TECU **and** as a percentage of baseline, peak time, and held-out RMSE and 95% coverage for baseline-only versus driver-informed prediction.
5. **Robustness check.** On synthetic data, the whole experiment is repeated over 8 random seeds with known planted lags.

## Repository layout

```
.
├── GP_Storm_TEC_ADIS_BJCO.ipynb   # main notebook
├── requirements.txt
├── README.md
└── data/                          # you create this (not included)
    ├── vtec_ADIS.csv
    ├── vtec_BJCO.csv
    └── omni.csv
```

## Quick start

```bash
pip install -r requirements.txt
jupyter notebook GP_Storm_TEC_ADIS_BJCO.ipynb
```

Run all cells. With `USE_REAL_DATA = False` (the default) no data files are needed.

## Using real data

Set `USE_REAL_DATA = True` in the configuration cell and provide three CSV files in `data/` with **UTC** timestamps:

| File | Columns | Notes |
|---|---|---|
| `vtec_ADIS.csv`, `vtec_BJCO.csv` | `utc,vtec` | VTEC in TECU, for example from GOPI GPS-TEC output converted to CSV. Resampled to 30 min in the notebook. |
| `omni.csv` | `utc,sym_h,bz,vsw` | SYM-H (nT), IMF Bz GSM (nT) and flow speed (km/s) from [OMNIWeb](https://omniweb.gsfc.nasa.gov/). Use the 1-min or 5-min data for SYM-H. |

Example rows:

```
utc,vtec
2015-06-22 18:00:00,14.8
2015-06-22 18:30:00,15.6

utc,sym_h,bz,vsw
2015-06-22 18:00:00,-3.0,2.1,455.0
```

Then check these settings:

- `STORM_START`, `TRAIN_STORM_UNTIL` and the date range (`START`, `END`).
- `QUIET_DAYS`: set explicitly (for example the international quiet days of the month) and repeat the analysis with several choices.
- `DRIVER`: `"sym_h"` or `"ey"`.
- Station coordinates, using the IGS site logs. Longitudes are **east-positive**.

GOPI output formats vary by version. If your files differ, edit `load_vtec` only.

## Method notes

- **Kernel.** `ExpSineSquared` (period bounded near 24 h) × `RBF` for a slowly evolving diurnal cycle, plus an `RBF` trend and `WhiteKernel` noise.
- **Prior mean for ΔTEC is zero** (target scaled but not centred), so with no driver the model predicts no storm effect.
- **Uncertainty.** Stage-2 predictive uncertainty and baseline latent uncertainty are added in quadrature (an approximation).
- **Validation.** Chronological hold-out (fit on early storm hours, test on later ones). Report RMSE **and** 95% interval coverage, since RMSE alone says nothing about whether the uncertainty is calibrated.

## Limitations

- One storm and two stations: any real-data conclusions are descriptive for this event.
- ΔTEC depends strongly on the quiet-day choice.
- A single lag per station and a simple two-input model. A joint model and more drivers (E<sub>y</sub>, pressure, AE) are natural extensions.
- Day-to-day baseline variability that is unrelated to the driver limits what any driver-based model can achieve on a held-out day. The synthetic robustness section illustrates this.

## References

- Seemala, G. K., and Valladares, C. E. (2011). Statistics of total electron content depletions observed over the South American continent for the year 2008. *Radio Science*, 46, RS5019.
- King, J. H., and Papitashvili, N. E. (2005). Solar wind spatial scales in and comparisons of hourly Wind and ACE plasma and magnetic field data. *Journal of Geophysical Research*, 110, A02104. (OMNI data)
- Rasmussen, C. E., and Williams, C. K. I. (2006). *Gaussian Processes for Machine Learning*. MIT Press.
- Liemohn, M. W., et al. (2021). RMSE is not enough: Guidelines to robust data-model comparisons for magnetospheric physics. *Journal of Atmospheric and Solar-Terrestrial Physics*, 218, 105624.

## Author

Ezekwueme Augustine Elozana · ORCID [0009-0003-7310-5217](https://orcid.org/0009-0003-7310-5217)

Licence: MIT
