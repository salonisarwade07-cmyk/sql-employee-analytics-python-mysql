SELECT * FROM employees_data;

**/* 1. Intra-Department Salary Ranking: Rank employees by their salary within their
respective departments using an explicit window function. Display the department, first
name, salary, and rank*/**

SELECT 
department, 
first_name, 
salary,
RANK() OVER(
PARTITION BY department
ORDER BY salary
)AS salary_rank
FROM employees_data


**/* 2. Peer Salary Delta Analysis: For each employee, calculate the exact salary difference
between them and the next highest-paid employee in the same department using a lag
or lead window function*/**

SELECT
first_name,
Salary, 
department,
LEAD(salary)                            #LEAD
OVER(
PARTITION BY department
ORDER BY salary DESC
)AS Next_Highest_Salary,

salary - LEAD(salary)
OVER(
PARTITION BY department
ORDER BY salary DESC
)AS Salary_Difference
FROM employees_data;

SELECT
first_name,
department,
salary,
LAG(salary)                                      #LAG
OVER (
PARTITION BY department
ORDER BY salary DESC
) AS next_higher_salary,

LAG(salary) OVER (
PARTITION BY department
ORDER BY salary DESC
) - salary AS salary_difference
FROM employees_data;

**/* 3. High-Expenditure Outlier Departments: Write a CTE to find departments whose total
salary expenditure exceeds the average total salary expenditure across all departments
in the company*/**

WITH DepartmentSalary AS
( 
		SELECT 
        department,
        SUM(salary) AS Total_Salary
        FROM employees_data
        GROUP BY department
)

SELECT 
department,
Total_Salary
FROM DepartmentSalary
WHERE Total_Salary >
(
	SELECT AVG(Total_Salary)
    FROM employees_data
);
        
        
**/* 4. Cross-Department Ceiling Breakers: Identify employees who earn more than the
maximum salary of the entire 'HR' department using a subquery */**
     
SELECT 
MAX(salary)
FROM employees_data
WHERE department = 'HR' 

SELECT
first_name,
salary,
department
FROM employees_data
WHERE salary > 
(
		SELECT MAX(salary)
		FROM employees_data
		WHERE department = 'HR'
);


**/*5. Cumulative Departmental Spending: Generate a running total of salaries for each
department, ordered by their hire_date*/**

SELECT
department,
salary,
hire_date,
SUM(salary) OVER(
PARTITION BY department
ORDER BY hire_date DESC
) AS Running_Total
from employees_data;
*/


**/* 6.  The Second-Highest Earner Problem: Identify the exact second-highest-paid
employee in each department using a CTE combined with a ranking function (do not use
LIMIT)*/**

WITH EmployeeRank AS
(
	SELECT
    first_name,
    department,
    salary,
    DENSE_RANK() OVER(
    PARTITION BY department
    ORDER BY salary DESC
    ) AS salary_rank
    from employees_data
    )
    
SELECT 
first_name,
department,
salary
from EmployeeRank
WHERE salary_rank = 2;
    
**/* 7. Aggressive Hiring Month Detection: Write a query using a subquery or CTE to find the
months (by name) where the company's total hiring volume exceeded the average
monthly hiring count for that calendar year */**

WITH MonthlyHiring AS (
SELECT 
YEAR(hire_date) AS hire_year,
MONTH(hire_date) AS month_no,
MONTHNAME(hire_date) AS month_name,
COUNT(*) AS hiring_count
from employees_data
GROUP BY YEAR(hire_date),
		MONTH(hire_date),
		MONTHNAME(hire_date)
)       

SELECT
hire_year,
month_name,
hiring_count
FROM MonthlyHiring m
WHERE hiring_count >
(
	SELECT
    AVG(hiring_count)
    FROM MonthlyHiring
    WHERE hire_year = m.hire_year
 )
 ORDER BY hire_year, month_no;
 
 **/* 8. Departmental Salary Quartiles: Divide employees within each department into 4
salary-based quartiles (NTILE). Show the employee details along with their assigned
quartile*/**


SELECT
employee_id,
salary,
department,
NTILE(4) OVER(
PARTITION BY department
ORDER BY salary DESC
) AS Salary_Quartile
from employees_data;

**/* 9. Budget Percentage Contribution: Use window functions to calculate the exact
percentage contribution of each employee's salary to their specific department's total
salary budget*/**


SELECT
first_name,
salary,
department,
SUM(salary) OVER(
PARTITION BY department
) AS Department_Total,
ROUND(
		(salary * 100) / SUM(salary) OVER(
        PARTITION BY department
		),
        2
 ) AS Percentage_Contribution       
from employees_data;


**/* 10. . Rolling 3-Employee Salary Moving Average: Within each department, calculate a
3-employee rolling/moving average of salaries ordered by hire_date (include current
employee, one preceding, and one succeeding record). */**

SELECT
first_name,
department,
hire_date,
salary,
AVG(salary) OVER (
PARTITION BY department
ORDER BY hire_date
ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
) AS Moving_Avg_Salary
from employees_data;


**/* 11. Volatility Analysis (Salary Spread): Identify the department that has the absolute
largest numeric spread (difference) between its highest-earning and lowest-earning
employee using a subquery */**


SELECT
department,
MAX(salary) - MIN(salary) AS Salary_Spread
FROM employees_data
GROUP BY department
HAVING Salary_Spread = 
(
	SELECT 
    MAX(Salary_Spread)
    FROM
    (
		SELECT
        department,
		MAX(salary) - MIN(salary) AS Salary_Spread
		FROM employees_data
		GROUP BY department
        
	) AS depart_spred
);


**/*12 . Anchor Earner Cohorts: Find all employees who were hired in the exact same month
and year as the employee who receives the absolute lowest salary in the entire
company*/**

SELECT
first_name,
salary,
department,
hire_date
FROM employees_data
WHERE YEAR(hire_date) = (
SELECT YEAR(hire_date)
FROM employees_data
WHERE salary= (
	SELECT MIN(salary)
    FROM employees_data
    )
)

AND MONTH(hire_date) = (
SELECT MONTH(hire_date)
FROM employees_data
WHERE salary= (
	SELECT MIN(salary)
    FROM employees_data
    )
);
    
    
**/* 13. . Dense Grouping for Elite Earners: Find the top 3 dense-ranked earners in each
department. Ensure that ties do not skip ranking numbers */  **

SELECT 
department, 
salary,
first_name,
salary_rank
FROM(
		SELECT
			department, 
			salary,
			first_name,
            DENSE_RANK() OVER(
            PARTITION BY department
			ORDER BY salary DESC
	) AS salary_rank
    FROM employees_data
    ) AS ranked
WHERE salary_rank <= 3
ORDER BY department, salary_rank, salary DESC;    

**/* 14. High-Value Operational Units: Using an EXISTS subquery, extract all unique
departments that house at least one employee earning over $120,000. */**

SELECT DISTINCT e1.department
FROM employees_data e1
WHERE EXISTS (
				SELECT *
				FROM employees_data e2
				WHERE e1.department = e2.department
				AND e2.salary > 120000
);

**/* 15.  Hiring Bookends Analysis: Write a CTE to identify both the earliest hired employee
and the latest hired employee for every department, returning their names and hire dates
in a unified dataset */**
 
WITH ranked_employees AS( 
SELECT first_name,
hire_date,
department,
ROW_NUMBER() OVER (
PARTITION BY department
ORDER BY hire_date ASC
) AS earliest_rank,

ROW_NUMBER() OVER (
PARTITION BY department
ORDER BY hire_date DESC
) AS lastest_rank
FROM employees_data
)

SELECT
first_name,
hire_date,
department,
'Earliest' AS hire_type
FROM ranked_employees
WHERE earliest_rank = 1

UNION ALL  
 
SELECT
first_name,
hire_date,
department,
'Lastest' AS hire_type
FROM ranked_employees
WHERE lastest_rank = 1

ORDER BY hire_date, department;

**/* 16. Dynamic Top-Earner Tagging: For every employee record, append a column showing
the name of the highest-paid individual within their department using the FIRST_VALUE
window function */**

SELECT
first_name,
department,
salary,
FIRST_VALUE(first_name) OVER(
PARTITION BY department
ORDER BY SALARY DESC
 )AS highest_paid_employee
FROM employees_data;

**/* 17. Year-Over-Year Hiring Trajectory: Use a CTE combined with LAG to calculate the
year-over-year change in headcount hiring volumes for every year present in the
dataset  */**

WITH yearly_hiring AS(
SELECT
YEAR(hire_date) as hire_year,
COUNT(*) AS headcount
FROM employees_data
GROUP BY YEAR(hire_date)
)

SELECT
hire_year,
headcount,
LAG(headcount) OVER(
ORDER BY hire_year
) AS previous_year_headcount,
 
headcount - LAG(headcount) OVER(
ORDER BY hire_year
) AS yoy_change
FROM yearly_hiring
ORDER BY hire_year;

**/* 18. Peak-Period Cohorts: Write a query to find all employees hired in a month where the
total number of hires across the entire company was at its historical peak. */**

WITH monthly_hire AS(
SELECT 
YEAR(hire_date) AS hire_year,
MONTH(hire_date) AS hire_month,
COUNT(*) AS total_hires
FROM employees_data 
GROUP BY 
YEAR(hire_date),
MONTH(hire_date)
),

peak_month AS(
SELECT
hire_year,
hire_month
FROM monthly_hire
WHERE total_hires = (
		SELECT MAX(total_hires)
        FROM monthly_hire
)
)
SELECT
e.first_name,
e.department,
e.hire_date
FROM employees_data e
JOIN peak_month p 
	ON YEAR(e.hire_date) = p.hire_year
    AND MONTH(e.hire_date) = p.hire_month
ORDER BY e.hire_date;

**/* 19. Polarization Index: For each department, find the difference between the average
salary of the top 10% highest earners and the bottom 10% lowest earners (Hint: Use
CTEs with PERCENT_RANK or ROW_NUMBER) */**

/*USE company;
SHOW TABLES;*/

WITH ranked AS (
    SELECT department, salary,
           PERCENT_RANK() OVER (
               PARTITION BY department ORDER BY salary
           ) AS salary_rank
    FROM employees_data
),
salary_groups AS (
    SELECT department,
           AVG(CASE WHEN salary_rank >= 0.9
                    THEN salary END) AS top_10_avg,
           AVG(CASE WHEN salary_rank <= 0.1
                    THEN salary END) AS bottom_10_avg
    FROM ranked
    GROUP BY department
)
SELECT department,
       ROUND(top_10_avg - bottom_10_avg, 2) AS polarization_index
FROM salary_groups;


/*DESCRIBE employee;*/

**/* 20. The Executive Master Summary: Write a single, highly optimized query utilizing a CTE that maps out every employee's name, department, and salary alongside their
department's average salary, their department rank, and a conditional flag (CASE WHEN) indicating if they are "Above" or "Below" the median salary of their group.*/**

WITH summary AS (
    SELECT first_name, last_name, department, salary,
           AVG(salary) OVER (
               PARTITION BY department
           ) AS dept_avg,
           RANK() OVER (
               PARTITION BY department ORDER BY salary DESC
           ) AS dept_rank,
           PERCENT_RANK() OVER (
               PARTITION BY department ORDER BY salary
           ) AS salary_percentile
    FROM employees_data
)
SELECT first_name, last_name, department, salary,
       ROUND(dept_avg, 2) AS department_avg_salary,
       dept_rank,
       CASE
           WHEN salary_percentile > 0.5 THEN 'Above'
           ELSE 'Below'
       END AS median_flag
FROM summary;
