# Hive Employee Data Analysis

## Project Overview

This project involves analyzing employee data using Apache Hive. The dataset contains information about employees, including their department, job role, salary, and project assignments. The analysis includes partitioning the data, filtering, aggregating, and ranking employees based on salary.

## Dataset Description

### **employees.csv**

- `emp_id` - Unique employee ID
- `name` - Employee's full name
- `age` - Employee's age
- `job_role` - Designation of the employee
- `salary` - Annual salary of the employee
- `project` - Assigned project (Alpha, Beta, Gamma, Delta, Omega)
- `join_date` - Date when the employee joined
- `department` - Department to which the employee belongs (Used for partitioning)

### **departments.csv**

- `dept_id` - Unique department ID
- `department_name` - Name of the department
- `location` - Location of the department

## Steps to Execute

### **1. Setup and Load Data into HDFS**


docker exec -it namenode /bin/bash  # Open an interactive bash shell inside the namenode container.
hdfs dfs -mkdir -p /data/employee_data  # Create a directory in HDFS for employee data.
mkdir -p /data/employee_data  # Create a local directory for CSV files.
ls -l /data/employee_data  # Verify directory structure.
exit  # Exit the container shell.



docker cp /workspaces/hive-employee-data-analysis/input_dataset/employees.csv namenode:/data/employee_data/employees.csv
docker cp /workspaces/hive-employee-data-analysis/input_dataset/departments.csv namenode:/data/employee_data/departments.csv



docker exec -it namenode /bin/bash  # Open a shell inside namenode container.
hdfs dfs -put /data/employee_data/employees.csv /data/employee_data/
hdfs dfs -put /data/employee_data/departments.csv /data/employee_data/
exit


### **2. Start Hive and Create Tables**


docker exec -it hive-server /bin/bash
hive  # Start Hive CLI



CREATE EXTERNAL TABLE IF NOT EXISTS employees (
    emp_id INT,
    name STRING,
    age INT,
    job_role STRING,
    salary DOUBLE,
    project STRING,
    join_date STRING
)
PARTITIONED BY (department STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/data/employee_data';

CREATE EXTERNAL TABLE IF NOT EXISTS departments (
    dept_id INT,
    department_name STRING,
    location STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/data/employee_data';

SET hive.exec.dynamic.partition=true;
SET hive.exec.dynamic.partition.mode=nonstrict;


### **3. Run Queries**


SELECT * FROM employees WHERE CAST(SUBSTRING(join_date, 1, 4) AS INT) > 2015;
SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department;
SELECT * FROM employees WHERE project = 'Alpha';
SELECT job_role, COUNT(*) AS total_employees FROM employees GROUP BY job_role;

SELECT e.* FROM employees e
JOIN (
    SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department
) dept_avg
ON e.department = dept_avg.department
WHERE e.salary > dept_avg.avg_salary;

SELECT department, COUNT(*) AS total_employees
FROM employees GROUP BY department
ORDER BY total_employees DESC LIMIT 1;

SELECT COUNT(*) FROM employees WHERE emp_id IS NULL OR name IS NULL OR age IS NULL
OR job_role IS NULL OR salary IS NULL OR project IS NULL OR join_date IS NULL OR department IS NULL;

SELECT e.*, d.location FROM employees e JOIN departments d ON e.department = d.department_name;

SELECT *, RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank FROM employees;

SELECT * FROM (
    SELECT *, DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
    FROM employees
) ranked_employees WHERE rank <= 3;


### **4. Execute Queries and Save Output**


hive -f queries.hql | tee query_results.txt
```

### **5. Push Output to GitHub**


git init
git add .
git commit -m "Added Hive queries and output"
git branch -M main
git remote add origin <your-repo-url>
git push -u origin main
```


