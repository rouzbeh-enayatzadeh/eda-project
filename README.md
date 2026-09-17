# King County Housing — EDA

An exploratory data analysis of 21,597 house sales in King County, Washington,
carried out for a seller who wants to know **when** to sell, **where** the
value is, and whether **renovating** is worth it.

**Client:** Charles Christensen, a seller looking for large returns.

## Key findings

**Timing is worth about 10%.** Median price per square foot peaks in April at
$260 and bottoms out in December at $233. Averaged by season, spring sells
roughly 8% above winter.

**Location is the biggest lever.** The ten highest zipcodes command 2.76x the
price per square foot of the ten lowest — $565/sqft in Medina (98039) against
$145/sqft in Federal Way (98023). Mapped by coordinates, the expensive
zipcodes form one tight cluster around Lake Washington and central Seattle.

**The renovation premium is mostly a neighborhood effect.** Compared within
grade bands, renovated homes look 29–79% more valuable. Compared within the
same zipcode, that premium collapses to a median of +2.7% and is positive in
only 39 of 68 zipcodes. Renovated homes simply cluster in expensive areas.

## Repository contents

| File | What it is |
| --- | --- |
| [`04_eda.ipynb`](04_eda.ipynb) | The analysis: cleaning, feature engineering, the three hypotheses, insights and recommendations |
| [`03_fetching_the_data_eda.ipynb`](03_fetching_the_data_eda.ipynb) | How the data is pulled from the course PostgreSQL database |
| [`01_assignment.md`](01_assignment.md) | The original project brief |
| [`02_workflow.md`](02_workflow.md) | The recommended EDA workflow |
| [`column_names.md`](column_names.md) | Description of every column in the dataset |
| `presentation.pdf` | The 10-minute client presentation |
| `data/` | Local data folder — CSV files here are not tracked by git |

## The data

The dataset lives in the `eda` schema of the course database, split across two
tables. `king_county_house_details` holds one row per house (size, grade,
location, build year) and `king_county_house_sales` holds one row per sale
(date, price), linked by `house_id`:

```sql
SELECT d.*, s.date, s.price
FROM eda.king_county_house_details AS d
INNER JOIN eda.king_county_house_sales AS s
        ON d.id = s.house_id
ORDER BY d.id, s.date;
```

This returns 21,597 sales of 21,420 distinct houses, covering May 2014 to
May 2015. The result is saved to `data/king_county_joined.csv`, which is not
tracked by git — re-run the query above to recreate it.

## Method notes

- **Neighborhood** is operationalised as `zipcode`, the only geographic unit in
  the data.
- **Return** is measured as price per square foot, so that house size does not
  drive the result.
- **176 houses sold more than once.** The shortest gap between two sales is 61
  days and no sale date is duplicated, so these are genuine resales rather than
  data-entry errors, and all are kept.
- **`yr_renovated` was systematically stored as the real year × 10.** All 744
  non-zero values were affected, with no valid four-digit value anywhere in the
  column. Corrected during cleaning.
- **No rows were removed.** 21,597 in, 21,597 out. Every headline figure is a
  median, which outliers do not move.

## Setup

### 1. Clone the repository

```bash
git clone git@github.com:rouzbeh-enayatzadeh/eda-project.git
cd eda-project
```

### 2. Install dependencies

This installs everything and creates a virtual environment in `.venv/`.

```bash
uv sync
```

> [!TIP]
> Need a library that is not installed yet? Add it with `uv add <package-name>`.
> This updates `pyproject.toml` and `uv.lock` and installs it into your `.venv`.

### 3. Set up your database credentials

The data-fetching notebook reads the connection details from a `.env` file.
Copy the template and fill in your own values:

```bash
cp .env.example .env
```

Open `.env` and replace the placeholders with the credentials for the King
County housing database. These values feed
[`03_fetching_the_data_eda.ipynb`](03_fetching_the_data_eda.ipynb).

> [!CAUTION]
> `.env` holds secrets and must never be committed. It is already listed in
> `.gitignore`. Only `.env.example`, with placeholder values, belongs in the
> repository.

### 4. Open the notebooks

```bash
code .
```

Open a notebook and select the Python environment created by `uv sync` as the
kernel.

## References

- [House Sales in King County dataset](https://www.kaggle.com/datasets/harlfoxem/housesalesprediction) — the source dataset and column descriptions
- [Pandas user guide](https://pandas.pydata.org/docs/user_guide/index.html)
- [Seaborn tutorial](https://seaborn.pydata.org/tutorial.html)
- [SQLAlchemy documentation](https://docs.sqlalchemy.org/en/20/)
- [EDA Checklist](https://github.com/neuefische/datascience-infographics/blob/main/EDA_Checklist.md)
- [Tips for data science presentations](https://www.dataknowsall.com/storytelling.html)
