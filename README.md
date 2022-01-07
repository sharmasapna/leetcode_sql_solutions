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
### 180. Consecutive Numbers
```
SELECT DISTINCT l1.num AS ConsecutiveNums
from 
Logs l1
JOIN Logs l2 on l2.id = l1.id +1 AND l1.num = l2.num
JOIN Logs l3 on l3.id = l1.id +2 AND l1.num = l3.num
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
