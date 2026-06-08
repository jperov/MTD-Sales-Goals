# MTD-Sales-Goals



<br>
<br>

The full report can be viewed [here](https://docs.google.com/spreadsheets/d/1MEWmhq633yTtfmOw41zC6IutEKzrT65g-0mpXVMxhu8/edit?gid=1316513782#gid=1316513782)


<br>
<br>

### SQL Query Used
```SQL
-- Gathers all daily goals data from the goals table.
WITH
daily_goals AS (
  SELECT
    SAFE_CAST(date AS DATE) AS date,
    Location,
    Goal
  FROM jacobperovichportfolio.goals
),


-- Aggregates total sales per store and day while ignoring bag fees and gift card sales.
net_sales_per_store AS (
  SELECT
    Date,
    Location,    
    ROUND(SUM(Net_sales), 2) AS Net_Sales
  FROM jacobperovichportfolio.sales
  WHERE
    lower(product_name) <> 'bag_fee' AND lower(product_name) <> 'gift_card'
  GROUP BY Date, Location
),


-- Daily goals data is joined to the aggregated daily sales table.
combined AS (
  SELECT
    g.Date,
    g.Location,
    COALESCE(g.goal, 0) AS Daily_Goal,
    COALESCE(ns.Net_Sales, 0) AS Net_Sales
  FROM daily_goals g
  LEFT JOIN net_sales_per_store ns
    ON g.date = ns.date
    AND g.location = ns.location
  WHERE
    g.date BETWEEN '2025-01-01' AND CURRENT_DATE - INTERVAL 1 day
    -- Updates to this report were made once at the start of a new day. In order to show the correct MTD goal amount for each store, the total goal needed to be calculated only up to the prior day.
),


-- Aggregates daily store data to MTD (one row per store).
mtd_summary AS (
  SELECT
    Location,
    SUM(Daily_Goal) AS MTD_Goal,
    ROUND(SUM(Net_Sales), 2) AS MTD_Sales,
    ROUND(SUM(Net_Sales) - SUM(Daily_Goal), 2) AS MTD_Difference
  FROM combined
  GROUP BY Location
)


SELECT
  Location,
  MTD_Goal,
  MTD_Sales,
  MTD_Difference,
  ROUND(MTD_Difference / NULLIF(MTD_Goal, 0), 4) AS MTD_Percent
FROM mtd_summary
WHERE
  Location <> 'Store Z'
ORDER BY MTD_Percent DESC;

```
