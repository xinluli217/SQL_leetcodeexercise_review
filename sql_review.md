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
or 第二种解法

    with ordered_tx as (
    select user_id, date(created_at)as tx_date,
    lag(date(created_at)) over (
    PARTITION BY user_id ORDER BY created_at)AS prev_tx_date 对每个用户，按时间排序，取“上一笔交易日期”
    FROM amazon_transactions)
    select distinct user_id
    from ordered_tx
    where prev_tx_date is not null
    and DATEDIFF(tx_date, prev_tx_date) > 0
    and  DATEDIFF(tx_date, prev_tx_date) <=7;
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

## Finding Updated Records
    SELECT id,
       first_name,
       last_name,
       department_id,
       salary
    FROM (
    SELECT *,
         ROW_NUMBER() OVER (PARTITION BY id ORDER BY salary DESC, department_id DESC) AS rn
     FROM ms_employee_salary
    ) s
    WHERE rn = 1
    ORDER BY id ASC;

## Highest Cost Orders
1.每个客户在每一天的总消费金额

    WITH customer_daily_totals AS (
    SELECT
    o.cust_id,
    o.order_date,
    SUM(o.total_order_cost) AS total_daily_cost
    FROM orders o
    WHERE o.order_date BETWEEN '2019-02-01' AND '2019-05-01'
    GROUP BY o.cust_id, o.order_date
    ),
2.对每一天的客户按消费金额从大到小排名

    ranked_daily_totals AS (
    SELECT
    cust_id,
    order_date,
    total_daily_cost,
    RANK() OVER (
      PARTITION BY order_date
      ORDER BY total_daily_cost DESC
    ) AS rnk  ##每一天单独排名
     FROM customer_daily_totals
    )
    
3Step 3 — 最终筛选

    SELECT
     c.first_name,
    rdt.order_date,
    rdt.total_daily_cost AS max_cost
    FROM ranked_daily_totals rdt
    JOIN customers c ON rdt.cust_id = c.id
    WHERE rdt.rnk = 1
    ORDER BY rdt.order_date;


## 最末尾插入排名

    select distinct from_user as user_id,count(*) as total_emails,    
    ROW_NUMBER() OVER (
        ORDER BY COUNT(*) DESC, from_user ASC
    ) AS activity_rank
    from google_gmail_emails
    group by from_user
    order by total_emails desc;

      DENSE_RANK() OVER (ORDER BY sum(n_messages) DESC) AS ranking,插入排名
      
##Risky Projects

    SELECT a.title,
       a.budget,
       CEILING(DATEDIFF(a.end_date, a.start_date) * SUM(c.salary) / 365) AS    向上取整。prorated_employee_expense
    FROM linkedin_projects a
    INNER JOIN linkedin_emp_projects b ON a.id = b.project_id
    INNER JOIN linkedin_employees c ON b.emp_id = c.id
    GROUP BY a.title,
         a.budget,
         a.end_date,
         a.start_date
    HAVING CEILING(DATEDIFF(a.end_date, a.start_date) * SUM(c.salary) / 365) > a.budget
    ORDER BY a.title ASC;

## 

     WITH cte1 AS
    (SELECT CASE
              WHEN REGEXP_LIKE(business_address, "^[0-9]") = 1 THEN   substring_index(substring_index(business_address, ' ', 2), ' ', -1)
              ELSE substring_index(business_address, ' ', 1)
          END AS street_name,
          business_postal_code AS postal_code
    FROM sf_restaurant_health_violations)

建立一个临时表 cte1，里面只有两列：
street_name（提取后的街道名）
postal_code
👉 把地址先处理干净，再统计。

    SELECT postal_code,
       count(DISTINCT street_name) AS num
    FROM cte1
    WHERE postal_code IS NOT NULL
    GROUP BY postal_code
    ORDER BY num DESC,
         postal_code ASC

##  Income By Title and Gender
    WITH bonus_totals AS (
    SELECT worker_ref_id,
           SUM(bonus) AS ttl_bonus
    FROM sf_bonus
    GROUP BY worker_ref_id
    ),
    compensation_data AS (
    SELECT e.employee_title,
           e.sex,
           e.salary + b.ttl_bonus AS total_compensation
    FROM sf_employee e
    INNER JOIN bonus_totals b
    ON e.id = b.worker_ref_id
    )
    SELECT employee_title,
       sex,
       AVG(total_compensation) AS avg_compensation
    FROM compensation_data
    GROUP BY employee_title, sex;

##Reviews of Categories

    with recursive num (n) as (
    -- create list 1 - 15
    select 1
    union all
    select n+1 from num where n<12
    )
    select 
    substring_index(substring_index(categories,';',n),';',-1) as category,
    sum(review_count) as review_cnt
    from 
    yelp_business
    inner join 
    num
    on 
    n <= char_length(categories) - char_length(replace(categories,';','')) + 1
    group by
    category

    
