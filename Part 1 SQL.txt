Q1.Write an SQL query using vendor neutral ANSI SQL to find the
user_id, current name and current email address for all users.Do not
worry too much about the performance of the query, favour
readability.

SELECT user_id, name, email
FROM user_changes
WHERE created IN (
    SELECT MAX(created)
    FROM user_changes
    GROUP BY user_id
)

--------------------------------------------------------------------------------------

Q2.Write an SQL query using vendor neutral ANSI SQLto find the
median time between the second and third profile edit.Do not worry
too much about the performance of the query, favour readability.

SELECT TOP 1
  FLOOR(PERCENTILE_CONT(0.5)
    WITHIN GROUP (ORDER BY Mins) OVER()/60) as Median_Hours,
   ROUND((PERCENTILE_CONT(0.5)
    WITHIN GROUP (ORDER BY Mins) OVER()/60 -
  FLOOR(PERCENTILE_CONT(0.5)
    WITHIN GROUP (ORDER BY Mins) OVER()/60))*60,0) as Median_Minutes
FROM (SELECT tbl_row2.user_id, tbl_row2.created as date1, tbl_row2.created as date2, DATEDIFF(minute,tbl_row2.created,tbl_row3.created) as Mins
FROM user_changes tbl_row2
FULL JOIN (SELECT user_id, created
FROM user_changes) tbl_row3 ON tbl_row2.user_id = tbl_row2.user_id
WHERE tbl_row2.created IN (
SELECT MAX(created)
FROM (SELECT user_id, created, ROW_NUMBER() OVER
      (PARTITION BY user_id ORDER BY created) as row_num FROM user_changes) sub_row1
WHERE row_num = 2
GROUP BY user_id)
AND tbl_row3.created IN (
SELECT MAX(created)
FROM (SELECT user_id, created, ROW_NUMBER() OVER
      (PARTITION BY user_id ORDER BY created) as row_num FROM user_changes) sub_row2
WHERE row_num = 3
GROUP BY user_id)) main_table
