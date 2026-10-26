# Solar Energy Production Forecasting

Forecasting the hourly electricity output of Edikli GES, a solar power plant in Niğde, Turkey, from weather data.

Group project (Group 21), IE 360 — Time Series Analysis, Boğaziçi University.

## The problem

Solar output cannot be stored easily and varies with weather, season and time of day, so a plant operator has to predict tomorrow's production in order to trade and distribute it. The forecast has to be made a day ahead, which rules out using the most recent production figures — at submission time, the previous day's production is not yet available.

## Data

- Hourly production data from the plant
- Weather data from 25 grid points surrounding the site, including temperature, cloud cover, relative humidity and downward shortwave radiation flux (DSWRF)
- Only the 16 daylight hours carry production, so the rest are excluded

## Approach

The forecast is built in two stages: predict the **daily** total first, then distribute it across hours. Hourly data is far noisier than daily totals, so modelling the day first reduces the variance the model has to absorb.

### Stage 1 — daily production

**Stationarity.** The daily series fails a KPSS test: the mean and variance move over time and there is clear seasonality. Differencing at lag 2 — rather than lag 1 — was chosen deliberately, because the previous day's production isn't available when the forecast is submitted. After differencing, KPSS confirms stationarity and the seasonal pattern disappears.

**ARIMA.** ACF and PACF of the differenced series suggested ARIMA(0,0,3); `auto.arima` returned ARIMA(1,0,0). The two were compared on BIC and on residual diagnostics. ARIMA(0,0,3) won on both — the ARIMA(1,0,0) residuals still showed significant partial autocorrelation out to lag 14, meaning information remained unextracted.

**Linear regression.** A regression on weather variables and seasonality terms was fitted as the alternative.

**Comparison.** Both models were evaluated on FBias, MAPE, RMSE, MAD, MADP and WMAPE. Linear regression gave the lower WMAPE and was selected; ARIMA's bias is explained by having no seasonal information available to it.

### Stage 2 — distributing the day across hours

Two methods were compared for turning a daily total into 16 hourly figures:

1. **One model for all hours** — a single regression on the full hourly dataset, used to derive each hour's share of the day
2. **One model per hour** — a separate regression for each hour of the day, with weather and time features

Both produce hourly ratios, which are then multiplied by the daily forecast.

### Evaluation

Models were compared on FBias, RMSE and WMAPE under k-fold cross-validation, with regression formulas adjusted iteratively to minimise test WMAPE. The single hourly model combined with the daily forecast was selected, giving the lowest test error with only slight overfitting. Daily WMAPE scores from the competition phase are reported at the end of the notebook.

## Files

- `IE360_Group21_ProjectReport.ipynb` — full analysis: data exploration, modelling, evaluation, prediction function
- `IE360_Group21_ProjectReport.html` — rendered version, readable without running anything

## Tools

R · ARIMA · linear regression · KPSS test · k-fold cross-validation
