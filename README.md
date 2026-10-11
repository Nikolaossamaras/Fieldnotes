# Fieldnotes

Fieldnotes is a small, local-first notebook for collecting measurements from physics experiments and exporting them as pandas-friendly CSV files. Its starter experiment is based on `grip12.txt`: measuring grip strength before and after squeezing an elastic ball, then recording recovery over time.

## Open the app

Open `index.html` in a modern browser. No install or account is needed.

## Use it

1. Start with **Grip strength recovery** or create a separate experiment.
2. Record one reading per observation. For a recovery curve, repeat the same trial at each recovery time; keep the participant, trial, protocol, baseline force, and condition fields consistent across those rows.
3. Add fields to match your protocol. The starter fields cover participant, hand, squeeze duration and intensity, squeeze count, recovery time, baseline and measured grip strength, food/meal condition, and other notes.
4. Search/filter the observation table, select the rows you want, and choose **Export selected**. **Export CSV** exports the whole experiment; **Export all data** combines experiments with an `experiment` column.
5. Use **Analyze** to view local summaries (mean, median, standard deviation, minimum and maximum), compare groups, and chart numeric fields. For advanced analysis, export a CSV and load it with `pandas.read_csv("grip_strength_recovery.csv")`.

Numbers are stored as numeric values in the CSV; text fields remain text. CSV cells are quoted and escaped, and a UTF-8 BOM is included for compatibility with spreadsheet software. A JSON backup can be downloaded and restored from **Backup / restore**.

## Storage and privacy

Data stays in the browser's local storage on the current device. Clearing browser data removes it, so keep a JSON backup if the records matter. This app does not upload your data. Analysis runs in your browser.

