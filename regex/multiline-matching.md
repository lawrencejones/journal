#!/usr/bin/env ../bin/run-md-bash
# Matches multiple lines, with optional run

Assuming we have a file that looks like below...

```bash
fileContent="""

-- Q1 returns (...)

SELECT * FROM table;

-- Q2 returns (state_name, name)

SELECT state.name AS state_name
     , place.name AS name
FROM place JOIN state ON place.state_code=state.code
WHERE
  place.name LIKE '%City'
  AND
  place.type <> 'city';

"""
```

If we wish to extract just the SQL query, to run through psql, then we can do this...

```bash

Q="2"  # question number

echo "$fileContent" \
  | ggrep -Pzo "\-\- Q$Q([^;])*;"
# | psql  # to run

```
