## Count Occurrences Of Words In Drafts
    SELECT LOWER(jt.word) AS word,
       COUNT(*) AS occurrences
    FROM google_file_store g
    JOIN JSON_TABLE(CONCAT('["', REPLACE(REGEXP_REPLACE(g.contents, '[[:punct:]]', ''), ' ', '","'), '"]'), '$[*]' COLUMNS (word VARCHAR(200) PATH '$')) jt
    GROUP BY LOWER(jt.word)
    ORDER BY occurrences DESC;
