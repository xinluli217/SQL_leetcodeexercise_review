## Users By Average Session Time
    with loads as(
    select user_id, DATE(timestamp) as day,
    max(timestamp) as load_time
    from facebook_web_log
    where action = 'page_load'
    group by user_id, DATE(timestamp)),

    exits as(
    select user_id, DATE(timestamp) as day,
    min(timestamp) as exit_time
    from facebook_web_log
    where action = 'page_exit'
    group by user_id, DATE(timestamp)),

    sessions as(
    select l.user_id,
    l.load_time, e.exit_time, TIMESTAMPDIFF(SECOND, l.load_time, e.exit_time) AS session_duration
    from loads l
    join exits e on l.user_id = e.user_id AND l.day = e.day
    WHERE l.load_time < e.exit_time)
  
    SELECT
    user_id,
    AVG(session_duration) AS avg_session_duration
    FROM sessions
    GROUP BY user_id;
