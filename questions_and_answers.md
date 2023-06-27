# UFC Fight Data 2009-2021

## Questions and Answers

Author: Justin Mayo

Email: jjustinmayo@gmail.com

LinkedIn: https://www.linkedin.com/in/jjustinmayo/

#### 1. How many total fights has there been since 2009? Of this number, how many were title/non-title fights?

```sql
SELECT
    count(*) AS "total number of fights",
    count(CASE WHEN title_bout = 'True' THEN date END) AS "total number of title fights",
    count(CASE WHEN title_bout = 'False' THEN date END) AS "total number of non-title fights"
FROM ufc_data.fight_data;
```

| total number of fights | total number of title fights | total number of non-title fights |
| ---------------------- | ---------------------------- | -------------------------------- |
| 4953                   | 297                          | 4656                             |

#### 2. Which location/venue does the UFC typically host title fights?

```sql
SELECT
	count(*) AS 'title fight locations',
    location
FROM
	ufc_data.fight_data
WHERE 
	title_bout = 'True'
GROUP BY
	location
ORDER BY 
	COUNT('title fight locations') DESC
```

| \# of title fights | location                                   |
| ------------------ | ------------------------------------------ |
| 147                | Las Vegas, Nevada, USA                     |
| 10                 | Atlantic City, New Jersey, USA             |
| 10                 | Toronto, Ontario, Canada                   |
| 9                  | Anaheim, California, USA                   |
| 8                  | Abu Dhabi, Abu Dhabi, United Arab Emirates |
| 8                  | Los Angeles, California, USA               |
| 7                  | Houston, Texas, USA                        |
| 7                  | New York City, New York, USA               |
| 7                  | Montreal, Quebec, Canada                   |
| 6                  | Birmingham, Alabama, USA                   |
| 146                | 68 other locations                         |

#### 3. What is the breakdown of wins/defences and draws for red corner fighters (i.e. champions) in Las Vegas?

```sql
SELECT 
    COUNT(Winner) as '# of fights',
	Winner as Result
FROM ufc_data.fight_data
WHERE
	title_bout = 'True'
GROUP BY 
	Winner;
```

| Winner | \# of fights |
| ------ | ------------ |
| Red    | 291          |
| Blue   | 69           |
| Draw   | 5            |

#### 4. What was the breakdown of victory type for each red corner fighter?

For this question, I join fight_data with raw_total_fight_data to get data such as type of win, the winner of each contest, etc.

```sql
-- First step: Create a temporary table from raw_total_fight_data with relevant columns. A primary key needs to be created since there is no unique primary key to connect both datasets.
DROP TABLE IF EXISTS final_results;
CREATE TEMPORARY TABLE final_results AS (
	SELECT
		R_fighter,
        B_fighter,
        Referee,
        DATE_FORMAT(STR_TO_DATE(date, '%M %d, %Y'), '%Y-%m-%d') AS formatted_date,
        win_by,
        last_round,
        R_SIG_STR_pct,
        R_TD_pct,
        R_CTRL,
        Winner
        
	FROM 
		ufc_data.raw_total_fight_data
);

-- Add a new column to 'final_results' table for the primary key
ALTER TABLE final_results
ADD COLUMN primary_key CHAR(100);

-- Update the column for the calculation of primary key
UPDATE final_results
SET primary_key = CONCAT(R_fighter, '_', B_fighter, '_', Referee, '_', formatted_date);

-- repeat primary key steps for the 'fight_data' table
ALTER TABLE ufc_data.fight_data
ADD COLUMN primary_key CHAR(100);

UPDATE fight_data
SET primary_key = CONCAT(R_fighter, '_', B_fighter, '_', Referee, '_', date);
```

With the new primary key now created, we are able to join the tables:

```sql
DROP TABLE IF EXISTS combined_table;
CREATE TEMPORARY TABLE combined_table AS (
	SELECT
		fd.R_fighter,
        fd.B_fighter,
        fd.Referee,
        fd.date,
        fr.Winner,
        fr.win_by,
        fd.R_Stance,
        fr.last_round,
        fr.R_SIG_STR_pct,
        fr.R_TD_pct,
        fr.R_CTRL
        
	FROM 
	ufc_data.fight_data AS fd
	JOIN final_results AS fr
    ON fd.primary_key = fr.primary_key
    
    WHERE 
		fr.Winner = fd.R_fighter
        AND fd.title_bout = 'TRUE'
);
```

| R_fighter            | B_fighter       | Referee      | date       | Winner               | win_by               | R_Stance | last_round | R_SIG_STR_pct | R_TD_pct | R_CTRL |
| -------------------- | --------------- | ------------ | ---------- | -------------------- | -------------------- | -------- | ---------- | ------------- | -------- | ------ |
| Jan Blachowicz       | Israel Adesanya | Herb Dean    | 2021-03-06 | Jan Blachowicz       | Decision - Unanimous | Orthodox | 5          | 55%           | 60%      | 7:06   |
| Amanda Nunes         | Megan Anderson  | Jason Herzog | 2021-03-06 | Amanda Nunes         | Submission           | Orthodox | 1          | 72%           | \---     | 0:42   |
| Kamaru Usman         | Gilbert Burns   | Herb Dean    | 2021-02-13 | Kamaru Usman         | KO/TKO               | Switch   | 3          | 61%           | \---     | 2:05   |
| Valentina Shevchenko | Jennifer Maia   | Herb Dean    | 2020-11-21 | Valentina Shevchenko | Decision - Unanimous | Southpaw | 5          | 52%           | 83%      | 9:34   |
| Deiveson Figueiredo  | Alex Perez      | Marc Goddard | 2020-11-21 | Deiveson Figueiredo  | Submission           | Orthodox | 1          | 62%           | \---     | 0:00   |

We are able to find summarize the total number of victory types for every red corner fighter since 1993

```sql
SELECT 
 	win_by as 'red corner wins by',
	COUNT(win_by) as 'number of wins'
FROM combined_table

GROUP BY
	win_by
ORDER BY
	COUNT('total wins') DESC;
```

| red corner wins by      | number of wins |
| ----------------------- | -------------- |
| KO/TKO                  | 103            |
| Decision - Unanimous    | 90             |
| Submission              | 71             |
| Decision - Split        | 17             |
| TKO - Doctor's Stoppage | 7              |
| Decision - Majority     | 3              |

#### 5. What did the red corner victors average across significant strikes landed, takedown success rate, and control time from 1993-2021?

```sql
SELECT 
	AVG(R_SIG_STR_pct) as 'avg sig. strikes landed (%)',
	AVG(R_TD_pct) as 'avg takedown (%)',
	AVG(R_CTRL) as 'avg control time (mins)' 
FROM 
	combined_table
```

| avg sig. strikes landed (5rds) | avg takedowns (5rds) | avg control time (5rds) |
| ------------------------------ | -------------------- | ----------------------- |
| 53.84%                         | 39.45%               | 3.56 mins               |