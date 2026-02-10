## 1Users By Average Session Time
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

## 2Finding User Purchases

① daily：把“同一天多次购买”压缩成一天一次
    
    WITH daily AS (
    SELECT DISTINCT user_id, DATE(created_at) AS purchase_date
    FROM amazon_transactions
    )

② ranked：给每个用户的购买日期排顺序

    ranked AS (
    SELECT
    user_id,
    purchase_date,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY purchase_date) AS rn
    FROM daily
    )

PARTITION BY user_id：每个用户单独算
ORDER BY purchase_date：按时间从早到晚
ROW_NUMBER()：
第一次购买 → rn = 1
第二次购买 → rn = 2

③ first_two：把前两次购买“横着摊开”

    first_two AS (
    SELECT
    user_id,
    MAX(CASE WHEN rn = 1 THEN purchase_date END) AS first_date,
    MAX(CASE WHEN rn = 2 THEN purchase_date END) AS second_date
    FROM ranked
    WHERE rn <= 2
    GROUP BY user_id
    )
④ 最终筛选：是否在 1–7 天内完成第二次购买

    SELECT user_id
    FROM first_two
    WHERE second_date IS NOT NULL
    AND DATEDIFF(second_date, first_date) BETWEEN 1 AND 7
    ORDER BY user_id;
    
## 3Acceptance Rate By Date
1.所有被接受的好友请求

    with a as(
    select *  
    from fb_friend_requests 
    where action='accepted'),
2. 所有发出的好友请求
   
       b as(
       select *  
       from fb_friend_requests  
       where action='sent')

3. RIGHT JOIN 把 sent 当分母
 RIGHT JOIN 的含义
以 b（sent）为主表
即使某个请求没被接受（a 中不存在）
也要保留这条 sent 记录


       select b.date,count(a.user_id_receiver)/count(b.user_id_sender)
       from a
       right join b on a.user_id_sender=b.user_id_sender
       and a.user_id_receiver=b.user_id_receiver group by date;

## 4Workers With The Highest Salaries

    SELECT b.worker_title AS best_paid_title
    FROM worker a
    JOIN title b
    ON a.worker_id = b.worker_ref_id
    WHERE a.salary = (
    SELECT MAX(w.salary)
    FROM worker w
    JOIN title t
    ON w.worker_id = t.worker_ref_id
    WHERE t.worker_title IS NOT NULL
    )
    ORDER BY best_paid_title;


## 
错

    with a as(
    select c.first_name,SUM(o.total_order_cost) as total_cost,o.order_date
    from customers c
    join orders o on c.id=o.cust_id
    where o.order_date between '2019-02-01' and '2019-05-01'
    group by o.order_date,c.first_name
    order by o.order_date)
    select first_name,order_date,MAX(total_cost) as max_cost
    from a
    group by order_date;

正确：

    WITH a AS (
    SELECT
    c.id AS cust_id,
    c.first_name,
    o.order_date,
    SUM(o.total_order_cost) AS total_cost
    FROM customers c
    JOIN orders o ON c.id = o.cust_id
    WHERE o.order_date BETWEEN '2019-02-01' AND '2019-05-01'
    GROUP BY c.id, c.first_name, o.order_date
    ),
    ranked AS (
     SELECT *,
         RANK() OVER (
           PARTITION BY order_date
           ORDER BY total_cost DESC
         ) AS rnk
    FROM a
    )
    SELECT first_name, order_date, total_cost AS max_cost
    FROM ranked
    WHERE rnk = 1
    ORDER BY order_date;
