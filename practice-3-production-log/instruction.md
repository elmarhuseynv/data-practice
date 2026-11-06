# Practice 3: Production Log and Machines

Same workflow as before (load, clean, analyze, plot, conclude), with three new skills: **dates**, **merging two tables**, and **monthly/weekday summaries**. The problems only state the goal. You decide how. Ask for a hint when you're stuck, not for the full code.

**Folder:** `practice-3-production-log/`

## The scenario

A factory has 6 machines (CNC, Press, Welder) in two plants. Each day, each machine logs how many units it made, how many were defective, and how many minutes it was down. You have two files:

- `production_log.csv`: one row per machine per day (the daily log)
- `machines.csv`: one row per machine (type, plant, install year)

Your job: clean the log, join it with the machine info, and tell a plant manager what you see.

## Columns

**production_log.csv**
- `date`: the day of the record
- `machine_id`: which machine (M01 to M06)
- `shift`: Day or Night
- `units`: units produced that day
- `defects`: how many of those units were defective
- `downtime_min`: minutes the machine was stopped

**machines.csv**
- `machine_id`: the machine
- `type`: CNC, Press or Welder
- `plant`: North or South
- `install_year`: the year the machine was installed

## Setup (run once)

Creates `machines.csv` and `production_log.csv` (about 2000 rows for the year 2025). The log has messy dates, messy text, missing values, impossible values, unknown machines and duplicates.

```python
import random
import pandas as pd

random.seed(11)

machines = pd.DataFrame({
    "machine_id": ["M01", "M02", "M03", "M04", "M05", "M06"],
    "type": ["CNC", "CNC", "Press", "Press", "Welder", "Welder"],
    "plant": ["North", "North", "North", "South", "South", "South"],
    "install_year": [2015, 2019, 2012, 2020, 2017, 2022],
})
machines.to_csv("machines.csv", index=False)

base_rate = {"CNC": 120, "Press": 200, "Welder": 80}

rows = []
for d in pd.date_range("2025-01-01", "2025-12-31"):
    for _, m in machines.iterrows():
        if random.random() < 0.05:
            continue
        shift = random.choice(["Day", "Night"])
        age = 2025 - m["install_year"]
        rate = base_rate[m["type"]] * (1 - 0.01 * age) * (0.92 if shift == "Night" else 1.0)
        units = max(1, round(random.gauss(rate, rate * 0.1)))
        defect_p = 0.02 + 0.004 * age + (0.01 if shift == "Night" else 0)
        defects = min(units, max(0, round(random.gauss(units * defect_p, 1.5))))
        downtime = max(0, round(random.gauss(30 + 5 * age, 15)))
        rows.append([d.strftime("%Y-%m-%d"), m["machine_id"], shift, units, defects, downtime])

log = pd.DataFrame(rows, columns=["date", "machine_id", "shift", "units", "defects", "downtime_min"])

for i in random.sample(range(len(log)), 25):
    log.loc[i, "date"] = pd.to_datetime(log.loc[i, "date"]).strftime("%d.%m.%Y")
for i in random.sample(range(len(log)), 60):
    log.loc[i, "shift"] = random.choice([log.loc[i, "shift"].lower(), log.loc[i, "shift"].upper()])
for i in random.sample(range(len(log)), 20):
    log.loc[i, "units"] = None
for i in random.sample(range(len(log)), 8):
    log.loc[i, "units"] = -1
for i in random.sample(range(len(log)), 15):
    log.loc[i, "machine_id"] = "M07"

log = pd.concat([log, log.sample(12, random_state=5)]).reset_index(drop=True)
log.to_csv("production_log.csv", index=False)
```

## Problems

### 1. Find the problems
Load both files. List every data problem you can find in the log (at least 5). For each one, write how you found it and how many rows are affected.

### 2. Clean the log
Write `clean_log(df)` that fixes the problems from problem 1. The `date` column must become a real datetime column. For the missing `units`, decide what to do (drop or fill) and write one sentence on why.

Check:
- `date` has the datetime type and no missing dates
- `shift` has exactly 2 unique values
- no duplicates
- no negative `units`

### 3. Merge with the machine table
Join the cleaned log with `machines.csv` so every row has its machine type, plant and install year. Some `machine_id` values in the log do not exist in the machine table. Find them, decide what to do with them, and write one sentence on why. Add a column `machine_age` (2025 minus install year).

### 4. Monthly output
How many units were produced in each month of 2025? Draw a line chart. Then write one sentence: is the output steady, rising or falling?

### 5. Defect rate
Add a column `defect_rate` (defects divided by units). Compare the average defect rate by machine type and by shift. Which type has the highest rate? Do night shifts make more defects? Support your answer with numbers and one chart.

### 6. Machine age
Does an older machine have more downtime? Use a chart and a correlation number. Check it per machine (6 points), not per day. Is the relationship strong, weak or none?

### 7. Weekday pattern
Which weekday has the highest average output, and which has the lowest? Draw a bar chart in the order Monday to Sunday.

### 8. Report function and dashboard
Write `machine_report(df, machine_id)` that returns a dict (total units, average defect rate, total downtime hours, number of days logged) and saves a chart `report_<machine_id>.png`. Loop over all machines and collect the results into one DataFrame.

Then make a 2×2 figure saved as `dashboard.png` with 4 charts you choose, and write 3 to 5 sentences: what would you tell a plant manager?

## New tools you will meet

- `pd.to_datetime(...)` and the `.dt` accessor (`.dt.month`, `.dt.day_name()`)
- `pd.merge(...)` and checking which rows did not match
- Grouping by month and by weekday

## Rules

- Spend about 15 minutes on a problem before asking for help.
- Ask for a hint, and I will guide you step by step.
- Keep the notebook in one folder with one heading per problem.

## Notion tasks (short)

- [ ] Clean production log
- [ ] Merge log with machines
- [ ] Production charts
- [ ] Machine report and dashboard

## Checklist

- [ ] Setup: generate both CSV files
- [ ] 1. Find data problems
- [ ] 2. clean_log(df)
- [ ] 3. Merge with machines
- [ ] 4. Monthly output
- [ ] 5. Defect rate
- [ ] 6. Machine age vs downtime
- [ ] 7. Weekday pattern
- [ ] 8. machine_report() + dashboard