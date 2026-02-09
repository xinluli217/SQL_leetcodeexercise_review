# Review 
## SecondHighestSalary

    select(
      select distinct salary 
      from employee 
      order by salary desc limit 1 OFFSET 1)as SecondHighestSalary;
    

 DISTINCT：removes duplicate salaries
 sorts salaries in descending order -> highest salary comes first.
 OFFSET N - 1 -> skips the top N−1 salaries
 LIMIT 1 -> select exactly one salary
 
## getNthHighestSalary(N INT)  

    CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
     BEGIN
     RETURN (
      SELECT MAX(salary)
      FROM Employee e1
     WHERE (
      SELECT COUNT(DISTINCT e2.salary)
      FROM Employee e2
      WHERE e2.salary > e1.salary
    ) = N - 1
    );
    END

core: a salary is the Nth highest if exactly N−1 distinct salaries are higher than it.
The outer query iterates over each salary in the table
.
For each salary, a correlated subquery counts how many distinct salaries are greater than the current one.
If that count equals N − 1, the salary satisfies the definition of the Nth highest salary.
The MAX(salary) in the outer query guarantees that only one value is returned, even if multiple rows share the same salary.
If no salary satisfies this condition, the function naturally returns NULL, which matches the problem requirement.

## visit without transaction
    select v.customer_id, COUNT(*) as count_no_trans
    from Visits v
    left join Transactions t on v.visit_id=t.visit_id
    where transaction_id is null
    group by v.customer_id

start: Visits table -> all visits are considered, including those without transactions.
A LEFT JOIN -> join the Transactions table on visit_id:  visits without a matching transaction will still appear in the result, with NULL values on the transaction side.
The WHERE transaction_id IS NULL :  filters the result to keep only visits with no corresponding transaction.
The query then groups the data by customer_id.
COUNT(*) is used to count how many non-transaction visits each customer has.
The result shows, for each customer, the total number of visits that did not lead to a transaction.

## Write a solution to find all dates' id with higher temperatures compared to its previous dates (yesterday).

    select w1.id as ID
    from Weather w1
    join Weather w2 on w1.recordDate = DATE_ADD(w2.recordDate, INTERVAL 1 DAY)
    where w1.temperature>w2.temperature

self join：w1 represents the current day, while w2 represents the previous day.
DATE_ADD to match each date in w1 with the date one day earlier in w2.
WHERE clause compares temperatures and keeps only the rows where the current day’s temperature is greater than the previous day’s temperature
id of those days that meet this condition.

##Average Time of Process per Machine

    select a1.machine_id, ROUND(AVG(a2.`timestamp` - a1.`timestamp`), 3)as processing_time
    from Activity a1
    join Activity a2 on a1.process_id =a2.process_id and a1.machine_id=a2.machine_id
    WHERE a1.activity_type = 'start'
    AND a2.activity_type = 'end'
    group by a1.machine_id
    
The processing time = the time difference between a start event and its corresponding end event.
The Activity table is joined to itself using a self join.
The WHERE clause filters a1 to only include rows where activity_type is 'start', and a2 where it is 'end'.
The time difference is calculated as a2.timestamp - a1.timestamp.
AVG is used to compute the average processing time for each machine.
ROUND(..., 3) rounds the result to three decimal places for readability.
Finally, the results are grouped by machine_id.

## Having 
### Managers with at Least 5 Direct Reports
   
    select e1.name
    from Employee e1
    join Employee e2 ON e1.id = e2.managerId
    group by e1.id
    HAVING COUNT(e2.id) >= 5;

The Employee table is joined to itself using a self join.
e1 represents the manager, while e2 represents the employees reporting to that manager.
The join condition e1.id = e2.managerId links each manager to their direct subordinates.
The query groups the results by the manager’s ID (e1.id).
COUNT(e2.id) counts how many employees report to each manager.
The HAVING clause filters the groups to keep only managers with five or more direct reports.
The query returns the names of those managers.

 ### largest single number 
    SELECT MAX(num) AS num
    FROM MyNumbers
    WHERE num IN (
    SELECT num
    FROM MyNumbers
    GROUP BY num
    HAVING COUNT(num) = 1
    );

 having: a number that appeared only once in the MyNumbers
 max -> largest single number.

 ### Customers Who Bought All Products
    select  customer_id
    from Customer 
    group by customer_id
    HAVING count(distinct product_key)= (select count(*) from Product

 
## Confirmation Rate

    select s.user_id, ROUND(
    IFNULL(SUM(CASE WHEN c.action = 'confirmed' THEN 1 ELSE 0 END) / COUNT(c.action),
      0
    ),2)
    as confirmation_rate
    from Signups s
    left join Confirmations c on s.user_id=c.user_id
    group by s.user_id

LEFT JOIN is used to join the Confirmations table on user_id.
CASE WHEN: how many times the action equals 'confirmed'.
SUM(...) : the number of confirmed actions per user.
COUNT(c.action) : the total number of actions per user.
IFNULL is used to handle cases where a user has no confirmation records, preventing division by NULL.
ROUND:two decimal places

## not boring movie
    select *
    from Cinema
    where id % 2 = 1 and  description != 'boring'
    order by rating DESC
id % 2 = 1: odd-numbered
!= not include

## Percentage of Users Attended a Contest
    order by percentage desc, r.contest_id asc

## Monthly Transactions 
    select DATE_FORMAT(trans_date, '%Y-%m') as month, country,count( id)as trans_count, SUM(CASE WHEN  state ='approved' THEN 1 ELSE 0 END) as approved_count, SUM(amount) as trans_total_amount,SUM(CASE WHEN  state ='approved' THEN amount ELSE 0 END) as approved_total_amount
    from Transactions
    group by DATE_FORMAT(trans_date, '%Y-%m'), country
DATE_FORMAT(trans_date, '%Y-%m') as month
2018-12-18 -> 2018-12

##  Immediate Food Delivery

    select round(
      sum(case when d.customer_pref_delivery_date=d.order_date then 1 else 0 end)/count(*)*100,2) as immediate_percentage
    from Delivery d
    join (
       select customer_id, MIN(order_date) AS first_order_date
       from Delivery d
       group by customer_id
    ) t
    on d.customer_id = t.customer_id
    and d.order_date = t.first_order_date;
    
##  fraction of players that logged in again on the day after the day they first logged in

    SELECT
     ROUND(
      SUM(
        CASE
         WHEN a2.event_date = DATE_ADD(f.first_date, INTERVAL 1 DAY)
         THEN 1 ELSE 0
       END
     ) * 1.0
     / COUNT(DISTINCT f.player_id),
     2
    ) AS fraction
    FROM (
    SELECT player_id, MIN(event_date) AS first_date
    FROM Activity
     GROUP BY player_id
    ) f
    LEFT JOIN Activity a2
    ON f.player_id = a2.player_id;
player who play one day and second day:      
SUM(
        CASE
         WHEN a2.event_date = DATE_ADD(f.first_date, INTERVAL 1 DAY)
         THEN 1 ELSE 0
       END
     ) * 1.0
     / COUNT(DISTINCT f.player_id),
     2
     
## OR
 ### Primary Department for Each Employee
    SELECT employee_id, department_id
    FROM Employee e
    WHERE primary_flag = 'Y'
     OR (
        primary_flag = 'N'
        AND NOT EXISTS (
            SELECT 1
            FROM Employee e2
            WHERE e2.employee_id = e.employee_id
              AND e2.primary_flag = 'Y'
        )
     );
## CASE WHEN A AND B THEN 1 ELSE 0 END

 ### triangle
    SELECT
     x,
     y,
     z,
    CASE
      WHEN x + y > z AND x + z > y AND y + z > x THEN 'Yes'
      ELSE 'No'
    END AS triangle
    FROM Triangle;
### Exchange Seats
    select case when id % 2 =1 and id+1 in (select id from Seat) then id+1
            when id % 2 =0 then id-1 else id
        end as id, student
        from Seat
        order by id;
        
## WITH
###  Product Price at a Given Date
    with cte as (
    select * from (
    select*,
    rank() over(partition by product_id order by change_date desc) as rnk
    from Products
    where change_date < '2019-08-17') sq1
    where rnk = 1)

    select distinct p.product_id,
    case when c.new_price is null then 10
    else c.new_price end as price
    from Products p left join cte c on P.product_id = C.product_id;
CTE: Products table to include only price changes before the given date.
RANK() is applied, partitioned by product_id and ordered by change_date in descending order
 where rnk = 1, meaning the latest valid price per product.
In the outer query, the Products table is left joined with the CTE on product_id.
CASE expression is used to handle missing prices:
-> If a product has no prior price record, its price defaults to 10.
-> the latest price from the CTE is used.
DISTINCT: each product_id appears only once in the final output.

### Last Person to Fit in the Bus
    WITH cte AS (
      SELECT
         person_name,
         turn,
         SUM(weight) OVER (ORDER BY turn) AS running_sum
      FROM Queue
    )
    SELECT person_name
    FROM cte
    WHERE running_sum <= 1000
    ORDER BY running_sum DESC
    LIMIT 1;
CTE= calculate a running total of weight.
 SUM(weight) OVER (ORDER BY turn)
 turn column represents the queue order, so the running sum reflects the total weight up to each person.
 less than or equal to 1000.
running_sum in descending order.
LIMIT 1 selects the person with the largest valid cumulative weight, meaning the last person who can board safely.

## UNION ALL
    SELECT 'Low Salary' AS category,
       COUNT(*) AS accounts_count
    FROM Accounts
    WHERE income < 20000
    UNION ALL
    SELECT 'Average Salary' AS category,
       COUNT(*) AS accounts_count
    FROM Accounts
    WHERE income BETWEEN 20000 AND 50000
    UNION ALL
    SELECT 'High Salary' AS category,
       COUNT(*) AS accounts_count
    FROM Accounts
    WHERE income > 50000;
    
### Movie Rating
     (
     SELECT u.name AS results
     FROM Users u
     JOIN MovieRating mr ON u.user_id = mr.user_id
     GROUP BY u.user_id, u.name
     ORDER BY COUNT(*) DESC, u.name ASC
    LIMIT 1
    )
    UNION ALL
    (
     SELECT m.title
     FROM Movies m
    JOIN MovieRating mr ON m.movie_id = mr.movie_id
    WHERE mr.created_at >= '2020-02-01'
    AND mr.created_at < '2020-03-01'
    GROUP BY m.movie_id, m.title
    ORDER BY AVG(mr.rating) DESC, m.title ASC
    LIMIT 1
    );

