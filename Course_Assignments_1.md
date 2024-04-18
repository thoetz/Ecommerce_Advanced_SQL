# Summary

This markdown file contains SQL queries for traffic analysis, optimization, and web performance evaluation. It includes sections for analyzing traffic sources, trend analysis, page performance analysis, bounce rate analysis, AB testing, and conversion funnel analysis. Each section addresses specific business objectives and provides valuable insights for decision-making and optimization strategies.

## Traffic Analysis and Optimization

```sql
use mavenfuzzyfactory;

-- Section 4 traffic analysis and optimization for optimizing marketing budgets
-- Business concept 1: traffic source analysis (direct, search, social, email)

SELECT website_sessions.utm_content,
COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
COUNT(DISTINCT orders.order_id) AS orders,
COUNT(DISTINCT orders.order_id)/COUNT(DISTINCT website_sessions.website_session_id) AS conversion_rate
FROM website_sessions
	LEFT JOIN orders
		ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.website_session_id BETWEEN 1000 AND 2000
GROUP BY utm_content
ORDER BY sessions DESC;

-- CEO wants to figure out where our web traffic is coming from, she wants UTM source, campaign and referring domain.
SELECT 
	utm_source, 
	utm_campaign, 
	http_referer,
	COUNT(DISTINCT website_session_id) AS sessions
FROM website_sessions
WHERE created_at < '2012-04-12'
GROUP BY 1, 2, 3
ORDER BY sessions DESC;
-- Based on our analysis, gsearch nonbrand is the biggest source

-- Our marketing manager Tom wants us to dig into gsearch nonbrand to see the conversion rate
SELECT 
	COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
    COUNT(DISTINCT orders.order_id) AS orders,
    COUNT(DISTINCT orders.order_id) / COUNT(DISTINCT website_sessions.website_session_id) AS conversion
FROM website_sessions
LEFT JOIN orders
ON  website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2012-04-14' 
	AND utm_source = 'gsearch'
    AND utm_campaign = 'nonbrand';

-- Grouping sessions by year and week to create trend analysis
SELECT 
    Year(created_at),
    Week(created_at),
    MIN(DATE(created_at)) AS week_start,
    COUNT(DISTINCT(created_at)) AS sessions
FROM website_sessions
WHERE website_session_id BETWEEN 100000 AND 115000
GROUP BY 1,2;

-- Pivoting with Count and Case
SELECT 
	primary_product_id,
    Count(DISTINCT CASE WHEN items_purchased = 1 THEN order_id ELSE NULL END) AS orders_1_item,
    Count(DISTINCT CASE WHEN items_purchased = 2 THEN order_id ELSE NULL END) AS orders_2_item,
    Count(DISTINCT order_id) AS total_orders
FROM orders
WHERE order_id BETWEEN 31000 AND 32000
GROUP BY 1;

-- our marketing manager wants us to pull gsearch non-brand trended session volume by week

SELECT 
    utm_source,
    utm_campaign,
    YEAR(created_at),
    Week(created_at),
    MIN(DATE(created_at)) AS start_of_week,
    COUNT(website_session_id) AS sessions
FROM website_sessions
WHERE created_at < '2012-05-10' AND utm_source = 'gsearch' AND utm_campaign = 'nonbrand'
GROUP BY  3,4;
-- It looks like volumes have gone down since he lowered the bid 

-- Traffic source bid optimization assignment
-- Our manager wants us to pull the conversion rates from sessions to order by device type
SELECT
	   device_type,
       COUNT(website_sessions.website_session_id) AS sessions,
       COUNT(orders.order_id) AS total_orders,
       COUNT(orders.order_id) / COUNT(website_sessions.website_session_id) AS conversion_rate
	FROM
		website_sessions
        LEFT JOIN orders
        ON website_sessions.website_session_id = orders.website_session_id
    WHERE website_sessions.created_at < '2012-05-11'    
		AND utm_source = 'gsearch'
        AND utm_campaign = 'nonbrand'
	GROUP BY device_type;
    -- it appears mobile does not perform that well. we should probably increase our bids on desktop
    
    -- Traffic source segment trending assignment
    -- our marketing manager wants us to pull the trends for desktop and mobile so we can see impact on volume
SELECT
    MIN(DATE(created_at)) AS week_start_date,
    Count(DISTINCT CASE WHEN device_type = 'mobile' THEN website_session_id ELSE NULL END) AS mob_sessions,
    Count(DISTINCT CASE WHEN device_type = 'desktop' THEN website_session_id ELSE NULL END) AS desktop_sessions
	FROM
		website_sessions
    WHERE website_sessions.created_at > '2012-04-15'  
		AND website_sessions.created_at < '2012-06-09'    
		AND utm_source = 'gsearch'
        AND utm_campaign = 'nonbrand'
	GROUP BY YEAR(created_at), WEEK(created_at);    
    
-- based off our data we can see that toms change in strategy did result in a positive outcome for desktop]


## Page Performance Analysis

```sql
-- Section 5 analyzing web performance
-- Top pages analysis

SELECT pageview_url,
COUNT(DISTINCT website_pageview_id) AS pvs
FROM website_pageviews
WHERE website_pageview_id < 1000 -- arbitrary
GROUP BY pageview_url
ORDER BY pvs DESC;

-- finding the top entry pages or first pageview that the website session_id sees

CREATE TEMPORARY TABLE first_pageview
SELECT website_session_id,
MIN(website_pageview_id) AS minpv
FROM website_pageviews
WHERE website_pageview_id < 1000 -- arbitrary
GROUP BY website_session_id;

SELECT first_pageview.website_session_id,
website_pageviews.pageview_url AS landing_page,
COUNT(DISTINCT first_pageview.website_session_id) AS sessions_hitting
FROM first_pageview
LEFT JOIN website_pageviews
ON first_pageview.minpv = website_pageviews.website_pageview_id
GROUP BY landing_page;

-- Website manager wants us to get the most viewed website pages ranked by session volume
SELECT 
	COUNT(DISTINCT website_pageview_id) AS total_views,
    pageview_url
FROM website_pageviews
WHERE created_at < '2012-06-09'
GROUP BY pageview_url
ORDER BY total_views DESC;

-- Manager wants the top entry pages on site and ranked by volume
-- First thing we need to do is find the first pageview for the session 

CREATE TEMPORARY TABLE first_pv_per_session -- Then we want to store this query in a temp table to use for a join in another query
SELECT 
	website_session_id,
    MIN(website_pageview_id) AS first_pv
FROM website_pageviews
WHERE created_at < '2012-06-12'
GROUP BY website_session_id;

-- now we will use our temp table to perform a join and execute our query
SELECT 
    pageview_url AS landing_page_url,
    COUNT(distinct first_pv_per_session.website_session_id ) AS sessions_on_landing_page
FROM first_pv_per_session
LEFT JOIN website_pageviews
ON website_pageviews.website_pageview_id = first_pv_per_session.first_pv
GROUP BY pageview_url
ORDER BY sessions_on_landing_page DESC;


## Bounce Rate Analysis

```sql
-- Section 6 analyzing bounce rates
-- Home page bounce rate assignment

SELECT 
	COUNT(website_sessions.website_session_id) AS sessions,
    COUNT(orders.order_id) AS orders,
    COUNT(orders.order_id) / COUNT(website_sessions.website_session_id) AS conversion_rate
FROM website_sessions
LEFT JOIN orders
ON  website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2012-05-12' 
	AND pageview_url = '/'
GROUP BY pageview_url;

-- CEO wants us to measure bounce rate of users who enter site via home page
-- The first thing we need to do is to determine whether or not the pageview was the first pageview of the session.

CREATE TEMPORARY TABLE first_pageview_per_session
SELECT 
	website_session_id,
    MIN(website_pageview_id) AS first_pageview_id
FROM website_pageviews
WHERE created_at < '2012-05-12'
GROUP BY website_session_id;

-- now we can join our temp table to website_pageviews to determine whether or not the session entered via the home page

SELECT 
    COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
    COUNT(DISTINCT CASE WHEN pageview_url = '/' THEN website_sessions.website_session_id ELSE NULL END) AS home_page_views,
    COUNT(DISTINCT CASE WHEN pageview_url = '/' THEN NULL ELSE website_sessions.website_session_id END) AS non_home_page_views,
    COUNT(DISTINCT CASE WHEN pageview_url = '/' THEN NULL ELSE website_sessions.website_session_id END) / COUNT(DISTINCT website_sessions.website_session_id) AS non_home_page_bounce_rate
FROM website_sessions
JOIN first_pageview_per_session
ON website_sessions.website_session_id = first_pageview_per_session.website_session_id
WHERE website_sessions.created_at < '2012-05-12';

-- website manager wants us to find the bounce rate of the home page
-- We will need to pull bounce rate for all pageviews of the home page and compare it to all pageviews of the site

SELECT
	( 
	SELECT 
        COUNT(DISTINCT website_sessions.website_session_id)
    FROM 
        website_sessions
    WHERE 
        website_sessions.created_at < '2012-05-12'
        AND pageview_url = '/'
    ) AS total_sessions_on_hp,
    (
    SELECT
        COUNT(DISTINCT website_sessions.website_session_id)
    FROM
        website_sessions
    WHERE
        website_sessions.created_at < '2012-05-12'
        AND pageview_url = '/'
        AND orders.order_id IS NULL
    ) AS sessions_with_bounce_on_hp,
    (
    SELECT
        COUNT(DISTINCT website_sessions.website_session_id)
    FROM
        website_sessions
    WHERE
        website_sessions.created_at < '2012-05-12'
    ) AS total_sessions,
    (
    SELECT
        COUNT(DISTINCT website_sessions.website_session_id)
    FROM
        website_sessions
    WHERE
        website_sessions.created_at < '2012-05-12'
        AND orders.order_id IS NULL
    ) AS total_sessions_with_bounce,
    (
    SELECT
        COUNT(DISTINCT website_sessions.website_session_id)
    FROM
        website_sessions
    WHERE
        website_sessions.created_at < '2012-05-12'
        AND pageview_url = '/'
        AND orders.order_id IS NULL
    ) / (
    SELECT
        COUNT(DISTINCT website_sessions.website_session_id)
    FROM
        website_sessions
    WHERE
        website_sessions.created_at < '2012-05-12'
        AND pageview_url = '/'
    ) AS home_page_bounce_rate;


## AB Testing

```sql
-- Section 7 ab testing
-- Tom wants to A/B test the new homepage design against the old one
-- The first thing we need to do is pull the performance of both versions of the page
SELECT 
	COUNT(DISTINCT CASE WHEN pageview_url = '/homepage' THEN website_session_id ELSE NULL END) AS old_page_sessions,
    COUNT(DISTINCT CASE WHEN pageview_url = '/new_homepage' THEN website_session_id ELSE NULL END) AS new_page_sessions
FROM website_pageviews
WHERE created_at < '2012-06-10';

-- now that we have the performance of both pages we can calculate conversion rate
-- The business wants us to analyze the impact of the new homepage on order conversion

SELECT
	( 
	SELECT 
		COUNT(DISTINCT CASE WHEN pageview_url = '/homepage' THEN website_sessions.website_session_id ELSE NULL END)
	FROM 
		website_sessions
	WHERE 
		created_at < '2012-06-10'
    ) AS old_page_sessions,
    (
    SELECT 
		COUNT(DISTINCT CASE WHEN pageview_url = '/new_homepage' THEN website_sessions.website_session_id ELSE NULL END)
	FROM 
		website_sessions
	WHERE 
		created_at < '2012-06-10'
    ) AS new_page_sessions,
    (
    SELECT 
		COUNT(DISTINCT orders.order_id)
	FROM 
		orders
	JOIN 
		website_sessions 
	ON 
		orders.website_session_id = website_sessions.website_session_id
	WHERE 
		created_at < '2012-06-10'
		AND pageview_url = '/homepage'
    ) AS old_page_orders,
    (
    SELECT 
		COUNT(DISTINCT orders.order_id)
	FROM 
		orders
	JOIN 
		website_sessions 
	ON 
		orders.website_session_id = website_sessions.website_session_id
	WHERE 
		created_at < '2012-06-10'
		AND pageview_url = '/new_homepage'
    ) AS new_page_orders;
-- The data shows that the new homepage did not perform well in terms of conversion


## Conversion Funnel Analysis

```sql
-- Section 8 conversion funnel analysis
-- Analysis of conversion funnel
-- Our marketing manager wants us to pull the trends for desktop and mobile so we can see impact on volume

SELECT 
	COUNT(DISTINCT CASE WHEN step = 1 THEN website_session_id ELSE NULL END) AS step_1,
    COUNT(DISTINCT CASE WHEN step = 2 THEN website_session_id ELSE NULL END) AS step_2,
    COUNT(DISTINCT CASE WHEN step = 3 THEN website_session_id ELSE NULL END) AS step_3,
    COUNT(DISTINCT CASE WHEN step = 4 THEN website_session_id ELSE NULL END) AS step_4,
    COUNT(DISTINCT CASE WHEN step = 5 THEN website_session_id ELSE NULL END) AS step_5
FROM funnel_events
WHERE created_at < '2012-06-10'
GROUP BY 1, 2;

-- CEO wants to see the trends of people going through our conversion funnel

SELECT
	YEAR(created_at),
    Week(created_at),
    COUNT(DISTINCT CASE WHEN step = 1 THEN website_session_id ELSE NULL END) AS step_1,
    COUNT(DISTINCT CASE WHEN step = 2 THEN website_session_id ELSE NULL END) AS step_2,
    COUNT(DISTINCT CASE WHEN step = 3 THEN website_session_id ELSE NULL END) AS step_3,
    COUNT(DISTINCT CASE WHEN step = 4 THEN website_session_id ELSE NULL END) AS step_4,
    COUNT(DISTINCT CASE WHEN step = 5 THEN website_session_id ELSE NULL END) AS step_5
FROM funnel_events
GROUP BY 1, 2;

-- Our marketing manager Tom wants to see the session funnel volume by week

SELECT
	YEAR(created_at),
    Week(created_at),
    COUNT(DISTINCT website_session_id) AS sessions
FROM funnel_events
GROUP BY 1, 2;


## Conclusion

These SQL queries provide valuable insights for traffic analysis, web performance evaluation, bounce rate analysis, AB testing, and conversion funnel analysis. By analyzing traffic sources, trend analysis, page performance, bounce rates, AB testing results, and conversion funnel trends, businesses can make informed decisions to optimize marketing strategies, improve website performance, and enhance overall conversion rates.



