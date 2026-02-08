# SecondHighestSalary

select(
    select distinct salary 
    from employee 
    order by salary desc limit 1 OFFSET 1)as SecondHighestSalary;
    
     limit 1 OFFSET 1

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
