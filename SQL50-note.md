## Length 
### Example:
    select tweet_id
    from Tweets
    where LENGTH(content)>15;

## Character length:
    CHAR_LENGTH() / CHARACTER_LENGTH()
  CHAR_LENGTH counts characters, while LENGTH may count bytes depending on the database.

### OCTET_LENGTH() / BYTE_LENGTH()
    OCTET_LENGTH() / BYTE_LENGTH()
  BYTE_LENGTH is useful when storage size matters rather than visible characters.

### TRIM() + LENGTH()
    TRIM() + LENGTH()
  Length without the gap before and after the character

### SUBSTRING() / LEFT() / RIGHT()
    SUBSTRING(content, 1, 15)
    LEFT(content, 15)
    RIGHT(content, 15)
truncate text/preview text

## DATE_
DATE_SUB
DATE_ADD
DARE_

## PARTITION BY

    FUNCTION() OVER (PARTITION BY column ORDER BY column)
    
PARTITION BY is used in window functions to divide rows into separate groups (partitions).

Each partition is processed independently, but rows are not collapsed like with GROUP BY.

It allows calculations within each group while keeping all rows.

GROUP BY reduces rows by grouping them.

PARTITION BY keeps all rows and adds calculated values per group.


## DENSE_RANK vs. ROW_NUMBER
DENSE_RANK is used to include all tied values when selecting the top N results, whereas ROW_NUMBER may arbitrarily exclude ties.
