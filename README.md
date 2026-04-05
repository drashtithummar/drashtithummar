# Sales Credit & SPIF Calculator

## Overview

This solution in solve.py file calculates monthly and quarterly sales performance metrics for each sales representative based on:

- Bookings data  
- Rep split allocations  
- Account industries  
- SPIF rules  
- Monthly quotas  

The outputs include:
- Total credited revenue  
- SPIF payout  
- Quota attainment  


---

## Code Structure & Logic


### 1. Data Cleaning
- All input CSV files are loaded
- Leading and trailing whitespace is removed from string columns
- Numeric columns are converted to appropriate types

---

### 2. Data Formatting
- Bookings are joined with accounts to bring in industry information
- Date fields are parsed to derive:
  - `month` (YYYY-MM)
  - `booking_quarter` (YYYYQX)

---

### 3. SPIF Rule Matching
For each booking:
- Match rules where:
  - `product_number` starts with `product_prefix`
  - industry matches OR rule industry is `*`
- If multiple rules match:
  - select the rule with the **longest prefix**


---

### 4. SPIF & Credit Calculation
- Booking-level SPIF: 
  booking_spif = amount × rate

- Split to reps using `split_pct`:
- credited revenue
- SPIF payout

---

### 5. Monthly Aggregation
Grouped by:
- rep_id
- month
- booking_quarter

Computed:
- total_credited
- spif_payout
- attainment = credited / quota

---

### 6. Quarterly Aggregation
- Quotas are aggregated per quarter
- Credited revenue and SPIF are summed per quarter
- Attainment is calculated as: total_credited / quarterly_quota



---

## How to Run

### Using a directory of CSV files:

```bash
python solve.py --data-dir <path_to_data> --out-month monthly.csv --out-quarter quarterly.csv
```

## Using ZIP file:
python solve.py --zip data.zip --out-month monthly.csv --out-quarter quarterly.csv


# Output Format
## Monthly Output
rep_id
month (YYYY-MM)
booking_quarter (YYYYQX)
total_credited (integer)
spif_payout (integer)
attainment (2 decimal places)

## Quarterly Output
rep_id
quarter (YYYYQX)
total_credited (integer)
spif_payout (integer)
attainment (2 decimal places)

## All outputs:

sorted correctly
UTF-8 encoded
written without index


# Debugging & Edge Case Observations

## During validation against the provided expected outputs:

- The quarterly output matched exactly
- The monthly output had a single edge case difference
SPIF payout differed by 1 (e.g., 228 vs 227)
Investigation Performed

# I explored several approaches to understand this behavior:

- Rounding at booking level
- Rounding at rep level
- Rounding before aggregation vs after aggregation
- Using different rounding strategies:
 -- standard .round()
 -- floor / ceil
 -- half-up rounding
- Handling floating point precision explicitly

# Key Finding
Adjusting rounding at different stages fixed that specific edge case
However, those changes introduced inconsistencies across other rows


# Final Decision
I chose to:

- follow the assignment instructions strictly
- apply rounding consistently at the final aggregation stage
- avoid introducing special-case or overfitted logic


# This keeps the solution:
- consistent
- maintainable
- aligned with general business expectations

Rather than forcing correctness for a single row at the cost of broader accuracy.


# Follow-up: Handling Data Larger Than Memory

If the dataset becomes too large to fit into memory, I would adjust the solution as follows:

## 1. Process data in chunks

```
pd.read_csv(..., chunksize=N)
```

to read the bookings dataset in smaller batches.

## 2. Keep lookup tables in memory

These datasets are relatively small and can be loaded once:

- accounts.csv
- spif_rules.csv
- quotas.csv

## 3. Process each chunk independently

For each chunk:

- join with accounts to get industry
- apply SPIF rule matching
- join with splits
- compute credited and SPIF values
- perform aggregation

## 4. Incremental aggregation

- Maintain running aggregates instead of storing all rows
- Combine chunk-level results progressively

## 5. Final aggregation

- Merge all chunk results into final monthly and quarterly outputs

## Optional Scaling

For very large datasets:

- Use distributed frameworks like PySpark
- parallel processing approaches


# Final Notes

This solution prioritizes:

- correctness based on instructions
- clarity and readability
- robustness for different input formats

It avoids overfitting to specific edge cases and instead focuses on a clean and generalizable implementation.



