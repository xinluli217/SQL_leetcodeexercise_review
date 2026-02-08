## Sorting
# SecondHighestSalary

    select(
      select distinct salary 
      from employee 
      order by salary desc limit 1 OFFSET 1)as SecondHighestSalary;
    

 DISTINCT：removes duplicate salaries
 sorts salaries in descending order -> highest salary comes first.
 OFFSET N - 1 -> skips the top N−1 salaries
 LIMIT 1 -> select exactly one salary
 
# getNthHighestSalary(N INT)  

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


