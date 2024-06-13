# Top 3 salaries by dept

## Dense Rank Window Function

I opted to use `DENSE_RANK` because I don't want to skip ranks if employees have the same salary.  
I also use the `ROW_NUMBER` function to get the row number of each employee.

I used a cte to get the rank for each employee as well as suplemental information about the department.

```sql
SELECT d.id,
    d.name AS Department,
    salary AS Salary,
    e.name AS Employee,
    DENSE_RANK()OVER(PARTITION BY d.id ORDER BY salary DESC) AS rnk,
FROM Department d
JOIN Employee e
ON d.id = e.departmentId
```

This will return a result that looks like the table below. You will see the employees are ranked and ordered by their salaries.

| department | employee | salary | rnk | row\_num |
| :--- | :--- | :--- | :--- | :--- |
| IT | Max | 90000 | 1 | 1 |
| IT | Randy | 85000 | 2 | 2 |
| IT | Joe | 85000 | 2 | 3 |
| IT | Will | 70000 | 3 | 4 |
| IT | Janet | 69000 | 4 | 5 |
| Sales | Henry | 80000 | 1 | 1 |
| Sales | Sam | 60000 | 2 | 2 |


## Getting the top 3 unique salaries per deptartment

Since we used `DENSE_RANK` each employee with have a rank associated with them. This makes getting the names of employees with the top 3 uniquie salaries. To do this I will wrap the ranking function into a CTE and then select anything from those results that has a rank <= 3

```sql
WITH employee_department AS
    (
    SELECT d.id,
        d.name AS Department,
        salary AS Salary,
        e.name AS Employee,
        DENSE_RANK()OVER(PARTITION BY d.id ORDER BY salary DESC) AS rnk,
        ROW_NUMBER() OVER (PARTITION BY d.id ORDER BY salary DESC) AS row_num

    FROM Department d
    JOIN Employee e
    ON d.id = e.departmentId
    )
SELECT Department, Employee, Salary, rnk, row_num
FROM employee_department
WHERE rnk <= 3
```
| department | employee | salary | rnk |
| :--- | :--- | :--- | :--- |
| IT | Max | 90000 | 1 |
| IT | Randy | 85000 | 2 |
| IT | Joe | 85000 | 2 |
| IT | Will | 70000 | 3 |
| Sales | Henry | 80000 | 1 |
| Sales | Sam | 60000 | 2 |

