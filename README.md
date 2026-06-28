# Pivot Table Summary Analysis

## Project Overview
This project analyzes the cleaned orders dataset using Excel Pivot Tables to summarize sales performance, profitability, customer behavior, shipping preferences, order status outcomes, and sales trends over time.

## Business Problem Summary
The objective of this analysis is to provide management with a high-level operational and financial overview of business performance. The pivot summaries help identify:
- High-performing regions and product categories
- Customer segments with stronger profitability
- Preferred shipping methods
- Regional patterns in cancelled/refunded orders
- Sales seasonality and monthly trends

## Dataset Description
The analysis uses the `cleaned_orders.xlsx` dataset containing transactional order data, including:
- Order ID
- Order Date
- Region
- Category and Sub-Category
- Customer Segment
- Ship Mode
- Sales
- Calculated Profit
- Order Status

## Deliverable
The file `pivot_summary.xlsx` contains six pivot-based summary worksheets.

## Pivot Summaries Created

### 1. Sales and Profit by Region
- Aggregation: Sum of Sales and Profit
- Purpose: Compare regional business performance
- Additional Feature: Sorted by Sales (descending)

### 2. Sales and Profit by Category and Sub-Category
- Aggregation: Sum of Sales and Profit
- Purpose: Identify strongest and weakest product segments
- Additional Feature: Sorted by Sales (descending)

### 3. Order Count by Ship Mode
- Aggregation: Count of Orders
- Purpose: Analyze shipping preferences

### 4. Profit Margin by Customer Segment
- Calculation:
  Profit Margin = Total Profit / Total Sales
- Purpose: Compare profitability across customer groups
- Additional Feature: Sorted by Profit Margin

### 5. Refunded/Cancelled/Failed Orders by Region
- Aggregation: Count of unsuccessful orders
- Purpose: Identify operational issues by geography

### 6. Monthly Sales Trend
- Aggregation: Monthly Sales totals
- Purpose: Understand seasonality and business growth patterns

## Filters and Sorting Used
The workbook includes:
- Excel AutoFilters on all summary sheets
- Sorting by Sales for regional and category analyses
- Sorting by Profit Margin for customer segment analysis

## Key Business Insights
- Regional performance varies significantly across markets.
- Product categories contribute unevenly to overall profitability.
- Certain customer segments generate stronger profit margins.
- Shipping mode usage patterns reveal customer preferences.
- Failed and cancelled orders exhibit regional concentration.
- Monthly sales trends provide insights into seasonality.

## Assumptions and Limitations
- Profit calculations rely on the provided `Calculated Profit` field.
- Order status analysis assumes consistent status labeling.
- Monthly trends are based solely on available historical data.
- No predictive or statistical analysis was performed.

## Files Included
- `cleaned_orders.xlsx`
- `pivot_summary.xlsx`
- `README.md`
