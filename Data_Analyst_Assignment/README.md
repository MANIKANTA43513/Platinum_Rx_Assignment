# HealthCare-Data-Analysis

## Repository Structure

```
Data_Analyst_Assignment/
├── SQL/
│   ├── 01_Hotel_Schema_Setup.sql   – CREATE TABLE + INSERT data (Hotel system)
│   ├── 02_Hotel_Queries.sql        – Part A Q1–Q5 (Hotel queries)
│   ├── 03_Clinic_Schema_Setup.sql  – CREATE TABLE + INSERT data (Clinic system)
│   └── 04_Clinic_Queries.sql       – Part B Q1–Q5 (Clinic queries)
├── Spreadsheets/
│   └── Ticket_Analysis.xlsx        – Three-sheet workbook (ticket, feedbacks, Analysis)
├── Python/
│   ├── 01_Time_Converter.py        – Minutes → human-readable converter
│   └── 02_Remove_Duplicates.py     – Remove duplicate chars from string (loop-based)
└── README.md
```

---

## Phase 1 – SQL

### Hotel Management (Part A)

| Q# | Question | Key Technique |
|----|----------|---------------|
| 1 | Last booked room per user | `ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY booking_date DESC)` |
| 2 | Total billing per booking – Nov 2021 | `JOIN` 3 tables + `SUM(quantity * rate)` grouped by booking |
| 3 | Bills > 1000 in Oct 2021 | `HAVING SUM(...) > 1000` |
| 4 | Most/Least ordered item per month | `RANK() OVER (PARTITION BY month ORDER BY qty DESC/ASC)` |
| 5 | Customer with 2nd highest bill per month | `DENSE_RANK() OVER (PARTITION BY month ORDER BY total_bill DESC)` → filter `rnk = 2` |

### Clinic Management (Part B)

| Q# | Question | Key Technique |
|----|----------|---------------|
| 1 | Revenue by sales channel (given year) | `GROUP BY sales_channel` + `SUM(amount)` |
| 2 | Top 10 customers by spend | `GROUP BY uid` + `ORDER BY SUM DESC LIMIT 10` |
| 3 | Month-wise profit/loss status | CTE for revenue + CTE for expenses → joined and `profit = revenue - expense` |
| 4 | Most profitable clinic per city (given month) | `RANK() OVER (PARTITION BY city ORDER BY profit DESC)` → filter rank 1 |
| 5 | 2nd least profitable clinic per state | `DENSE_RANK() OVER (PARTITION BY state ORDER BY profit ASC)` → filter rank 2 |

**How to run:** Open any SQL client (MySQL Workbench / DB Fiddle / pgAdmin).  
Run `01_Hotel_Schema_Setup.sql` first, then `02_Hotel_Queries.sql`.  
Similarly run `03_Clinic_Schema_Setup.sql` then `04_Clinic_Queries.sql`.

---

## Phase 2 – Spreadsheet

File: `Ticket_Analysis.xlsx`

### Sheet: ticket
Raw ticket data with columns: `ticket_id`, `created_at`, `closed_at`, `outlet_id`, `cms_id`.

### Sheet: feedbacks
Raw feedback data. Column `ticket_created_at` is auto-populated via:

```excel
=IFERROR(INDEX(ticket!$B:$B, MATCH(A2, ticket!$E:$E, 0)), "Not Found")
```

This uses `cms_id` (column A) as the lookup key to fetch `created_at` from the `ticket` sheet.

### Sheet: Analysis
Helper columns and summary table for Q2:

| Column | Formula | Purpose |
|--------|---------|---------|
| Same Day? | `=IF(INT(created_at)=INT(closed_at),"Yes","No")` | Compares date portion |
| Same Hour? | `=IF(AND(INT(B)=INT(C),HOUR(B)=HOUR(C)),"Yes","No")` | Same day AND same hour |

The summary table shows per-outlet counts of same-day and same-hour closures.

---

## Phase 3 – Python

### 01_Time_Converter.py
```python
hours          = total_minutes // 60   # integer division
remaining_mins = total_minutes %  60   # modulo remainder
```
Output: `"2 hrs 10 minutes"`, `"1 hrs 50 minutes"`, etc.

### 02_Remove_Duplicates.py
```python
result = ""
for char in input_string:
    if char not in result:
        result += char
```
Preserves first-occurrence order; case-sensitive.

**How to run:**
```bash
python Python/01_Time_Converter.py
python Python/02_Remove_Duplicates.py
```

---

## Assumptions

1. All SQL queries target year 2021 for Hotel and 2023 for Clinic (parameterised via `SET @target_year`).
2. `item_quantity` can be fractional (DECIMAL) as shown in sample data (0.5).
3. Profit = Revenue − Expenses; negative profit = Not-Profitable.
4. For Q4/Q5 in Hotel: ties are handled by RANK (all tied items get the same rank).
5. Spreadsheet formulas use INDEX-MATCH instead of VLOOKUP for robustness across column insertions.
