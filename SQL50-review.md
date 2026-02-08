# Sorting
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

##


