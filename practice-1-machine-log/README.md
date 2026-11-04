# Python + pandas + Matplotlib: Practice Set

One dataset (a machine sensor log), 9 problems, easy to hard. Do them in order. Each problem builds on the previous one.

## Setup (run once)

Creates `machine_log.csv` with 3 machines, 30 days of hourly data, plus some missing values and duplicate rows to clean.

```python
import random
import pandas as pd

random.seed(42)
rows = []
for machine in ["M1", "M2", "M3"]:
    for h in range(30 * 24):
        ts = pd.Timestamp("2026-09-01") + pd.Timedelta(hours=h)
        temp = random.gauss(70 + (8 if machine == "M3" else 0), 5)
        vib = abs(random.gauss(2.0 + (0.8 if machine == "M2" else 0), 0.6))
        power = random.gauss(15, 2)
        defects = max(0, int(random.gauss(2 + vib, 1.5)))
        rows.append([ts, machine, temp, vib, power, defects])

df = pd.DataFrame(rows, columns=["timestamp", "machine", "temperature_C", "vibration_mm_s", "power_kW", "defects"])

for i in random.sample(range(len(df)), 40):
    df.loc[i, "temperature_C"] = None

df = pd.concat([df, df.sample(15, random_state=1)]).reset_index(drop=True)
df.to_csv("machine_log.csv", index=False)
```

---

## Core problems

### 1. Load and inspect
Load `machine_log.csv`. Print: shape, column dtypes, number of missing values per column, number of duplicate rows. Is `timestamp` a real datetime right now?

### 2. Clean (as a function)
Write `clean(df)` that returns a new DataFrame with:
- duplicates removed
- `timestamp` converted to datetime
- missing `temperature_C` filled with the **median of that machine** (not the global median)

Check: no missing values and no duplicates left.

### 3. Status labels (plain Python logic)
Write a function `get_status(temp, vib)` that returns:
- `"overheat"` if temp > 80
- `"high_vibration"` if vib > 3.0
- `"ok"` otherwise

Apply it to every row to make a `status` column. Print how many rows of each status there are.

### 4. GroupBy summary
For each machine, compute: mean temperature, max vibration, total defects, number of `overheat` rows. Then answer in one line each:
- Which machine has the most defects?
- Which machine overheats most?

### 5. Line chart over time
Compute the **daily average temperature per machine**. Plot all 3 machines on one chart with a legend, axis labels, title, and a horizontal dashed line at 80 °C. Rotate the date labels so they are readable.

### 6. Scatter plot + correlation
Plot vibration (x) vs defects (y), one color per machine (use a loop over machines). Then print the correlation between vibration and defects. Write one sentence: what does the plot suggest?

---

## Harder problems

### 7. Distributions side by side
Make one figure with 3 subplots in a row (one per machine), each a histogram of `temperature_C`. All subplots must share the same x-axis range so they can be compared fairly. Add the mean as a vertical line in each.

### 8. Dashboard
Build a 2×2 figure and save it as `dashboard.png`:
- top-left: daily average temperature (line, per machine)
- top-right: total defects per machine (bar)
- bottom-left: vibration vs defects (scatter)
- bottom-right: share of each status (bar or pie)

### 9. Mini report generator (capstone)
Write `machine_report(df, machine)` that:
- filters one machine
- returns a dict: `{"machine", "avg_temp", "max_vib", "total_defects", "overheat_hours"}`
- saves a chart `report_<machine>.png` (temperature over time + 80 °C line)

Loop over all machines, collect the dicts into a new DataFrame, and print it sorted by `total_defects` (highest first). Finish with 3 to 5 sentences of findings: which machine needs maintenance first, and why?

---

## Stretch (optional)
- Find the 5 worst hours (highest vibration) for each machine using `groupby` + `nlargest`.
- Add a 24-hour rolling average of temperature to the problem 5 chart.
- Find which hour of the day has the most defects overall. Plot defects by hour.

## Rules for yourself
- Try each problem without looking things up first, then check the docs.
- Keep your code in one file or notebook, one section per problem.
- When done, send me your code for any problem and I will review it.