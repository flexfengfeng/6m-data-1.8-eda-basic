# 📚 Lesson 1.8: EDA Basic — Exploratory Data Analysis

## Session Overview

| | |
|---|---|
| **Duration** | 150 minutes (including 2 × 10-min breaks) |
| **Format** | Flipped Classroom + Guided Coding in Jupyter |
| **Tools** | VS Code + `pds` conda environment |
| **Notebooks** | `notebooks/Part_1_descriptive_statistics.ipynb` → `Part_4_reading_writing_data.ipynb` (4 parts, see below) |
| **Dataset** | `data/cafe_june_raw.csv` — one messy till export, used from start to finish |

## Agenda

| Time | Part | Topic |
|------|------|-------|
| 0:00 – 0:05 | Setup | Imports, load `cafe_june_raw.csv`, the "why this matters" hook |
| 0:05 – 0:38 | Part 1 | Descriptive Statistics — the 5-move first look, then `.describe()` unpacked *(incl. 8-min Group Exercise 1)* |
| 0:38 – 0:48 | ☕ | **Break** |
| 0:48 – 1:30 | Part 2 | Data Quality — the type fix, missing values, duplicates, impossible values *(incl. 12-min Group Exercise 2)* |
| 1:30 – 1:40 | ☕ | **Break** |
| 1:40 – 2:15 | Part 3 | Transformation — mapping, labels, strings/regex, categories, dates, grouping *(incl. 10-min Group Exercise 3)* |
| 2:15 – 2:30 | Part 4 | Reading & writing — CSV, JSON, Excel, databases |

**The notebook follows the four learning outcomes in order.** Reading files is deliberately *last*:
nothing in Parts 1–3 depends on file I/O, so learners meet cleaning before parsing.

---

## 🎬 The business problem (do not skip this)

> **The Daily Grind**, a four-outlet café chain. Revenue has been flat for two quarters and the
> Marina Bay lease is up for renewal. The owner asks for the sales data; what arrives is the **raw
> till export** — one row per outlet, per day, per daypart, untouched.

The hook is three cells and takes two minutes:

1. `raw["outlet"].value_counts()` → **twelve** spellings for four cafés.
2. `raw["daypart"].value_counts()` → **nine** labels for three dayparts.
3. `raw["revenue_raw"].dropna().sort_values().iloc[[0, -1]]` → the "smallest" value is
   `" 1,006.71 "` and the "largest" is `"S$94.41"`, because the money column is **text** and is being
   compared alphabetically.

> **Land the point:** any average, chart or model built on this file is wrong before you start — and
> none of it would *look* wrong. It would produce numbers, with decimal places, and nobody in the
> meeting would know. Part 1 is the two-minute routine that finds problems like these.

Ask the class to write down the three findings. The notebook ticks them off.

---

## 🧭 The four beats

Every fix in Parts 2 and 3 follows the same order:

**find it → decide → apply → verify**

Say it out loud at each fix. By Part 3 the class should be saying it for you. Lesson 1.9 builds its
own four beats for *summarising* on top of this one, so the habit transfers.

### Instructor notes

- **Do not skip the hook.** Twelve cafés, nine dayparts, and money that sorts alphabetically. Two
  minutes, and it pays for the rest of the session.
- **Each section is spine-then-drills.** The spine block works on `raw` / `clean` and carries the
  narrative; the small hand-built tables that follow are mechanics. If you are running late, cut
  drills — never the spine.
- **Section 2.0 is new and load-bearing.** You cannot check whether takings are plausible while they
  are text, so the money column is converted first, with one regex line and a forward reference to
  3.3. The instant `.describe()` works, the -999s and the \$98,000 appear. That is the payoff for
  move 4 of the first look (`.dtypes`).
- **The decision table in 2.1 is the teaching moment:** `notes` stays `NaN`, `manager_email` stays
  `NaN`, `staff_on_shift` and `items` get the median, and `revenue`/`tickets` must **wait**. Six
  columns, four different right answers.
- **The order-matters cell is more honest than the usual version of this lecture.** It prints the
  median and the mean, before and after masking. The median barely moves (\$456 → \$463) — which is
  *why* a median is the safe default. The mean moves \$742 → \$486, so the same mistake with a mean
  would have made all eight filled shifts 53% too high. The rule to give them: **mask first, then
  compute, because you cannot tell from the outside which case you are in.**
- **The fake blanks in `notes`** (`N.A.`, `-`) are worth a minute: `.isna()` cannot see them. Ask who
  has seen a spreadsheet where "empty" was typed as a word.
- **The duplicate check in 2.2 deliberately returns a reassuring 0** and the notebook explains why
  that is not reassurance: while "Marina Bay" and "marina bay" look like different outlets, a genuine
  double-submission would slip straight through. You cannot de-duplicate on keys you have not
  standardised. The check is re-run at the end of 3.1, where it means something.
- **The payoff is section 3.5** — the four-row `groupby` summary, shown beside the same summary on the
  raw data (twelve rows, one of them \$107k).
- **Breaks are load-bearing**, not padding. Both break cells state where the class is and what is next.
- Deep dives (categorical internals, `.cat`, awkward CSV parsing, pickle) live in `reference.md`.
  Point learners there rather than teaching them live.
- Group exercises fade: (a) is worked or blank-filling, later parts are from scratch, (d) is
  explain-only. Expected outputs are stated, so groups self-check without waiting for you.

---

## 🎯 Learning Objectives

By the end of this session, you will be able to:

1. Summarise a dataset using descriptive statistics and identify its shape, data types, and distributions.
2. Handle missing values, duplicates, and outliers using appropriate Pandas methods.
3. Transform data through type conversion, string cleaning, and categorical encoding.
4. Read and write data across multiple file formats (CSV, JSON, Excel, databases).

---

## Before You Start

**Have you completed the pre-class reading?**
- ✓ Understand what EDA is and why it matters
- ✓ Review basic statistics concepts (mean, median, standard deviation)
- ✓ Familiar with regex basics
- ✓ `pds` conda environment is set up

Open `notebooks/Part_1_descriptive_statistics.ipynb` in VS Code, then select the `pds` conda
environment as the kernel. Each part notebook ends with a pointer to the next one
(`Part_2_data_quality.ipynb`, `Part_3_data_transformation.ipynb`, `Part_4_reading_writing_data.ipynb`).

---

## 🏃 Part 1: Descriptive Statistics (33 min)

Open the notebook and follow along with **Part 1** — understanding what you have before you analyse it.

**1.1 — the five-move first look**, in this order, every time:

| Move | Question it answers |
|---|---|
| `.head()` | What do the rows actually look like? |
| `.shape` | How big is it? |
| `.info()` | What type is each column, and where are the holes? |
| `.dtypes` | Is anything stored as the wrong type? |
| `.describe()` | Are the numbers plausible? |

Then `.describe(include="object")` for the text columns — that is where spelling variants show up.

> **Two things to point at in the output.** First, `revenue_raw` is **missing** from `.describe()`
> entirely, because it is text — a column you cannot summarise is a column you cannot trust.
> Second, `tickets` is a *decimal*. A count stored as a float is a symptom: it means the column has
> holes, because there is no integer that means "missing".

**1.2a — the same statistics unpacked** on eight rows you can add up by hand: reductions
(`.sum()`, `.mean()`), `skipna`, `.idxmax()` / `.idxmin()`, `.cumsum()`, then `.describe()` on text,
`.unique()`, `.value_counts()`.

**1.2b — the `axis` sandbox**, on a deliberately meaningless table, because on real data one of the
two directions is nonsense (`tickets + staff_on_shift` is not a quantity).

> The first-look output *is* the Part 2 to-do list. Have the class write down the five problems before
> the break.

---

## 🏃 Part 2: Data Quality (42 min)

Continue in the notebook with **Part 2**. Every fix follows the four beats:
**find it → decide → apply → verify.**

**2.0 — one type fix first.** `to_numeric(clean["revenue_raw"].str.replace(r"[^0-9.\-]", "", regex=True))`.
One line, and now `.describe()` shows a minimum of **-999** and a maximum of **98,000** in a business
whose busiest shift takes about \$1,000.

**Key topics:**
- Missing values — `isna()`, the fake blanks in `notes`, `dropna()` (`how`, `thresh`, `axis`),
  `fillna()` (scalar, per-column dictionary, `bfill`, median)
- Duplicates — `duplicated()`, `drop_duplicates()` (`subset`, `keep`), and why the subset check is
  meaningless until Part 3
- Impossible values — boolean filtering, `.mask()`, sentinel codes, capping vs marking missing

> **The comparison cell at the end of 2.3 is the one to dwell on:**
> raw June = **\$267,987**, cleaned June = **\$174,753**, difference **\$93,234**.
> One mis-keyed shift did most of it, six duplicated rows added more, four sentinels pulled the other
> way. A report built on the raw file would have told the owner she'd had her best month ever.

---

## 🏃 Part 3: Data Transformation (35 min)

Continue in the notebook with **Part 3**.

**Key topics:**
- 3.1 Mapping — `.map()` with a dictionary, `.replace()`; twelve outlet spellings → four cafés, nine
  daypart labels → three dayparts, then the duplicate check re-run
- 3.2 Renaming axis labels — `.rename()`, `.index.map()`
- 3.3 Strings & regex — `.str.strip()`, `.str.title()`, `.str.split()`; regex built up in three steps
  on the managers' email addresses
- 3.4 Categorical encoding — `astype('category')`, `pd.cut()` shift-size bands, `pd.get_dummies()`
- 3.5 Type conversion & grouping — `pd.to_datetime(..., dayfirst=True)`, `.dt`, `groupby().agg()`

> `.map()` silently turns unlisted values into NaN; `.replace()` leaves them alone. Make the class say
> which behaviour they want before they pick — then show the `.isna().sum()` check that catches a
> thirteenth spelling.
>
> **The regex step 3 cell has a deliberate trap:** Daniel's address is `@dailygrind.com.sg`, so the
> pattern puts `dailygrind.com` in the domain group and `sg` in the suffix. The regex is doing exactly
> what it was asked, on data whose shape nobody checked. Test patterns against the awkward cases.
>
> **3.5 is the payoff:** a four-row per-outlet summary. Run the same `groupby` on the raw data to get
> twelve "outlets", including two Marina Bays (one with a trailing space) and a \$107k row.
> `groupby` returns in depth in Lesson 1.9 — this is the first taste.

---

## 🏃 Part 4: Reading and Writing Data (15 min)

Continue in the notebook with **Part 4**.

**Key topics:**
- CSV in — `read_csv` with `header`, `names`, `index_col`, `comment`, `na_values`
- CSV out — `to_csv`, and why you almost always want `index=False`
- JSON — `to_json(orient="records")`, `read_json`
- Excel — `pd.ExcelFile`, `.parse()`, `read_excel`, `to_excel` with two sheets
- Databases — `sqlalchemy.create_engine`, `read_sql_table`, `read_sql` with real SQL, `to_sql`

> The section saves `clean` to `data/cafe_june_clean.csv` — the session's actual output — then reads
> it back and checks it. Point out that the date came back as **text**: CSV and JSON forget dtypes;
> databases and pickle do not.

---

## 🎯 Wrap-Up

**Key Takeaways:**
1. Always run the five-move first look on a new file — head, shape, info, dtypes, describe.
2. Wrong types hide everything. The \$98,000 and the -999s were invisible while revenue was text.
3. Cleaning decisions depend on what the value *means*, not on what is convenient.
4. Order of operations matters: mask sentinels before you compute the statistic you impute with —
   and know that a median forgives the mistake where a mean does not.
5. Standardise your keys before you trust a duplicate check.
6. A clean table is not the goal; a *trustworthy answer* is. Section 3.5 is what all the cleaning
   was for.

**Next Steps:**
- Complete the [Assignment](./assignment.md) — audit the May export.
- Next lesson: **1.9 EDA Advanced** opens `daily_sales.csv` — the same export for all 18 months,
  cleaned the way you just cleaned June — and asks what the pattern is.

> **The handoff, in one number.** Cleaned June = \$174,753. Lesson 1.9's June = \$175,669. A \$916
> gap — 0.5% — and the class should be able to name its cause: **eight shifts** (four sentinels, one
> mis-key, three blanks) where a median stood in for a figure the export had destroyed.
