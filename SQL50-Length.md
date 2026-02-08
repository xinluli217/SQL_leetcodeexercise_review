## Length 
# Example:
    select tweet_id
    from Tweets
    where LENGTH(content)>15;

# Character length:
    CHAR_LENGTH() / CHARACTER_LENGTH()
  CHAR_LENGTH counts characters, while LENGTH may count bytes depending on the database.

# OCTET_LENGTH() / BYTE_LENGTH()
    OCTET_LENGTH() / BYTE_LENGTH()
  BYTE_LENGTH is useful when storage size matters rather than visible characters.

# TRIM() + LENGTH()
    TRIM() + LENGTH()
  Length without the gap before and after the character

# SUBSTRING() / LEFT() / RIGHT()
    SUBSTRING(content, 1, 15)
    LEFT(content, 15)
    RIGHT(content, 15)
truncate text/preview text
