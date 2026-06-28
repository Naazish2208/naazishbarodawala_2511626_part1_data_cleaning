# Data Cleaning Log — raw_orders.xlsx
**Project:** Retail Sales Data Quality & Cleaning  
**Analyst:** Business Analytics Team  
**Date:** 2025-07-09  
**Source file:** `raw_orders.xlsx` (sheet: `raw_orders`, 932 rows × 21 columns)  
**Output file:** `cleaned_orders.xlsx`

---

## Task 1: Preserve Raw Data
- The original file `raw_orders.xlsx` has **not been modified**.
- All cleaning is performed on an in-memory copy and saved to `cleaned_orders.xlsx`.

---

## Task 2: Text Field Cleaning

### Fields cleaned
All text fields were processed with the following standard pipeline:
1. `.strip()` — remove leading/trailing whitespace
2. `' '.join(s.split())` — collapse all internal multiple spaces to single space
3. `.title()` — standardize to Title Case

| Field | Issues Found | Values Before | Values After |
|---|---|---|---|
| `segment` | 16 variants | `'  Small Business '`, `'SMALL BUSINESS'`, `'consumer'`, `'Small  Business'`, `'Home Office '` | 5 values: Consumer, Corporate, Home Office, Small Business |
| `region` | 16 variants | `'  North '`, `'NORTH'`, `'west'`, `'WEST'`, `'EAST'`, `'east'` | 5 values + Unknown |
| `category` | 13 variants | `'FURNITURE'`, `'  Furniture '`, `'OFFICE SUPPLIES'`, `'Office  Supplies'`, `'  Technology '` | 3 values: Furniture, Office Supplies, Technology |
| `sub_category` | 23 variants | `'machines'`, `'LABELS'`, `'storage'`, `'COPIERS'`, `'MACHINES'`, `'  Accessories '` | 13 standard values |
| `ship_mode` | 12 variants | `'STANDARD CLASS'`, `'Standard  Class'`, `'FIRST CLASS'`, `'standard class'`, `'Second Class '` | 5 values: First Class, Same Day, Second Class, Standard Class, Unknown |
| `order_status` | 10 variants | `'COMPLETED'`, `'completed'`, `'  Completed '`, `'  Cancelled '`, `'cancelled'` | 4 values: Cancelled, Completed, Returned |
| `payment_status` | 8 variants | `'PENDING'`, `'failed'`, `'Paid '`, `'  Pending '` | 4 values: Failed, Paid, Pending, Refunded |
| `customer_name` | Extra spaces, ALL CAPS, double spaces | `'PRIYA MENON'`, `'Vikram  Iyer'`, `'Ananya Rao '` | Title Case, single-space names |
| `state` | Minor whitespace only | TRIM applied | Clean |
| `city` | Minor whitespace only | TRIM applied | Clean |

**Special normalization rules applied:**
- `'Small  Business'` (double space) → `'Small Business'`
- `'Standard  Class'` (double space) → `'Standard Class'`

---

## Task 3: Date Cleaning and Validation

### Date formats encountered in raw data
All of the following formats appeared across `order_date` and `ship_date`:
- `DD MMM YYYY` (e.g., `21 Jul 2024`)
- `DD-MM-YYYY` (e.g., `28-11-2024`)
- `MM/DD/YYYY` (e.g., `07/27/2024`)
- `YYYY-MM-DD` (e.g., `2024-05-24`)

**Decision:** A multi-format parser was used. All dates were converted to a consistent Python `datetime` object and output to Excel in `DD-MMM-YYYY` format.

### Date flags
| Flag | Count | Action |
|---|---|---|
| `flag_missing_order_date` | 0 | No missing order dates found |
| `flag_missing_ship_date` | 0 | No missing ship dates found |
| `flag_ship_before_order` | 21 | Flagged with `flag_ship_before_order = TRUE`. Records retained; excluded from completed sales. |

### `shipping_delay_days` column
- Calculated as `ship_date − order_date` in calendar days.
- Negative values indicate ship date is before order date (invalid).
- NULL when either date could not be parsed.

---

## Task 4: Duplicate Handling

### Exact Duplicates
- **Definition:** All 21 raw columns are byte-for-byte identical.
- **Found:** 20 exact duplicate rows.
- **Action:** First occurrence retained; subsequent occurrences removed.
- **Records removed:** 20

### Conflicting Duplicates (Same order_id, Different Data)
- **Definition:** Same `order_id`, but at least one column differs.
- **Found:** 12 unique `order_id` values with conflicts → 24 records.
- **Action:** All records retained. Each record marked with `flag_conflict_duplicate = TRUE`.
- **Records removed:** 0 (no silent deletion)
- **Records flagged for review:** 24

**Examples of conflicts found:**
| Order ID | Conflict Type |
|---|---|
| ORD-2025-10091 | sales = 9445.10 vs 9549.44; order_status = Completed vs Returned |
| ORD-2024-10124 | Three instances — sales differ (634.13, 634.13, 808.67); order_status differ |
| ORD-2024-10143 | profit value differs; order_status = Completed vs Returned |
| ORD-2025-10171 | sales = 5133.96 vs 5206.53; order_status = Completed vs Returned |

**Rationale:** These conflicts likely reflect amendment or reversal events in the source systems. Silently removing any record could destroy evidence of financial reversals. A human reviewer must decide the correct record.

---

## Task 5: Business Rules Applied

| Rule Area | Condition | Action Taken |
|---|---|---|
| **Missing region** | 25 records had blank `region` | Filled as `Unknown`; `flag_missing_region = TRUE` |
| **Missing ship_mode** | 21 records had blank `ship_mode` | Filled as `Unknown`; `flag_missing_ship_mode = TRUE` |
| **Missing discount** | 19 records had blank `discount` | Set to `0` where all numeric sales fields (unit_price, quantity, sales, cost, profit) were valid and non-null. Remaining unfillable: 0 |
| **Negative discount** | 16 records (e.g., -0.19, -0.23, -0.09) | Flagged as `flag_negative_discount = TRUE`; excluded from completed sales summary |
| **Discount > 50%** | 14 records (e.g., 70%, 85%, 0.65) | Flagged as `flag_discount_exceeds_max = TRUE`; excluded from completed sales summary. Threshold: 50% |
| **Cancelled orders** | `order_status = Cancelled` | Excluded from Completed Sales Summary sheet |
| **Failed payments** | `payment_status = Failed` | Excluded from Completed Sales Summary sheet |
| **Refunded orders** | `payment_status = Refunded` | Reported separately in "Refunds Summary" sheet in cleaned_orders.xlsx |
| **Ship before order** | 21 records where ship_date < order_date | Flagged as `flag_ship_before_order = TRUE`; excluded from completed sales |

---

## Calculated Columns Added

| Column | Formula / Logic |
|---|---|
| `shipping_delay_days` | `ship_date − order_date` in days |
| `calculated_sales` | `unit_price × quantity × (1 − discount)` |
| `calculated_profit` | `sales − cost` |
| `profit_margin` | `profit ÷ sales` (null if sales = 0 or null) |
| `order_month` | Month-Year label (e.g., "January 2024") |
| `order_year` | Year extracted from `order_date` |
| `data_quality_flag` | `"Issues Found"` if any flag = TRUE; else `"Clean"` |

---

## Final Record Count

| Stage | Count |
|---|---|
| Raw records | 932 |
| Exact duplicates removed | -20 |
| **Net records in cleaned_orders.xlsx** | **912** |
| Records with Issues Found | 136 |
| Clean records | 776 |

---

## Assumptions & Decisions

1. **Discount threshold:** Set at 50% (0.50). Discounts above this are unusual in retail and flagged as potential data-entry errors (e.g., "70%" and "85%" appear to be percentage-format mistakes).
2. **Percentage discounts:** Values like `70%` and `85%` in the discount column were converted from percentage format to decimal (e.g., 70% → 0.70) before flagging.
3. **Sales mismatch tolerance:** A tolerance of ₹1.00 is used. Differences larger than ₹1 are flagged as `flag_sales_mismatch`. Small rounding differences are accepted.
4. **Conflict duplicate resolution:** Retained both records. The analyst or finance team must reconcile — this log documents the decision not to silently discard any record.
5. **"Unknown" for missing fields:** Per business rules, blanks in `region` and `ship_mode` are filled as the string `"Unknown"` (not NULL) so these records can still be grouped in reports.
6. **Refunded with "Cancelled" order_status:** Some refunded orders have `order_status = Cancelled`. These are captured in the Refunds Summary by filtering on `payment_status = Refunded` regardless of order_status.
7. **Date format ambiguity:** Where a date like `01/07/2024` could be Jan-7 or Jul-1, the format parser resolves it as `MM/DD/YYYY` per the detected pattern. Any irresolvable dates would be left null (none found in this dataset).

