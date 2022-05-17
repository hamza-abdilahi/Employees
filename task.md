# Task - Employee Records

The major corporation BusinessCorp&#8482; wants to do some analysis of varioius metrics around its employees and have contracted the job out to you. They have provided you with two SQL files, you will need to write the queries which will extract the data they are looking for.

## Setup

- Create a database to run the files against
- Use the `psql -d database -f file` command to run the SQL files
- Before starting to write the queries take some time to look at the data and figure out the relationships between the tables - maybe even draw an ERD to help.

## Tasks

### CTEs

1) Calculate the average salary of all employees

```sql
select
avg(salary) as company_average
from employees;
```

2) Calculate the average salary of the employees in each team (hint: you'll need to `JOIN` and `GROUP BY` here)

```sql
select
departments.id,
departments.name,
avg(salary) as department_avg_salary
from employees
inner join departments
on employees.department_id = departments.id
group by departments.id;

```

3) Using a CTE find the ratio of each employees salary to their team average, eg. an employee earning £55000 in a team where the average is £50000 has a ratio of 1.1

```sql
with department_averages(id, name, department_avg_salary) as (
select
departments.id,
departments.name,
avg(salary) as department_avg_salary
from employees
inner join departments
on employees.department_id = departments.id
group by departments.id
)
select
employees.first_name,
employees.last_name,
employees.salary,
department_averages.name,
department_averages.department_avg_salary,
employees.salary / department_averages.department_avg_salary as ratio
from employees
inner join department_averages
on employees.department_id = department_averages.id;
```

4) Find the employee with the highest ratio in Argentina

```sql
with department_averages(id, name, department_avg_salary) as (
select
departments.id,
departments.name,
avg(salary) as department_avg_salary
from employees
inner join departments
on employees.department_id = departments.id
group by departments.id
)
select
employees.first_name,
employees.last_name,
employees.salary,
department_averages.name,
department_averages.department_avg_salary,
employees.salary / department_averages.department_avg_salary as ratio
from employees
inner join department_averages
on employees.department_id = department_averages.id
where employees.country = 'Argentina'
order by ratio desc
limit 1;
```

5) **Extension:** Add a second CTE calculating the average salary for each country and add a column showing the difference between each employee's salary and their country average

```sql

```

---

### Window Functions

1) Find the running total of salary costs as the business has grown and hired more people

```sql
select
start_date,
salary,
sum(salary) over (order by start_date) as total_salary
from employees;

```

2) Determine if any employees started on the same day (hint: some sort of ranking may be useful here)

```sql
select
start_date,
dense_rank() over (order by start_date) as ranking
from employees
order by ranking desc;
```

3) Find how many employees there are from each country

```sql
select
distinct(country),
count(*) over (partition by country)
from employees;
```

4) Show how the average salary cost for each department has changed as the number of employees has increased

```sql
select
employees.first_name,
employees.last_name,
departments.name,
employees.start_date,
employees.salary,
avg(employees.salary) over(partition by employees.department_id order by employees.start_date) as average_salary
from employees 
inner join departments
on employees.department_id = departments.id;
```

5) **Extension:** Research the `EXTRACT` function and use it in conjunction with `PARTITION` and `COUNT` to show how many employees started working for BusinessCorp&#8482; each year. If you're feeling adventurous you could further partition by month...

```sql
<!--Copy solution here-->
```

---

### Combining the two

1) Find the maximum and minimum salaries

```sql
select
min(salary) as min_salary,
max(salary) as max_salary
from employees;
```

2) Find the difference between the maximum and minimum salaries and each employee's own salary

```sql
select
first_name,
last_name,
salary - (max(salary) over()) as max_offset;
```

3) Order the employees by start date. Research how to calculate the **median** salary value and the **standard deviation** in salary values and show how these change as more employees join the company

```sql
select
first_name,
last_name,
start_date,
salary,
stddev(salary) over (order by start_date) as avg_stddev
from employees;
```

4) Limit this query to only Research & Development team members and show a rolling value for only the 5 most recent employees.

```sql
select
first_name,
last_name,
departments.name,
start_date,
salary
avg(salary) over (order by start_date rows 5 preceding) as avg_salary
from employees
inner join departments
on employees.department_id = departments.id
where departments.id = 'Research and Development;
```

