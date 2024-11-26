# SQL Mentor User Performance Analysis 

**Project Overview**
This project is designed to help beginners understand SQL querying and performance analysis using real-time data from SQL Mentor datasets.The goal is to solve a series of SQL problems to extract meaningful insights from user data.

**Objectives**
--Learn how to use SQL for data analysis tasks such as aggregation, filtering, and ranking.
--Understand how to calculate and manipulate data in a real-world dataset.
--Gain hands-on experience with SQL functions like COUNT, SUM, AVG, EXTRACT(), and DENSE_RANK().
--Develop skills for performance analysis using SQL by solving different types of data problems related to user performance.

**SQL Mentor User Performance Dataset**
The dataset consists of information about user submissions for an online learning platform. Each submission includes:

User ID
Question ID
Points Earned
Submission Timestamp
Username

**#Key SQL Concepts Covered**
**Aggregation:** Using COUNT, SUM, AVG to aggregate data.
**Date Functions:** Using EXTRACT() and TO_CHAR() for manipulating dates.
**Conditional Aggregation:** Using CASE WHEN to handle positive and negative submissions.
**Ranking:** Using DENSE_RANK() to rank users based on their performance.
**Group By:** Aggregating results by groups (e.g., by user, by day, by week).



 **-- Q.1 List all distinct users and their stats (return user_name, total_submissions, points earned)**
```sql
select distinct username, count(id) as Total_submissions, sum(points) as points_earned
from user_submissions
group by username
order by 2 desc;
```
  
**-- Q.2 Calculate the daily average points for each user.
--each user and their daily average point**
```sql
select 
extract(day from submitted_at) as day, extract(month from submitted_at) as month,
username, avg(points) as daily_average
from user_submissions
group by 1,2,3;
```
**-- Q.3 Find the top 3 users with the most correct submissions for each day.
--each day, max right answers/correct submissions**
```sql
WITH daily_submissions AS (
    SELECT 
        TO_CHAR(submitted_at, 'DD-MM') AS daily,
        username,
        SUM(CASE WHEN points > 0 THEN 1 ELSE 0 END) AS correct_submissions
    FROM user_submissions
    GROUP BY 1, 2
),
users_rank AS (
    SELECT 
        daily,
        username,
        correct_submissions,
        DENSE_RANK() OVER(PARTITION BY daily ORDER BY correct_submissions DESC) AS rank
    FROM daily_submissions
)
SELECT 
    daily,
    username,
    correct_submissions
FROM users_rank
WHERE rank <= 3;
```

**-- Q.4 Find the top 5 users with the highest number of incorrect submissions.**

```sql
select 
username,
sum(
case when points > 0 then 1
else 0
end) as correct_submissions, 
sum(
case when points < 0 then 1
else 0 
end) as incorrect_submissions,
sum
( case when points < 0 then points
else 0
end) as incorrect_submissions_points,
sum(points) as total_points,
sum
( case when points > 0 then points
else 0
end) as correct_submissions_points
from user_submissions
group by username
order by incorrect_submissions desc
limit 5;
```
**-- Q.5 Find the top 10 performers for each week.**

```sql
select *
from
(select 
extract(week from submitted_at) as week_number,
username,
sum(points) as total_points,
DENSE_RANK() OVER(PARTITION BY EXTRACT(WEEK FROM submitted_at) ORDER BY SUM(points) DESC) AS rank
from user_submissions
group by 1,2
order by week_number asc, total_points desc
)
where rank  <=10;
```
