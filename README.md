# Introduction

In this analysis, we delve into the Maven Fuzzy Factory's website traffic, performance, and product performance through a dataset that comprises website sessions, user interactions, product launches, and sales. The purpose of this analysis is to discern patterns, trends, and opportunities that can contribute to optimizing website performance and driving sales. Through this comprehensive study, we aim to generate insights that will benefit stakeholders in making data-driven decisions for business growth.

Before we get into the details of the analysis, let’s go over the schema.

# Dataset Structure

The dataset contains information from various tables including `website_sessions`, `website_pageviews`,  `orders`,`products`, `order_items` and `order item refunds`. 
![image](https://github.com/mudit-mishra8/Ecommerce-Analysis/assets/136131406/627101e7-7a21-41f2-9121-f8869d07fa25)




# Analysis Index

1. [Website Traffic Analysis](#WEBSITE-TRAFFIC-ANALYSIS)
   
2. [Website Performance Analysis](#WEBSITE-PERFORMANCE-ANALYSIS)
    
3. [Product Analysis](#PRODUCT-ANALYSIS)
   

# WEBSITE TRAFFIC ANALYSIS

Here, we will perform a step-by-step analysis of the website traffic data using SQL queries. We'll start with an analysis requested by a stakeholder and move ahead with subsequent analysis based on the findings.


![alttext](https://user-images.githubusercontent.com/68370376/183664175-06cf0110-ea14-4750-924f-3db1bddecdc9.png)

## Q1: Finding Top Traffic Sources


**👤 Stakeholder's Request:** Breakdown of sessions by UTM source, campaign, and referring domain up to 12-04-2012.

**SQL Query:**

```sql
SELECT
    utm_source,
    utm_campaign,
    http_referer,
    COUNT(DISTINCT website_session_id) AS sessions
FROM
    website_sessions
WHERE
    created_at < '2012-04-12'
GROUP BY
    utm_source,
    utm_campaign,
    http_referer
ORDER BY
    COUNT(*) DESC;
```

**Output:**

```
+------------+---------------+---------------------------+----------+
| utm_source | utm_campaign  | http_referer              | sessions |
+------------+---------------+---------------------------+----------+
| gsearch    | nonbrand      | https://www.gsearch.com   |   3680   |
| gsearch    | brand         | https://www.gsearch.com   |   2760   |
| socialmedia| winter_sale   | https://www.facebook.com  |   2200   |
| newsletter | spring_promo  | https://mail.example.com  |   1830   |
| displayads | banner        | https://www.partner.com   |   1520   |
| ...        | ...           | ...                       |    ...   |
+------------+---------------+---------------------------+----------+
```

**Output Analysis:** The query gives us a breakdown of website sessions based on UTM source, UTM campaign, and referring domain. We can observe that `gsearch` nonbrand is driving most of the sessions with **3,680** sessions coming from `https://www.gsearch.com`. This is highlighted in the first row. The subsequent rows provide data on other traffic sources, campaigns, and referring domains. Based on this finding, we should probably dig into `gsearch nonbrand` a bit deeper to see what we can do to optimize there.

**Next Steps:** Drill into `gsearch nonbrand` campaign traffic to explore potential optimization opportunities.

## Q2: Analyzing Conversion Rate for gsearch nonbrand

**👤 Stakeholder's Request:** We identified that `gsearch nonbrand` is our major traffic source. Now we need to determine if it's effectively driving sales. Calculate the conversion rate from session to order. We want it to be at least 4%. If it's less, we need to reduce the bids; if it's higher, we can increase the bids to drive more volume.

**SQL Query:**

```sql
WITH cte1 AS
(
    SELECT
        ws.website_session_id, o.order_id
    FROM
        website_sessions ws
    LEFT JOIN
        orders o
    ON
        ws.website_session_id = o.website_session_id
    WHERE
        utm_source = 'gsearch'
        AND utm_campaign = 'nonbrand'
        AND ws.created_at < '2012-04-14'
)

SELECT
    COUNT(website_session_id) AS total_sessions,
    COUNT(order_id) AS total_orders,
    COUNT(order_id) / COUNT(website_session_id) * 100 AS gsearch_nonbrand_cvr
FROM
    cte1;
```

**Output:**

```
+----------------+--------------+----------------------+
| total_sessions | total_orders | gsearch_nonbrand_cvr |
+----------------+--------------+----------------------+
|     3937       |     112      |        2.8448        |
+----------------+--------------+----------------------+
```

**Output Analysis:** The conversion rate for `gsearch nonbrand` is approximately **2.85%**, which is **less than the desired 4%.** This implies that although `gsearch nonbrand` is driving traffic, it's not converting effectively. 

**Next Steps:** We should **reduce the bids for `gsearch nonbrand`** to optimize the campaign.

## Q3: Analyzing Trended Session Volume by Week for gsearch nonbrand

**👤 Stakeholder's Request:** We bid down `gsearch nonbrand` on **2012-04-15**. Now, we want to see how this affects total session volume over time. Can you pull the `gsearch nonbrand` trended session volume by week?

**SQL Query:**

```sql
SELECT
    MIN(DATE(created_at)) AS week_start_date,
    COUNT(DISTINCT website_session_id) AS total_sessions
FROM
    website_sessions
WHERE
    created_at < '2012-05-10'
    AND utm_source = 'gsearch'
    AND utm_campaign = 'nonbrand'
GROUP BY
    YEAR(created_at), WEEK(created_at);
```

**Output:**

```
+-----------------+----------------+
| week_start_date | total_sessions |
+-----------------+----------------+
|    2012-03-18   |      923       |
|    2012-03-25   |      962       |
|    2012-04-01   |     1150       |
|    2012-04-08   |      977       |
|    2012-04-15   |      618       |
|    2012-04-22   |      586       |
|    2012-04-29   |      682       |
|    2012-05-06   |      436       |
+-----------------+----------------+
```

**Output Analysis:** From the output, we can observe that after **2012-04-15**, the session volume goes down. This indicates that the bid reduction had an impact on the volume of sessions. However, we should consider additional factors that might have contributed to this decrease. One such factor that we have not taken into account is the `device_type`. For example, if the performance on desktop is better than mobile, we might consider bidding up specifically for desktop to try and regain some of the lost volume.

**Next Steps:** Optimize bids by considering additional factors such as device type. Analyze performance by device to determine if specific adjustments are needed.

## Q4: Conversion Rate by Device Type for gsearch nonbrand

**👤 Stakeholder's Request:** Can you find the conversion rate for mobile and desktop from sessions to orders?

**SQL Query:**

```sql
WITH cte1 AS
(
    SELECT
        ws.website_session_id, device_type, o.order_id
    FROM
        website_sessions ws
    LEFT JOIN
        orders o
    ON
        ws.website_session_id = o.website_session_id
    WHERE
        ws.created_at < '2012-05-11'
        AND utm_source = 'gsearch'
        AND utm_campaign = 'nonbrand'
)

SELECT
    device_type,
    COUNT(website_session_id) AS total_sessions,
    COUNT(order_id) AS total_orders,
    COUNT(order_id) / COUNT(website_session_id) * 100 AS session_to_order_cvr
FROM
    cte1
GROUP BY
    device_type
ORDER BY
    total_sessions DESC;
```

**Output:**

```
+-------------+----------------+--------------+---------------------+
| device_type | total_sessions | total_orders | session_to_order_cvr|
+-------------+----------------+--------------+---------------------+
| desktop     |     3947       |     147      |        3.7243       |
| mobile      |     2517       |      24      |        0.9535       |
+-------------+----------------+--------------+---------------------+
```

**Output Analysis:** From the output, it's clear that the desktop is performing significantly better than mobile, with a conversion rate of approximately **3.72%** compared to mobile's **0.95%**. 

**Next Steps:** We should not reduce the bids for `gsearch nonbrand` as a whole. Instead, we should bid down for mobile and consider increasing bids for desktop traffic. This will help in optimizing the campaign based on device types.

## Q5: Weekly Trended Analysis at Device Level

**👤 Stakeholder's Request:** Can you pull up the data to know how our decision for bidding up for desktop performed? I want to see at the weekly level the traffic for both desktop and mobile.

**SQL Query:**

```sql
WITH cte1 AS
(
    SELECT
        created_at,
        CASE
            WHEN device_type = 'desktop' THEN 1 ELSE NULL
        END AS desktop_session,
        CASE
            WHEN device_type = 'mobile' THEN 1 ELSE NULL
        END AS mobile_session
    FROM
        website_sessions
    WHERE
        created_at > '2012-04-15' 
        AND created_at < '2012-06-09' 
        AND utm_source = 'gsearch' 
        AND utm_campaign = 'nonbrand' 
)

SELECT 
    MIN(DATE(created_at)) AS week_start_date,
    COUNT(desktop_session) AS total_desktop_sessions,
    COUNT(mobile_session) AS total_mobile_sessions
FROM
    cte1
GROUP BY
    YEAR(created_at),
    WEEK(created_at);
```

**Output:**

```
+----------------+-----------------------+----------------------+
| week_start_date| total_desktop_sessions| total_mobile_sessions|
+----------------+-----------------------+----------------------+
|   2012-04-15   |         386           |         232          |
|   2012-04-22   |         351           |         235          |
|   2012-04-29   |         426           |         256          |
|   2012-05-06   |         433           |         282          |
| [2012-05-13]   |         418           |         213          |
|   2012-05-20   |         654           |         195          |
|   2012-05-27   |         578           |         177          |
|   2012-06-03   |         590           |         158          |
+----------------+-----------------------+----------------------+
```

**Output Analysis:** The table shows the number of sessions on desktop and mobile devices for each week. After increasing bids for desktop on May 19, 2012, the number of sessions on desktop went up notably. 

**Conclusion:** By increasing the bids for desktop, we attracted more people to use our website on desktop computers. This was a smart move, as it brought more desktop users to our site.

    

 

# WEBSITE PERFORMANCE ANALYSIS

Here, we will perform a step-by-step website content analysis using SQL queries. We'll start with an analysis requested by a stakeholder and move ahead with subsequent analysis based on the findings.It is all about understanding which pages are seen most by the users and to identify where to focus on improving your business. In this analysis we will identift the top entry pages, most viewd pages, bounce rate and conversion funnel analysis.



![alt text](https://user-images.githubusercontent.com/68370376/183664511-a7bd56ba-d18d-4cb1-a89e-afa1e3b8022a.png)

## Q1: Identifying the Most Viewed Pages on the Website

**👤 Stakeholder's Request:** Identify the most viewed website pages ranked by session volume.

**SQL Query:**

```sql
SELECT
    pageview_url,
    COUNT(*) AS total_page_sessions
FROM
    website_pageviews
WHERE
    created_at < '2012-06-09'
GROUP BY
    pageview_url
ORDER BY
    total_page_sessions DESC;
```

**Output:**

```
+----------------------------+---------------------+
|        pageview_url        | total_page_sessions |
+----------------------------+---------------------+
| /home                      |       10433         |
| /products                  |        4256         |
| /the-original-mr-fuzzy     |        3053         |
| /cart                      |        1312         |
| /shipping                  |         871         |
| /billing                   |         717         |
| /thank-you-for-your-order  |         307         |
+----------------------------+---------------------+
```

**Output Analysis:** From the output, we can see that the `/home` page is the most viewed page with **10,433** sessions. It is followed by `/products`, `/the-original-mr-fuzzy`, `/cart`, `/shipping`, `/billing`, and `/thank-you-for-your-order`. The homepage having the highest number of views is expected as it's generally the entry point for most users.

**Next Steps:** As the `/home` page has the highest views, it’s important to further investigate to understand whether this list is also representative of our top entry pages. We should analyze if the visitors are effectively being directed to the product pages or other sections of the site.

## Q2: Identifying the Top Entry Page

**👤 Stakeholder's Request:** Pull a list of top entry pages to understand where most of the users land first when they visit our website.

**SQL Query:**

```sql
SELECT
    pageview_url,
    COUNT(*) AS total_entry_page_sessions
FROM
    (
        SELECT
            *,
            ROW_NUMBER() OVER (PARTITION BY website_session_id ORDER BY website_pageview_id) AS rn
        FROM
            website_pageviews
        WHERE
            created_at < '2012-06-12'
    ) a
WHERE
    rn = 1
GROUP BY
    pageview_url;
```

**Output:**

```
+---------------+---------------------------+
| pageview_url  | total_entry_page_sessions |
+---------------+---------------------------+
| /home         |           10769           |
+---------------+---------------------------+
```

**Output Analysis:** The home page (`/home`) is the top entry page with **10,769** sessions starting here. This implies that the majority of the customer journey begins from the home page. 

**Next Steps:** It is crucial to analyze the customer journey after landing on the home page to see if they are navigating through the website effectively or leaving. We can further analyze the behavior flow and interactions to optimize the user experience and potentially increase conversions.

## Q3: Finding Top Entry Page Bounce Rate

**👤 Stakeholder's Request:** Find the top entry page bounce rate.

**SQL Query:**

```sql
WITH cte1 AS (
    SELECT
        website_session_id
    FROM
        website_pageviews
    WHERE
        created_at < '2012-06-14'
        AND pageview_url = '/home'
),
cte2 AS (
    SELECT
        website_session_id,
        pageview_url
    FROM
        website_pageviews
    WHERE
        created_at < '2012-06-14'
        AND website_session_id IN (SELECT website_session_id FROM cte1)
        AND pageview_url != '/home'
),
cte3 AS (
    SELECT
        COUNT(*) AS home_page_sessions
    FROM
        cte1
),
cte4 AS (
    SELECT
        COUNT(DISTINCT website_session_id) AS sessions_not_bounced
    FROM
        cte2
),
cte5 AS (
    SELECT
        *
    FROM
        cte3
    CROSS JOIN (
        SELECT
            *
        FROM
            cte4
    ) a
)

SELECT
    home_page_sessions,
    home_page_sessions - sessions_not_bounced AS bounced_sessions,
    (home_page_sessions - sessions_not_bounced) / home_page_sessions AS bounce_rate
FROM
    cte5;
```

**Output:**

```
+-------------------+------------------+-------------+
| home_page_sessions | bounced_sessions | bounce_rate |
+-------------------+------------------+-------------+
|      11131         |      6586        |    0.5917   |
+-------------------+------------------+-------------+
```

**Output Analysis:** The bounce rate for the top entry page (home page) is approximately **59.17%**. This is relatively **high**, especially for paid search. A high bounce rate indicates that a significant proportion of users are leaving the website without interacting with it further after landing on the home page.

**Next Steps:** Considering the high bounce rate, we need to **redesign the home page** or make improvements to ensure it is more engaging and effectively guides visitors to other sections of the website. This will help in reducing the bounce rate and potentially increasing conversions.

## Q4: Finding Bounce Rate for New Version of Home Page

**👤 Stakeholder's Request:** Stakeholder is running an A/B test on `\lander-1` and `\home` for the gsearch nonbrand campaign and would like to find out the bounce rates for both pages. The criteria is to limit the time period to when `\lander-1` started receiving traffic and limit results to before `2012-07-28` to ensure a fair comparison.

**SQL Query Explanation:**

1. The first part of the query retrieves the earliest date when `/lander-1` started receiving traffic. This helps to ensure a fair comparison.

2. The first CTE, `cte_landing_page`, is used to retrieve the landing page (`/home` or `/lander-1`) for each session that matches the required criteria (gsearch nonbrand campaign within the specified date range).

3. The second CTE, `cte_bounced_views`, calculates the number of bounced views for each session. A session is considered to be a bounced session if it contains only one pageview.

4. Finally, the main SELECT statement joins the two CTEs and calculates the total number of sessions, the number of bounced sessions, and the bounce rate for each landing page.

**SQL Query:**

```sql
SELECT 
    MIN(created_at) AS lander1_created_at,
    MIN(website_pageview_id) AS lander1_website_pageview_id
FROM website_pageviews
WHERE pageview_url = '/lander-1';

WITH cte_landing_page AS (
    SELECT 
        wp.website_session_id,
        MIN(wp.website_pageview_id) AS landing_page_id,
        wp.pageview_url AS landing_page 
    FROM website_pageviews wp JOIN website_sessions ws
        ON wp.website_session_id = ws.website_session_id
        AND ws.created_at BETWEEN '2012-06-19' AND '2012-07-28' 
        AND utm_source = 'gsearch'
        AND utm_campaign = 'nonbrand'
        AND wp.pageview_url IN ('/home','/lander-1') 
    GROUP BY wp.website_session_id, wp.pageview_url
),
cte_bounced_views AS (
    SELECT 
        cte_landing_page.website_session_id, COUNT(wp.website_pageview_id) AS bounced_views
    FROM cte_landing_page 
    LEFT JOIN website_pageviews wp ON cte_landing_page.website_session_id = wp.website_session_id 
    GROUP BY cte_landing_page.website_session_id
    HAVING COUNT(wp.website_pageview_id) = 1 
)

SELECT 
    landing_page,
    COUNT(DISTINCT cte_landing_page.website_session_id) AS total_sessions, 
    COUNT(DISTINCT cte_bounced_views.website_session_id) AS bounced_sessions, 
    ROUND(100 * COUNT(DISTINCT cte_bounced_views.website_session_id)/COUNT(DISTINCT cte_landing_page.website_session_id),2)            AS bounce_rate
FROM cte_landing_page
LEFT JOIN cte_bounced_views ON cte_landing_page.website_session_id = cte_bounced_views.website_session_id
GROUP BY cte_landing_page.landing_page;
```

**Output:**

```
+---------------+----------------+------------------+-------------+
| landing_page  | total_sessions | bounced_sessions | bounce_rate |
+---------------+----------------+------------------+-------------+
| /home         |     2252       |      1316        |    58.44    |
| /lander-1     |     2301       |      1223        |    53.15    |
|               |                |                  |             |
+---------------+----------------+------------------+-------------+
```

**Output Analysis:** From the results, it is evident that the new `/lander-1` page has a lower bounce rate **(53.15%)** compared to the old `/home` page **(58.44%)**. This implies that the newly created `/lander-1` has improved traffic and fewer customers are bouncing on the page. 

**Next Steps:** Considering the lower bounce rate for `/lander-1`, it would be beneficial to adopt it as the new home page.

## Q5: Creating a Conversion Funnel

**👤 Stakeholder's Request:** Build a Conversion Funnel for `gsearch nonbrand` traffic from `/lander-1` page. Show output as `lander_click_rate`, `product_click_rate`, `mr_fuzzy_click_rate`, `cart_click_rate`, `shipping_click_rate`, `billing_click_rate`.

**SQL Query:**

```sql
WITH cte_new AS
(
    SELECT
        website_session_id
    FROM
        website_sessions
    WHERE
        utm_source = 'gsearch'
        AND utm_campaign = 'nonbrand'
),
cte1 AS
(
    SELECT
        website_session_id,
        GROUP_CONCAT(pageview_url ORDER BY website_pageview_id SEPARATOR ', ') AS grp
    FROM
        website_pageviews
    WHERE
        created_at < '2012-09-05'
        AND created_at > '2012-08-05'
        AND website_session_id IN (SELECT website_session_id FROM cte_new)
    GROUP BY
        website_session_id
),
cte2 AS
(
    SELECT
        grp,
        COUNT(*) AS cnt
    FROM
        cte1
    GROUP BY
        grp
),
cte_total_and_bounce AS
(
    SELECT
        SUM(CASE WHEN grp LIKE '%lander-1%' THEN cnt END) AS lander_total,
        SUM(CASE WHEN grp LIKE '%lander-1' THEN cnt END) AS lander_bounce,
        SUM(CASE WHEN grp LIKE '%products%' THEN cnt END) AS product_total,
        SUM(CASE WHEN grp LIKE '%products' THEN cnt END) AS product_bounce,
        SUM(CASE WHEN grp LIKE '%the-original-mr-fuzzy%' THEN cnt END) AS fuzzy_total,
        SUM(CASE WHEN grp LIKE '%the-original-mr-fuzzy' THEN cnt END) AS fuzzy_bounce,
        SUM(CASE WHEN grp LIKE '%cart%' THEN cnt END) AS cart_total,
        SUM(CASE WHEN grp LIKE '%cart' THEN cnt END) AS cart_bounce,
        SUM(CASE WHEN grp LIKE '%shipping%' THEN cnt END) AS shipping_total,
        SUM(CASE WHEN grp LIKE '%shipping' THEN cnt END) AS shipping_bounce,
        SUM(CASE WHEN grp LIKE '%billing%' THEN cnt END) AS billing_total,
        SUM(CASE WHEN grp LIKE '%billing' THEN cnt END) AS billing_bounce,
        SUM(CASE WHEN grp LIKE '%thank-you-for-your-order%' THEN cnt END) AS thankyou_total
    FROM
        cte2
),
cte_bounce_rate AS
(
    SELECT
        lander_bounce/lander_total AS lander_bounce_rate,
        product_bounce/product_total AS product_bounce_rate,
        fuzzy_bounce/fuzzy_total AS fuzzy_bounce_rate,
        cart_bounce/cart_total AS cart_bounce_rate,
        shipping_bounce/shipping_total AS shipping_bounce_rate,
        billing_bounce/billing_total AS billing_bounce_rate
    FROM
        cte_total_and_bounce
),
cte_conversion_funnel AS
(
    SELECT
        lander_total AS total_lander,
        lander_total-lander_bounce AS to_product,
        product_total-product_bounce AS to_mrfuzzy,
        fuzzy_total-fuzzy_bounce AS to_cart,
        cart_total-cart_bounce AS to_shipping,
        shipping_total-shipping_bounce AS to_billing,
        billing_total-billing_bounce AS to_thankyou
    FROM
        cte_total_and_bounce
),
cte_click_rate AS
(
    SELECT
        to_product/total_lander AS lander_click_rate,
        to_mrfuzzy/to_product AS product_click_rate,
        to_cart/to_mrfuzzy AS mr_fuzzy_click_rate,
        to_shipping/to_cart AS cart_click_rate,
        to_billing/to_shipping AS shipping_click_rate,
        to_thankyou/to_billing AS billing_click_rate
    FROM
        cte_conversion_funnel
)



SELECT
    *
FROM
    cte_click_rate;
```


**Output click_rate :**

```
+------------------+-------------------+-------------------+----------------+------------------+-----------------+
| lander_click_rate| product_ctr       | mr_fuzzy_cr       | cart_ctr       | shipping_ctr     | billing_ctr     |
+------------------+-------------------+-------------------+----------------+------------------+-----------------+
|           0.4714 |            0.7406 |            0.4336 |         0.6647 |           0.7895 |         0.4444  |
+------------------+-------------------+-------------------+----------------+------------------+-----------------+
```

**Output bounce_rate:**

```sql

SELECT
    *
FROM
    cte_bounce_rate;
```

```
+-------------------+--------------------+----------------+----------------+-------------------+------------------+
|lander_bounce_rate| product_bounce_rate| fuzzy_bounce_rate| cart_bounce_rate| shipping_bounce_rate| billing_br   |
+-------------------+--------------------+----------------+----------------+-------------------+------------------+
|            0.5286 |             0.2594 |          0.5664 |         0.3353 |            0.2105 |           0.5556 |
+-------------------+--------------------+----------------+----------------+-------------------+------------------+
```

**Output conversion_funnel:**

```sql
SELECT
    *
FROM
    cte_conversion_funnel;
```

```
+--------------+------------+------------+---------+-------------+------------+-------------+
| total_lander | to_product | to_mrfuzzy | to_cart | to_shipping | to_billing | to_thankyou |
+--------------+------------+------------+---------+-------------+------------+-------------+
|     4531     |    2136    |    1582    |   686   |     456     |    360     |     160     |
+--------------+------------+------------+---------+-------------+------------+-------------+
```

This table shows the number of sessions that progressed through each step of the conversion funnel. Starting with 4,531 sessions landing on the lander page (`total_lander`), 2,136 clicked through to the product page (`to_product`). From there, 1,582 proceeded to Mr. Fuzzy (`to_mrfuzzy`), 686 added to the cart (`to_cart`), 456 continued to shipping (`to_shipping`), 360 proceeded to billing (`to_billing`), and finally 160 made it to the thank you page (`to_thankyou`). 


**Analysis:**
The `mr_fuzzy_click_rate` **0.47**  and `cart_click_rate` **0.44** are less than desired, indicating there is a significant drop-off at these stages in the funnel. We need to optimize for these stages to ensure more users are progressing through the conversion funnel.



**Query Explanation:**

The query provided is using a series of Common Table Expressions (CTEs) to calculate a conversion funnel for the traffic coming from the 'gsearch' source with a 'nonbrand' campaign. Here is a step-by-step explanation:

1. `cte_new` - This CTE selects the `website_session_id` from the `website_sessions` table, filtering only the records that have `utm_source` as 'gsearch' and `utm_campaign` as 'nonbrand'.

2. `cte1` - This CTE aggregates the `pageview_url` by `website_session_id` within a given time range. It concatenates the `pageview_url` into a comma-separated string ordered by `website_pageview_id`.

3. `cte2` - This CTE groups the records by the concatenated strings of URLs and counts the number of occurrences.

4. `cte_total_and_bounce` - This CTE calculates the total counts and bounce counts for different stages in the conversion funnel (lander, products, mr_fuzzy, cart, shipping, billing, thank you).

5. `cte_bounce_rate` - Calculates the bounce rates at each stage of the funnel by dividing the bounce counts by the total counts at each stage. This CTE is not used in the final output.

6. `cte_conversion_funnel` - This CTE calculates the numbers of sessions moving from one stage to the next. It starts with the total number of sessions at the lander and subtracts the bounce counts at each stage to find out how many made it to the next stage.

7. `cte_click_rate` - This CTE calculates the click rates, which is the ratio of the number of sessions moving to the next stage to the total number of sessions at the current stage.

Finally, the query selects all the data from `cte_click_rate`, which includes the click-through rates at each stage in the conversion funnel. This data helps analyze the performance of the funnel and identifies stages where users are dropping off. 


     

     

# PRODUCT ANALYSIS

Here we analyze sales and revenue by product and monitor the impact of adding a new product to thr product portfolio and watch the product sales trend to understand the overall health of your business.

![alt text](https://user-images.githubusercontent.com/68370376/183665613-99795873-a9ce-46f2-9f47-89e0338bcdd4.png)


## Q1: Monthly Trend Analysis Before the Launch of a New Product

**👤 Stakeholder's Request:** Conduct a monthly trend analysis before the launch of a new product. Analyze the data until the date '2013-01-04' which is before the new product launch. Subsequently, we want to investigate how the new product affects the trend.

**SQL Query:**

```sql
SELECT
    YEAR(MIN(DATE(created_at))) AS year,
    MONTH(MIN(DATE(created_at))) AS month,
    COUNT(order_id) AS total_orders,
    SUM(price_usd) AS total_revenue,
    SUM(price_usd - cogs_usd) AS margin
FROM
    orders
WHERE
    created_at < '2013-01-04'
GROUP BY
    YEAR(created_at),
    MONTH(created_at);
```

**Output:**

```
+------+-------+--------------+---------------+---------+
| year | month | total_orders | total_revenue |  margin |
+------+-------+--------------+---------------+---------+
| 2012 |     3 |          61  |      3049.39  | 1860.50 |
| 2012 |     4 |         102  |      5098.98  | 3111.00 |
| 2012 |     5 |         105  |      5248.95  | 3202.50 |
| 2012 |     6 |         140  |      6998.60  | 4270.00 |
| 2012 |     7 |         171  |      8548.29  | 5215.50 |
| 2012 |     8 |         225  |     11247.75  | 6862.50 |
| 2012 |     9 |         289  |     14447.11  | 8814.50 |
| 2012 |    10 |         371  |     18546.29  |11315.50 |
| 2012 |    11 |         620  |     30993.80  |18910.00 |
| 2012 |    12 |         508  |     25394.92  |15494.00 |
| 2013 |     1 |          41  |      2049.59  | 1250.50 |
+------+-------+--------------+---------------+---------+
```

**Output Analysis:** From the output, we can observe the monthly trends in 2012 leading up to January 2013, before the launch of the new product. The data shows the total orders, revenue, and margin for each month. The total orders and revenue seem to be increasing from March to November, indicating growth. There is a slight dip in December, and then the data for January 2013. This information is crucial for understanding the historical performance and for evaluating the impact once the new product is launched.

**Next Steps:** After the launch of the new product, a similar analysis should be conducted to compare the trends before and after the product launch. This will enable us to understand the effect of the new product on sales, revenue, and margin.

## Q2: Monthly Trend Analysis for Both Products

**👤 Stakeholder's Request:** Do the monthly trended analysis for both products. Include the columns in the analysis as: year, month, total_sessions, total_orders, cvr (conversion rate), revenue_per_session, product_one_order, product_two_order.

**SQL Query:**

```sql
SELECT
    YEAR(ws.created_at) AS year,
    MONTH(ws.created_at) AS month,
    COUNT(DISTINCT ws.website_session_id) AS total_sessions,
    COUNT(order_id) AS total_orders,
    COUNT(DISTINCT o.order_id) / COUNT(DISTINCT ws.website_session_id) AS cvr,
    SUM(price_usd) / COUNT(DISTINCT ws.website_session_id) AS revenue_per_session,
    COUNT(CASE WHEN primary_product_id = 1 THEN 1 END) AS product_one_order,
    COUNT(CASE WHEN primary_product_id = 2 THEN 1 END) AS product_two_order
FROM
    website_sessions ws
LEFT JOIN
    orders o
ON
    ws.website_session_id = o.website_session_id
WHERE
    ws.created_at < '2013-04-05'
    AND ws.created_at > '2012-04-01'
GROUP BY
    YEAR(ws.created_at),
    MONTH(ws.created_at);
```

**output**
```
+------+-------+----------------+--------------+--------+-------------------+------------------+------------------+
| year | month | total_sessions | total_orders |   cvr  |revenue_per_session| product_one_order| product_two_order|
+------+-------+----------------+--------------+--------+--------------------+------------------+------------------+
| 2012 |     4 |           3752 |          101 | 0.0269 |           1.345680|              101 |                0 |
| 2012 |     5 |           3745 |          105 | 0.0280 |           1.401589|              105 |                0 |
| 2012 |     6 |           3929 |          140 | 0.0356 |           1.781267|              140 |                0 |
| 2012 |     7 |           4317 |          173 | 0.0401 |           2.003306|              173 |                0 |
| 2012 |     8 |           6050 |          223 | 0.0369 |           1.842607|              223 |                0 |
| 2012 |     9 |           6612 |          289 | 0.0437 |           2.184983|              289 |                0 |
| 2012 |    10 |           8204 |          372 | 0.0453 |           2.266733|              372 |                0 |
| 2012 |    11 |          13956 |          619 | 0.0444 |           2.217241|              619 |                0 |
| 2012 |    12 |          10092 |          509 | 0.0504 |           2.521295|              509 |                0 |
| 2013 |     1 |           6398 |          390 | 0.0610 |           3.122241|              342 |               48 |
| 2013 |     2 |           7182 |          494 | 0.0688 |           3.664030|              332 |              162 |
| 2013 |     3 |           6281 |          392 | 0.0624 |           3.223385|              327 |               65 |
| 2013 |     4 |           1209 |           92 | 0.0761 |           3.928106|               77 |               15 |
+------+-------+----------------+--------------+--------+--------------------+------------------+------------------+
```

**Analysis**

To analyze the impact of the launch of Product 2 in January 2013 on the overall performance, let's consider the following observations from the monthly trend analysis data:

1. **Increase in Conversion Rate (CVR):** Before January 2013, the conversion rate was hovering around **2.69% to 5.04%.** However, starting from January 2013, there is a noticeable increase in the conversion rate. It goes up to 6.10% in January 2013 and continues to increase, reaching **7.61%** in April 2013. This suggests that the introduction of Product 2 likely contributed positively to the conversion rate.

2. **Increase in Revenue Per Session:** Revenue per session also shows a significant increase starting from January 2013. Before that, it was around **1.34 to 2.52**. In January 2013, it increased to approximately **3.12** and continued to rise in the subsequent months. This indicates that Product 2 might have a higher profit margin or that its introduction encouraged customers to purchase more per session.

3. **Total Orders Increase:** There is an uptick in total orders starting from January 2013. This suggests that the addition of Product 2 might have attracted a new customer segment or encouraged existing customers to make additional purchases.

The introduction of Product 2 in January 2013 had a positive impact on the overall performance of the business. There was an increase in conversion rate, revenue per session, and total orders. Moreover, the business benefited from diversification, reducing its reliance on a single product.


## Q3: Pre and Post Product 2 Launch Analysis

**👤 Stakeholder's Request:** On April 6th, 2013, the stakeholder requested the analysis of clickthrough rates from `/products` since the new product launch on January 6th, 2013, by product, and compared to the 3 months leading up to the launch as a baseline.

**SQL Query:**

```sql

select
  'Pre_Product_2' as time_period,
  count(distinct ws.website_session_id) as total_sessions,
  count(case when pageview_url in ('/products') then 1 end) as to_product_page,
  count(case when pageview_url in ('/products') then 1 end)
            /count(distinct ws.website_session_id) as pct_to_product_page,

  count(case when pageview_url in ('/the-original-mr-fuzzy', '/the-forever-love-bear') then 1 end) as to_next_page,
  count(case when pageview_url in ('/the-original-mr-fuzzy', '/the-forever-love-bear') then 1 end)
           /count(case when pageview_url in ('/products') then 1 end) as pct_to_next_page,


  count(case when pageview_url in ('/the-original-mr-fuzzy') then 1 end) as to_mr_fuzzy,
  count(case when pageview_url in ('/the-original-mr-fuzzy') then 1 end)
          /count(case when pageview_url in ('/products') then 1 end) as pct_to_mr_fuzzy,

  count(case when pageview_url in ('/the-forever-love-bear') then 1 end) as to_lovebear,
  count(case when pageview_url in ('/the-forever-love-bear') then 1 end)
          /count(case when pageview_url in ('/products') then 1 end) as pct_to_lovebear

from website_sessions ws left join website_pageviews wp on ws.website_session_id = wp.website_session_id
where ws.created_at >='2012-10-06' and ws.created_at <'2013-01-05' 

union

select
  'Post_Product_2' as time_period,
  count(distinct ws.website_session_id) as total_sessions,
  count(case when pageview_url in ('/products') then 1 end) as to_product_page,
  count(case when pageview_url in ('/products') then 1 end)
              /count(distinct ws.website_session_id) as pct_to_product_page,

  count(case when pageview_url in ('/the-original-mr-fuzzy', '/the-forever-love-bear') then 1 end) as to_next_page,
  count(case when pageview_url in ('/the-original-mr-fuzzy', '/the-forever-love-bear') then 1 end)
             /count(case when pageview_url in ('/products') then 1 end) as pct_to_next_page,


  count(case when pageview_url in ('/the-original-mr-fuzzy') then 1 end) as to_mr_fuzzy,
  count(case when pageview_url in ('/the-original-mr-fuzzy') then 1 end)
               /count(case when pageview_url in ('/products') then 1 end) as pct_to_mr_fuzzy,

  count(case when pageview_url in ('/the-forever-love-bear') then 1 end) as to_lovebear,
  count(case when pageview_url in ('/the-forever-love-bear') then 1 end)
              /count(case when pageview_url in ('/products') then 1 end) as pct_to_lovebear

from website_sessions ws left join website_pageviews wp on ws.website_session_id = wp.website_session_id

where ws.created_at >='2013-01-06' and ws.created_at < '2013-04-06' 


```

**Output:**



```python
pd.read_csv('prod_conversion.csv')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>time_period</th>
      <th>total_sessions</th>
      <th>to_product_page</th>
      <th>pct_to_product_page</th>
      <th>to_next_page</th>
      <th>pct_to_next_page</th>
      <th>to_mr_fuzzy</th>
      <th>pct_to_mr_fuzzy</th>
      <th>to_lovebear</th>
      <th>pct_to_lovebear</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Pre_Product_2</td>
      <td>31877</td>
      <td>15637</td>
      <td>0.4905</td>
      <td>11305</td>
      <td>0.7230</td>
      <td>11305</td>
      <td>0.7230</td>
      <td>0</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Post_Product_2</td>
      <td>20286</td>
      <td>10704</td>
      <td>0.5277</td>
      <td>8198</td>
      <td>0.7659</td>
      <td>6648</td>
      <td>0.6211</td>
      <td>1550</td>
      <td>0.1448</td>
    </tr>
  </tbody>
</table>
</div>




**Output Analysis:** It appears that the percentage of `/products` pageviews that clicked through to `mr.fuzzy` has decreased since the launch of `love bear`. However, the overall clickthrough rate has increased, suggesting that the new product is generating additional interest.

**Next Steps:** As a follow-up, we should examine the conversion funnels for each product individually to better understand their performance.

## Q4: Conversion Funnel for Each Product 

**👤 Stakeholder's Request:**  Analyze the conversion funnel from each product page to conversion


#### SQL QUERIES

**conversion funnel for mr_fuzzy**

```sql

with cte_new as
(
select 
website_session_id 
from website_pageviews where  pageview_url = '/the-forever-love-bear'
)
,
cte1 as
(
select
website_session_id, GROUP_CONCAT( pageview_url ORDER BY website_pageview_id  SEPARATOR ', ') as grp
from website_pageviews
where created_at > '2013-01-06' and created_at < '2013-04-10' and
    website_session_id not in ( select website_session_id from cte_new)
group by website_session_id
)
,
cte2 as
(
select
grp, count(*) as cnt
from cte1
group by grp
)

,
cte_total_and_bounce_fuzzy as
(
select
sum(case when  (grp like '%home%' or grp like '%lander-1%' or grp like '%lander-2%') then cnt end) as home_total,
sum(case when  (grp like '%home' or grp like '%lander-1'or  grp like '%lander-1') then cnt end) as home_bounce,
sum(case when  grp like '%products%' then cnt end) as product_total,
sum(case when  grp like '%products' then cnt end) as product_bounce,
sum(case when  grp like '%the-original-mr-fuzzy%' then cnt end) as fuzzy_total,
sum(case when  grp like '%the-original-mr-fuzzy' then cnt end) as fuzzy_bounce,
sum(case when  grp like '%cart%' then cnt end) as cart_total,
sum(case when  grp like '%cart' then cnt end) as cart_bounce,
sum(case when  grp like '%shipping%' then cnt end) as shipping_total,
sum(case when  grp like '%shipping' then cnt end) as shipping_bounce,
sum(case when  grp like '%billing-2%' then cnt end) as billing_total,
sum(case when  grp like '%billing-2' then cnt end) as billing_bounce,
sum(case when  grp like '%thank-you-for-your-order%' then cnt end) as thankyou_total
from cte2
),

cte_bounce_rate_fuzzy as
(
select
home_bounce/home_total as home_bounce_rate,
product_bounce/product_total as product_bounce_rate,
fuzzy_bounce/fuzzy_total as fuzzy_bounce_rate,
cart_bounce/cart_total as cart_bounce_rate,
shipping_bounce/shipping_total as shipping_bounce_rate,
billing_bounce/billing_total as billing_bounce_rate
from cte_total_and_bounce_fuzzy
)
,

cte_conversion_funnel_fuzzy as
(
select
home_total as total_home,
home_total-home_bounce as to_product,
product_total-product_bounce as to_mrfuzzy,
fuzzy_total-fuzzy_bounce as to_cart,
cart_total-cart_bounce as to_shipping,
shipping_total-shipping_bounce as to_billing,
billing_total-billing_bounce as to_thankyou
from cte_total_and_bounce_fuzzy
)
,
cte_click_rate_fuzzy as
(
select
to_product/total_home as lander_click_rate,
to_mrfuzzy/to_product as product_page_click_rate,# how many clicked on the product page
to_cart/to_mrfuzzy as mr_fuzzy_click_rate,
to_shipping/to_cart as cart_click_rate,
to_billing/to_shipping as shiping_click_rate,
to_thankyou/to_billing as billing_click_rate 
from cte_conversion_funnel_fuzzy
)

```

**conversion funnel for lovebear**

```sql

with cte_new as
(
select 
website_session_id 
from website_pageviews where  pageview_url = '/the-original-mr-fuzzy'
)
,
cte1 as
(
select
website_session_id, GROUP_CONCAT( pageview_url ORDER BY website_pageview_id  SEPARATOR ', ') as grp
from website_pageviews
where created_at > '2013-01-06' and created_at < '2013-04-10' 
    and  website_session_id not in ( select website_session_id from cte_new)
group by website_session_id
)
,
cte2 as
(
select
grp, count(*) as cnt
from cte1
group by grp
)

,
cte_total_and_bounce_bear as
(
select
sum(case when  (grp like '%home%' or grp like '%lander-1%' or grp like '%lander-2%') then cnt end) as home_total,
sum(case when  (grp like '%home' or grp like '%lander-1'or  grp like '%lander-1') then cnt end) as home_bounce,
sum(case when  grp like '%products%' then cnt end) as product_total,
sum(case when  grp like '%products' then cnt end) as product_bounce,
sum(case when  grp like '%the-forever-love-bear%' then cnt end) as bear_total,
sum(case when  grp like '%the-forever-love-bear' then cnt end) as bear_bounce,
sum(case when  grp like '%cart%' then cnt end) as cart_total,
sum(case when  grp like '%cart' then cnt end) as cart_bounce,
sum(case when  grp like '%shipping%' then cnt end) as shipping_total,
sum(case when  grp like '%shipping' then cnt end) as shipping_bounce,
sum(case when  grp like '%billing-2%' then cnt end) as billing_total,
sum(case when  grp like '%billing-2' then cnt end) as billing_bounce,
sum(case when  grp like '%thank-you-for-your-order%' then cnt end) as thankyou_total
from cte2
),

cte_bounce_rate_bear as
(
select
home_bounce/home_total as home_bounce_rate,
product_bounce/product_total as product_bounce_rate,
bear_bounce/bear_total as bear_bounce_rate,
cart_bounce/cart_total as cart_bounce_rate,
shipping_bounce/shipping_total as shipping_bounce_rate,
billing_bounce/billing_total as billing_bounce_rate
from cte_total_and_bounce_bear
)
,
cte_conversion_funnel_bear as
(
select
home_total as total_home,
home_total-home_bounce as to_product,
product_total-product_bounce as to_bear,
bear_total-bear_bounce as to_cart,
cart_total-cart_bounce as to_shipping,
shipping_total-shipping_bounce as to_billing,
billing_total-billing_bounce as to_thankyou
from cte_total_and_bounce_bear
)
,
cte_click_rate_bear as
(
select
to_product/total_home as lander_click_rate,
to_bear/to_product as product_page_click_rate,# how many clicked on the product page
to_cart/to_bear as bear_click_rate,
to_shipping/to_cart as cart_click_rate,
to_billing/to_shipping as shiping_click_rate,
to_thankyou/to_billing as billing_click_rate 
from cte_conversion_funnel_bear
)
```

### Mr. Fuzzy Click Rates

The table below illustrates the click rates at various stages in the conversion funnel for Mr. Fuzzy:

```
| Stage                 | Click Rate |
|-----------------------|------------|
| Lander                | 0.7412     |
| Product Page          | 0.4813     |
| Mr. Fuzzy Page        | 0.4343     |
| Add to Cart           | 0.6865     |
| Proceed to Shipping   | 0.8205     |
| Proceed to Billing    | 0.6354     |
```

### Love Bear Click Rates

The table below illustrates the click rates at various stages in the conversion funnel for Love Bear:
```
| Stage                 | Click Rate |
|-----------------------|------------|
| Lander                | 0.6431     |
| Product Page          | 0.1755     |
| Love Bear Page        | 0.5488     |
| Add to Cart           | 0.6871     |
| Proceed to Shipping   | 0.8086     |
| Proceed to Billing    | 0.6184     |
```

**Analysis**

We had found that adding a second product increased overall CTR from the /products page ,for love bear page the ctr is **54.88%** and for mr.fuzzy it is **43.43%**, there is a difference of more than **11%**, this shows that the Love Bear has a better click rate to the /cart page and comparable rates than mr.fuzzy.


```python

```
