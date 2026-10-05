# Exercise: Community Population Summary

**Time:** About 1 hour
**You'll need:** VS Code with the Python and Jupyter extensions, and the file `community_populations.csv`

---

## The Scenario

It's your first week as a junior data analyst at a regional planning office. Your manager, Dana, sends you this message:

> Hi! Before Thursday's planning meeting, can you put together a quick summary of the populations of the 25 communities in our region? I'd like to know the typical community size, how spread out the numbers are, and whether any communities are unusually large or small. Please send me a notebook with your code and a short note explaining what you found. Thanks!

Your job is to build that notebook.

---

## Part 1: Set Up Your Notebook (10 min)

1. Create a new folder for this exercise and put `community_populations.csv` inside it.
2. Open the folder in VS Code (**File → Open Folder**).
3. Create a new Jupyter notebook and save it in the same folder as `firstname_lastname_population_summary.ipynb`.
4. Select a Python kernel.
5. Add a **Markdown cell** at the top with:
   - A title (use `#` for a heading)
   - Your name
   - Today's date
   - One sentence describing the purpose of this notebook
6. Add a **code cell** that imports pandas, numpy, and matplotlib. Run it.

✅ **Check:** The cell runs with no errors and a number appears beside it.

---

## Part 2: Load and Explore the Data (10 min)

Add a Markdown heading for this section, then use a separate code cell for each step.

1. Load `community_populations.csv` into a DataFrame called `df`.
2. Display the first 5 rows.
3. Find how many rows and columns the dataset has.
4. Check the data types and look for missing values.
5. List the column names.
6. Select just the `Population` column.

💡 **Hints:** `pd.read_csv()`, `.head()`, `.shape`, `.info()`, `.columns`

📝 **In a Markdown cell, answer:** How many communities are in the dataset? Are there any missing values?

---

## Part 3: Summary Statistics (20 min)

Add a Markdown heading for this section. For each statistic, add a code cell with a short comment explaining what it calculates.

1. Mean
2. Median
3. Mode
4. Minimum
5. Maximum
6. Range
7. Variance
8. Standard deviation
9. First quartile (Q1) and third quartile (Q3)
10. Interquartile range (IQR)
11. Use `.describe()` to check your answers

💡 **Hints:** `.mean()`, `.median()`, `.mode()`, `.min()`, `.max()`, `.var()`, `.std()`, `.quantile()`, `.describe()`

📝 **In a Markdown cell, answer:**
- What is a typical community size? Would you report the mean or the median to Dana? Why?
- Do your results match the output of `.describe()`?

---

## Part 4: Z-scores and Outliers (10 min)

Add a Markdown heading for this section.

1. Calculate the Z-score of the first community in the dataset.
2. Find the row with the largest population and calculate its Z-score.
3. Calculate the lower and upper outlier boundaries using the IQR method:
   - Lower bound = Q1 − 1.5 × IQR
   - Upper bound = Q3 + 1.5 × IQR
4. Filter the DataFrame to show only the outlier rows.

💡 **Hints:**
- Z = (value − mean) / standard deviation
- Use `.iloc[]` to pick a row by its position
- Use `|` to mean "or," and put each condition in its own parentheses

📝 **In a Markdown cell, answer:** Which communities are outliers? Are they unusually large or unusually small?

---

## Part 5: Write Your Note to Dana (5 min)

At the bottom of your notebook, add a Markdown cell titled **Summary for Dana**. In 3 to 5 sentences, explain:

- The typical community size
- How spread out the populations are
- Which communities are outliers

Write it for someone who doesn't code. Use plain language, not variable names.

---

## Part 6: Final Check (5 min)

1. Click **Restart**, then **Run All**.
2. Make sure every cell runs from top to bottom with no errors.
3. Save your notebook.

---

## ⭐ Bonus: Add a Chart

If you finish early:

1. Create a histogram of the `Population` column.
2. Add a Markdown cell describing what the chart shows. Can you spot the outliers?

💡 **Hints:** `df["Population"].plot(kind="hist")` and `plt.show()`

**Extra challenge:** Look up how to add a title and axis labels to your chart.

---

## Submission Checklist

- [ ] Notebook is saved as `firstname_lastname_population_summary.ipynb`
- [ ] Title cell includes your name, the date, and the purpose
- [ ] Each part has a Markdown heading
- [ ] Code cells have short comments
- [ ] All the 📝 questions are answered in Markdown cells
- [ ] Summary for Dana is complete
- [ ] Notebook runs from top to bottom after **Restart** and **Run All**
- [ ] (Bonus) Histogram with a description
