# BingePlay — Streaming Analytics Minor Project 4

## Project Overview

**BingePlay** is a fictional Indian OTT streaming service analytics project from **The Unlox Academy — Data Analytics & Data Science Track, Week 4: Advanced SQL**.

The project uses a MySQL database containing five tables — `users`, `subscriptions`, `shows`, `watch_sessions`, and `ratings` — and asks **12 business questions** across three SQL skill tiers:

- **Tier 1 — Foundations:** Q1–Q5
- **Tier 2 — Joins & Subqueries:** Q6–Q10
- **Tier 3 — Advanced SQL:** Q11–Q12

The official project requires **one Jupyter notebook named `bingeplay_<yourname>.ipynb`**, connected to the MySQL `bingeplay` database using **SQLAlchemy + PyMySQL**, with one SQL code cell per question executed using `pandas.read_sql()`, printed query output, and one markdown cell per question containing the final answer and interpretation where requested.

---

## 1. Project Objective

The objective is to answer real-world streaming analytics questions involving:

- Monthly recurring revenue
- User signup trends
- Device usage
- Rating distribution
- Original vs acquired content
- Binge behaviour
- Users who never watched
- Plan/value analysis
- Upgrade cohorts
- Cliffhanger comebacks
- Consecutive-week engagement
- Early churn signals

The project is designed to test both basic SQL and advanced analytical SQL patterns.

---

## 2. Database Schema

### `users`

| Column | Description |
|---|---|
| `user_id` | Primary key |
| `name` | Full name |
| `signup_date` | Signup date |
| `city` | Indian city |
| `age_group` | User age group |
| `referral_source` | Signup referral source |

### `subscriptions`

| Column | Description |
|---|---|
| `subscription_id` | Primary key |
| `user_id` | User foreign key |
| `plan` | Basic / Premium / Family |
| `start_date` | Subscription start |
| `end_date` | End date; NULL means open-ended |
| `status` | active / expired / cancelled |
| `monthly_price_inr` | Monthly price |

**Important:** One user can have multiple subscription records.

### `shows`

| Column | Description |
|---|---|
| `show_id` | Primary key |
| `title` | Show title |
| `category` | Show category |
| `language` | Show language |
| `release_year` | Release year |
| `imdb_rating` | IMDb rating |
| `is_original` | 1 = BingePlay Original, 0 = acquired |
| `min_plan` | Minimum plan required |

### `watch_sessions`

| Column | Description |
|---|---|
| `session_id` | Primary key |
| `user_id` | Watching user; can be NULL |
| `show_id` | Watched show |
| `session_date` | Session date |
| `watch_minutes` | Minutes watched |
| `device_type` | Mobile / TV / Laptop / Tablet |
| `completed` | 1 = completed, 0 = incomplete |

**Important:** Exactly 2 rows have `user_id IS NULL`.

### `ratings`

| Column | Description |
|---|---|
| `rating_id` | Primary key |
| `user_id` | Rating user |
| `show_id` | Rated show |
| `stars` | 1–5 |
| `rated_date` | Rating date |

---

## 3. Dataset Verification

After loading the setup SQL file, verify the database with:

```sql
USE bingeplay;

SELECT 'users' AS tbl, COUNT(*) AS row_count FROM users
UNION ALL
SELECT 'subscriptions', COUNT(*) FROM subscriptions
UNION ALL
SELECT 'shows', COUNT(*) FROM shows
UNION ALL
SELECT 'watch_sessions', COUNT(*) FROM watch_sessions
UNION ALL
SELECT 'ratings', COUNT(*) FROM ratings;
```

Expected dataset size:

| Table | Rows |
|---|---:|
| users | 3,000 |
| subscriptions | 4,497 |
| shows | 100 |
| watch_sessions | 100,351 |
| ratings | 5,000 |

Verify the NULL trap:

```sql
SELECT COUNT(*)
FROM watch_sessions
WHERE user_id IS NULL;
```

Expected:

```text
2
```

---

# 4. How to Run the Project in Google Colab

> **This project is intended to be run in Google Colab. VS Code is not required.**

## Step 1 — Open the notebook

Open:

```text
bingeplay_abhishek.ipynb
```

in Google Colab.

The final submission filename should follow:

```text
bingeplay_<yourname>.ipynb
```

For example:

```text
bingeplay_abhishek.ipynb
```

## Step 2 — Install MySQL and Python packages

Run:

```python
!apt-get -qq update
!DEBIAN_FRONTEND=noninteractive apt-get -qq install -y mysql-server
!pip -q install pandas sqlalchemy pymysql
```

## Step 3 — Start MySQL

Run:

```python
!service mysql start
!mysql --version
```

## Step 4 — Upload the BingePlay setup SQL file

Run:

```python
from google.colab import files

uploaded = files.upload()

sql_file = next(
    (name for name in uploaded if name.lower().endswith(".sql")),
    None
)

if sql_file is None:
    raise FileNotFoundError("Please upload the BingePlay .sql setup file.")

print("SQL setup file:", sql_file)
```

When Colab opens the file picker, select the supplied BingePlay database setup file.

The project brief refers to this setup file as:

```text
bingeplay_setup.sql
```

## Step 5 — Load the SQL database

Run:

```python
!mysql -u root < "{sql_file}"
```

This creates:

```text
bingeplay
```

and loads all five tables.

## Step 6 — Connect using SQLAlchemy + PyMySQL

Use:

```python
from sqlalchemy import create_engine
import pandas as pd

engine = create_engine(
    "mysql+pymysql://root:@127.0.0.1:3306/bingeplay"
)
```

If TCP connection gives a connection error in Colab, use the MySQL Unix socket:

```python
from sqlalchemy import create_engine
import pandas as pd

engine = create_engine(
    "mysql+pymysql://root@localhost/bingeplay",
    connect_args={
        "unix_socket": "/var/run/mysqld/mysqld.sock"
    }
)
```

Test the connection:

```python
test = pd.read_sql(
    "SELECT COUNT(*) AS total_users FROM users;",
    engine
)

print(test.to_string(index=False))
```

Expected:

```text
 total_users
        3000
```

## Step 7 — Run Q1 to Q12

Run every question cell in order.

Do **not** skip the setup cells.

Each question should have:

1. A markdown cell with the answer/interpretation.
2. One SQL code cell.
3. `pd.read_sql(query, engine)`.
4. Printed output.

---

# 5. Final Results and Outputs

## Q1 — Active Revenue

**Final answer:**

- Active subscriptions: **2,340**
- Monthly recurring revenue: **₹784,260**

**Interpretation:** There are 2,340 currently open active subscriptions as of 30 June 2024, generating ₹784,260 in monthly recurring revenue.

---

## Q2 — Signup Momentum

| Month | Signup count |
|---|---:|
| January | 350 |
| February | 400 |
| March | 500 |
| April | 550 |
| May | 600 |
| June | 600 |

**Highest:** May and June, tied at **600 signups** each.

---

## Q3 — Device Analytics

| Device | Sessions | Watch minutes | Avg minutes/session | Completion rate % |
|---|---:|---:|---:|---:|
| device_type   |   total_sessions |   total_watch_minutes |   avg_watch_minutes |   completion_rate_pct |
|:--------------|-----------------:|----------------------:|--------------------:|----------------------:|
| Laptop        |            15105 |                453434 |               30.02 |                 60.51 |
| Mobile        |            50172 |               1504355 |               29.98 |                 60.24 |
| TV            |            27981 |                840595 |               30.04 |                 59.98 |
| Tablet        |             7091 |                210733 |               29.72 |                 59.79 |

**Highest session volume:** **Mobile — 50,172 sessions**.

NULL `user_id` sessions are excluded.

---

## Q4 — Rating Distribution

| Stars | Rating count | Percentage |
|---:|---:|---:|
|   stars |   rating_count |   rating_percentage |
|--------:|---------------:|--------------------:|
|       1 |            234 |                4.68 |
|       2 |            352 |                7.04 |
|       3 |            847 |               16.94 |
|       4 |           1781 |               35.62 |
|       5 |           1786 |               35.72 |

**Percentage of 4 or 5 star ratings:** **71.34%**.

---

## Q5 — Originals vs Acquired

| Content group | Number of shows | Average IMDb | Average release year |
|---|---:|---:|---:|
| content_group   |   number_of_shows |   avg_imdb_rating |   avg_release_year |
|:----------------|------------------:|------------------:|-------------------:|
| Acquired        |                70 |              6.63 |            2020.73 |
| Originals       |                30 |              7.92 |            2020.37 |

**Interpretation:** BingePlay Originals perform better on ratings by **1.29 IMDb points** on average.

---

## Q6 — Binge Day Detection

**Final answer:**

- Total Q2 binge days: **414**
- Top user: **U02956**
- Top user's binge days: **8**

A binge day requires the same user to watch the same show at least 5 times on the same date.

---

## Q7 — Q1 Signups Who Never Watched

**Final answer:**

- Total Q1 signups: **1250**
- Q1 signups who never watched: **226**

**Interpretation:** The solution uses `LEFT JOIN` + `IS NULL` rather than `NOT IN`, correctly handling the NULL values in `watch_sessions.user_id`.

---

## Q8 — Over-Paying Premium/Family Users

**Final answer:** **204 users**

**Interpretation:** These Premium/Family users only watched shows available on the Basic tier and therefore represent potential downgrade targets.

---

## Q9 — Upgrade Success Cohort

**Final answer:**

- Users: **55**
- Average days from signup to first upgrade: **64.96 days**

**Interpretation:** These users signed up in January, started with Basic, later upgraded to Premium/Family, and were still active as of 30 June 2024.

---

## Q10 — Cliffhanger Comebacks

**Final answer:**

- Total comeback events: **4345**
- Top show ID: **S088**
- Top show: **Rayalaseema Raga**
- Comeback events for top show: **64**

**Interpretation:** A comeback occurs when a user has an incomplete session and watches the same show again within 1–7 days.

---

## Q11 — Consecutive-Week Engagement

**Final answer:**

- Users with 4+ consecutive weeks: **1675**
- Longest streak: **26 weeks**
- One user with the longest streak: **U00213**

**Interpretation:** This uses the **gaps-and-islands** technique with ISO week numbering to identify consecutive weekly engagement streaks.

There can be multiple users tied for the longest streak; `U00213` is one valid user from the dataset.

---

## Q12 — Churn Signal Detection

**Final answer:** **522 users**

A churn signal means:

```text
May watch minutes > 0
AND
June watch minutes dropped by 50% or more compared with May
```

The complete Q12 result contains:

- `user_id`
- `name`
- `may_watch_minutes`
- `june_watch_minutes`
- `drop_percentage`

The complete 522-row Q12 output is also saved as:

```text
q12_churn_signal_output.csv
```

This CSV is useful because Q12 returns one row per qualifying user.

---

# 6. SQL Concepts Demonstrated

## Tier 1 — Foundations

### Q1
- `SELECT`
- `COUNT`
- `SUM`
- `WHERE`
- NULL handling

### Q2
- `MONTH`
- `MONTHNAME`
- `GROUP BY`
- `ORDER BY`
- Aggregation

### Q3
- `COUNT`
- `SUM`
- `AVG`
- `CASE`
- Percentage calculation

### Q4
- Subquery
- `COUNT`
- Percentage calculation
- `GROUP BY`

### Q5
- `CASE`
- `GROUP BY`
- `AVG`
- Content comparison

## Tier 2 — Joins & Subqueries

### Q6
- CTE
- `GROUP BY`
- `HAVING`
- Window ranking

### Q7
- `LEFT JOIN`
- `IS NULL`
- NULL handling
- Anti-join logic

### Q8
- CTE
- `ROW_NUMBER()`
- `NOT EXISTS`
- Multiple-table join

### Q9
- CTEs
- `ROW_NUMBER()`
- Plan ranking
- `DATEDIFF`
- Cohort analysis

### Q10
- Self-join
- Date range conditions
- `DISTINCT`
- CTE

## Tier 3 — Advanced SQL

### Q11
- `YEARWEEK(..., 3)`
- `ROW_NUMBER()`
- Gaps-and-islands
- CTEs
- Consecutive-week streak detection

### Q12
- Monthly aggregation
- `LAG()`
- CTE
- Percentage change
- Churn signal detection

---

# 7. Important SQL Traps

### Q7 — Never use naive `NOT IN`

Because `watch_sessions.user_id` contains NULL values, this can fail:

```sql
WHERE user_id NOT IN (
    SELECT user_id
    FROM watch_sessions
)
```

Use:

```sql
LEFT JOIN ... IS NULL
```

or:

```sql
NOT EXISTS
```

instead.

### Q8 — Current subscription

Do not simply use:

```sql
status = 'active'
```

The project defines a currently open subscription as:

```sql
status = 'active'
AND (end_date IS NULL OR end_date > '2024-06-30')
```

### Q9 — Multiple subscription rows

A user can have multiple subscription records, so the first subscription and first upgrade must be determined from the ordered subscription history.

### Q10 — Correct comeback definition

The self-join must use:

```text
same user
same show
incomplete first session
later session
1–7 days later
```

### Q11 — Consecutive weeks

The query must use distinct user/week combinations and a gaps-and-islands method.

### Q12 — LAG and WHERE

A window function such as `LAG()` should be calculated in a CTE first and filtered in an outer query. Do not try to reference the window-function result directly in the same `WHERE` clause.

---

# 8. Submission Checklist

Before submitting, verify:

- [ ] Notebook filename is `bingeplay_<yourname>.ipynb`
- [ ] MySQL database is named `bingeplay`
- [ ] All 5 tables are loaded
- [ ] Row counts are verified
- [ ] SQLAlchemy is used
- [ ] PyMySQL is used
- [ ] `pandas.read_sql()` is used
- [ ] Q1 SQL cell is present
- [ ] Q2 SQL cell is present
- [ ] Q3 SQL cell is present
- [ ] Q4 SQL cell is present
- [ ] Q5 SQL cell is present
- [ ] Q6 SQL cell is present
- [ ] Q7 SQL cell is present
- [ ] Q8 SQL cell is present
- [ ] Q9 SQL cell is present
- [ ] Q10 SQL cell is present
- [ ] Q11 SQL cell is present
- [ ] Q12 SQL cell is present
- [ ] Each question has a markdown answer cell
- [ ] Query outputs are printed
- [ ] No setup/error cells are left showing failed execution
- [ ] Restarting Colab and running from the top works correctly
- [ ] The uploaded setup SQL file is the supplied BingePlay database file

---

# 9. Project Scoring

| Component | Questions | Marks |
|---|---:|---:|
| Tier 1 — Foundations | 5 | 30 |
| Tier 2 — Joins & Subqueries | 5 | 40 |
| Tier 3 — Advanced | 2 | 20 |
| Code quality | — | 10 |
| **Total** | **12** | **100** |

The project grades each question on correctness of the numeric answer, SQL approach, and interpretation where requested.

---

# 10. Files

### Main submission

```text
bingeplay_abhishek.ipynb
```

### Database setup

```text
bingeplay_setup.sql
```

### Additional output

```text
q12_churn_signal_output.csv
```

---

## Final Result

**All 12 BingePlay business questions are covered:**

| Question | Result |
|---|---|
| Q1 | 2,340 active subscriptions; ₹784,260 MRR |
| Q2 | May & June — 600 signups each |
| Q3 | Mobile — 50,172 sessions |
| Q4 | 71.34% are 4/5-star ratings |
| Q5 | Originals lead by 1.29 IMDb points |
| Q6 | 414 binge days; U02956 had 8 |
| Q7 | 1,250 Q1 signups; 226 never watched |
| Q8 | 204 over-paying users |
| Q9 | 55 users; 64.96 days average |
| Q10 | 4345 comeback events; S088 had 64 |
| Q11 | 1675 users; 26-week longest streak; U00213 |
| Q12 | 522 churn-signal users |

**Recommended submission:** `bingeplay_abhishek.ipynb`
