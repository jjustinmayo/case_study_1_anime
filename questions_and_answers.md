# UFC Fight Data 2009-2021

## Questions and Answers

Author: Justin Mayo

Email: jjustinmayo@gmail.com

LinkedIn: https://www.linkedin.com/in/jjustinmayo/

### How many total fights has there been since 2009? Of this number, how many were title/non-title fights?

```
SELECT
	count(*) AS "total number of fights",
    count(CASE WHEN title_bout = 'True' THEN date END) AS "total number of title fights",
    count(CASE WHEN title_bout = 'False' THEN date END) AS "total number of non-title fights"
FROM ufc_data.fight_data;
```

| total number of fights | total number of title fights | total number of non-title fights |
| ---------------------- | ---------------------------- | -------------------------------- |
| 4953                   | 297                          | 4656                             |

### 