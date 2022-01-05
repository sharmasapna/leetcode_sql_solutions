# Solution to Leetcode SQL Questions 

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
### 1097. Game Play Analysis V
Sirst we want to find the min date any player logged.  
secondly we want to find who all logged on the next day so we join the previous result with Activity with a left join as we want all the day1 logged players.  
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
