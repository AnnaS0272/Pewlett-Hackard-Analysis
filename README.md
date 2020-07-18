## Pewlett Hackard Analysis
---

At Pewlett Hackard like in most companies human capital matters for the ongoing company's success. The problem identified was to understand the amount of employees retiring in the next little while, specifically in which departments, what their titles were and how many were senior levels managers/employees. The latter particularly important to address mentoring program strategy. 

The data was initially only available in separate csv files. First, I built an ERD to understand the relationship between the data, i.e., how is the data connected through primary key (unique values in main table) and foreign keys (unique values in other tables).

Pewlett Hackard Employee Database

![Employee Database](https://github.com/AnnaS0272/Pewlett-Hackard-Analysis/blob/master/EmployeeDB.png)

Then I added the data to a Postgres database, created multiple tables by connecting primary key and foreign keys to access relevant information from various tables. In order to do the latter, I used 'Inner Join' method and performed specific data requests such as determining current employees retiring soon. For example, using the below code we first determed list of current employees born between Jan. 1, 1952 and Dec. 31, 1955. I joined three tables, the first one -- employees tables, that provided unique employee number ('emp_no'), first name and last name, then I took 'title' and 'from date' from titles table by using the unique (foreign) key 'emp_no', and finally 'salary' from salaries table also using the unique (foreign) key 'emp_no'.

```
-- Challenge Part 1: List of retiring employees grouped by title
SELECT e.emp_no,
	e.first_name,
	e.last_name,
	ti.title,
	ti.from_date,
	s.salary
INTO emp_title
FROM employees as e
INNER JOIN titles as ti
ON (e.emp_no = ti.emp_no)
INNER JOIN salaries as s
ON (e.emp_no = s.emp_no)
WHERE (e.birth_date BETWEEN '1952-01-01' AND '1955-12-31')
AND (e.hire_date BETWEEN '1985-01-01' AND '1988-12-31')
GROUP BY ti.title, e.emp_no, ti.from_date, s.salary;
```
While this quiery returned the table I needed, upon quick examination it was clear there were duplicates as throughout their careers people had more than one title. I then used partitioning method to filter out the duplicates.

```
-- Challenge Part 1: Partition the data to show only most recent title per employee
SELECT emp_no,
 first_name,
 last_name,
 salary,
 title
INTO emp_title_recent
FROM 
 (SELECT emp_no,
 first_name,
 last_name,
 salary,
 title, ROW_NUMBER() OVER
 (PARTITION BY (emp_no)
 ORDER BY from_date DESC) rn
 FROM emp_title
 ) tmp WHERE rn = 1
ORDER BY emp_no;
```
As I got my tables back I wanted to sense check if the total number of line items reduced or not, i.e., were the duplicates filtered out or not. for that, I have used the below code which evidently confirmed my total line items were reduced from 65,427 to 41,380:

```
--Challenge Part 1: Count the original number of rows in emp_title(recorded result:65427)
SELECT
count(*)
FROM
emp_title

--Challenge Part 1: Count the original number of rows in emp_title_recent(recorded result:41380)
SELECT
count(*)
FROM
emp_title_recent
  ```
I also used this approach to double check there were no duplicates left, and as I ran this quiery the tbale returned was empty, which signified the duplicated were removed.

```
--Challenge Part 1: Check if new table still has duplicates
SELECT
emp_no,
first_name,
last_name,
count(*)

FROM emp_title_recent

GROUP BY
emp_no,
first_name,
last_name
HAVING count(*) > 1;
```

Next I looked at how many people are retiring within each title to get a more summarized view on data.

```
-- Create summary table showing number of titles retiring
SELECT 
COUNT (DISTINCT title) as "Number of Titles Retirees"
INTO count_emp_title_unique
FROM emp_title_recent;

-- Create one table showing number of employees with each title or how many
SELECT
title as "Title",
COUNT (title) as "Number of Pending Retirees"
INTO count_emp_title_unique_summary
FROM emp_title_recent
GROUP BY title
ORDER BY title ASC;

```

**NOTE: I was confused what the Challenge was asking for at the end, under submission remarks, when I read : "One showing number of [titles] retiring." From the above code I could clearly see there were just 7 titles, I wasn't sure why we needed a separate csv file for 1 row that says "7" but i still egnerated is as "count_emp_title_unique. My other classmates found it confusing too."**

We then determined the total number of employees per title who will be retiring, and identify employees who are eligible to participate in a mentorship program. 


In your second paragraph, summarize the steps that you took to solve the problem, as well as the challenges that you encountered along the way. This is an excellent spot to provide examples and descriptions of the code that you used.

In your final paragraph, share the results of your analysis and discuss the data that you’ve generated. Have you identified any limitations to the analysis? What next steps would you recommend?
