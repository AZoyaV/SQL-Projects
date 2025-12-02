# Calculation of the Percentage of Emails Sent to Each Account within Each Month
# Determination of the First and Last Email Sending Dates for Each Account per Month
```sql
SELECT
  DISTINCT DATE_TRUNC(sent_month1, month) AS sent_month,
  id_account,
  COUNT(id_message) OVER (PARTITION BY id_account ORDER BY DATE_TRUNC(sent_month1, month)) / COUNT(id_message) OVER (PARTITION BY DATE_TRUNC(sent_month1, month)) AS sent_msg_percent_from_this_month,
  MIN(sent_month1) OVER (PARTITION BY id_account ORDER BY DATE_TRUNC(sent_month1, month)) AS first_sent_date,
  MAX(sent_month1) OVER (PARTITION BY id_account ORDER BY DATE_TRUNC(sent_month1, month)) AS last_sent_date
FROM (
  SELECT
    id_account,
    DATE_ADD(s.date, INTERVAL es.sent_date day) AS sent_month1,
    es.id_message AS id_message
  FROM
    `DA.email_sent` es
  JOIN
    `DA.account` a
  ON
    es.id_account = a.id
  JOIN
    `DA.account_session` acs
  ON
    es.id_account = acs.account_id
  JOIN
    `DA.session` s
  ON
    acs.ga_session_id = s.ga_session_id) sub
ORDER BY
  sent_month desc,
  id_account

