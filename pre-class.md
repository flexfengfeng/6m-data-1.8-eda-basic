# 📚 Pre-Class: Fundamentals of Exploratory Data Analysis (EDA)

⏱️ **Estimated Time:** 35–40 minutes
**Prerequisites:** Lesson 1.7 — Introduction to Pandas

> In Lesson 1.7 you learned to load and manipulate data with Pandas. This lesson covers what you do
> *first* when a file lands on your desk: investigating it, spotting problems, and cleaning it before
> any analysis begins. This is called Exploratory Data Analysis (EDA).

🎯 **Goal:** understand the core ideas of investigating and cleaning data, so the code in class is
about *decisions* rather than syntax.

🎬 Watch this video: [[EDA Basic: The Data Health Check]](https://youtu.be/PlOwlaqjLBY?si=a_G-OlKDWTwQdUX0)

---

## **0. The problem you will be working on (3 minutes)**

> **The Daily Grind** is a four-outlet café chain in Singapore. Revenue has been flat for two
> quarters, and the owner has to decide whether to renew the Marina Bay lease. She asks her assistant
> to send over the sales data.
>
> What arrives is a **raw till export**: one row per outlet, per day, per part of the day, straight
> out of the point-of-sale system. Nobody has looked at it. Your job is to make it trustworthy.

This is the first of three lessons on the same problem:

| Lesson | The question | What you do |
|---|---|---|
| **1.8 — this one** | **Can I trust this data?** | clean one month of the raw export |
| 1.9 | What is the pattern? | 18 months, cleaned: time, joins, grouping |
| 1.10 | How do I make them act? | one chart, one slide, one decision |

Here is a taste of what is waiting in that file: **twelve different spellings for four cafés**, and a
takings column stored as text like `"S$1,240.50"` — so pandas cannot add it up, and sorts \$98,000
below \$99. The raw file claims June took **\$268,000**. It actually took about **\$175,000**.

---

## **1. What is EDA? (5 minutes)**

**Exploratory Data Analysis (EDA)** is the first step in any data project. Think of it as
"interviewing" your data. Before you can build models or draw charts, you have to know what you are
holding.

**Why do we do it?**

* **To check assumptions:** is the data what you think it is?
* **To spot anomalies:** errors, missing values, impossible values, duplicates.
* **To find patterns:** what are the basic shapes and trends?

**Analogy:** EDA is a doctor's check-up. You take the vitals before diagnosing anything — and if the
thermometer turns out to be broken, you find that out *first*, not after prescribing.

---

## **2. The "health check": inspection and summary (10 minutes)**

When a file first loads, answer three questions.

### **A. How big is it?**

The **dimensions** — rows × columns. *Pandas tool:* `.shape`

### **B. What type is each column?**

* **Numerical:** integers (1, 2, 100) or floats (1.5, 3.14).
* **Categorical / text:** `"Marina Bay"`, `"Morning"`, `"PayNow"`.
* *Pandas tool:* `.info()` shows types and non-null counts together.

**This one matters more than it sounds.** In the café export, the takings column is **text**, because
someone exported it with dollar signs and thousands separators: `"S$1,240.50"`. Every consequence
follows from that:

* `.describe()` ignores the column entirely — you cannot see a mean, a min or a max.
* Sorting is alphabetical, so `"S$94.41"` counts as *bigger* than `" 1,006.71 "`.
* A mis-keyed `98000` sits there invisibly, because there is no numeric range to fall outside of.

Checking types is not clerical work. It is how you find the errors that hide.

### **C. What do the numbers look like?**

**Summary statistics** give you a snapshot without reading every row.

* **Mean:** the average. **Median:** the middle value. **Min/Max:** the range.
* *Pandas tool:* `.describe()` computes these for every numeric column at once.

> **Read the min and max rows first.** That is where impossible values live. In the café file the
> takings run from **-999** to **98,000** — for shifts that normally take a few hundred dollars.

---

## **3. The "cleanup": handling dirty data (10 minutes)**

### **A. Missing data (NaN)**

Sometimes data was not collected, or a system failed. Pandas shows this as `NaN` (Not a Number).

* **Option 1: drop.** Throw the row away (`dropna`). *Risk:* you lose data, and if the missingness is
  not random you introduce bias.
* **Option 2: impute (fill).** Put a reasonable value in (`fillna`) — often the median. *Risk:* you
  have invented a number, and it will be treated as real by everything downstream.

**There is a third option people forget: leave it missing.** For the café export, "no note recorded"
and "no email on file" are honestly blank. Inventing a value there would be worse than the hole.

> **Watch out for fake blanks.** The café's `notes` column uses `N.A.` and `-` to mean empty. Those
> are ordinary *text* to pandas — `.isna()` cannot see them, so they quietly survive every check you
> run. Real-world data is full of these.

### **B. Duplicates**

The same record appears twice — a customer clicked submit twice, or a till re-sent a batch.

* **Why fix it?** It inflates every total built from it.
* **Solution:** `drop_duplicates()`, after deciding *which columns identify a row*.

> **The subtle part:** you cannot reliably de-duplicate on a column you have not cleaned. While
> "Marina Bay" and "marina bay" look like different cafés to pandas, a genuine double-submission
> under two spellings slips through untouched.

### **C. Outliers and impossible values**

An outlier is a value far from the others. The useful question is not "is it extreme?" but
**"is it possible?"**

* **A real extreme:** a genuinely record-breaking Saturday. Keep it — it is the most interesting row
  in the file.
* **A typo:** `98000` where someone meant `980.00`. Not real.
* **A sentinel:** `-999`, which the old till wrote when a shift failed to close off. It does not mean
  "minus nine hundred and ninety-nine dollars", it means "no reading" — and if you leave it in, it
  silently drags down every average you compute.

**Order matters here.** Mask the sentinels *first*, then compute the statistic you are going to fill
with, then fill. In class you will measure what that ordering is worth: with a **median** it is worth
very little (\$456 vs \$463 — a median ignores extremes, which is exactly why we prefer it), but with a
**mean** the same mistake makes every filled value 53% too high. You cannot tell from the outside which
case you are in, so you order the steps properly and never have to. Nothing here raises an error, which
is why this bug reaches real reports.

---

## **4. Refinement: pattern matching with regex (5 minutes)**

Sometimes the mess is inside the text itself: `"S$1,240.50"`, `"  Aisha Rahman "`,
`"daniel.lim@dailygrind.com.sg"`.

* **Regular expressions (regex):** a pattern that describes the *shape* of text rather than its exact
  content. You use it to find, extract or replace.
* **In class you will use it to:**
  * **Clean:** strip everything that is not a digit, dot or minus sign, turning `"S$1,240.50"` into a
    number.
  * **Extract:** pull the user, domain and suffix out of an email address.
  * **Validate:** ask whether a value even looks like an email.

**Analogy:** regex is "Find" on steroids. Instead of searching for the exact word "cat", you can
search for "any three-letter word starting with c and ending with t".

You do not need to memorise the syntax. You need to know that it exists and roughly what it is for.

---

## **5. Preparation checklist (3 minutes)**

1. **Environment:** the `pds` conda environment from Lesson 1.7. If it is not set up, run
   `conda env create -f environment.yml` from the course repository, then `conda activate pds`.
2. **Libraries:** confirm pandas and numpy work — run `import pandas as pd; import numpy as np` in a
   notebook cell. No error means you are ready.
3. **Data:** already in `data/`. Nothing to download.
4. **Mindset:** be ready to look at ugly data and *make decisions about it*. Most of today has no
   single right answer, only defensible ones — and being able to say why you chose is the skill.

---

## **🧠 Quick Self-Check Quiz**

Try these without looking back, then check below.

1. **Scenario:** you have 1,000 shifts of café sales. The takings column has 50 missing values. Would
   you drop those 50 rows or fill them with the average? Why?
2. **True or false:** `.describe()` gives you the same kind of output for a column of café names as
   for a column of takings.
3. **Scenario:** a takings column contains `-999` in four rows. You fill the missing values with the
   column median and then remove the -999s. What went wrong?
4. **Definition:** what is the difference between an integer and a float — and why might a column of
   *counts* show up as a float?
5. **Regex:** you want to turn `"S$1,240.50"` into a number. What kind of pattern would you look for?

<details>
<summary>Suggested answers</summary>

**Q1:** It depends — which is exactly the point. Dropping 50 of 1,000 loses 5% of the month, which
may be acceptable. Filling with an average keeps the row count but invents 50 numbers that everything
downstream will treat as real. Ask *why* they are missing first: if the till failed at random, imputing
the median is defensible; if they are all Sunday evenings at one outlet, imputing a global average
quietly erases the very pattern you were hired to find.

**Q2:** False. On a numeric column you get count, mean, std, min, quartiles and max. On a text column
you get count, unique, top (most frequent) and freq. Completely different outputs — and the text
version is how you discover twelve spellings of four café names.

**Q3:** The statistic was computed *while the -999s were still in the column*, and that contaminated
figure was then written into the genuine holes. Removing the sentinels afterwards does not undo it. The
sequence is: sentinels out → *then* compute the statistic → then fill.

How bad it is depends on the statistic, and this is worth knowing precisely. A **median** is barely
affected (in the café file, \$456 vs \$463) because it looks at the middle of the sorted values. A
**mean** is wrecked (\$742 vs \$486). So the median is the safer default *and* the reason careless code
often gets away with this — until the day the file has thirty sentinels instead of four.

**Q4:** An **integer** is a whole number (5, 100, -3); a **float** has a decimal part (5.0, 3.14).
A column of counts shows up as a float when it contains missing values, because there is no integer
that means "missing" — so pandas promotes the whole column. A count stored as a decimal is therefore a
*clue*: it usually means there are holes in it.

**Q5:** Look for the characters you want to *remove* rather than the ones you want to keep: anything
that is not a digit, a dot or a minus sign. The pattern `[^0-9.\-]` reads "any character that is not
in this set", and replacing every match with nothing leaves `"1240.50"`, which pandas can then convert
to a number.

</details>

---

## **Reference**

- [EDA Process Overview](https://su-ntu-ctp.github.io/6m-data-1.8-eda-basic/)
- [Beginner's Guide to Statistics](https://www.analyticsvidhya.com/blog/2021/08/a-beginners-guide-to-statistics-for-machine-learning/)
- [Introduction to Regular Expressions in Python](https://developers.google.com/edu/python/regular-expressions)
- [Regex Cheatsheet](https://www.dataquest.io/blog/regex-cheatsheet/)
