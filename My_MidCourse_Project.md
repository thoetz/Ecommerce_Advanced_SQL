# Monthly Trends Analysis Summary

## Objective
The objective of this analysis is to assess the monthly trends in website sessions and orders, focusing primarily on Gsearch as the source. The analysis aims to provide insights into the performance of Gsearch sessions and orders over time, as well as to explore specific segments within Gsearch traffic, such as nonbrand and brand campaigns, and device types. Additionally, the analysis examines the contribution of Gsearch traffic compared to other channels and evaluates the impact of website performance improvements and landing page tests on conversion rates and revenue.

## Questions to be Answered

- Q1 Gsearch seems to be the biggest driver of our business. Could you pull monthly trends for gsearch sessions and orders so that we can showcase the growth there?
- Q2 Next, it would be great to see a similar monthly trend for Gsearch, but this time splitting out nonbrand and brand campaigns separately. I am wondering if brand is picking up at all. If so, this is a good story to tell.
- Q3 While we’re on Gsearch, could you dive into nonbrand, and pull monthly sessions and orders split by device type? I want to flex our analytical muscles a little and show the board we really know our traffic sources.
- Q4 I’m worried that one of our more pessimistic board members may be concerned about the large % of traffic from Gsearch. Can you pull monthly trends for Gsearch, alongside monthly trends for each of our other channels?
- Q5 I’d like to tell the story of our website performance improvements over the course of the first 8 months. Could you pull session-to-order conversion rates, by month?
- Q6 For the landing page test you analyzed previously, it would be great to show a full conversion funnel from each of the two pages to orders. You can use the same time period you analyzed last time (Jun 19 Jul 28).
- Q7 For the Gsearch lander test, please estimate the revenue that test earned us. Hint: Look at the increase in CVR from the test (Jun 19 Jul 28), and use nonbrand sessions and revenue since then to calculate incremental value.

# SQL Programming Concepts and Techniques

1. **SELECT statements**: Used to retrieve data from the database tables.
2. **YEAR() and MONTH() functions**: Extract year and month components from dates.
3. **COUNT() function**: Counts the number of rows or values in a specified column.
4. **DISTINCT keyword**: Filters out duplicate values in the result set.
5. **LEFT JOIN**: Joins two tables based on a condition specified after the ON keyword, returning all rows from the left table and matching rows from the right table.
6. **GROUP BY**: Groups rows that have the same values into summary rows, typically used with aggregate functions like COUNT() to perform operations on each group.
7. **Temporary Tables**: Created using the `CREATE TEMPORARY TABLE` statement to store intermediate results for further processing within the session.
8. **CASE WHEN statements**: Used for conditional logic, allowing different actions to be taken based on specified conditions.
9. **Aggregate Functions**: Used to perform calculations on a set of values and return a single value. In this case, COUNT() is used as an aggregate function.
10. **Subqueries**: Not explicitly used in the provided code, but could be utilized in future queries to perform nested queries within the main query.
11. **Mathematical Operations**: Used to calculate conversion rates and other derived metrics.
12. **Date Filtering**: Filtering data based on specific date ranges using the WHERE clause.

These concepts and techniques demonstrate a strong understanding of SQL fundamentals and advanced data analysis capabilities. You can list them in your portfolio to showcase your proficiency in SQL programming and data analysis.

## SQL Queries

### Q1: Monthly Trends for Gsearch Sessions and Orders
- Identifies monthly trends for Gsearch sessions and orders, showcasing growth.
```sql
SELECT 
	year(website_sessions.created_at) AS year,
    MONTH(website_sessions.created_at) AS month,
    COUNT(DISTINCT website_sessions.website_session_id) AS total_sessions,
    COUNT(DISTINCT orders.order_id) AS total_orders,
    COUNT(DISTINCT orders.order_id) / COUNT(DISTINCT website_sessions.website_session_id) AS conv_rt
FROM website_sessions
LEFT JOIN orders
	ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
AND website_sessions.utm_source = 'gsearch'
GROUP BY 1,2;
```

### Q2: Monthly Trends for Gsearch, Split by Nonbrand and Brand Campaigns
- Examines monthly trends for Gsearch, distinguishing between nonbrand and brand campaigns.
```sql
SELECT 
	year(website_sessions.created_at) AS year,
    MONTH(website_sessions.created_at) AS month,
    COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN website_sessions.website_session_id ELSE NULL END) as nonbrand_sessions,
    COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN website_sessions.website_session_id ELSE NULL END) as brand_sessions,
    COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN order_id ELSE NULL END) as nonbrand_orders,
    COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN order_id ELSE NULL END) as brand_orders
FROM website_sessions
LEFT JOIN orders
	ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
AND website_sessions.utm_source = 'gsearch'
AND website_sessions.utm_campaign IN ('nonbrand','brand')
GROUP BY 1,2;
```

### Q3: Monthly Sessions and Orders Split by Device Type for Gsearch Nonbrand
- Provides monthly sessions and orders split by device type for Gsearch nonbrand traffic.
```sql
SELECT 
	year(website_sessions.created_at) AS year,
    MONTH(website_sessions.created_at) AS month,
    COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_sessions.website_session_id ELSE NULL END) as mobile_sessions,
    COUNT(DISTINCT CASE WHEN device_type = 'desktop' THEN website_sessions.website_session_id ELSE NULL END) as desktop_sessions,
    COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN order_id ELSE NULL END) as mobile_orders,
    COUNT(DISTINCT CASE WHEN device_type = 'desktop' THEN order_id ELSE NULL END) as desktop_orders
FROM website_sessions
LEFT JOIN orders
	ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
AND website_sessions.utm_source = 'gsearch'
GROUP BY 1,2;
```

### Q4: Monthly Trends for Gsearch Compared to Other Channels
- Compares monthly trends for Gsearch with other channels, including paid and organic search.
```sql
SELECT distinct
	utm_source,
    utm_campaign,
    http_referer
FROM website_sessions
WHERE website_sessions.created_at < '2012-11-27';

SELECT 
	year(website_sessions.created_at) AS year,
    MONTH(website_sessions.created_at) AS month,
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' THEN website_sessions.website_session_id ELSE NULL END) as gsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' THEN website_sessions.website_session_id ELSE NULL END) as bsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN utm_source is NULL AND http_referer IS NOT NULL THEN website_sessions.website_session_id ELSE NULL END) as organic_search_sessions,
    COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NULL THEN website_sessions.website_session_id ELSE NULL END) as direct_search_sessions
FROM website_sessions
LEFT JOIN orders
	ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
GROUP BY 1,2;
```

### Q5: Monthly Session to Order Conversion Rates
- Calculates session to order conversion rates by month to demonstrate website performance improvements.
```sql
SELECT 
	year(website_sessions.created_at) AS year,
    MONTH(website_sessions.created_at) AS month,
    COUNT(DISTINCT website_sessions.website_session_id) AS total_sessions,
    COUNT(DISTINCT orders.order_id) AS total_orders,
	COUNT(DISTINCT orders.order_id) / COUNT(DISTINCT website_sessions.website_session_id) AS sessions_orders_conv
FROM website_sessions
LEFT JOIN orders
	ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
GROUP BY 1,2;
```

### Q6: Conversion Funnel Analysis for Landing Page Test
- Analyzes the conversion funnel from two landing pages to orders, evaluating performance.
```sql
-- For the landing page test you analyzed previously, it would be great to show a
-- full conversion funnel from each of the two pages to orders . You can use the same time period you analyzed last time (Jun 19 Jul 28).

SELECT 
	MIN(website_pageviews.website_pageview_id) AS test_min_pv
    FROM website_pageviews
    WHERE pageview_url = '/lander-1';
-- id = 23504

CREATE TEMPORARY TABLE first_test_pvs
SELECT 
	website_pageviews.website_session_id,
    MIN(website_pageviews.website_pageview_id) AS min_pv_id
    FROM website_pageviews
    LEFT JOIN website_sessions
    ON website_pageviews.website_session_id = website_sessions.website_session_id
    WHERE website_pageview_id >= 23504 -- our first pageview of the test
    AND website_sessions.created_at < '2012-07-28'
    AND utm_source = 'gsearch'
    AND utm_campaign = 'nonbrand'
    GROUP BY website_pageviews.website_session_id;

CREATE temporary table sessions_landingpage
SELECT 
		first_test_pvs.website_session_id,
        pageview_url AS landing_page
	FROM first_test_pvs
    LEFT JOIN website_pageviews
    ON first_test_pvs.min_pv_id = website_pageviews.website_pageview_id
    WHERE website_pageviews.pageview_url IN ('/lander-1','/home');

CREATE TEMPORARY TABLE sessions_orders_landingpage    
SELECT 
	sessions_landingpage.website_session_id,
    sessions_landingpage.landing_page,
    orders.order_id
FROM sessions_landingpage
	LEFT JOIN orders
    ON sessions_landingpage.website_session_id = orders.website_session_id;
    
SELECT 
		landing_page,
        COUNT(DISTINCT website_session_id) AS total_sessions,
        COUNT(DISTINCT order_id) AS total_orders,
        COUNT(DISTINCT order_id) / COUNT(DISTINCT website_session_id) AS conversion_rate
FROM sessions_orders_landingpage
GROUP BY landing_page;

-- our analysis shows that our landing page was producing a slightly higher conversion rate
-- finidng the msot recent pageview for gsearch nonbrand where traffic was sent to /home

SELECT 
	MAX(website_sessions.website_session_id) AS recent_home_pv
    FROM website_sessions
    JOIN website_pageviews
    ON website_pageviews.website_session_id = website_sessions.website_session_id
    WHERE utm_source = 'gsearch'
		AND utm_campaign = 'nonbrand'
        AND pageview_url = '/home'
        AND website_sessions.created_at < '2012-11-27'; 
-- id = 17145
SELECT 
	COUNT(website_session_id) AS sessions_since_test
FROM website_sessions
WHERE utm_source = 'gsearch'
		AND utm_campaign = 'nonbrand'
        AND website_session_id > 17145 -- found in previous query as our home session
        AND website_sessions.created_at < '2012-11-27'; 
 -- 22,972 sessions since       
-- so becuase we found this we can multiply it by our previously found incremental conversion rate of 0.0087 (0.0406 - 0.0318)
-- and if we do that 0.0087 * 22972 = 202 
-- So we have gained 202 orders from making that change! pretty good 
```

### Q7: Revenue Estimation for Gsearch Lander Test
- Estimates revenue earned from the Gsearch lander test based on increased conversion rates.
```sql
-- lets make flags along our funnel
DROP TABLE session_made_it_flags;
CREATE TEMPORARY TABLE session_made_it_flags
SELECT 
	website_session_id,
	MAX(home_view) AS reachedhome,
    MAX(lander_1_view) AS reachedlander,
    MAX(products_view) AS reachedproduct,
    MAX(fuzzy_view) AS reachedfuzzy,
    MAX(cart_view) AS reachedcart,
    MAX(shipping_view) AS reachedshipping,
    MAX(billing_view) AS reachedbilling,
    MAX(ty_view) AS reachedty
FROM(
SELECT
	website_sessions.website_session_id,
	website_pageviews.pageview_url,
	CASE WHEN pageview_url = '/home' THEN 1 ELSE NULL END AS home_view,
	CASE WHEN pageview_url = '/lander-1' THEN 1 ELSE NULL END AS lander_1_view,
    CASE WHEN pageview_url = '/products' THEN 1 ELSE NULL END AS products_view,
    CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE NULL END AS fuzzy_view,
    CASE WHEN pageview_url = '/cart' THEN 1 ELSE NULL END AS cart_view,
    CASE WHEN pageview_url = '/shipping' THEN 1 ELSE NULL END AS shipping_view,
    CASE WHEN pageview_url = '/billing' THEN 1 ELSE NULL END AS billing_view,
    CASE WHEN pageview_url = '/thank-you-for-your-order' THEN 1 ELSE NULL END AS ty_view
FROM website_sessions 
LEFT JOIN website_pageviews
ON website_sessions.website_session_id = website_pageviews.website_session_id
WHERE website_sessions.utm_source = 'gsearch'
	AND website_sessions.utm_campaign = 'nonbrand'
    AND website_sessions.created_at > '2012-06-19'
    AND website_sessions.created_at < '2012-07-28'
ORDER BY website_sessions.website_session_id,
website_pageviews.created_at
) AS pageview_level
GROUP BY website_session_id;

SELECT
	CASE WHEN reachedhome = 1 THEN 'saw_homepage'
		WHEN reachedlander = 1 THEN 'saw_lander'
        ELSE 'check_logic'
        END AS segment,
	COUNT(DISTINCT website_session_id) AS total_sessions,
	COUNT(DISTINCT CASE WHEN reachedproduct = 1 THEN website_session_id ELSE NULL END) AS count_product,
    COUNT(DISTINCT CASE WHEN reachedfuzzy = 1 THEN website_session_id ELSE NULL END) AS count_fuzzy,
    COUNT(DISTINCT CASE WHEN reachedcart = 1 THEN website_session_id ELSE NULL END) AS count_cart,
    COUNT(DISTINCT CASE WHEN reachedshipping = 1 THEN website_session_id ELSE NULL END) AS count_shipping,
    COUNT(DISTINCT CASE WHEN reachedbilling = 1 THEN website_session_id ELSE NULL END) AS count_billing,
    COUNT(DISTINCT CASE WHEN reachedty = 1 THEN website_session_id ELSE NULL END) AS count_ty
FROM session_made_it_flags
GROUP BY segment;

-- to find click through rates we would divide the pages by the previous page counts. to save some time i will do this in excel. looks like the new lander page out performed the home page
/*				home	product	fuzzy	cart	shipping 	billing
saw_homepage	41.66%	72.61%	43.27%	67.57%	84.00%	42.86%
saw_lander		46.76%	71.28%	45.08%	66.38%	85.28%	47.72%
*/

```

## Conclusion
The analysis reveals significant insights into the performance of Gsearch traffic and website improvements over time. It demonstrates the effectiveness of different campaigns, devices, and landing pages in driving conversions and revenue growth.
```
