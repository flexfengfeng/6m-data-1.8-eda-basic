# 🎓 Instructor Guide — Lesson 1.8: EDA Basic

> **Branch:** `feature/instructor-guide`
> **Audience:** Instructors and teaching assistants
> **Companion to:** `lesson.md`, `pre-class.md`, `assignment.md`

---

## 1. Lesson Overview & Instructor Objectives

| | |
|---|---|
| **Duration** | 3 hours |
| **Format** | Flipped Classroom + Guided Coding in Jupyter (notebook-led) |
| **Notebook** | `notebooks/eda_basic.ipynb` |
| **Learner entry point** | Completed 1.7 (Pandas); can create/select/filter DataFrames |

By the end of this lesson learners should be able to:

1. Run a "Health Check" on any new dataset — summarise shape, types, nulls, and distributions.
2. Handle missing values with appropriate strategies (fill vs. drop), and justify the choice.
3. Detect and remove duplicates.
4. Identify outliers using the IQR method and z-score.
5. Transform data through type conversion, string cleaning, and file I/O.

**Instructor's primary job:** EDA is where data science *actually starts* in real projects. More than 80% of professional data work is this lesson. The goal is not to teach syntax — the notebook does that. Your job is to build judgment: *why* fill vs. drop, *why* that outlier threshold, *why* encode categoricals. Frame every technique as a decision with trade-offs, not a rule to memorise.

---

## 2. Concept Analogies

### EDA as a Medical Checkup

> "When a new patient walks in, a good doctor doesn't immediately prescribe medication. They do a health check first: weight, blood pressure, blood test, family history. Only then do they make a diagnosis. EDA is the health check for your dataset before you prescribe any analysis."

**The Health Check checklist:**
- `.info()` → Blood test (data types, null counts, memory)
- `.describe()` → Vital signs (mean, std, min, max, quartiles for numeric columns)
- `.value_counts()` → Symptom frequency (most common values in categorical columns)
- `.shape` → Patient size (how many rows and columns)

This framing makes it natural to always run these four checks on every new dataset, rather than skipping straight to analysis.

---

### Missing Values — "Triage Decisions"

> "A hospital triage nurse makes a fast judgment: 'Can we treat this patient immediately? Do we need more information? Is the situation too severe?' Data cleaning is triage — every missing value needs a decision."

| Triage Call | Data Decision | When |
|-------------|--------------|------|
| "Treat immediately" | Fill with median/mode | When the column is important and missing % is small (<20%) |
| "Needs more investigation" | Fill with model-predicted value | When the column is critical and has a predictable pattern |
| "Remove from care" | Drop the row | When the row is missing data critical to your analysis |
| "Accept as is" | Leave as NaN | When NaN *is* meaningful (e.g., "no transaction yet") |

**The key question to ask for every missing value decision:** "Why is this value missing?" If the answer is "random data entry failure" → fill or drop. If the answer is "this customer has never purchased" → NaN is meaningful, keep it.

---

### Duplicates — "The Photocopier Problem"

> "Imagine you photocopied every receipt in the filing cabinet by mistake. The cabinet looks twice as full. If you calculate average transaction value, you'll get the right answer — but only by accident. Other aggregations will be wrong. Duplicates are silent errors."

**The real-world cause:** Duplicates usually come from system joins (if a customer appears in two source systems and the import runs twice) or pipeline re-runs (batch jobs that process data that was already loaded). This is why de-duplication is one of the first steps in any data pipeline.

---

### Outliers — "The Extremes Inspection"

> "An outlier is like a fire alarm. It might mean: (1) there's actually a fire (a genuine extreme value), (2) someone burned toast (a real but unimportant spike), or (3) the alarm is broken (data error). Your job is to find out which."

**The three responses to an outlier:**
1. Investigate: "Is this possible?" (A person aged 150 is not possible → data error.)
2. Cap: "This is real but extreme." Winsorization caps at 99th percentile.
3. Remove: "This will distort my model and isn't representative."

**IQR method:** The 1.5×IQR rule is a statistical convention, not a law. In some domains (finance, biomedical) extreme values are expected and removing them would be wrong. Teach the technique AND the judgment.

---

### Type Conversion — "Translating Between Languages"

> "If you store a date as text, the database knows the characters '2023-01-15' but has no idea that 2023-01-16 is one day later. Type conversion is translation — you're telling Python: 'Stop treating this as text. It's a date. Now you can do date math on it.'"

**The silent error analogy:** Storing a number as text looks fine until you try to average it — you get an error or incorrect result. These are "silent errors" that don't throw exceptions but produce wrong answers. Type conversion prevents them.

---

## 3. Real-World Use Cases

### EDA in Every Data Project

EDA is not a "beginner" skill — it's the first step in every professional data project. Netflix, Spotify, and Amazon run EDA on every new dataset before any modelling. The value: finding problems early is exponentially cheaper than finding them after building a model on bad data.

### Missing Data in Healthcare

A study at a major hospital found that 30% of electronic health records had at least one missing field. The strategy:
- Height/weight: fill with population median (non-critical for most analyses)
- Diagnosis code: cannot fill — must trace back to source system (critical field)
- Admission date: drop the row if missing (date is required for time-series analysis)

The decision on how to handle each missing field required clinical domain knowledge — not just statistics. This is the "why" behind the triage analogy.

### Outlier Handling in Fraud Detection

Fraud datasets are inherently imbalanced and outlier-heavy. A transaction 1000× the customer's average is an outlier — but it might be the fraud you're trying to detect. Removing all outliers would remove the signal. This is why "context determines the strategy" is the most important lesson in Part 2.

### String Cleaning in Production Pipelines

A UK e-commerce company found that their customer database had "London", "london", "LONDON", "London, UK" and "London (England)" all referring to the same city — stored in 5 different formats by 5 different sales teams. The `.str.lower().str.strip()` chain is the first step in fixing this before geographic analysis.

---

## 4. Activity Facilitation Notes

### Part 1: The Health Check (60 min)

**Establish the ritual:** Before opening the notebook, tell learners:
> "Every single time you receive a new dataset — for the rest of your career — you will run these four commands first. Not sometimes. Every time."

Then run `.info()`, `.describe()`, `.value_counts()`, `.shape` in sequence and narrate what each output tells you.

**What to look for in `.info()` output:**
- Non-null counts: anything less than total rows → missing data
- Dtypes: `object` when you expected `int64` → type mismatch or mixed data
- Memory usage: large values → consider `float32` or categorical types

**What to look for in `.describe()` output:**
- `min`/`max` outliers (e.g., `age = -3` or `age = 999`)
- High `std` relative to `mean` → high variance, possible outliers
- `25%` and `75%` quartiles far from `mean` → skewed distribution

**The `.value_counts()` discussion:** Show `value_counts(normalize=True)` for percentage distributions. Ask: "If one category makes up 95% of your data, what does that mean for your analysis?" → Class imbalance, potential selection bias, may need stratified sampling.

---

### Part 2: Data Quality (60 min)

**Missing value strategy — don't just show the syntax, show the decision:**
For each missing value scenario in the notebook, pause and ask:
- "What percentage is missing?" (5% vs. 40% → very different decisions)
- "Is this column used in our analysis?" (Critical vs. optional)
- "Why is it missing?" (Random vs. systematic)

**The fill vs. drop table to draw on the board:**

| % Missing | Importance | Strategy |
|-----------|-----------|----------|
| <5% | Any | Fill with median/mode or drop rows |
| 5–20% | Low | Fill or drop depending on analysis |
| 5–20% | High | Fill with model/forward-fill |
| >30% | Any | Flag for data collection improvement |
| >50% | Any | Consider dropping the column |

**Duplicate detection demonstration:**
Show `df.duplicated().sum()` (count) then `df[df.duplicated()]` (inspect). Ask: "Before we drop them, should we check if they're real duplicates?" → Yes. Two customers with the same name and birthdate could be different people. Always inspect before dropping.

**Outlier hands-on:** Have learners calculate the IQR manually before using the formula:
1. Calculate Q1 and Q3 with `.quantile(0.25)` and `.quantile(0.75)`.
2. Calculate IQR = Q3 - Q1.
3. Calculate lower bound = Q1 - 1.5 × IQR.
4. Calculate upper bound = Q3 + 1.5 × IQR.
5. Flag rows outside these bounds.

This manual walk-through is more memorable than just running a function.

---

### Part 3: Data Transformation (60 min)

**Type conversion order matters:** Convert types after you've handled missing values — trying to convert a column with NaN to int will fail. Teach this dependency explicitly.

**String operations demonstration:** Use `.str.lower()` + `.str.strip()` + `.str.replace()` as a three-step cleaning pipeline. Show before/after `value_counts()` to demonstrate how the number of unique values collapses (e.g., "London", "london", "LONDON" → all become "london").

**Categorical encoding — `get_dummies()` the ML preview:**
> "Machine learning models don't understand 'Red', 'Green', 'Blue' — they only understand numbers. One-hot encoding converts 'Color: Red' into three columns: `color_Red: 1, color_Green: 0, color_Blue: 0`. Each category becomes its own binary column."

Don't go deep — this is a preview of what they'll use in ML lessons. Just demonstrate and name the technique.

**File I/O — emphasise the round-trip:**
Save a cleaned DataFrame to CSV, then reload it. Show that datetime columns lose their type on save and need `parse_dates` on reload. This is not a bug — it's how CSV works (CSV has no type system). The `parse_dates` parameter is the professional solution.

---

## 5. Timing & Pacing Notes

| Part | Planned | Common Overrun | Mitigation |
|------|---------|---------------|-----------|
| Part 1: Health Check | 60 min | `.describe()` exploration generates many tangent questions | Set a discovery timer — "5 minutes to explore, then debrief together" |
| Part 2: Data Quality | 60 min | Outlier discussion runs long when learners ask about domain-specific decisions | Keep to IQR method as the framework; park domain-specific questions as "it depends on your industry" |
| Part 3: Transformation | 60 min | Regex section is unexpectedly complex for some learners | Treat regex as optional enrichment; focus on `.str` methods as the practical minimum |

---

## 6. Common Learner Questions

**Q: "When should I fill missing values vs. drop rows?"**
A: If the missing data is random and the column is important → fill with median/mode. If the missing data is in a column critical to your analysis (e.g., the target variable) → drop the row. If more than 30% of a column is missing → consider dropping the column entirely and flagging the data collection process.

**Q: "Is the 1.5 × IQR rule always the right threshold?"**
A: No — it's a convention from John Tukey's work in 1977. In some industries (finance, biomedical) you'd use 2 or 3 standard deviations, or domain-specific thresholds. Always ask: "Does this threshold make sense for my data and my analysis goal?"

**Q: "What's the difference between `dropna()` and `fillna()`?"**
A: `dropna()` removes rows (or columns) with NaN values. `fillna()` replaces NaN with a specified value. They're not alternatives — use `fillna()` when you want to keep the row but estimate the missing value; use `dropna()` when the row is unusable without that value.

**Q: "What is regex and do I need to learn it?"**
A: Regular expressions are a pattern language for text matching. You don't need to memorise syntax, but you should know they exist and when to reach for them (extracting postcodes from addresses, validating email formats, parsing log files). There are online tools (regex101.com) that help build patterns interactively.
