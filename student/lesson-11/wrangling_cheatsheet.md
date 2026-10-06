# Data Wrangling Cheatsheet
SDEV2501 · Days 09-11 · Student reference

Everything from the three wrangling days in one place. It's a reference, not a tutorial: skim for the thing you need, copy the pattern, change the names. Nothing in here is new. If a method looks unfamiliar, you've seen it in a notebook; you just haven't needed it badly enough to remember it yet.

Two assumptions throughout: `import pandas as pd` and `import numpy as np` have been run, and your DataFrame is called `df`.

---

## 1. The workflow

**Inspect → Clean → Transform → Combine → Summarize.** In that order, and inspect means *look, don't touch*.

| Stage | Question you're answering | Changes `df`? |
|---|---|---|
| Load | Did it read correctly? | no |
| Inspect | What's wrong with it? (six checks, below) | **no** |
| Clean | Fix what you *know*, flag what you'd be *guessing*, leave the rest | yes |
| Transform | New columns, right types, common scale | yes |
| Combine | Stack or join with other tables | makes a new one |
| Summarize | Group, count, total, average | makes a new one |

Three habits that save you every time (I say this from experience, most of it painful):
- **Check after every fix.** A `value_counts()` or `describe()` straight after the change. A fix you didn't check is a hope.
- **Assign it back.** Most pandas methods return a *new* object. `df.drop_duplicates()` on its own does nothing you can see. `df = df.drop_duplicates()` does.
- **Restart and Run All before you call it done.** If it only works because you ran cell 14 before cell 9, it doesn't work.

---

## 2. Definitions

| Term | Plain meaning |
|---|---|
| **Data quality** | How well the data serves its purpose. The same file can be great for one question and useless for another |
| **Validation** | Rules that keep bad data *out* (at entry, at import). Validation prevents, cleaning repairs |
| **Missing value / null / `NaN`** | No value recorded. `NaN` ("not a number") is how pandas stores "nothing". Watch for missing-in-disguise: `"N/A"`, `"unknown"`, `0` where 0 isn't valid |
| **Duplicate** | The same record more than once. *Exact* duplicates repeat every column; *hidden* duplicates repeat the thing (same person, same booking) with different details |
| **Imputation** | Filling a missing value with a reasonable estimate (median, a lookup from another column). It's a guess, so flag it |
| **Boolean mask** | A True/False Series, one per row, built from a comparison. `df[mask]` keeps the True rows |
| **Normalization / scaling** | Putting numbers on a common scale (min-max, z-score). Not the database meaning of the word |
| **Transformation** | Changing the shape or form of data: a new column from arithmetic, a month from a date, text to a number |
| **Outlier** | A value far from the rest. Far by the numbers, which is not the same as wrong |
| **Merge** | Add *columns* from another table using a shared key. Excel's VLOOKUP, if you've met it |
| **Concat** | Add *rows* by stacking tables with the same columns |
| **Key** | The column two tables share, used to line rows up in a merge (`facility_id`, `member_id`) |

### The five dimensions of data quality

| Dimension | Asks | Example of a failure |
|---|---|---|
| Accuracy | Is it correct? | duplicate rows; `$4.50` that should be `$45` |
| Completeness | Is anything missing? | blank hours |
| Consistency | Does it agree with itself? | `AB` vs `Alberta`; `2026-09-01` vs `09/01/26` |
| Timeliness | Is it current? | a business listed as open that closed in 2023 |
| Validity | Does it follow the rules? | age 217; satisfaction 6 on a 1-5 scale; `2026-10-32` |

### Fix, flag, exclude or leave?

The one question: **do I know the right answer, or would I be guessing?**

| Call | When | Example |
|---|---|---|
| **Fix** | You have enough information to be sure | `edmonton`, `EDMONTON` → `Edmonton`; a blank activity when that facility only hosts one activity; a rate from the price list |
| **Flag** | It's wrong or doubtful, but you can't know the right value | 0 hours; a date in 2027; an imputed value (`hours_imputed = True`) |
| **Exclude** | The data can't answer the question at all | a category that's 40% "Other" |
| **Leave** | It's legitimate, or fixing it would invent data | a member who chose not to give their income; a booking with no member ID (the money is still real) |

---

## 3. The six inspection checks

Run all six before you fix anything. Fixing as you go means you fix the first three things you see and miss the sneaky ones.

| Check | Looking for | How |
|---|---|---|
| 1. Missing values | blanks, `NaN`, and missing-in-disguise | `df.isna().sum()` then `value_counts(dropna=False)` on text columns |
| 2. Duplicates | exact repeats, and repeated IDs with different details | `df.duplicated().sum()` vs `df["id"].duplicated().sum()`. If the numbers differ, you have the hidden kind |
| 3. Unexpected categories | spelling, case, trailing spaces, typos | `df["col"].value_counts(dropna=False)` |
| 4. Impossible values | zero, negative, out of range | `pd.to_numeric(df["col"], errors="coerce").describe()`, look at `min` and `max` |
| 5. Wrong data types | numbers or dates stored as text | `df.info()`; then find what won't convert |
| 6. Suspicious distributions | values that are *possible* but odd | a histogram **and** `value_counts()`. They catch different things |

On check 6: a histogram finds values far from the rest. `value_counts()` finds values that shouldn't exist at all, even when they sit right next to a normal one. A `4.50` beside a wall of `6.00`s looks fine on a histogram and screams in a value count.

---

## 4. pandas reference

### Load and look

| Job | Code |
|---|---|
| Load a CSV | `df = pd.read_csv("file.csv")` |
| First rows | `df.head(10)` |
| Rows × columns | `df.shape` |
| Column names as a list | `list(df.columns)` |
| Types and non-null counts | `df.info()` |
| Summary statistics (numeric columns) | `df.describe()` |
| Distinct values | `df["col"].unique()` |
| Count each value (include blanks) | `df["col"].value_counts(dropna=False)` |
| Save without the index | `df.to_csv("clean.csv", index=False)` |

### Boolean masks and selecting

| Job | Code |
|---|---|
| One column (a Series) | `df["col"]` |
| Several columns (a DataFrame) | `df[["a", "b"]]` |
| Rows where a condition holds | `df[df["hours"] <= 0]` |
| Two conditions | `df[(df["status"] == "completed") & (df["hours"] > 2)]` |
| Either condition | `(mask1) \| (mask2)` |
| Not | `~mask` |
| Value in a list | `df["activity"].isin(["Lane Swim", "Public Skate"])` |
| Rows and columns by label | `df.loc[mask, "col"]` |
| Assign to only the masked rows | `df.loc[mask, "col"] = value` |

Use `&`, `|` and `~`, never `and`, `or`, `not`, and put brackets around each comparison. `ValueError: The truth value of a Series is ambiguous` means you forgot one of those.

### Missing values

| Job | Code |
|---|---|
| Count per column | `df.isna().sum()` |
| Mask of blanks / non-blanks | `df["col"].isna()`, `df["col"].notna()` |
| Flag before you fill | `df["hours_imputed"] = df["hours"].isna()` |
| Fill with the median | `df["hours"] = df["hours"].fillna(df["hours"].median())` |
| Fill from a lookup, blanks only | `mask = df["rate"].isna()` then `df.loc[mask, "rate"] = df.loc[mask, "activity"].map(rates)` |
| What does each group contain? | `df.groupby("facility_id")["activity"].unique()` |
| Set a value to missing | `df.loc[mask, "col"] = np.nan` |

### Duplicates

| Job | Code |
|---|---|
| Count exact duplicate rows | `df.duplicated().sum()` |
| Count repeated IDs | `df["booking_id"].duplicated().sum()` |
| Show every copy, together | `df[df["booking_id"].duplicated(keep=False)].sort_values("booking_id")` |
| Drop exact duplicates | `df = df.drop_duplicates()` |
| Drop by one column, keep first | `df = df.drop_duplicates(subset="booking_id", keep="first")` |
| Confirm | `df["booking_id"].is_unique` |

### Text clean-up (the `.str` accessor)

| Job | Code |
|---|---|
| Trim ends | `df["col"].str.strip()` |
| Collapse repeated spaces | `df["col"].str.replace(r"\s+", " ", regex=True)` |
| Case | `.str.lower()`, `.str.upper()`, `.str.title()` |
| Remove a literal character | `df["rate"].str.replace("$", "", regex=False)` |
| Does it contain…? (blanks count as no) | `df["hours"].str.contains("min", na=False)` |
| Fix the ones rules can't | `df["col"] = df["col"].replace({"Gym Drop In": "Gym Drop-In", "complete": "completed"})` |
| Rename columns | `df = df.rename(columns={"rate ($/hr)": "hourly_rate"})` |

Chain them: `df["activity"] = df["activity"].str.strip().str.replace(r"\s+", " ", regex=True).str.title()`. Rules first (strip, case), then a dictionary for the leftovers.

### Types and dates

| Job | Code |
|---|---|
| Text → number, bad values become `NaN` | `pd.to_numeric(df["col"], errors="coerce")` |
| Find what didn't convert | `df[check.isna() & df["col"].notna()]` where `check` is the coerced copy |
| Text → dates, mixed formats | `pd.to_datetime(df["date"], format="mixed", errors="coerce")` |
| Rows that failed | `df[parsed.isna()]` |
| Compare dates to a string | `df["date"] < "2026-09-01"` |
| Pull a part out | `df["date"].dt.month_name()`, `.dt.day_name()`, `.dt.year` |
| Change a type | `df["col"] = df["col"].astype("float")` |

Convert into a *temporary variable* first (`parsed = pd.to_datetime(...)`), look at the failures, then assign it to the column. Once the text is gone you can't see what it was.

### New columns and arithmetic

| Job | Code |
|---|---|
| From other columns | `df["total"] = df["hours"] * df["hourly_rate"]` |
| Same value for every row | `df["export"] = "October"` |
| True/False flag | `df["needs_review"] = (df["hours"] <= 0) \| (df["hours"] > 12)` |
| Change some rows only | `df.loc[df["status"] != "completed", "total"] = 0` |

### Combine two tables

| | `pd.concat([a, b], ignore_index=True)` | `a.merge(b, on="key", how="left", indicator=True)` |
|---|---|---|
| Picture it as | stacking two sheets on top of each other | VLOOKUP: looking up extra columns from another sheet |
| Adds | **rows** | **columns** |
| Needs | the **same column names** | a shared **key** column |
| Gotcha | different headers → half-empty columns; overlapping records → duplicates you couldn't see before | keys that don't match → `NaN` in the new columns |
| Check it | `.shape`, then `duplicated()` on the ID | `merged["_merge"].value_counts()` |

`how="left"` keeps every row from the left table whether or not it matched. `indicator=True` adds a `_merge` column that says `both`, `left_only` or `right_only`, which is how you find the rows that didn't match.

### Group and summarize

| Job | Code |
|---|---|
| Total per group | `df.groupby("facility")["total"].sum()` |
| Several stats at once | `df.groupby("month")["total"].agg(["count", "sum", "mean"])` |
| Count rows per group | `df.groupby("activity")["booking_id"].count()` |
| Biggest first | `.sort_values("sum", ascending=False)` (or `.sort_values(ascending=False)` on a Series) |
| Only some rows first | `good = df[(df["status"] == "completed") & (~df["needs_review"])]` then group `good` |

---

## 5. Formulas

### Centre: mean vs median

| | What it is | When it misleads |
|---|---|---|
| **Mean** | sum ÷ count | a few big values drag it up (or down) |
| **Median** | the middle value when sorted | almost never; it ignores how big the extremes are |

`df["col"].mean()`, `df["col"].median()`. If the mean is well above the median, the data has a long tail on the high side. Quote the median as "typical" and mention the tail.

### Scaling

| Method | Formula | Result | Use when |
|---|---|---|---|
| **Min-max** | `(x - min) / (max - min)` | 0 to 1 | you want a bounded scale, or the data isn't bell-shaped (a price list, a rating scale) |
| **Z-score** | `(x - mean) / std` | mostly −3 to +3, centred on 0 | the data is roughly bell-shaped and you want "how unusual is this?" |

```python
col = df["hourly_rate"]
df["rate_minmax"] = (col - col.min()) / (col.max() - col.min())
df["rate_z"] = (col - col.mean()) / col.std()
```

Neither changes the order of the values, only the ruler. A z-score reads as "how many standard deviations from average": 0 is average, +2 is well above, −1 is a bit below.

### Outliers

| Rule | Flag anything… | Code |
|---|---|---|
| **IQR** | above `Q3 + 1.5 × IQR` or below `Q1 − 1.5 × IQR`, where `IQR = Q3 − Q1` | `q1 = col.quantile(0.25)`, `q3 = col.quantile(0.75)`, `fence = q3 + 1.5 * (q3 - q1)`, `df[col > fence]` |
| **Z-score** | with `|z| > 3` | `z = (col - col.mean()) / col.std()`, `df[z.abs() > 3]` |

They disagree, often by a lot. IQR is built from the middle 50%, so it doesn't care how big the extremes are and tends to flag more. Z-score uses the standard deviation, which the extremes have already inflated, so it flags fewer. Pick one, say which, and remember: **an outlier is a value to look at, not a value to delete.** Delete when you can show it's wrong, not when it's inconvenient.

---

## 6. Errors you'll see

| You see | It usually means |
|---|---|
| `FileNotFoundError` | The CSV isn't in the same folder as the notebook |
| `KeyError: 'col'` | No column by that name. Check `list(df.columns)` for spelling, case, spaces, or a rename you haven't done yet |
| `NameError: name 'df' is not defined` | You restarted the kernel and didn't re-run the cells above |
| `ValueError: The truth value of a Series is ambiguous` | `and` / `or` instead of `&` / `\|`, or missing brackets around a comparison |
| `ValueError: Unable to parse string "90 min"` | Text that isn't a number. Clean it, or use `errors="coerce"` to find it |
| `TypeError: Cannot perform reduction 'mean' with string dtype` | The column is text. Check `.info()`, fix what won't convert, then `pd.to_numeric` |
| `AttributeError: Can only use .str accessor with string values` | The column is already numeric, or already converted. You probably ran the cell twice |
| `SettingWithCopyWarning` | You changed a slice of a DataFrame. Use `df.loc[mask, "col"] = value` on the original |
| "It ran, but nothing changed" | You didn't assign the result back |
| Half-empty columns after `concat` | Column names didn't match. Rename first, then stack |
| `NaN` in every merged column | The keys don't match: `F7` vs `F07`, different case, trailing spaces |
