# Displaying the Percentage of Cumulative Income Achievement against Cumulative Targets (Predict) by Day
```sql
WITH
  union_day AS (
  SELECT
    s.date,
    SUM(p.price) AS revenue,
    0 AS revenue_pr
  FROM
    `DA.order` o
  JOIN
    `DA.product` p
  ON
    o.item_id = p.item_id
  JOIN
    `DA.session` s
  ON
    o.ga_session_id = s.ga_session_id
  GROUP BY
    s.date
  UNION ALL
  SELECT
    rp.date,
    0 AS revenue,
    rp.predict AS revenue_pr
  FROM
    `DA.revenue_predict` rp),

  union_day2 AS (
  SELECT
    date,
    DATE_TRUNC(date, day) AS day_date,
    SUM(revenue) AS revenue,
    SUM(revenue_pr) AS revenue_pr
  FROM
    union_day
  GROUP BY
    1, 2)

  SELECT
    day_date,
    date,
    revenue,
    SUM(revenue) OVER(ORDER BY date) AS acc_revenue,
    revenue_pr,
    SUM(revenue_pr) OVER(ORDER BY date) AS acc_revenue_rp,
    SUM(revenue) OVER(ORDER BY date) / SUM(revenue_pr) OVER(ORDER BY date) * 100 AS revenue_percent
  FROM
    union_day2
