# Rec Centre Bookings: Exercise Guide
SDEV2501 · Days 10-11 · Goes with `bookings_wrangling_starter.ipynb`

This guide walks you through the notebook one step at a time, and the section numbers match. Each step tells you what to do and lists the functions, methods and operators you'll need. Working out how to put them together is your job. That's where the learning happens, annoyingly.

When you've finished a step, check yourself against the **answer key** at the end. Try first, then look. Reading the answers before you've written any code is like reading the last page of a mystery: technically faster, and you learn nothing.

If your result doesn't match the answer key, stop and fix it before going on. Every step builds on the last one, so a mistake in step 3 turns into five mistakes by step 8, and by then good luck finding the first one.

---

## Before you start

Put the notebook (`bookings_wrangling_starter.ipynb`), `bookings_sept_messy.csv` and `facilities.csv` in **one folder**. Open that folder in VS Code (File > Open Folder), open the notebook, and pick your Python kernel in the top right.

Run the first code cell (the imports). If you get `ModuleNotFoundError`, you've picked the wrong kernel. Fix that before anything else.

---

# Part 1: Inspect & Clean (Day 10)

One rule for today: **look before you touch.** In section 2 you only look. Section 3 is where you finally get to change things.

## 1. Load

**1.1** Load `bookings_sept_messy.csv` into a DataFrame called `df` and show the first 10 rows. Before you move on, count how many things already look wrong in just those 10 rows.

You'll need: `pd.read_csv()`, `.head()` (it takes the number of rows as an argument)

**1.2** Find out how many rows and columns there are, and list the column names. Look closely at the `Hours` column name. What's different about it? What happens if you try to select it with `df["Hours"]`?

You'll need: the `.shape` attribute, the `.columns` attribute, `list()`

## 2. Inspect

Nothing you do in this section should change `df`. Keep a running list of every problem you find. You'll need it for the findings table, and future-you will not remember.

### 2.1 Missing values
Count the missing values in every column. Which columns have them? And is a blank cell the only way a value can be "missing"?

You'll need: `.isna()`, `.sum()` (chain them)

### 2.2 Duplicates
**2.2a** Count the rows that are exact copies of an earlier row. Then show every copy, sorted by `Booking ID` so the pairs sit next to each other.

You'll need: `.duplicated()` (and its `keep` argument, set to `False` to mark every copy), `.sum()`, boolean indexing (`df[mask]`), `.sort_values()`

**2.2b** Now check the `Booking ID` column on its own for repeated IDs. Do you get the same number as 2.2a? Find any extra one. Why didn't 2.2a catch it, and which of its rows do you believe?

You'll need: the same methods as 2.2a, called on a single column (a Series)

### 2.3 Unexpected categories
Count every distinct value in `Activity`, `Status` and `Facility ID`, one cell each. How many activities are there *really*? Is there a facility code that looks out of place?

You'll need: `.value_counts()`

### 2.4 Impossible values
An impossible value can't be true no matter what.

**2.4a** `Hours ` is stored as text, so you can't get statistics from it yet. Make a temporary numeric copy called `hours_check` (leave `df` alone), then look at its summary statistics. Is the minimum possible? Compare the `count` to the number of hours that aren't blank. Do they match? Why not?

You'll need: `pd.to_numeric()` with the `errors` argument set to `"coerce"` (anything that won't convert becomes `NaN`), `.describe()`

**2.4b** Show the rows where `hours_check` is zero or less.

You'll need: a comparison operator to build a boolean mask, boolean indexing

### 2.5 Wrong data types
**2.5a** Show the data type of every column. Which ones *should* be numbers or dates?

You'll need: `.info()`

**2.5b** Find the hours that have a value in `df` but are missing in `hours_check`. Those are the values that didn't convert. Then list every distinct value in `Hourly Rate`. What's stopping each of these columns from being numbers?

You'll need: `.isna()`, `.notna()`, the `&` operator to combine two masks, `.unique()`

### 2.6 Suspicious distributions
A suspicious value is possible, but odd enough that someone should check it.

**2.6a** Plot a histogram of `hours_check`.

You'll need: `.plot()` with `kind="hist"` and a `bins` argument, `plt.show()`

**2.6b** Make `rate_check`, a numeric copy of `Hourly Rate`, and plot its histogram too. You'll have to deal with the `$` first. Which bars sit far away from the rest? Are those values *impossible*, or just *suspicious*?

You'll need: `.str.replace()` (pass `regex=False` so `$` is treated as a plain character), `pd.to_numeric()`, `.plot()`

### Findings
Fill in the findings table in the notebook as you go (it's a markdown cell, so double-click to edit it). Write one row per problem: what it is, which column it's in, and which step found it.

## 3. Clean

Now you get to change things. For every fix, ask: **do I know the right answer, or am I guessing?** If you know, fix it. If you'd be guessing, flag it or leave it.

> **Assign it back.** Most pandas methods return a *new* object and leave `df` exactly as it was. If you don't assign the result back (`df = ...` or `df["col"] = ...`), nothing changes, and pandas won't tell you. This will be the bug you hit most today.

### 3.1 Rename columns
Clean every column name in one go: strip the spaces, make them lowercase, and swap spaces for underscores. Then rename `member_#` to `member_id`.

You'll need: the `.str` accessor on `df.columns` with `.strip()`, `.lower()` and `.replace()` chained together, `.rename()` with its `columns` argument (it takes a dictionary of `{old: new}`)

### 3.2 Remove duplicates
**3.2a** Drop the exact duplicates and print how many rows were removed.

You'll need: `len()` before and after, `.drop_duplicates()`

**3.2b** Remove the 20-hour copy of booking 5071, then check that every `booking_id` is unique. Careful: `hours` is still text at this point, so what type does the 20 need to be in your comparison?

You'll need: two comparisons combined with `&`, the `~` operator to invert a mask (keep everything *except* that row), the `.is_unique` attribute

### 3.3 Fix categories
**3.3a** Clean `activity`: strip the spaces at each end, turn any run of spaces into a single space, and put it in Title Case. How many values are left?

You'll need: `.str.strip()`, `.str.replace()` with a regular expression (`r"\s+"` means "one or more whitespace characters", so pass `regex=True`), `.str.title()`

**3.3b** Some variants need a human decision. Turn `Swim`, `Skate` and `Gym Drop In` into their proper names.

You'll need: a dictionary, `.replace()` (pass it the dictionary)

**3.3c** Clean `status` the long way first: loop over the column, strip and lowercase each value, and collect them in a list. Show the first 8.

You'll need: a `for` loop, an empty list, the string methods `.strip()` and `.lower()`, `.append()`

**3.3d** Now do the same to the column the vectorized way (one instruction for the whole column), then merge the spelling variants. Which version would you rather write for a table with a million rows? (Trick question. You'd rather not write the loop for ten.)

You'll need: `.str.strip()`, `.str.lower()`, a dictionary, `.replace()`

**3.3e** Make every `facility_id` uppercase.

You'll need: `.str.upper()`

### 3.4 Fix data types
Convert `hours` to numbers. Then remove the `$` from `hourly_rate` and convert that too. Check the types. Why is it safe to coerce `hours` now, when it wasn't safe to do it blindly before section 2?

You'll need: `pd.to_numeric()`, `.str.replace()`, the `.dtypes` attribute

### 3.5 Missing values
**3.5a** Every activity has a standard hourly rate. Put the rates in a dictionary, then fill *only* the missing rates from it. (Not sure what the rates are? Group the rates by activity and look at the unique values.)

You'll need: a dictionary, `.isna()` to build a mask, `.loc[rows, column]` to select and assign, `.map()` (pass it the dictionary), `.groupby()` and `.unique()` if you need to look up the rates

**3.5b** We don't know how long the missing bookings were, so we'll guess, and label the guess. Add a True/False column called `hours_filled` that marks the missing hours, *then* fill them with the median. Why can we *fix* the rate but only *guess* the hours?

You'll need: `.isna()`, `.median()`, `.fillna()`

### 3.6 Impossible and suspicious values
Before you write any code, decide: fix, flag or leave the -2 hour booking, the 24-hour booking and the $450 rate? Write one line of reasoning for each in the notebook. If you're working with a partner, see if you agree.

Then add a True/False `needs_review` column for hours that are 0 or less, or over 12. Set the hours that are 0 or less to missing, so they don't count in totals, and fix the $450 court rate. Check your work with summary statistics.

You'll need: comparison operators, the `|` operator ("or"), `.loc[]` to assign, `np.nan`, `.describe()`

### 3.7 Save
Save `df` to `bookings_cleaned.csv` without the index, and print its shape. You need this file next class, so make sure it's actually in your folder.

You'll need: `.to_csv()` with `index=False`, `.shape`

---

# Part 2: Transform & Reshape (Day 11)

Fresh start: restart the kernel, run the imports cell at the top, then jump to Part 2.

**Reload.** Load `bookings_cleaned.csv` into `df` and check the column types. Look at `booking_date`. What type is it now, and why didn't the CSV remember?

You'll need: `pd.read_csv()`, `.info()`

## 4. Select
**4.1** Select the `activity` column on its own. Then select `booking_id`, `activity` and `hours` together. Show the first 5 rows of each. What type of object does each one give you?

You'll need: single square brackets with a column name, double square brackets with a list of names, `.head()`, `type()` if you're curious

**4.2** Get rows 0 to 4 of just `booking_id` and `activity`. How many rows do you get, and why?

You'll need: `.loc[]` with a row slice and a list of columns

## 5. Filter
**5.1** Show only the court rentals.

You'll need: `==` to build a boolean mask, boolean indexing

**5.2** Three more filters: show the completed court rentals (two conditions), count the bookings that are Lane Swim **or** Public Skate, and count the bookings that are **not** flagged for review.

Every condition needs its own brackets, and Python's `and`/`or` won't work on a Series. Pandas needs the operators below.

You'll need: `&`, `.isin()` (pass it a list), `~`, `len()`

## 6. Create and modify columns
**6.1** Create `booking_total`, the cost of each booking (hours × hourly rate), and a True/False `is_long` column for bookings over 2 hours.

You'll need: column arithmetic (assigning to a new column name creates it), a comparison operator

**6.2** Cancelled bookings and no-shows didn't earn the City a cent. Set their `booking_total` to 0, then total `booking_total` by status to check it worked. Is "no-shows don't pay" a fair assumption? Who would you ask?

You'll need: `!=`, `.loc[mask, column]` to assign, `.groupby()`, `.sum()`

## 7. Dates and data types
**7.1** Convert `booking_date` to real dates. Store the result in a variable called `parsed` first (not the column), and show the rows that didn't convert. Why didn't they?

You'll need: `pd.to_datetime()` with `format="mixed"` and `errors="coerce"`, `.isna()`, boolean indexing

**7.2** Now save `parsed` into the column and print the earliest and latest date. Then flag (`needs_review`) any booking with a missing date or a date after September 30, 2026. The latest date is odd: should you fix it or flag it?

You'll need: `.min()`, `.max()`, `.isna()`, `|`, a date comparison (you can compare a date column to a string like `"2026-09-30"`), `.loc[]` to assign

**7.3** Add a `day_of_week` column from the dates. Then convert `activity`, `status` and `facility_id` to the `category` type, and check the types.

You'll need: the `.dt` accessor and `.day_name()`, a `for` loop over a list of column names, `.astype()`, `.info()`

## 8. Normalize
Scale `booking_total` two ways and compare them. Save the min-max version as `total_minmax` and the z-score version as `total_z`, then look at both with summary statistics. The median of `total_minmax` is tiny. Which booking is squashing everything else?

- Min-max: `(x - min) / (max - min)`
- Z-score: `(x - mean) / standard deviation`

You'll need: `.min()`, `.max()`, `.mean()`, `.std()`, column arithmetic, `.describe()`

## 9. Merge
**9.1** Load `facilities.csv` into a DataFrame called `facilities`. Left-merge it onto `df` using `facility_id`, and add the indicator column so you can see which rows matched. Save the result as `merged` and count how many rows matched.

You'll need: `pd.read_csv()`, `.merge()` with the `on`, `how` and `indicator` arguments, `.value_counts()` on the `_merge` column

**9.2** Show the bookings that didn't match a facility. You've seen this facility code before. Where? Should these bookings stay?

You'll need: boolean indexing on `_merge`, a list of columns to keep the output readable

## 10. Group and summarize
**10.1** Keep only the completed bookings that aren't flagged for review, and call that `good`. Then show the number of bookings and the total `booking_total` for each facility, biggest total first. Does the winner have the most bookings? If not, why does it make the most money?

You'll need: `&`, `~`, `.groupby()`, `.agg()` (pass it a list of function names, like `["count", "sum"]`), `.sort_values()` with `ascending=False`

**10.2** Count the completed bookings for each day of the week, busiest first.

You'll need: `.groupby()`, `.count()`, `.sort_values()`

**Try it:** go back to 10.1 and remove the "not flagged" condition. How much does the top facility change, and which booking is responsible?

## Wrap-up
Answer the two questions in the last cell of the notebook: two reasons wrangling has to happen *before* analysis, each backed by something you saw today.

---

## Stuck?

| You see | It usually means |
|---|---|
| `FileNotFoundError` | The CSV isn't in the same folder as the notebook |
| `KeyError: 'Hours'` | The column name has a space in it (`'Hours '`), or you renamed it already |
| "It ran, but nothing changed" | You didn't assign the result back to `df` or the column |
| `ValueError: The truth value of a Series is ambiguous` | You used `and` / `or` instead of `&` / `\|` |
| `TypeError` when combining conditions | Missing brackets around each condition |
| `TypeError: Cannot perform reduction 'mean' with string dtype` | That column is still text. Convert it first |
| Numbers look wrong after a few steps | Restart the kernel and run every cell from the top, in order |

---

## Answer key

Try each step before you look. If your answer is different, work out *why* before you change anything. Sometimes you'll find a mistake, and sometimes you'll find you made a different (defensible) call.

### Part 1

| Step | What you should see |
|---|---|
| 1.1 | A table with columns like `Booking ID`, `Member #`, `Activity`. Already visible: `Lane  Swim` with two spaces, `LANE SWIM`, rates with and without `$`, three different date formats |
| 1.2 | `(146, 8)`. `'Hours '` has a trailing space, so `df["Hours"]` raises a `KeyError` |
| 2.1 | `Hours ` 4 missing, `Hourly Rate` 2 missing. And no, blanks aren't the only kind of missing: there's an `"unknown"` hiding in `Hours ` (you'll find it in 2.5) |
| 2.2a | 5 exact duplicates |
| 2.2b | 6 repeated IDs. The extra one is **booking 5071**: one row says 1.5 hours of lane swim, the other says 20. The rows aren't identical, so a whole-row check misses it. The 1.5-hour row is the believable one |
| 2.3 | `Activity` has 20 spellings of 5 real activities. `Status` has 9 spellings of 3. `Facility ID` has lowercase `f01`-`f03`, and `F07`, which isn't one of the six facilities |
| 2.4a | `count` 141, min **-2**, max 24. There are 142 non-blank hours, so one value didn't convert |
| 2.4b | One row: booking 5034 at -2 hours |
| 2.5a | Everything except `Booking ID` is `str`. `Hours ` and `Hourly Rate` should be numbers, `Booking Date` should be a date |
| 2.5b | One row: booking 5041, hours `"unknown"`. The rates are `str` because some have a `$`, and there's a `450.00` in the list |
| 2.6 | Hours: lonely bars at 20 (the 5071 copy) and 24 (a court rental). Rate: one bar out at 450, when court rentals are normally $45. The -2 hours is *impossible*; 24 hours and $450 are *suspicious* |
| Findings | At least: messy column names; missing hours and rates; 5 exact duplicates; booking 5071; spelling variants in activity, status and facility ID; negative hours; numbers stored as text (`"unknown"`, `$`); dates stored as text in mixed formats; the 24-hour booking and $450 rate |
| 3.1 | `['booking_id', 'member_id', 'facility_id', 'activity', 'booking_date', 'hours', 'hourly_rate', 'status']` |
| 3.2a | 5 removed, 141 left |
| 3.2b | 140 rows, `True`. The 20 has to be the string `"20"` |
| 3.3a | 8 values |
| 3.3b | Lane Swim 50, Court Rental 25, Gym Drop-In 25, Public Skate 25, Fitness Class 15 |
| 3.3d | completed 121, no-show 10, cancelled 9 |
| 3.3e | F01-F06, plus F07 (3 rows) |
| 3.4 | `hours` and `hourly_rate` are both `float64`. Coercing is safe now because we already looked in 2.5: the only value it throws away is `"unknown"` |
| 3.5a | Standard rates: Lane Swim $8, Public Skate $6, Gym Drop-In $10, Fitness Class $12, Court Rental $45. 0 missing rates left |
| 3.5b | 5 rows filled with 1.5. The rate comes from a price list, so we *know* it. Nobody wrote down how long those bookings were, so the hours are a guess |
| 3.6 | Suggested calls: -2 hours, flag and set to missing; 24 hours, flag (could be a tournament); $450, fix to $45 (the rate table proves it). After `.describe()`: hours min 1, max 24; rate min 6, max 45. 2 rows flagged |
| 3.7 | `(140, 10)` |

### Part 2

| Step | What you should see |
|---|---|
| Reload | 140 rows. `booking_date` is `str` again: a CSV only stores text, so the types get guessed fresh on every load |
| 4.1 | Single brackets give a Series (one column). Double brackets give a DataFrame |
| 4.2 | 5 rows. `.loc` slices by label and *includes* the end |
| 5.2 | 22 completed court rentals, 75 Lane Swim or Public Skate, 138 not flagged |
| 6.1 | The first booking is 2 hours at $8, so its total is 16 |
| 6.2 | cancelled 0, no-show 0, completed 4,161.50. Whether no-shows pay is a business rule, so you'd ask the rec department, not the data |
| 7.1 | One row: booking 5105, `2026-09-31`. September has 30 days |
| 7.2 | Earliest 2026-09-01, latest **2027-09-14** (booking 5121). Flag it, don't fix it: changing it to 2026 would be a guess. 4 bookings flagged in total |
| 7.3 | `booking_date` is `datetime64`; `activity`, `status` and `facility_id` are `category` |
| 8 | `total_minmax` runs from 0 to 1 with a tiny median, because the flagged 24-hour booking ($1,080) sets the max. `total_z` has a mean of about 0 and a standard deviation of 1 |
| 9.1 | 137 `both`, 3 `left_only` |
| 9.2 | All three are **F07**, the code from 2.3 that isn't in the facilities table. Keep them (a left merge does that for you) and tell whoever maintains the facilities list |
| 10.1 | **Strathcona Sports Hall, $1,777.50** from 20 bookings. Next is Oliver Fitness Hub, $342.00 from 19. Strathcona doesn't have the most bookings; it wins on price, because court rentals cost $45/hour versus $6-12 for everything else |
| 10.2 | Tuesday 24, Wednesday 20, Friday 20, Thursday 16, Sunday 14, Saturday 13, Monday 11 |
| Try it | Strathcona jumps by over $1,000, almost all from the one flagged 24-hour booking |
