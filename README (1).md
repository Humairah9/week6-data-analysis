# From NumPy to SQL: What I Learned (and Broke) in My First Data Analyst Internship Week

This week I went from working with small arrays in Python to querying a live cloud database, and I learned as much from the things that failed as from the things that worked. Here's what I learned, what I solved, and what I'd tell anyone starting out.

## What I learned

The week had three parts: NumPy and pandas fundamentals, an exploratory analysis of weather data, and an intermediate SQL assessment.

On the Python side, I practised NumPy's element-wise checks (`isnan`, `iscomplex`, `allclose`, `isclose`) and comparison functions, plus how much memory an array uses. In pandas, I worked with Series: converting them to lists and arrays, doing arithmetic, handling mixed-type data with `pd.to_numeric(errors="coerce")`, flattening a Series of lists with `explode()`, and combining Series with `pd.concat` (since `Series.append` no longer exists in current pandas). I also practised filtering a DataFrame of WHO tuberculosis data using conditions like `|` and `&`.

On the SQL side, I used `INFORMATION_SCHEMA` to explore a database, conditional counting with `CASE`, `GROUP BY` with `HAVING`, joins across two sources, an anti-join to find non-matching records, `UNION ALL` to stack tables, CTEs with `ROW_NUMBER()` to find the top row per group, and a running total with a window function.

## What I solved

**Weather across six airports.** I loaded 2014 weather data for Beijing, Brasília, Cape Town, Delhi, London and Moscow into pandas (2,190 rows, one per city per day) and compared them. Brasília was the warmest city (a mean of 22.9 °C) and the most stable through the year, while Moscow was the coldest (6.0 °C) with the biggest seasonal swing. Cape Town's seasons were the mirror image of the northern cities, as expected for the southern hemisphere. Brasília was the wettest, with a clear wet season from November to April, while London had the most rainy days (170) but modest totals, which suggests frequent, light rain all year.

**A business question.** For the SQL assessment, the Head of Credit asked how loan approval differs by property area and credit history. The result was striking: every application with a credit history of 0 was rejected (166 applications, all Rural), and every application with a credit history of 1 was approved (167 Urban, 167 Semiurban). A rate of exactly 0% and 100% is unrealistic for real lending. It also meant area and credit history were completely tied together, so I could not separate their effects. My recommendation was to treat this data with caution, because it may be synthetic or may leak the decision into the features, before anyone builds a model on it.

## Challenges I faced

Most of my learning came from things that broke.

1. **Setting up the environment.** My first `pip install` failed with "pip is not recognized", and I had also copied a full stop into the command. Using `py -m pip install ...` fixed the first problem, and removing the stray character fixed the second.

2. **The `.env` file.** My database connection failed with a `KeyError` because Python could only see one of the four settings. When I looked at my file, I found the lines had been joined together, and the server name had picked up link formatting and an `http://` prefix. Rewriting it as four clean lines fixed it. Even then, I had to restart the Jupyter kernel, because Python had cached the old environment.

3. **A dropped connection.** Later I hit a `PendingRollbackError`. It turned out to be a side effect of an earlier network failure, which left a broken connection in memory. I fixed it by resetting the engine and wrapping my query function in a retry.

4. **Inconsistent data.** The weather tables called the date column `Date` in five cities but `GMT` in London, and the data had stray spaces and an HTML tag in a column name. I handled this by renaming the first column consistently.

5. **Wrong assumptions in my SQL.** Several queries returned nothing or failed because I guessed instead of checking. The trade data used `'Exports'`, not `'Export'`. The country column was `country`, not `Country Name`. The loan table's approval column was `Loan_Approved`, not `Loan_Status`. Each time, the fix was to stop guessing and query `INFORMATION_SCHEMA` or look at a few rows first.

## How I overcame them

The pattern that worked every time was the same: read the actual error message, check the real names and values in the data, and change one thing at a time. A query that returns zero rows is a clue, not a dead end. It usually means a filter does not match the data. I also learned to distinguish between a root-cause error and the secondary errors it produces.

## What I'd do differently

I'd inspect any new table (columns, data types, a few rows) before writing the first query. I'd also look at the data for plausibility before trusting it. Moscow showed 0 mm of rain all year, which is almost certainly missing data and not a dry climate. Delhi's temperatures, with a January average near −2 °C, also looked wrong. Both would have been easy to miss if I had only looked at the totals.

## Advice for others learning this

- Never guess column names or values. Look them up.
- Restart your kernel after changing environment files.
- Be suspicious of results that look too clean, such as a 100% approval rate.
- Keep your own notes on errors and fixes. Most of them repeat.
