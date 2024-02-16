# HR-Analysis-Report-

create database hr
use hr

select * from hr_data

UPDATE hr_data
SET termdate = FORMAT(CONVERT(DATETIME, LEFT(termdate, 19), 120), 'yyyy-MM-dd');

-- Update from nvachar to date
-- First, add a new date column
Alter table hr_data
Add new_termdate Date;


-- Update the new date column with the converted values
begin tran
Update hr_Data
set new_termdate= case 
 when termdate is not null and isdate(termdate)=1 then cast(termdate as datetime) else null end; 

rollback

commit

select termdate from hr_data
order by termdate desc;

-- create new column "age"
Alter table hr_data
add age nvarchar(50);

begin tran
update hr_data
set age= DATEDIFF(year, birthdate, getdate());

select @@TRANCOUNT
select * from hr_data
rollback
commit

select birthdate, age from hr_data
order by age desc;

--Age distribution
select min(age) Youngest , max(age) oldest from hr_data

 select age_group, count(*) as count 
 from(
	select 
	  case
	    when age<=22 and age<=30 then '22 to 30'
	    when age<=31 and age<=40 then '31 to 40'
	    when age<=41 and age<=50 then '41 to 50'
	    else '50+' 
	   end as age_group
	 from hr_data
	 where new_termdate is null
	  ) as subquery 
	 group by age_group
	 order by age_group;

--  Age group by Gender

select age_group,gender,count(*) as count
from (
select
case
	when age<=22 and age<=30 then '22 to 30'
	when age<=31 and age<=40 then '31 to 40'
	when age<=41 and age<=50 then '41 to 50'
	else '50+'
	end as age_group, gender
	from hr_data
	where new_termdate is null
	)as subquery
	group by age_group, gender
	order by age_group, gender

--Gender breakdown  in company
select gender, count(gender) as count from hr_data
where new_termdate is null
group by gender
order by gender

select * from hr_data

--gender across departments 

select  department, gender, count (*) as count
from hr_data
where new_termdate is null
group by  department, gender
order by department,gender

--gender across department and job titles
select department, jobtitle, gender, count (*) as count from hr_data
where new_termdate is null
group by department, jobtitle, gender
order by department, jobtitle, gender

-- race distribution in company
select * from hr_data
select race, count(race) as count from hr_data
where new_termdate is null
group by race
order by race desc;

-- Average length of employment in the company
select avg(datediff(year, hire_date, new_termdate)) as tenure from hr_data
where new_termdate is not null and new_termdate <= getdate()

--department's highest turnover rate

SELECT
 department,
 total_count,
 terminated_count,
 (round(CAST(terminated_count AS FLOAT)/total_count, 2))*100 AS turnover_rate
FROM 
   (SELECT
   department,
   count(*) AS total_count,
   SUM(CASE
        WHEN new_termdate IS NOT NULL AND new_termdate <= getdate()
		THEN 1 ELSE 0
		END
   ) AS terminated_count
  FROM hr_data
  GROUP BY department
  ) AS Subquery
ORDER BY turnover_rate DESC;

--tenure of each department
select department, 
avg(datediff(year, hire_date, new_termdate)) as tenure from hr_data
where new_termdate is not null and new_termdate <= getdate()
group by department
order by department

-- remote work for each department

select location, count(location) as count
from hr_data
where new_termdate is null 
group by location

-- Distribution of employees across states
select location_state, count(*) as count
from hr_data
where new_termdate is null
group by location_state
order by count desc

-- job titles distributed in the company
select jobtitle, count (*) as count from hr_data
where new_termdate is null
group by jobtitle
order by count desc

--  employee hire counts varied over time
SELECT
    hire_yr,
    hires,
    terminations,
    hires - terminations AS net_change,
    (round(CAST(hires - terminations AS FLOAT) / NULLIF(hires, 0), 2)) *100 AS percent_hire_change
FROM  
    (SELECT
        YEAR(hire_date) AS hire_yr,
        COUNT(*) AS hires,
        SUM(CASE WHEN new_termdate IS NOT NULL AND new_termdate <= GETDATE() THEN 1 ELSE 0 END) terminations
    FROM hr_data
    GROUP BY YEAR(hire_date)
    ) AS subquery
ORDER BY hire_yr ASC;

