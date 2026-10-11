# Fieldnotes

Fieldnotes is a small, local-first notebook for collecting measurements from physics experiments and exporting them as pandas-friendly CSV files. Its starter experiment is based on `grip12.txt`: measuring grip strength before and after squeezing an elastic ball, then recording recovery over time.

## Open the app

Open `index.html` in a modern browser. No install or account is needed.

## Use it

1. Start with **Grip strength recovery** or create a separate experiment.
2. Record one reading per observation. For a recovery curve, repeat the same trial at each recovery time; keep the participant, trial, protocol, baseline force, and condition fields consistent across those rows.
3. Add fields to match your protocol. The starter fields cover participant, hand, squeeze duration and intensity, squeeze count, recovery time, baseline and measured grip strength, food/meal condition, and other notes.
4. Search/filter the observation table, select the rows you want, and choose **Export selected**. **Export CSV** exports the whole experiment; **Export all data** combines experiments with an `experiment` column.
5. Use **Simulate grip experiment** or **Simulate** to explore a synthetic fatigue/recovery curve. Adjust the baseline, squeeze protocol, fatigue/recovery time constants and maximum fatigue. The output is labeled as simulated; you can optionally add its time-series rows to the current experiment. Use **Analyze** to view local summaries (mean, median, standard deviation, minimum and maximum), compare groups, and chart numeric fields. For advanced analysis, export a CSV and load it with `pandas.read_csv("grip_strength_recovery.csv")`.

Numbers are stored as numeric values in the CSV; text fields remain text. CSV cells are quoted and escaped, and a UTF-8 BOM is included for compatibility with spreadsheet software. A JSON backup can be downloaded and restored from **Backup / restore**.

## Storage and privacy

Data stays in the browser's local storage on the current device. Clearing browser data removes it, so keep a JSON backup if the records matter. This app does not upload your data. Analysis runs in your browser.



## Simulator model and parameter guide

In the simulator, select a CSV file, map its time and measured-force columns, optionally map a baseline column, then choose **Fit simulation to CSV**. For files with multiple participants or trials, use the filters so one recovery series is fitted at a time. The simulator estimates baseline (when supplied), immediate post-fatigue force, and recovery τ, plots the fitted curve over the CSV observations, and updates the simulator inputs. CSV time values must be numeric seconds and force values numeric in the same units.

The simulator produces synthetic values. It is a transparent, adjustable model, not a medical model and not a prediction for a particular participant. Fit its parameters to repeated measurements before comparing its curve with real observations.

The linked 2025 study used 10 maximal handgrip contractions of 3 seconds each, with 5 seconds of rest, then repeated the set after one hour. It calculated `Fmax/Fmean` across the ten dynamometer grips (fatigue ratio) and `Fmean2/Fmean1` across the two sessions (recovery ratio). Those are useful measures for that repeated-dynamometer protocol. They do not directly describe the recovery curve after squeezing an elastic ball, so the simulator does not claim to reproduce its results.

The simulation uses a normalized fatigue state `x` between 0 and 1. For each squeeze:

```text
x_after_squeeze = 1 - (1 - x_before) * exp(-((intensity / 100)^p * contraction_seconds / fatigue_tau))
x_before_next_squeeze = x_after_squeeze * exp(-rest_seconds / between_squeeze_tau)
fatigue_fraction_at_end = max_fatigue_fraction * x_after_last_squeeze
F_min = baseline_force * (1 - fatigue_fraction_at_end)
F(t) = baseline_force - (baseline_force - F_min) * exp(-t / recovery_tau)
```

Parameters and their effects:

- **Baseline force (`F0`)** sets the participant’s unfatigued grip strength. It scales the simulated force in newtons; it does not change the percent loss in this model.
- **Contraction time (`T`)** increases the modeled fatigue state as the exposure term `T / fatigue_tau` grows.
- **Intensity (`I`, %MVC)** increases fatigue nonlinearly as `(I / 100)^p`. `p` is an adjustable shape assumption, not a universal physiological constant.
- **Number of squeezes (`n`)** repeats the fatigue update, so more squeezes generally increase fatigue.
- **Rest between squeezes (`R`)** reduces fatigue before the next squeeze by `exp(-R / between_squeeze_tau)`. More rest generally means less accumulated fatigue. This reflects why contraction/rest pattern matters beyond total squeeze time.
- **Fatigue time (`fatigue_tau`)** controls the speed at which the model accumulates fatigue; smaller values increase fatigue faster.
- **Between-squeeze recovery time (`between_squeeze_tau`)** controls how much recovery occurs during each rest interval; smaller values mean faster modeled recovery during the protocol.
- **Maximum fatigue (`A`)** sets the upper scale of the modeled force loss. It is a calibration parameter, not a value supplied by the paper.
- **Starting fatigue** sets the initial state `x`, for example if the participant has not fully rested before the protocol.
- **Delay to first test** shifts all sampled recovery times after ball release. This matters because the muscle is already recovering while the dynamometer test is set up.
- **Measurement noise SD** adds random normal measurement variation, expressed as a percentage of `F0`; it changes simulated observed values, not the underlying fatigue state. Set it to zero for the mean curve.
- **Recovery time (`recovery_tau`)** controls the post-exercise curve. After `3 * recovery_tau`, the model has recovered 95% of the force deficit present at time zero.

The fatigue and recovery equations are deliberately separate so you can calibrate them from data. A simple exponential recovery curve is a starting approximation; published handgrip recovery work has also found multi-exponential recovery, so inspect residuals and do not treat one `tau` as universal. Hand posture, hand dominance, dynamometer, ball stiffness, exact force history, prior activity, sleep, and participant characteristics should be recorded or held constant. The current simulator does not assign unsupported numeric multipliers to those factors. Food/meal effects are not given a default coefficient because the cited study does not quantify them.

### Research sources

- Kapan et al. (2025), [Fatigue and recovery assessed by repetitive handgrip strength measurement as predictors of fall risk in older adults](https://journals.sagepub.com/doi/full/10.1177/02692155251355881). Source for the 10 × 3 s, 5 s rest protocol and fatigue/recovery ratio definitions.
- Soo et al. (2012), [Quantitative estimation of muscle fatigue on cyclic handgrip tasks](https://doi.org/10.1016/j.ergon.2011.10.004). Models fatigue accumulation and recovery over cyclic contraction/rest tasks; supports treating task timing and rest as model inputs.
- Adamo et al. (2009), [Low mean level sustained and intermittent grip exertions: influence of age on fatigue and recovery](https://doi.org/10.1080/00140130902984935). Reports shorter recovery after intermittent than sustained handgrip exertions, showing that rest distribution matters.
- Stull and Kearney (1978), [Recovery of muscular endurance following submaximal, isometric exercise](https://pubmed.ncbi.nlm.nih.gov/692299/). Reports a three-component exponential recovery pattern in their endurance task, a reason to treat a single-exponential curve as an approximation.
