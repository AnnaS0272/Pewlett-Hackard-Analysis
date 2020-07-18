## Pewlett Hackard Analysis


At Pewlett Hackard like in most companies human capital matters for the ongoing company's success. The problem identified was to understand the amount of employees retiring in the next little while, specifically in which departments, what their titles were and how many were senior levels managers/employees. The latter particularly important to address mentoring program strategy. 

The data was initially only available in separate csv files. First, I built an ERD to understand the relationship between the data, i.e., how is the data connected through primary keys (unique values in the main table) and foreign keys (unique values in other tables).

Pewlett Hackard Employee Database

![Employee Database](https://github.com/AnnaS0272/Pewlett-Hackard-Analysis/blob/master/EmployeeDB.png)

Then I added the data to a PostgresSQL database, created multiple tables by connecting primary key and foreign keys to access relevant information from various tables. In order to do the latter, I used 'Inner Join' method and performed specific data requests such as determining current employees retiring soon. For example, using the below code I first determed list of current employees born between Jan. 1, 1952 and Dec. 31, 1955. I joined three tables, the first one -- **employees table**, that provided unique employee number ('emp_no'), first name and last name, then I took 'title' and 'from date' from **titles table** by using the unique (foreign) key 'emp_no', and finally 'salary' from **salaries table** also using the unique (foreign) key 'emp_no'.

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
As I got my tables back, I wanted to sense check if the total number of line items reduced or not, i.e., were the duplicates filtered out or not. In order to accomplish that, I have used the below code which evidently confirmed my total line items were reduced from 65,427 to 41,380:

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
I also used this approach to double check there were no duplicates left, and as I ran this quiery the table returned was empty, which signified the duplicated were removed.

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

**NOTE: I was confused what the Challenge was asking for at the end, under submission remarks, when I read : "One showing number of [titles] retiring." From the above code I could clearly see there were just 7 titles, I wasn't sure why we needed a separate csv file for 1 row that states "7" but I still generated it as "count_emp_title_unique.csv". My other classmates found it confusing too."**

I then determined employees who are eligible to participate in a mentorship program. To be eligible to participate in the mentorship program, employees will need to have a date of birth that falls between January 1, 1965 and December 31, 1965.

```
--Challenge Part 2: Mentorship Eligibility with duplicates
SELECT e.emp_no,
	e.first_name,
	e.last_name,
	ti.title,
	ti.from_date,
	ti.to_date
INTO emp_mentors
FROM employees as e
INNER JOIN titles as ti
ON (e.emp_no = ti.emp_no)
INNER JOIN dept_emp as de
ON (e.emp_no = de.emp_no)
WHERE (e.birth_date BETWEEN '1965-01-01' AND '1965-12-31')
--AND ti.to_date = ('9999-01-01')
ORDER BY e.emp_no;
```
Again, I encountered duplicates and I addressed them by using the following code:

```
--Challenge Part 2: Count before removing duplicates(3125)
SELECT
count(*)
FROM
emp_mentors
	
--Challenge Part 2: Mentorship Eligibility - without duplicates
SELECT e.emp_no,
	e.first_name,
	e.last_name,
	ti.title,
	ti.from_date,
	ti.to_date
INTO emp_mentors_clean
FROM employees as e
INNER JOIN titles as ti
ON (e.emp_no = ti.emp_no)
WHERE (e.birth_date BETWEEN '1965-01-01' AND '1965-12-31')
AND ti.to_date = ('9999-01-01')
ORDER BY e.emp_no;

--Challenge Part 2: Count after duplicates(1549)
SELECT
count(*)
FROM
emp_mentors_clean
```

---

From the data generated throughout the analysis, there are some things that require clarification before any human succession strategy plan takes place. For example, in the summary table by title, only 2 managers were identified, suggesting that either big chunk of data is missing or there are some inconsistencies at the data entry point. Therefore, it would be avisable to revisit the original csv files, where the data was pulled fom, who pulled the data and when. At the moment, the largest category retiring is with the title **"Senior Engineer"**, which means that a technical mentoring program needs to be put in place for this category. Scrolling through the results of cleaned up mentorship eligibility, I can see a lot of **"Senior Engineer"** titles, therefore suggesting the company has enough internal capabilities to set up the program but it has to be done promptly. My next steps would be to further test how a **mentor-mentee** program can be created, i.e., how many young engineers (or other titles as identified accordingly) are in the company and how many senior engineers, if there is an option for on-on-one mentoriship or, perhaps, assign several mentees to one mentor.

