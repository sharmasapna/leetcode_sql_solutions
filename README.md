# Solution to Leetcode SQL Questions

I have learnt a lot from the youtube videos of [Fredric Muller](https://www.youtube.com/watch?v=DoGrxxa6kow&list=PLdrw9_aIADIPAMJW8I_S-S747oyiRtzpS) who has an amazing way of breaking down the problems which seem hard initially. Do watch his videos for more indepth analysis.



### 175. Combine Two Tables
```
SELECT firstName,lastName,city,state from
Person
LEFT JOIN
Address ON Person.personId = Address.personId

```

### 176. Second Highest Salary
```
# Using subquery and dense_rank()
SELECT IFNULL((SELECT  DISTINCT salary 
FROM
    (SELECT  salary, dense_rank() over( ORDER by salary DESC) AS rnk
    FROM Employee) temp
    WHERE rnk = 2) , NULL) AS  SecondHighestSalary 
    
    
# using subquery
SELECT 
IFNULL(  (SELECT MAX(salary) FROM Employee WHERE salary <
     (SELECT MAX(salary)
    FROM Employee))  , NULL) as SecondHighestSalary
```

### 177. Nth Highest Salary
```
# METHOD: 1 using LIMIT AND OFFSET
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
SET N = N-1;
  RETURN (
        SELECT DISTINCT salary 
      FROM Employee
      ORDER BY salary DESC
      LIMIT 1 OFFSET N
      
  );
END
#########
#METHOD: 2 USING CORRELATED SUB QUERY
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN ( SELECT DISTINCT salary FROM Employee e1
           WHERE N-1 = 
                      (SELECT COUNT(DISTINCT(salary))
                       FROM Employee e2
                       WHERE e2.salary > e1.salary)
      )   ;
  END
#######  
#METHOD: 3 USING CTE AND DENSE_RANK
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN ( WITH nthHighestSalary AS
                (SELECT salary,
                 DENSE_RANK() OVER(ORDER BY salary DESC) AS    rnk FROM Employee)
                SELECT DISTINCT salary FROM nthHighestSalary
                WHERE rnk = N
                 
                )   ;
END
##########
# METHOD: 4 using subquery and dense_rank()
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN (
      SELECT DISTINCT salary 
      FROM 
            ( SELECT salary,
              DENSE_RANK() OVER(ORDER BY salary DESC) AS rnk
              FROM Employee
            ) as a
        WHERE rnk = N
      
  );
END
######
#METHOD 5 USING TOP CLAUSE (although it doest run in MySQL but the logic is great!)
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN ( SELECT TOP 1 salary FROM 
                      (SELECT DISTINCT TOP N salary
                       FROM Employee
                       ORDER BY salary DESC) A
            ORDER BY salary
      )   ;
  END


```
### 178. Rank Scores
```
SELECT score,
       DENSE_RANK() OVER( ORDER BY  score DESC) AS 'rank'
FROM Scores
```
### 180. Consecutive Numbers
```
SELECT DISTINCT l1.num AS ConsecutiveNums
FROM 
Logs l1
JOIN Logs l2 ON l2.id = l1.id +1 AND l1.num = l2.num
JOIN Logs l3 ON l3.id = l1.id +2 AND l1.num = l3.num
```
### 181. Employees Earning More Than Their Managers

```
SELECT e1.name as Employee
FROM Employee e1
JOIN Employee e2 ON e1.managerId = e2.id
WHERE e1.salary > e2.salary
```
### 183. Customers Who Never Order

```
SELECT name as Customers from 
Customers
LEFT JOIN Orders ON Customers.id = Orders.customerId
WHERE Orders.id IS null
```

### 184. Department Highest Salary

```
SELECT Department, Employee,salary
FROM 
    (SELECT Department.name AS `Department`, 
            Employee.name AS `Employee`,  
            Employee.salary, 
            DENSE_RANK() OVER(PARTITION BY departmentId ORDER BY salary DESC) AS 'rank'
     FROM Employee 
     JOIN Department ON Employee.departmentId = Department.id) T
WHERE T.rank = 1
```
### 185. Department Top Three Salaries

```
SELECT Department, 
       Employee,
       Salary
       
FROM 
    (
    SELECT Department.name AS Department, 
           Employee.name AS Employee,
           Salary,
           DENSE_RANK() OVER (PARTITION BY Department.name ORDER BY Salary DESC) AS rnk
    FROM Employee
    LEFT JOIN Department ON Department.id = Employee.departmentId
    ) temp
WHERE rnk <4
```
### 197. Rising Temperature

```
SELECT w2.id FROM weather w1
JOIN weather w2 ON DATEDIFF(w1.recordDate, w2.recordDate) = -1
WHERE w1.temperature < w2.temperature 
```
### 262. Trips and Users

```SELECT 
request_at AS Day,
ROUND((SUM(IF(status != 'completed' ,1 ,0 ))/count(status)),2) AS 'Cancellation Rate'
FROM Trips
WHERE request_at BETWEEN "2013-10-01" AND '2013-10-03'
  AND (client_id in (SELECT users_id FROM Users WHERE banned = 'No') )
  AND (driver_id in (SELECT users_id FROM Users WHERE banned = 'No') )
GROUP BY Day
```

### 511. Game Play Analysis I

```
SELECT player_id, min(event_date) AS first_login
FROM Activity
GROUP BY player_id
```

### 512. Game Play Analysis II

```
SELECT  player_id,device_id
FROM Activity
WHERE (player_id,event_date ) 
IN
    (SELECT player_id,
            MIN(event_date) as event_date 
     FROM Activity 
     GROUP BY player_id)
```

### 534. Game Play Analysis III
We use window fiunction to sum as the dates increment
```

SELECT player_id, 
       event_date,
       SUM(games_played) OVER(PARTITION BY player_id ORDER BY event_date) AS  games_played_so_far
FROM Activity
```
### 550. Game Play Analysis IV
```
SELECT 
ROUND(COUNT(b.player_id)/COUNT(DISTINCT a.player_id),2) AS fraction
FROM 
(SELECT player_id,MIN(event_date) as event_date FROM Activity GROUP BY player_id) a
LEFT JOIN  Activity b ON a.event_date +1 = b.event_date 
                     AND a.player_id = b.player_id
```
### 569. Median Employee Salary

```
SELECT id, 
       company, 
       salary 
FROM    ( 
        SELECT * , 
        RANK() OVER (PARTITION BY company ORDER BY salary,id) AS rnk,
        COUNT(*) OVER (PARTITION BY company) AS rc
        FROM Employee
        ) temp
WHERE
rnk IN (rc/2, (rc+1)/2, ( rc+2)/2)
```

### 574. Winning Candidate
```
SELECT name
FROM Candidate 
WHERE ID = 
            (SELECT candidateId
            FROM Vote
            GROUP BY candidateId
            ORDER BY COUNT(candidateId) DESC
            LIMIT 1)
```

### 578. Get Highest Answer Rate Question
```
SELECT question_id as survey_log
FROM (
    
    (SELECT question_id,
            SUM(CASE WHEN action = "answer"
                     THEN 1 ELSE 0
                 END ) AS num ,
            SUM(CASE WHEN action = "show"
                     THEN 1 ELSE 0
                 END ) AS den 
    FROM SurveyLog
    GROUP BY question_id)) temp
ORDER BY (num/den) DESC
LIMIT 1
```

### 602. Friend Requests II: Who Has the Most Friends

```
SELECT id,COUNT(id) AS num
FROM
    (SELECT requester_id as id FROM RequestAccepted
    UNION ALL
    SELECT accepter_id as id FROM RequestAccepted) temp
GROUP BY id
ORDER BY COUNT(id) DESC
LIMIT 1
```
### 608. Tree Node

```
SELECT id,
       CASE
         WHEN p_id IS NULL THEN 'Root'
         WHEN (id IN (SELECT p_id FROM Tree )) THEN 'Inner'
         ELSE 'Leaf'
       END AS type
FROM Tree
ORDER BY id
```
### 1068. Product Sales Analysis I
```
SELECT product_name, 
       year, 
       price
FROM
Sales
JOIN
Product USING (product_id)

```
### 1069. Product Sales Analysis II
```
SELECT product_id,
       SUM(quantity) as total_quantity
FROM Sales
GROUP BY product_id
```

### 1070. Product Sales Analysis III

```
SELECT product_id, 
       year AS first_year, 
       quantity,
       price
FROM Sales
WHERE (product_id, year) IN (SELECT product_id,MIN(year) FROM Sales
                             GROUP BY product_id)
```
### 1075. Project Employees I

```
SELECT project_id, 
       ROUND(AVG(experience_years),2) AS average_years
FROM Project
JOIN Employee USING (employee_id)
GROUP BY project_id
```
### 1076. Project Employees II
```
SELECT project_id 
FROM Project
GROUP BY project_id
HAVING COUNT(employee_id) = (
                                SELECT COUNT(employee_id)
                                FROM Project
                                GROUP BY project_id
                                ORDER BY COUNT(employee_id) DESC
                                LIMIT 1
                             )
```

### 1082. Sales Analysis I

```
SELECT  seller_id
FROM
        (
        SELECT seller_id,
            DENSE_RANK() OVER( ORDER BY SUM(price) DESC) AS rnk
        FROM Sales
        GROUP BY seller_id
        ) temp
WHERE rnk = 1
```
### 1083. Sales Analysis II
```
WITH S8 AS (
        SELECT DISTINCT buyer_id
        FROM Sales 
        JOIN Product USING (product_id)
        WHERE product_name = 'S8'
            ),
    iPhone AS (
        SELECT DISTINCT buyer_id
        FROM Sales
        JOIN Product USING (product_id)
        WHERE product_name = 'iPhone'
            )
SELECT buyer_id  FROM S8
WHERE buyer_id NOT IN (SELECT buyer_id FROM iPhone)
```

### 1097. Game Play Analysis V
First we want to find the min date any player logged.  
Secondly we want to find who all logged on the next day so we join the previous result with Activity with a left join as we want all the day1 logged players.  
Thirdly we count the install as total number of player_id logged on a day from the Activity tble
```

SELECT  a.event_date AS install_dt, 
        COUNT(a.player_id) as installs,
        ROUND(COUNT(b.player_id)/ COUNT(a.player_id),2) AS Day1_retention
FROM 
    (SELECT player_id, MIN(event_date) AS event_date 
    FROM Activity
    GROUP BY player_id) a
    LEFT JOIN Activity b ON a.player_id = b.player_id AND a.event_date +1 = b.event_date
    GROUP BY a.event_date
```
###
```
WITH cte1 AS
        (SELECT * ,sum(quantity) AS qty
        FROM Orders 
        WHERE dispatch_date > '2018-06-23'
        GROUP BY book_id
         )
        ,
     cte2 AS 
        ( SELECT * FROM Books WHERE available_from < '2019-05-23'
        )
SELECT cte2.book_id,name
FROM cte2 
LEFT JOIN cte1 USING (book_id)
WHERE qty < 10 OR qty IS NULL
```

### 1179. Reformat Department Table

```
SELECT id ,
       sum(case when month = "Jan" then  revenue end) as Jan_Revenue,
       sum(case when month = "Feb" then  revenue end) as Feb_Revenue,
       sum(case when month = "Mar" then  revenue end) as Mar_Revenue,
       sum(case when month = "Apr" then  revenue end) as Apr_Revenue,
       sum(case when month = "May" then  revenue end) as May_Revenue,
       sum(case when month = "Jun" then  revenue end) as Jun_Revenue,
       sum(case when month = "Jul" then  revenue end) as Jul_Revenue,
       sum(case when month = "Aug" then  revenue end) as Aug_Revenue,
       sum(case when month = "Sep" then  revenue end) as Sep_Revenue,
       sum(case when month = "Oct" then  revenue end) as Oct_Revenue,
       sum(case when month = "Nov" then  revenue end) as Nov_Revenue,
       sum(case when month = "Dec" then  revenue end) as Dec_Revenue
FROM Department
GROUP BY id
```
### 1251. Average Selling Price

```
SELECT UnitsSold.product_id, 
       ROUND((SUM(units*price)/ SUM(units)),2) AS average_price
FROM UnitsSold
JOIN Prices ON Prices.product_id = UnitsSold.product_id
           AND UnitsSold.purchase_date  BETWEEN start_date AND end_date
GROUP BY product_id
```
### 1285. Find the Start and End Number of Continuous Ranges

```
SELECT MIN(log_id) AS start_id,
       MAX(log_id) AS end_id
FROM (
        SELECT log_id,
            ROW_NUMBER() OVER(ORDER BY log_id) AS rnk
        FROM Logs
        ) temp
GROUP BY log_id -rnk
```
### 1303. Find the Team Size

```
SELECT employee_id, 
       COUNT(team_id) OVER(PARTITION BY team_id )AS team_size
FROM Employee
```
### 1384. Total Sales Amount by Year

```
# start-> end
# 2018 -> 2018- We don't care (2018,2019,2020)

# 2019 -> 2018-2019
# 2019 -> 2019- We don't care(2019,2020)


# 2020 -> 2018-2020
# 2020 -> 2019-2020
# 2020 -> 2020- We don't care (2020)


WITH cte AS
        (
        SELECT *, "2018" AS report_year,
            CASE 
                 WHEN YEAR(period_start) = '2018' 
                 THEN (DATEDIFF(LEAST(period_end,"2018-12-31"),period_start)+1)*average_daily_sales
                 END AS total_amount
        FROM Sales
        UNION ALL 
                SELECT *, "2019" AS report_year,
            CASE 
                 WHEN YEAR(period_start) = '2019' 
                 THEN (DATEDIFF(LEAST(period_end,"2019-12-31"),period_start)+1)*average_daily_sales
                 WHEN YEAR(period_start) = '2018' AND YEAR(period_end) >= '2019'
                 THEN (DATEDIFF(LEAST(period_end,"2019-12-31"),"2019-01-01")+1)*average_daily_sales
                 
                 END AS total_amount
        FROM Sales
        UNION ALL 
                SELECT *, "2020" AS report_year,
            CASE 
                 WHEN YEAR(period_start) = '2020'
                 THEN (DATEDIFF(LEAST(period_end,"2020-12-31"),period_start)+1)*average_daily_sales
                 WHEN YEAR(period_start) = '2018' AND YEAR(period_end) = '2020'
                 THEN (DATEDIFF(period_end,"2020-01-01")+1)*average_daily_sales
                 WHEN YEAR(period_start) = '2019' AND YEAR(period_end) = '2020'
                 THEN (DATEDIFF(period_end,"2020-01-01")+1)*average_daily_sales
                 END AS total_amount
        FROM Sales
            
        )
SELECT product_id,product_name,report_year,total_amount 
FROM cte 
JOIN Product USING (product_id)
WHERE total_amount IS NOT NULL
ORDER BY product_id, report_year
```
### 1501. Countries You Can Safely Invest In

```
WITH temp AS
        (SELECT caller_id AS user,duration FROM Calls
        UNION ALL
        SELECT callee_id AS user, duration FROM Calls) 
SELECT Country.name AS country
FROM temp
JOIN Person on Person.id = temp.user
JOIN Country on left(Person.phone_number,3) = Country.country_code
GROUP BY Country.name
HAVING avg(duration) >  (SELECT AVG(duration) FROM Calls) 
```
### 1965. Employees With Missing Information

```
WITH cte as 
            (SELECT employee_id FROM Employees
            UNION SELECT employee_id FROM Salaries)
SELECT employee_id FROM cte
LEFT JOIN Employees USING (employee_id)
LEFT JOIN Salaries USING (employee_id)
WHERE name IS NULL OR salary IS NULL
ORDER BY employee_id
```
