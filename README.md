# SQL-data-challange-datalemur- DB used - Postgree SQL

# Linkedin - Data science skills question Linkedin 

With T as (SELECT candidate_id, count(skill) FROM candidates
where skill IN ('Python', 'Tableau', 'PostgreSQL') 
GROUP BY candidate_id
HAVING COUNT(skill) = 3
ORDER BY candidate_id)
select Candidate_id from T;



# Facebook - pages with no likes question

with T as (SELECT pages.page_id, COUNT(page_likes.user_id) 
FROM pages LEFT JOIN page_likes
on pages.page_id = page_likes.page_id
GROUP BY pages.page_id
HAVING COUNT(page_likes.user_id) < 1)
select page_id from T;



#Tesla unfinished part question

SELECT distinct(part) FROM parts_assembly
where finish_date is NULL;



# New york times - Mobile vs Laptop viewership question

select count (DISTINCT user_id) where device_type = 'laptop' as laptop_views,
count (DISTINCT user_id) where device_type in ('tablet', 'phone') as mobile_views
from viewership;



# Linkedin - Duplicate job listings question

with T as (select company_id, title, description, count(title)
from job_listings
group by company_id, title, description
having COUNT(title)>1)
SELECT count(title) as duplicate_companies
from T;



# Facebook - Average post hiatus question

SELECT user_id, EXTRACT(day from (max(post_date)-min(post_date))) as duration
FROM posts
where EXTRACT(year from (post_date)) = 2021
group by user_id
having count(post_id) > 1;



# Microsoft - Team power users question

SELECT sender_id, count(message_id)
FROM messages
where EXTRACT(month from sent_date) = 08 and EXTRACT(year from (sent_date)) = 2022
group by sender_id
order by count(message_id) DESC
limit 2;


# Robinhood - Cities with completed trade

SELECT users.city, count(trades.user_id) FROM trades JOIN users
on trades.user_id = users.user_id
where status = 'Completed'
group by users.city
ORDER BY count(trades.user_id) desc
limit 3;



# Amazon - Average Review rating

SELECT EXTRACT(month from submit_date) as mth, product_id, ROUND(AVG(stars), 2)
from reviews
GROUP BY EXTRACT(month from submit_date), product_id
order by EXTRACT(month from submit_date), product_id;



# Facebook - App click through rate question

with T as (select app_id, 
count(case when event_type = 'impression' then event_type end) as imp,
count(case when event_type = 'click' then event_type  end) as clk
from events
where EXTRACT(year from timestamp) = 2022
group by  app_id)
select app_id, round(100.0*(CAST(clk as float)/cast(imp as float))::numeric,2) as ctr
from T;



# Tik-Tok second day confirmation

SELECT user_id FROM emails
JOIN texts
on emails.email_id = texts.email_id
where signup_action = 'Confirmed' and (action_date::date - signup_date::date) = 1;


# J.P. Morgan Chase - Cards Isuued difference question

SELECT card_name, 
case 
when card_name = 'Chase Freedom Flex' then MAX(issued_amount) - MIN(issued_amount)
when card_name = 'Chase Sapphire Reserve' then MAX(issued_amount) - MIN(issued_amount)
end as Cnt 
FROM monthly_cards_issued
GROUP BY card_name
order by cnt desc;


# Alibaba - Compressed mean question

with T as
(SELECT cast(sum(item_count * order_occurrences) as decimal) as total_items,
cast(SUM(order_occurrences) as decimal) as cnt  FROM items_per_order)
select ROUND((total_items/cnt),1) as mean from T;



# CVS - health : Pharmacy Analytics 1 Question

SELECT drug, sum(total_sales-cogs) as total_profit
FROM pharmacy_sales
GROUP BY drug
having COUNT( DISTINCT manufacturer) = 1
order by total_profit desc
limit 3;



# CVS - health : Pharmacy Analytics - 2 Question

select manufacturer, count(product_id), sum(cogs - total_sales) as total_loss
from pharmacy_sales
where cogs - total_sales > 0
group by manufacturer
order by sum(cogs - total_sales) desc;



# CVS - health : Pharmacy Analytics - 3 Question

SELECT manufacturer, '$'|| ROUND(sum(total_sales/1000000),0) ||' '|| 'million' FROM pharmacy_sales
group by manufacturer
order by sum(total_sales) desc;



c

with t as 
(SELECT policy_holder_id, count(case_id) 
FROM callers
group by policy_holder_id
having count(case_id) >= 3)
select count(policy_holder_id)
from t;



# United Health - Patient Support Analysis - 2

select ROUND(100.0*(select Count(case_id) from callers
where call_category = 'n/a' or call_category is NULL)/count(case_id),1)
from callers;



# Uber - User's third Transaction 

with t as
(select user_id, spend, transaction_date,
RANK() over(partition by user_id order by transaction_date) as rnk
from transactions) select user_id, spend, transaction_date
from t
where rnk = 3;



# Snapchat - Sending VS Opening Snaps

WITH S as (SELECT age_bucket,
sum(case when activity_type = 'send'
then time_spent else 0 end) as totalsendtime,
sum(case when activity_type = 'open'
then time_spent else 0 end) as totalopentime,
sum(time_spent) as total_time
from activities 
JOIN age_breakdown 
on activities.user_id = age_breakdown.user_id
where activity_type IN('send', 'open')
GROUP BY age_bucket)

select age_bucket, 
ROUND(100.0*totalsendtime/total_time, 2) as send_perc, ROUND(100.0*totalopentime/total_time, 2) as open_perc
from S;



# Twitter - 3 day rolling tweets

with t as 
(SELECT user_id,  tweet_date,
COUNT(tweet_id) cnt
from tweets
GROUP BY user_id, tweet_date
order by user_id, tweet_date) 

select user_id, tweet_date,
ROUND(avg(cnt) over(partition by user_id order by tweet_date rows between 2 preceding and current row ), 2)
from t;



# Amazon - Highest grossing items

select category, product, total_spend
from
(SELECT category, product, sum(spend) total_spend,
dense_rank() over(PARTITION BY category order by sum(spend) desc)
FROM product_spend
where date_part('year', transaction_date) = 2022 
group by category, product
) 
as T
where dense_rank in (1,2)



# Spotify - Top 5 artists

with t as
(select artists.artist_id a, artists.artist_name b, songs.song_id c, global_song_rank.day d, global_song_rank.rank e
FROM artists
join songs
on artists.artist_id = songs.artist_id
join global_song_rank 
on songs.song_id = global_song_rank.song_id
where global_song_rank.rank <11),
n as
(select b, count(c), dense_rank() over(order by count(c) desc ) artist_rank
from t  
group by b)
select b, artist_rank 
from n  
where artist_rank <= 5




























