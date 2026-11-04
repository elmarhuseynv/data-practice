# Practice 2: Tensile Test Data

Same workflow as practice 1 (load, clean, analyze, plot, conclude), but the problems only state the goal. You decide how. Ask for a hint when you're stuck, not for the full code.

**Folder:** `practice-2-tensile-tests/`

## Setup (run once)

Creates `tensile_tests.csv`: 600 tests of 3 materials, 3 batches, 4 test temperatures. It has messy text, missing values, impossible values and duplicates.

```python
import random
import pandas as pd

random.seed(7)
base = {"Steel": (350, 520, 22), "Aluminum": (270, 310, 12), "Titanium": (830, 900, 14)}
batch_factor = {"A": 1.0, "B": 0.95, "C": 1.05}

rows = []
for i in range(600):
    material = random.choice(list(base))
    y0, u0, e0 = base[material]
    batch = random.choice(["A", "B", "C"])
    thickness = round(random.uniform(1.0, 6.0), 2)
    temp = random.choice([20, 100, 200, 300])
    k = (1 - 0.0007 * (temp - 20)) * batch_factor[batch]
    yield_ = random.gauss(y0 * k, y0 * 0.04)
    ultimate = yield_ + random.gauss(u0 - y0, 15)
    elong = max(1, random.gauss(e0 * (1 + (temp - 20) / 800), 2))
    rows.append([f"T{i:03d}", material, batch, thickness, temp,
                 round(yield_, 1), round(ultimate, 1), round(elong, 1)])

df = pd.DataFrame(rows, columns=["sample_id", "material", "batch", "thickness_mm",
                                 "test_temp_C", "yield_MPa", "ultimate_MPa", "elongation_pct"])

for i in random.sample(range(len(df)), 30):
    df.loc[i, "yield_MPa"] = None
for i in random.sample(range(len(df)), 40):
    df.loc[i, "material"] = df.loc[i, "material"].lower()
for i in random.sample(range(len(df)), 20):
    df.loc[i, "material"] = " " + df.loc[i, "material"] + " "
for i in random.sample(range(len(df)), 5):
    df.loc[i, "ultimate_MPa"] = -999

df = pd.concat([df, df.sample(10, random_state=3)]).reset_index(drop=True)
df.to_csv("tensile_tests.csv", index=False)
```

## Problems

### 1. Find the problems
Load the file. List every data problem you can find (at least 4). For each one, write how you found it.

### 2. Clean it
Write `clean(df)` that fixes all the problems from problem 1. For the missing `yield_MPa`, decide what to do (drop, or fill) and write one sentence on why.

Check:
- `material` has exactly 3 unique values
- no duplicates
- no negative strengths

### 3. Pass rate
Add a column `passed`: `True` if `yield_MPa >= 300`. Which material has the highest pass rate, and which the lowest? (Think: is the 300 MPa limit fair for aluminum?)

### 4. Temperature effect
How does test temperature change the yield strength of each material? Choose a chart that answers it, and write one sentence on the trend.

### 5. Batch effect
Does the batch matter? Compare the batches and decide: is one batch weaker? Support your answer with a number and a chart.

### 6. Thickness
Does thickness affect elongation? Use a scatter plot and a correlation number. Is the relationship strong, weak or none?

### 7. Report function
Write `material_report(df, material)` that returns a dict (average yield, average ultimate, pass rate, number of tests) and saves a chart `report_<material>.png`. Loop over all materials and collect the results into one DataFrame.

### 8. Dashboard and findings
A 2×2 figure saved as `dashboard.png` with 4 charts you choose. Then write 3 to 5 sentences: what would you tell a quality manager about these materials?

## Rules

- Spend about 15 minutes on a problem before asking for help.
- Ask for a hint, and I will guide you step by step.
- Keep the notebook in one folder with one heading per problem.

## Notion tasks (short)

- [ ] Clean tensile data
- [ ] Tensile charts
- [ ] Material report