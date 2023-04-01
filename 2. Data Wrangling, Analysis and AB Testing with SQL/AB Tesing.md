## Assignment Tasks
We are running an experiment at an item-level, which means all users who visit will see the same page, but the layout of different item pages may differ.

### Question 1
Compare the final_assignments_qa table to the assignment events we captured for user_level_testing. Write an answer to the following question: Does this table have everything you need to compute metrics like 30-day view-binary?
```
SELECT 
  * 
FROM 
  dsv1069.final_assignments_qa
``` 
- **ANSWER:** No, In order to ensure precise calculation of metrics such as the 30-day view-binary, we require some additional data such as the `date and time` of the assignment. 

### Question 2
 Write a query and table creation statement to make final_assignments_qa look like the final_assignments table. If you discovered something missing in part 1, you may fill in the value with a place holder of the appropriate data type. 
```
SELECT item_id,
       COALESCE(test_a, CAST(NULL AS BIGINT)) AS test_assignment,
       'item_test_1' AS test_number,
       CAST('2020-01-01 00:00:00' AS timestamp) AS test_start_date
FROM dsv1069.final_assignments_qa
UNION ALL
SELECT item_id,
       COALESCE(test_b, CAST(NULL AS BIGINT)) AS test_assignment,
       'item_test_2' AS test_number,
       CAST('2020-01-01 00:00:00' AS timestamp) AS test_start_date
FROM dsv1069.final_assignments_qa
UNION ALL
SELECT item_id,
       COALESCE(test_c, CAST(NULL AS BIGINT)) AS test_assignment,
       'item_test_3' AS test_number,
       CAST('2020-01-01 00:00:00' AS timestamp) AS test_start_date
FROM dsv1069.final_assignments_qa
UNION ALL
SELECT item_id,
       COALESCE(test_d, CAST(NULL AS BIGINT)) AS test_assignment,
       'item_test_4' AS test_number,
       CAST('2020-01-01 00:00:00' AS timestamp) AS test_start_date
FROM dsv1069.final_assignments_qa
UNION ALL
SELECT item_id,
       COALESCE(test_e, CAST(NULL AS BIGINT)) AS test_assignment,
       'item_test_5' AS test_number,
       CAST('2020-01-01 00:00:00' AS timestamp) AS test_start_date
FROM dsv1069.final_assignments_qa
UNION ALL
SELECT item_id,
       COALESCE(test_f, CAST(NULL AS BIGINT)) AS test_assignment,
       'item_test_6' AS test_number,
       CAST('2020-01-01 00:00:00' AS timestamp) AS test_start_date
FROM dsv1069.final_assignments_qa
```

### Question 3
- Use the final_assignments table to calculate the order binary for the 30 day window after the test assignment for item_test_2 (You may include the day the test started)
```
SELECT ob.test_assignment,
       COUNT(DISTINCT ob.item_id) AS num_orders,
       SUM(ob.orders_bin_30d) AS sum_orders_bin_30d
FROM
    (SELECT fa.item_id,
            fa.test_assignment,
            MAX(CASE
                    WHEN (DATE(o.created_at) - DATE(fa.test_start_date)) BETWEEN 1 AND 30 THEN 1
                    ELSE 0
					END) AS orders_bin_30d
    FROM dsv1069.final_assignments AS fa
    LEFT JOIN dsv1069.orders AS o
        ON fa.item_id = o.item_id
    WHERE fa.test_number = 'item_test_2'
    GROUP BY fa.item_id, fa.test_assignment) AS ob
GROUP BY ob.test_assignment;
```
 - Query Result 

 |test_assignment | num orders | sum orders bin 30d|
 |----------------|------------|-------------------|
 |        0       |    1130    |      331          |
 |        1       |     1060   |     306           |   

### Question 4
- Use the final_assignments table to calculate the view binary, and average views for the 30 day window after the test assignment for item_test_2. (You may include the day the test started)
```
SELECT v.test_assignment,
       COUNT(DISTINCT v.item_id) AS num_views,
       SUM(v.view_bin_30d) AS sum_view_bin_30d,
       AVG(v.view_bin_30d) AS avg_view_bin_30d
FROM
  (SELECT fa.item_id,
          fa.test_assignment,
          MAX(CASE
                  WHEN EXTRACT(day FROM views.event_time - fa.test_start_date) BETWEEN 1 AND 30 THEN 1
                  ELSE 0
              END) AS view_bin_30d
   FROM dsv1069.final_assignments AS fa
   LEFT JOIN dsv1069.view_item_events AS views
     ON fa.item_id = views.item_id
   WHERE fa.test_number = 'item_test_2'
   GROUP BY fa.item_id,
            fa.test_assignment
   ORDER BY fa.item_id) AS v
GROUP BY v.test_assignment;
```

 - Query Result 

 |test_assignment | num orders | sum view bin 30d | avg view bin 30d
 |----------------|------------|-------------------|----------|
 |        0       |    1130    |      915          |  0.8097  |
 |        1       |     1060   |     881           |  0.8249  |

### Question 5
Use the https://thumbtack.github.io/abba/demo/abba.html to compute the lifts in metrics and the p-values for the binary metrics ( 30 day order binary and 30 day view binary) using a interval 95% confidence. 

### 30 day order binary 
<img src="images/Orders bin.png" width="600" height="350" alt="Orders bin" />

### 30 day view binary
<img src="images/Views bin.png" width="600" height="350" alt="Views bin" />

### Question 6
Use Mode’s Report builder feature to write up the test. Your write-up should include a title, a graph for each of the two binary metrics you’ve calculated. The lift and p-value (from the AB test calculator) for each of the two metrics, and a complete sentence to interpret the significance of each of the results.