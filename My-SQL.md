# My Notes in My-SQL

## 1. Introduction to Databases
- **What is a database?**
  - A database is an organized collection of structured information, or data, typically stored electronically in a computer system.
- **RDBMS vs. DBMS**
  - **DBMS**: Database Management System, manages databases (e.g., file-based, hierarchical, network).
  - **RDBMS**: Relational Database Management System, stores data in tables with relationships (e.g., MySQL, PostgreSQL).
- **MySQL overview and installation**
  - MySQL is a popular open-source RDBMS. Installation varies by OS; download from [mysql.com](https://www.mysql.com/downloads/).

## 2. Basic SQL Commands
- `CREATE DATABASE dbname;`
- `USE dbname;`
- `CREATE TABLE tablename (...);`
- `DROP TABLE tablename;`
- `ALTER TABLE tablename ...;`
- `TRUNCATE TABLE tablename;`
- **Data types in MySQL**: INT, VARCHAR, DATE, DATETIME, TEXT, FLOAT, DOUBLE, etc.

### MySQL Example Code
```sql
-- Create a database
CREATE DATABASE company_db;

-- Select the database
USE company_db;

-- Create a table
CREATE TABLE employees (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  department VARCHAR(50),
  salary DECIMAL(10,2),
  hire_date DATE
);

-- Alter table
ALTER TABLE employees ADD COLUMN email VARCHAR(100);

-- Drop table
DROP TABLE employees;

-- Truncate table
TRUNCATE TABLE employees;
```

## 3. Data Manipulation
- `INSERT INTO tablename (columns) VALUES (values);`
- `SELECT columns FROM tablename;`
- `UPDATE tablename SET column=value WHERE condition;`
- `DELETE FROM tablename WHERE condition;`

### MySQL Example Code
```sql
-- Insert data
INSERT INTO employees (name, department, salary, hire_date, email)
VALUES ('John Doe', 'HR', 50000, '2023-01-15', 'john@company.com');

-- Select data
SELECT * FROM employees;

-- Update data
UPDATE employees SET salary = 55000 WHERE name = 'John Doe';

-- Delete data
DELETE FROM employees WHERE name = 'John Doe';
```

## 4. Filtering and Sorting
- `WHERE` clause: filter rows
- Comparison operators: `=`, `!=`, `>`, `<`, `BETWEEN`, `LIKE`, `IN`
- `ORDER BY column [ASC|DESC]`
- `LIMIT n`

### MySQL Example Code
```sql
-- Where clause and comparison
SELECT * FROM employees WHERE salary > 40000 AND department = 'HR';

-- Order by and limit
SELECT * FROM employees ORDER BY salary DESC LIMIT 5;

-- Like and in
SELECT * FROM employees WHERE name LIKE 'J%' OR department IN ('HR', 'IT');
```

## 5. Basic Functions
- **Aggregate functions:**
  - `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`
- `GROUP BY column`
- `HAVING` (filter groups)

### MySQL Example Code
```sql
-- Aggregate functions
SELECT COUNT(*) FROM employees;
SELECT AVG(salary) FROM employees;
SELECT MIN(hire_date), MAX(hire_date) FROM employees;

-- Group by and having
SELECT department, COUNT(*) FROM employees GROUP BY department HAVING COUNT(*) > 2;
```

## 6. Joins
- `INNER JOIN`: rows with matching values in both tables
- `LEFT JOIN`: all rows from left, matched from right
- `RIGHT JOIN`: all rows from right, matched from left
- `FULL JOIN`: not directly supported, can be emulated with `UNION`
- `CROSS JOIN`: Cartesian product

### MySQL Example Code
```sql
-- Assume another table: departments
CREATE TABLE departments (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(50)
);

-- Inner join
SELECT e.name, d.name AS department_name
FROM employees e
INNER JOIN departments d ON e.department = d.name;

-- Left join
SELECT e.name, d.name AS department_name
FROM employees e
LEFT JOIN departments d ON e.department = d.name;

-- Right join
SELECT e.name, d.name AS department_name
FROM employees e
RIGHT JOIN departments d ON e.department = d.name;

-- Full join (emulated)
SELECT e.name, d.name AS department_name
FROM employees e
LEFT JOIN departments d ON e.department = d.name
UNION
SELECT e.name, d.name AS department_name
FROM employees e
RIGHT JOIN departments d ON e.department = d.name;

-- Cross join
SELECT e.name, d.name AS department_name
FROM employees e
CROSS JOIN departments d;
```

## 7. Advanced Querying
- **Subqueries**: Query within another query
- **Nested queries**: Subqueries inside subqueries
- **Correlated subqueries**: Subquery references outer query
- `EXISTS`, `ANY`, `ALL`: used with subqueries

### MySQL Example Code
```sql
-- Subquery
SELECT name FROM employees WHERE department = (SELECT name FROM departments WHERE id = 1);

-- Correlated subquery
SELECT name FROM employees e WHERE salary > (SELECT AVG(salary) FROM employees WHERE department = e.department);

-- EXISTS
SELECT name FROM employees WHERE EXISTS (SELECT 1 FROM departments WHERE employees.department = departments.name);
```

## 8. Set Operations
- `UNION`, `UNION ALL`
- `INTERSECT`, `EXCEPT`: not directly supported, can be simulated

### MySQL Example Code
```sql
-- Union
SELECT name FROM employees WHERE department = 'HR'
UNION
SELECT name FROM employees WHERE department = 'IT';

-- Union all
SELECT name FROM employees WHERE department = 'HR'
UNION ALL
SELECT name FROM employees WHERE department = 'IT';
```

## 9. Indexes
- **Types:**
  - `PRIMARY`, `UNIQUE`, `FULLTEXT`, `SPATIAL`
- Creating: `CREATE INDEX idx_name ON tablename (column);`
- Dropping: `DROP INDEX idx_name ON tablename;`
- Index performance: speeds up queries, but slows down writes

### MySQL Example Code
```sql
-- Create index
CREATE INDEX idx_dept ON employees(department);

-- Drop index
DROP INDEX idx_dept ON employees;
```

## 10. Constraints
- `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `NOT NULL`, `CHECK`, `DEFAULT`

### MySQL Example Code
```sql
-- Primary key and unique
CREATE TABLE projects (
  id INT PRIMARY KEY,
  name VARCHAR(100) UNIQUE
);

-- Foreign key
ALTER TABLE employees ADD CONSTRAINT fk_dept FOREIGN KEY (department) REFERENCES departments(name);

-- Not null, check, default
ALTER TABLE employees MODIFY salary DECIMAL(10,2) NOT NULL DEFAULT 30000 CHECK (salary > 0);
```

## 11. Views
- Creating: `CREATE VIEW viewname AS SELECT ...;`
- Updating: `CREATE OR REPLACE VIEW ...`
- Limitations: Some views are not updatable

### MySQL Example Code
```sql
-- Create view
CREATE VIEW high_earners AS SELECT * FROM employees WHERE salary > 60000;

-- Update view
CREATE OR REPLACE VIEW high_earners AS SELECT * FROM employees WHERE salary > 70000;
```

## 12. Stored Procedures & Functions
- Creating procedures: `CREATE PROCEDURE proc_name (...) BEGIN ... END;`
- Input/output parameters
- Custom SQL functions: `CREATE FUNCTION ...`
- Error handling: `DECLARE ... HANDLER`

### MySQL Example Code
```sql
-- Stored procedure
DELIMITER //
CREATE PROCEDURE GiveRaise(IN emp_id INT, IN amount DECIMAL(10,2))
BEGIN
  UPDATE employees SET salary = salary + amount WHERE id = emp_id;
END //
DELIMITER ;

-- Function
DELIMITER //
CREATE FUNCTION GetDepartmentCount(dept VARCHAR(50)) RETURNS INT
BEGIN
  DECLARE cnt INT;
  SELECT COUNT(*) INTO cnt FROM employees WHERE department = dept;
  RETURN cnt;
END //
DELIMITER ;
```

## 13. Triggers
- `BEFORE` and `AFTER` triggers on INSERT, UPDATE, DELETE
- Use cases: auditing, validation
- Restrictions: cannot call certain statements

### MySQL Example Code
```sql
-- Before insert trigger
DELIMITER //
CREATE TRIGGER before_employee_insert
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
  SET NEW.hire_date = IFNULL(NEW.hire_date, CURDATE());
END //
DELIMITER ;
```

## 14. Transactions
- `START TRANSACTION;`, `COMMIT;`, `ROLLBACK;`
- **ACID:** Atomicity, Consistency, Isolation, Durability
- Savepoints: `SAVEPOINT sp_name;`, `ROLLBACK TO sp_name;`

### MySQL Example Code
```sql
START TRANSACTION;
UPDATE employees SET salary = salary + 1000 WHERE department = 'HR';
SAVEPOINT before_bonus;
UPDATE employees SET salary = salary + 500 WHERE department = 'IT';
ROLLBACK TO before_bonus;
COMMIT;
```

## 15. Performance Tuning
- Query optimization: `EXPLAIN SELECT ...;`
- Index tuning
- Query caching (if available)
- Slow query log

### MySQL Example Code
```sql
-- Explain
EXPLAIN SELECT * FROM employees WHERE department = 'HR';

-- Enable slow query log
SET GLOBAL slow_query_log = 'ON';
```

## 16. User Management and Security
- Creating users: `CREATE USER 'user'@'host' IDENTIFIED BY 'pass';`
- Granting/revoking: `GRANT ... ON ... TO ...;`, `REVOKE ...;`
- Roles and privileges
- Securing data access

### MySQL Example Code
```sql
-- Create user
CREATE USER 'alice'@'localhost' IDENTIFIED BY 'password123';

-- Grant privileges
GRANT SELECT, INSERT ON company_db.* TO 'alice'@'localhost';

-- Revoke privileges
REVOKE INSERT ON company_db.* FROM 'alice'@'localhost';
```

## 17. Normalization & Schema Design
- **Normal forms:** 1NF, 2NF, 3NF, BCNF, 4NF, 5NF
- Denormalization strategies
- ER modeling concepts

### MySQL Example Code
```sql
-- Example: 1NF (atomic values)
CREATE TABLE contacts (
  id INT PRIMARY KEY,
  phone VARCHAR(20)
);

-- Example: 3NF (no transitive dependencies)
CREATE TABLE orders (
  id INT PRIMARY KEY,
  customer_id INT,
  order_date DATE
);
```

## 18. Replication
- Master-slave replication
- Master-master replication
- GTID-based replication

### MySQL Example Code
```sql
-- Show master status
SHOW MASTER STATUS;

-- Show slave status
SHOW SLAVE STATUS\G
```

## 19. Backup and Restore
- Logical backups: `mysqldump`
- Physical backups: `mysqlhotcopy`, InnoDB backup tools
- Point-in-time recovery

### MySQL Example Code
```sql
-- Logical backup
mysqldump -u root -p company_db > backup.sql

-- Restore
mysql -u root -p company_db < backup.sql
```

## 20. Partitioning
- Horizontal partitioning
- Types: range, list, hash, key
- Partition pruning

### MySQL Example Code
```sql
-- Range partitioning
CREATE TABLE sales (
  id INT,
  sale_date DATE
)
PARTITION BY RANGE (YEAR(sale_date)) (
  PARTITION p2023 VALUES LESS THAN (2024),
  PARTITION p2024 VALUES LESS THAN (2025)
);
```

## 21. Advanced MySQL Features
- JSON data type & functions
- Full-text search
- Spatial data support (GIS)
- Common Table Expressions (CTEs)
- Window functions: `RANK()`, `ROW_NUMBER()`

### MySQL Example Code
```sql
-- JSON data type
CREATE TABLE json_test (data JSON);

-- Full-text search
CREATE FULLTEXT INDEX idx_text ON employees(name);

-- CTE and window function
WITH dept_counts AS (
  SELECT department, COUNT(*) AS cnt FROM employees GROUP BY department
)
SELECT department, cnt, RANK() OVER (ORDER BY cnt DESC) AS dept_rank FROM dept_counts;
