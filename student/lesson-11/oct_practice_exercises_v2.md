# October Bookings: Exercise Guide
SDEV2501 · Day 11 · Practice · Goes with `oct_practice_starter_v2.ipynb`

Today you clean a messy file with a lot less hand-holding. Work on your own or with a partner, whichever you prefer. The section numbers match the notebook. Each step tells you what to do and lists the functions, methods and operators you'll need; putting them together is up to you.

What this guide won't tell you is what's wrong with the October file. That's not me being mean: spotting the problems is the skill you're practising.

When you've finished a step, check yourself against the **answer key** at the end. Try first, then look. The answer key gives away every problem in the file, so if you read it before you start, you've just turned a detective story into a shopping list.

---

## Before you start

Put the notebook (`oct_practice_starter_v2.ipynb`), `bookings_oct_messy.csv`, `bookings_sept_clean.csv` and `facilities.csv` in **one folder**. Open the folder in VS Code, open the notebook, pick your kernel, and run the imports cell.

---

# Part A: Inspect & Clean October

## 1. Load

**1.1** Load `bookings_oct_messy.csv` into a DataFrame called `df`. Show the first 10 rows, the shape and the column names.

You'll need: `pd.read_csv()`, `.head()`, the `.shape` attribute, `list()` on the `.columns` attribute

**1.2** Load `bookings_sept_clean.csv` into `sept` and list its columns too. Put the two lists side by side. Which names don't match? Which columns does September have that October doesn't?

You'll need: the same as 1.1

## 2. Inspect

Look, don't touch. Nothing in this section should change `df`. Every problem you find goes in the findings table at the end of section 2, because future-you will not remember.

### 2.1 Missing values
Count the missing values in each column. Which columns have them?

You'll need: `.isna()`, `.sum()`

### 2.2 Duplicates
Count the exact duplicate rows, then count the repeated values in `booking_id` on its own. Are the two numbers the same? What would it mean if they weren't?

You'll need: `.duplicated()`, `.sum()`, called on the DataFrame and then on a single column

### 2.3 Unexpected categories
Count every distinct value in `activity`, `status` and `facility`, one cell each. By default, `value_counts()` quietly skips missing values, so pass it `dropna=False` to include them. How many activities are there *really*? Is there a facility code that's wrong in a way that changing its case won't fix?

You'll need: `.value_counts()` with the `dropna` argument

### 2.4 Impossible values
`hours` is stored as text. Make a temporary numeric copy called `hours_check` (leave `df` alone) and look at its summary statistics. Then show any rows with zero or negative hours.

Look at the `count` in the summary, then work out how many hours aren't blank. Do those two numbers match?

You'll need: `pd.to_numeric()` with `errors="coerce"`, `.describe()`, a comparison operator and boolean indexing

### 2.5 Wrong data types
Check the type of every column. Then show the hours that have a value in `df` but are missing in `hours_check`; those are the ones that didn't convert. Finally, list every distinct rate.

Look closely at the hours that didn't convert. What are they, really? How would you turn them into hours?

You'll need: `.info()`, `.isna()`, `.notna()`, `&` to combine the two masks, `.unique()`

### 2.6 Suspicious distributions
Make a numeric copy of the rate called `rate_check` (you'll have to deal with the `$` first). Plot its histogram, then count its values. Does the histogram show anything odd? Do the value counts? Why would one check catch something the other misses?

You'll need: `.str.replace()` with `regex=False`, `pd.to_numeric()`, `.plot()` with `kind="hist"` and `bins`, `plt.show()`, `.value_counts()`

### Findings
Fill in the findings table (double-click the markdown cell to edit it). Write one row per problem, including whether you plan to fix, flag or leave it.

## 3. Clean

For every fix, ask: **do I know the right answer, or am I guessing?** If you know, fix it. If you'd be guessing, flag it or leave it. And check every fix straight after you make it.

> **Assign it back.** Most pandas methods return a *new* object and leave `df` exactly as it was. If you don't assign the result back (`df = ...` or `df["col"] = ...`), nothing changes, and pandas won't tell you.

### 3.1 Rename columns
Rename the October columns so they match September's names exactly.

You'll need: `.rename()` with its `columns` argument (a dictionary of `{old: new}`)

### 3.2 Remove duplicates
Drop the exact duplicates and check that every booking ID is now unique.

You'll need: `.drop_duplicates()`, the `.is_unique` attribute

### 3.3 Fix categories
Clean `activity`, `status` and `facility_id` so each value has exactly one spelling. Rules (strip, change case) handle most of it; a dictionary handles anything the rules can't.

You'll need: `.str.strip()`, `.str.replace()` with a regular expression (`r"\s+"`, with `regex=True`), `.str.title()`, `.str.lower()`, `.str.upper()`, a dictionary, `.replace()`

### 3.4 Fix data types
Convert `hours` to numbers, including the odd values you found in 2.5, then convert `hourly_rate`. Check your work with summary statistics.

The odd hours need three moves, in this order. First, build a boolean mask of the rows that need converting and save it to a variable, *before* you change the column (once the text is gone, you can't tell which rows they were). Second, strip the unit off so the whole column converts. Third, use the mask to change only those rows.

You'll need: `.str.contains()` (pass `na=False` so blank values count as `False` instead of `NaN`), `.str.replace()`, `pd.to_numeric()`, `.loc[mask, column]` to select and assign, arithmetic

### 3.5 Missing values
Deal with each column that has blanks, one at a time:

1. **activity:** group the activities by facility and look at the unique values. Can the facility tell you what the missing activity should be?
2. **hourly_rate:** fill the blanks from a dictionary of standard rates.
3. **hours:** add a True/False column called `hours_imputed` that marks the blanks, *then* fill them with the median.
4. **member_id:** decide what to do, and write down why.

Fill `activity` before `hourly_rate`, because the rate lookup needs the activity. Only change the blank rows. Then count what's still missing. Three columns had blanks and you made three different decisions: what made each one different?

You'll need: `.groupby()`, `.unique()`, `.isna()`, `.loc[]`, a dictionary, `.map()`, `.median()`, `.fillna()`

### 3.6 Impossible and suspicious values
Add a True/False `needs_review` column for hours that are 0 or less, or over 12. Then set the impossible hours to missing so they don't count in totals, and fix the bad rate. Check with summary statistics.

You'll need: comparison operators, `|`, `.loc[]` to assign, `np.nan`, `.describe()`

### 3.7 Save
Check that October's column names now match September's exactly, then save to `bookings_oct_cleaned.csv` without the index.

You'll need: `list()` and `==` to compare the two column lists, `.to_csv()` with `index=False`, `.shape`

## Merge or concatenate?
Before you start Part B, stop and think. You have September bookings, October bookings, and a table of facility names. Which job is a **merge** and which is a **concat**? Write your answer in the notebook, then check it against the answer key.

---

# Part B: Challenge

Two clean months, one question: what happens when you put them together? Restart the kernel, run the imports cell, then jump to section 4.

## 4. Stack the two months

**4.1** Load both clean files into `sept` and `october`. (Don't call it `oct`: that's already a built-in Python function, and you'd be overwriting it.) Add an `export` column to each that says `"September"` or `"October"`, then stack them into one DataFrame called `both`. Check its shape.

You'll need: `pd.read_csv()`, assigning a single value to a new column, `pd.concat()` (pass it a list of DataFrames, plus `ignore_index=True` so the row labels restart from 0)

**4.2** Every booking should appear once. Check for repeated booking IDs in `both` and look at them. Decide which copy to keep, then remove the others. These weren't duplicates in Part A. Why do they only show up now?

You'll need: `.duplicated()` with `keep=False`, `.sort_values()`, `.drop_duplicates()` with the `subset` and `keep` arguments

## 5. Dates
Convert `booking_date` to real dates, storing the result in a variable called `parsed` first, and look at the rows that failed. Then save `parsed` into the column, and flag any booking with a missing date or a date outside September-October 2026. Finally, add a `month` column.

You'll need: `pd.to_datetime()` with `format="mixed"` and `errors="coerce"`, `.isna()`, `|`, date comparisons against strings like `"2026-09-01"`, `.loc[]` to assign, the `.dt` accessor with `.month_name()`, `.value_counts()` with `dropna=False`

## 6. Totals and facility names
Create `booking_total` (hours × hourly rate), and set it to 0 for bookings that weren't completed. Then load `facilities.csv` and left-merge it onto `both` using `facility_id`, with the indicator column, and count how many rows matched. You fixed `F7` in Part A. Why doesn't it match now?

You'll need: column arithmetic, `!=`, `.loc[]`, `pd.read_csv()`, `.merge()` with `on`, `how` and `indicator`, `.value_counts()`

## 7. Questions
Use only the completed bookings that aren't flagged for review, and save them in a variable called `good`.

**Q1.** Which facility made the most money in October?

You'll need: boolean indexing on `month`, `.groupby()`, `.agg()` with a list of function names, `.sort_values()` with `ascending=False`

**Q2.** Did revenue go up or down from September to October, and by how much? Before you blame October, though: is the October export a full month? How would you check?

You'll need: `.groupby()`, `.agg()` or `.sum()`, `.max()` on the October dates

**Q3 (stretch).** Which activity had the most no-shows across both months? Is "most no-shows" the same thing as "worst no-show rate"?

You'll need: boolean indexing, `.groupby()`, `.count()`, `.sort_values()`

## Wrap-up
Answer the last cell in the notebook: the hardest problem to find, and which check found it.

---

## Stuck?

| You see | It usually means |
|---|---|
| `FileNotFoundError` | The CSV isn't in the same folder as the notebook |
| `KeyError: 'facility_id'` | You haven't renamed the October columns yet (or you're using the old name) |
| `ValueError` from `pd.to_numeric` on hours | There's still text in the column. Look at what didn't convert in 2.5 |
| `Columns match? False` | A rename was missed, or `hours_imputed` / `needs_review` wasn't created |
| `both` has extra half-empty columns | You stacked before renaming. Column names must match exactly |
| `NameError` in Part B | You restarted the kernel but didn't re-run the imports cell |
| "It ran, but nothing changed" | You didn't assign the result back |
| `ValueError: The truth value of a Series is ambiguous` | You used `and` / `or` instead of `&` / `\|` |

---

## Answer key

Spoilers below: this lists every problem in the October file. Try each step before you look. If your answer is different, work out *why* before changing anything. Sometimes you'll find a mistake; sometimes you'll find you made a different call you can defend.

### Part A

| Step | What you should see |
|---|---|
| 1.1 | `(117, 8)` |
| 1.2 | Three names don't match: October has `facility`, `date` and `rate ($/hr)` where September has `facility_id`, `booking_date` and `hourly_rate`. September also has `hours_imputed` and `needs_review`, which were added when it was cleaned |
| 2.1 | `member_id` 1, `activity` 1, `hours` 3, `rate ($/hr)` 1 |
| 2.2 | 4 and 4: every repeated ID is an exact duplicate. If the ID count were higher, you'd have the same booking recorded with different details, and you'd have to decide which one is right |
| 2.3 | `activity` has 20 spellings (some with trailing spaces) plus one missing value, for 5 real activities. `status` has 10 spellings of 3. `facility` has `f03` and **`F7`**: no amount of uppercasing adds the missing zero |
| 2.4 | `count` is 109 but 114 hours aren't blank, so 5 didn't convert. Min is **0** (booking 5168): a booking with no time is impossible |
| 2.5 | Everything except `booking_id` is `str`. The 5 hours that didn't convert are recorded in **minutes**: `60 min`, `90 min`, `120 min` (bookings 5147, 5157, 5158, 5227, 5239). Strip ` min`, convert, divide by 60. The rates have `$` signs and a suspicious `4.50` |
| 2.6 | The histogram looks normal: `4.50` sits right next to the $6 bar. The value counts give it away: every rate appears 15+ times except one `4.5`. It's booking 5185, a court rental, so $45 with a slipped decimal. No single check catches everything |
| Findings | Renamed headers; 4 duplicates; spelling variants (activity, status); bad facility keys (`f03`, `F7`); missing activity, hours, rate and member; zero hours; hours in minutes; `$` in rates; the $4.50 rate; dates stored as text in mixed formats |
| 3.1 | `list(df.columns)` shows `facility_id`, `booking_date`, `hourly_rate` |
| 3.2 | 113 rows, `True` |
| 3.3 | Lane Swim 38, Public Skate 22, Court Rental 20, Gym Drop-In 17, Fitness Class 15, plus 1 missing. completed 91, no-show 13, cancelled 9. Facility IDs F01-F06 plus one F07 |
| 3.4 | Hours max is **3**. If yours is 120, you converted the minutes but didn't divide. If you divided every row, your max will be tiny. Rate min is still 4.5 until 3.6 |
| 3.5 | F04 only ever hosts Court Rental, so the missing activity (booking 5153) is Court Rental: that's a **fix** from another column, not a guess. The rate is a **fix** from the price list. The hours are a **guess** (median 1.5, 3 rows). `member_id`: **leave it**. The booking happened and the money is real; you just can't count it per member. Afterwards only `member_id` has a missing value |
| 3.6 | 1 row flagged (5168). Hours min 1, max 3; rate min 6, max 45 |
| 3.7 | `True`, `(113, 10)` |
| Merge or concat | **Concat** stacks September and October (adds rows; needs the same column names). **Merge** adds the facility names (adds columns; needs a shared key) |

### Part B

| Step | What you should see |
|---|---|
| 4.1 | `(253, 11)` |
| 4.2 | 3 repeated IDs: **5130, 5135, 5139**. They're September bookings that also turned up in the October export. Inside October they were unique, so you could only catch them once the files were combined. Keep the September copy (`keep="first"`, since September was stacked first): 250 rows, all unique |
| 5 | Two rows fail: 5105 has no date at all (it was blanked when September was cleaned) and **5207 is `2026-10-32`**. The range check also catches 2027-09-14 (5121). 6 flagged in total. Months: September 139, October 109, missing 2 |
| 6 | 246 `both`, 4 `left_only`. All 4 are F07: fixing `F7` made it *consistent*, but F07 isn't in the facilities table, so it still can't match |
| Q1 | **Strathcona Sports Hall, $1,237.50** from 14 bookings. Next is Oliver Fitness Hub, $341.00 from 20. Same story as September: court rentals at $45/hour |
| Q2 | September **$3,028.50** (118 bookings), October **$2,160.50** (86). Down $868, because there were fewer completed bookings, not cheaper ones. The October dates run to October 31, so it is a full month |
| Q3 | Lane Swim 6, Court Rental 5, Public Skate 5, Gym Drop-In 4, Fitness Class 3. But Lane Swim also has the most bookings, so a no-show *rate* (no-shows ÷ bookings) could tell a different story |
