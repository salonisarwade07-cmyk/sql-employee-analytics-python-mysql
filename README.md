# sql-employee-analytics-python-mysql
Employee data ingestion and business analytics using Python and MySQL. Includes CSV ingestion, advanced SQL queries, CTEs, window functions, subqueries, and optional Pandas-based result verification.


# SQL-Powered Employee Analytics Using Python and MySQL

## Project Overview

This project demonstrates relational data ingestion and SQL-based business analytics using Python and MySQL. Employee records are loaded from a CSV file into a MySQL database, where business questions are solved using SQL queries.

Python is used to connect to the database, execute queries, and print the returned records. Optional Pandas verification is used only to inspect query-result structure and row counts, subject to the assignment rules.

## Objectives

- Ingest CSV data into a MySQL relational database.
- Create and manage employee tables using SQL.
- Solve 20 business analysis problems using SQL.
- Apply Common Table Expressions (CTEs), window functions, subqueries, grouping, and aggregations.
- Perform filtering, sorting, and calculations inside MySQL.
- Print raw query results using Python database cursors.
- Optionally verify query-result structure using Pandas.

## Technologies Used

- Python
- MySQL
- mysql-connector-python
- SQL
- Pandas (optional result verification only)

## Project Structure

- `ingest_data.py` — CSV ingestion and table creation.
- `employee_analytics.py` — SQL business analysis queries.
- `verification.py` — optional query-result verification.
- `employees_data.csv` — input dataset.

## Key Learning Outcomes

- Relational database design and data ingestion.
- SQL query writing and optimization fundamentals.
- Window functions, CTEs, and subqueries.
- Python–MySQL connectivity.
- Database-driven business analytics.

## How to Run

1. Install MySQL Server and create the `company_db` database.
2. Install the required Python packages.
3. Configure your MySQL credentials securely.
4. Place `employees_data.csv` in the project directory.
5. Run `ingest_data.py` to load the data.
6. Run `employee_analytics.py` to execute the business queries.
7. Run optional verification only if permitted by the assignment instructions.
