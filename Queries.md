# HR Employee Detail project Queries.


### create Database.
```create database project;```


### use project databse.

```use project;```

# Data Cleaning.
```sql
--Turn off the safe update mode.

 set sql_safe_updates = 0;

-- Converted text in hire_date to dates and changed the date to "YY-MM-DD" format.

update hr
set hire_date = case 
     when hire_date like '%/%' then date_format(str_to_date(hire_date,'%m/%d/%Y'), '%Y-%m-%d')
	 when hire_date like '%-%' then date_format(str_to_date(hire_date,'%m-%d-%Y'), '%Y-%m-%d')
     else null
   end;  
 
--  Changed datatype of hire_date column to date.

alter table hr
modify column hire_date date;


-- Converted text in birthdate to dates and changed the date to "YY-MM-DD" format.

update hr 
set birthdate = case 
     when birthdate like '%/%' then date_format(str_to_date(birthdate,'%m/%d/%Y'), '%Y-%m-%d')
	 when birthdate like '%-%' then date_format(str_to_date(birthdate,'%m-%d-%Y'), '%Y-%m-%d')
     else null
   end;  
 

 --  Changed datatype of birthdate column to date.

alter table hr
modify column birthdate date;
  

-- Converted text into termdate column to dates and changed format to '%Y-%m-%d %H:%i:%s UTC'

UPDATE hr
SET termdate = IF(termdate IS NOT NULL AND termdate != '', date(str_to_date(termdate, '%Y-%m-%d %H:%i:%s UTC')), '0000-00-00')
WHERE true; 

SET sql_mode = 'ALLOW_INVALID_DATES';

-- Changed datatype of termdate to date.

alter table hr
modify column termdate date;


-- Adding age column
alter table hr add column age int;
set sql_safe_updates = 0;


-- Calculating Age.
update hr 
set age = timestampdiff(year,birthdate,curdate());

select 
  min(age) as youngest,
  max(age) as oldest
 from hr; 
 
 select count(*) from hr where age <18;
 ```

# Data Analysis Using SQL
 ```sql
--  QUESTIONS:-

-- 1.What is the gender breakdown of employee in the company?

SET sql_mode = '';

select gender,count(*) as count
from hr 
where age >=18 and termdate ='0000-00-00'
group by gender;


-- 2.What is the Race/Ethnicity breakdown of the employee in the company?

select race,count(*) as Count
from hr
where age >=18 and termdate ='0000-00-00'
group by race 
order by count(*) desc;

-- 3. What is the age distribution of the empolyees in the compnay?

select 
  min(age) as Youngest,
  max(age) as Oldest
from hr
where age >=18 and termdate ='0000-00-00';

select 
     case
          when age >=18 and age <=24 Then '18-24'
		  when age >=25 and age <=34 Then '25-34'
          when age >=35 and age <=44 Then '35-44'
          when age >=45 and age <=54 Then '44-54'
          when age >=55 and age <=64 Then '54-64'
          else '65+'
        end as Age_Group,
        count(*) as count
from hr
where age >=18 and termdate ='0000-00-00'
group by Age_Group
order by Age_Group;  

select 
     case
          when age >=18 and age <=24 Then '18-24'
		  when age >=25 and age <=34 Then '25-34'
          when age >=35 and age <=44 Then '35-44'
          when age >=45 and age <=54 Then '44-54'
          when age >=55 and age <=64 Then '54-64'
          else '65+'
        end as Age_Group, Gender,
        count(*) as count
from hr
where age >=18 and termdate ='0000-00-00'
group by Age_Group,Gender
order by Age_Group;       
          

-- 4. How many employee works at headquaters versus remote locations?

select location,count(*)as count
from hr
where age >=18 and termdate ='0000-00-00'
group by location;


-- 5. what is the average length of employment for the employees who have been terminated?

select 
round(avg(datediff(termdate,hire_date))/365) as Avg_length_employment
from hr
where termdate <=curdate() and termdate !='0000-00-00' and age>=18;


-- 6. How does the gender distribution vary across department and job titles?

select department,gender,count(*) as count
from hr
where age >=18 and termdate ='0000-00-00'
group by department,gender
order by department;


-- 7.what is the distribution of job titles across the company?

select jobtitle,count(*) as count
from hr
where age >=18 and termdate ='0000-00-00'
group by jobtitle
order by jobtitle desc;


-- 8. which department has the higest turnover rate?

select department,
  total_count,
  terminated_count,
  terminated_count/total_count as Termination_Rate
from (
	select department,count(*) as total_count,
    sum(case when termdate != '0000-00-00' and termdate <= curdate() then 1 else 0 end) as terminated_count
    from hr
    where age >=18
    group by department
    ) as subquery
   order by termination_rate desc;


-- 9. what is the distribution of employeesacross locations by city and state?

select location_state,count(*) as count
from hr
where age >=18 and termdate ='0000-00-00'
group by location_state
order by count desc;


-- 10. how has the company's employee count changed over time based on term dates?

select 
   year,
   hires,
   terminations,
   hires - terminations as net_change,
   round((hires - terminations)/hires * 100,2) as net_change_percent
  from (
        select year(hire_date) as year,
        count(*)as hires,
        sum(case when termdate != '0000-00-00' and termdate <=curdate() then 1 else 0 end) as terminations
        from hr
        where age>= 18
        group by year(hire_date)
        ) as subquery
order by year asc;


-- 11. what is the tenure distribution for each department?
  
  select department,round(avg(datediff(termdate,hire_date)/365),0) as avg_tenure
  from hr
  where termdate <=curdate() and termdate !='0000-00-00' and age <=18
  group by department;
