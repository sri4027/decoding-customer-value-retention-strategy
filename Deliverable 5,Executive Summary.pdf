DROP TABLE IF EXISTS customers;
 
CREATE TABLE customers (
    customer_id             INT,
    age                     INT,
    gender                  TEXT,
    item_purchased          TEXT,
    category                TEXT,
    purchase_amount_usd     NUMERIC,
    location                TEXT,
    size                    TEXT,
    color                   TEXT,
    season                  TEXT,
    review_rating           NUMERIC,
    subscription_status     INT,
    shipping_type           TEXT,
    discount_applied        INT,
    promo_code_used         INT,
    previous_purchases      INT,
    payment_method          TEXT,
    frequency_of_purchases  TEXT,
    is_promo_buyer          TEXT,
    freq_score              NUMERIC,
    spend_norm              NUMERIC,
    freq_norm               NUMERIC,
    prev_norm               NUMERIC,
    value_score             NUMERIC,
    value_tier              TEXT,
    satisfied               TEXT,
    loyal_a                 INT,
    loyal_b                 INT,
    segment                 TEXT,
    age_band                TEXT,
    discount_cost           NUMERIC
);


TRUNCATE TABLE customers;

COPY customers(customer_id, age, gender, item_purchased, category, purchase_amount_usd, location, size, color, season, review_rating, subscription_status, shipping_type, discount_applied, promo_code_used, previous_purchases, payment_method, frequency_of_purchases, is_promo_buyer, freq_score, spend_norm, freq_norm, prev_norm, value_score, value_tier, satisfied, loyal_a, loyal_b, segment, age_band, discount_cost)
FROM 'C:/temp/engineered_dataset.csv'
WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');






/*SQL Q1:What separates high-value customers from low-value ones, and which profiles show 
the strongest repeat purchase behaviore */
SELECT
    value_tier,
    COUNT(*)                                                        AS total_customers,
    ROUND(AVG(purchase_amount_usd), 2)                             AS avg_spend,
    ROUND(AVG(freq_score), 1)                                      AS avg_frequency,
    ROUND(AVG(previous_purchases), 1)                              AS avg_tenure,
    SUM(CASE WHEN previous_purchases >= 25 THEN 1 ELSE 0 END)      AS repeat_buyers_count,
    ROUND(SUM(CASE WHEN previous_purchases >= 25 THEN 1 ELSE 0 END)::numeric 
          / COUNT(*) * 100, 1)                                      AS repeat_buyer_pct,
    ROUND(AVG(CASE WHEN is_promo_buyer = 'True' THEN 1 ELSE 0 END)
          * 100, 1)                                                 AS promo_rate_pct,
    ROUND(AVG(review_rating), 2)                                   AS avg_rating
FROM customers
GROUP BY value_tier
ORDER BY avg_spend ASC;


/* value ones, and which profiles show the strongest repeat purchase behavior?
 Logic: We group customers by value_tier and look at how spend, frequency, tenure and repeat purchase rate change as tier increases.
 repeat_buyer_pct shows what % of each tier has 25+ previous purchases.

 Result:
 As tier goes from Low to Premium:
 Spend     : $41 → $57 → $65 → $76
 Frequency : 7.2 → 11.6 → 18.2 → 32.8 visits/year
 Tenure    : 14.4 → 23.1 → 28.7 → 35.2 purchases
 Repeat buyers : 18.1% → 43.5% → 63.7% → 81.3%

 The repeat buyer % shows the clearest separation of all.
Only 18% of Low tier customers are repeat buyers, but 81% of 
 Premium customers are. Frequency also jumps 4.5x from Low to 
 Premium. These two frequency and repeat purchase rate — are 
 the strongest separators between high and low value customers.

Promo rate stays flat (~43%) across all tiers,discount usage
does NOT separate high value from low value customers*/


-----------------SQL Q2 — Which seasons and categories are associated with lower-tenure vs high-tenure customers?------------------------

SELECT
    season,
    category,
    COUNT(*)                                                    AS total_customers,
    ROUND(AVG(previous_purchases), 1)                          AS avg_tenure,
    SUM(CASE WHEN previous_purchases < 25 THEN 1 ELSE 0 END)  AS low_tenure_count,
    SUM(CASE WHEN previous_purchases >= 25 THEN 1 ELSE 0 END) AS high_tenure_count,
    ROUND(AVG(purchase_amount_usd), 2)                         AS avg_spend,
    ROUND(AVG(CASE WHEN is_promo_buyer = 'True' 
              THEN 1 ELSE 0 END) * 100, 1)                     AS promo_rate_pct
FROM customers
GROUP BY season, category
ORDER BY avg_tenure ASC;
/* Which seasons and categories are associated with lower-tenure vs high-tenure customers?

Logic: We group by season + category and look at avg_tenure to see which combinations attract new customers (low tenure) vs which ones retain long-term customers (high tenure).

Result:
Lowest tenure combinations (new customer entry points):Spring Footwear (24.2) and Summer Outerwear (24.3)
These are likely the first categories new customers try.
Highest tenure combinations (retention categories):
Summer Accessories (26.8), Winter Footwear (26.7) and Winter Clothing (26.1)
Long-term customers keep coming back for these.
Fall Clothing has the lowest promo rate (35.6%) with 427 customers the strongest organic demand signal in the entire table. Customers buy this without needing discounts.

Key takeaway:
Footwear/Outerwear in Spring/Summer : entry point for new customers
Accessories/Footwear/Clothing in Winter:retention categories
Fall Clothing should be protected from discounts ,it already has strong organic demand.*/



---------------------SQL Q3: Which geographies signal organic demand vs discount driven volume?-----------------
/* SQL Q3: Which geographies signal organic demand vs discount driven volume?

 METHOD: Lift on two dimensions per state, relative to national
 baselines (overall organic loyalist rate = 15.4%, overall promo
 rate = 43.0%):
   - loyalist_lift = state's organic loyalist rate ÷ 15.4%
   - promo_lift    = state's promo rate ÷ 43.0%

 Classification:
   Organic Demand  : loyalist_lift >= 1.2 AND promo_lift <= 1.0
   Discount Driven : promo_lift >= 1.15 AND loyalist_lift <= 0.9
   Mixed           : everything else
*/

SELECT
    location,
    COUNT(*) AS total_customers,
    ROUND(AVG(purchase_amount_usd), 2) AS avg_spend,
    ROUND(AVG(CASE WHEN is_promo_buyer='True' THEN 1 ELSE 0 END)*100, 1) AS promo_rate_pct,
    ROUND(AVG(CASE WHEN is_promo_buyer='True' THEN 1 ELSE 0 END)
          / (SELECT AVG(CASE WHEN is_promo_buyer='True' THEN 1 ELSE 0 END) FROM customers), 2) AS promo_lift,
    ROUND(AVG(loyal_a::numeric)*100, 1) AS loyalist_rate_pct,
    ROUND(AVG(loyal_a::numeric)
          / (SELECT AVG(loyal_a::numeric) FROM customers), 2) AS loyalist_lift,
    ROUND(SUM(discount_cost), 2) AS total_discount_cost,
    CASE
        WHEN AVG(loyal_a::numeric) / (SELECT AVG(loyal_a::numeric) FROM customers) >= 1.2
             AND AVG(CASE WHEN is_promo_buyer='True' THEN 1 ELSE 0 END)
             / (SELECT AVG(CASE WHEN is_promo_buyer='True' THEN 1 ELSE 0 END) FROM customers) <= 1.0
            THEN 'Organic Demand'
        WHEN AVG(CASE WHEN is_promo_buyer='True' THEN 1 ELSE 0 END)
             / (SELECT AVG(CASE WHEN is_promo_buyer='True' THEN 1 ELSE 0 END) FROM customers) >= 1.15
             AND AVG(loyal_a::numeric) / (SELECT AVG(loyal_a::numeric) FROM customers) <= 0.9
            THEN 'Discount Driven'
        ELSE 'Mixed'
    END AS demand_signal
FROM customers
GROUP BY location
ORDER BY loyalist_lift DESC;

/*
Organic Demand : 8 states (616 customers, $2,740 discount cost)
These states have an above-average share of genuinely loyal customers and below or average promo reliance. 
Alaska leads on both dimensions 53% more loyalists than the national rate, paired with the highest average 
spend ($67.60) of any state in the dataset
. Kansas has the lowest promo rate in the country (23.8%, well below the national 43%)
but the lowest average spend among this group ($54.56),its loyalty is real but not yet commercially valuable.

Discount Driven :4 states (315 customers, $1,970 discount cost)
Indiana has the highest promo reliance in the dataset (33% above the national average) and below average loyalty.
Missouri and West Virginia both sit at 0.64 loyalist lift only 
about two-thirds the national loyalty rate despite West Virginia having a relatively
high average spend ($63.88), meaning its higher spending customers are disproportionately discount-dependent.

Mixed :38 states (2,969 customers, $15,172 discount cost)
This group accounts for 76% of all customers and 78% of total geographic discount spend,
but no individual state shows an extreme signal in either direction.
Pennsylvania ($66.57 spend, loyalist lift 1.23) and Connecticut (loyalist lift 1.16) sit 
just outside the Organic Demand cutoff on promo lift, making them the closest near misses.

*/
 


-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

/* KEY QUESTION 1: Who are the genuinely loyal customers vs discount buyers?*/

SELECT 
    segment,
    COUNT(*)                                    AS total_customers,
    ROUND(AVG(CASE WHEN is_promo_buyer = 'True' THEN 1 ELSE 0 END) * 100, 1)  AS promo_rate_pct,
    ROUND(AVG(purchase_amount_usd), 2)         AS avg_spend,
    ROUND(AVG(review_rating), 2)               AS avg_rating,
    ROUND(AVG(previous_purchases), 1)          AS avg_tenure,
    SUM(loyal_a)                               AS loyal_a_count,
    SUM(loyal_b)                               AS loyal_b_count
FROM customers
GROUP BY segment
ORDER BY promo_rate_pct ASC;
/* Logic: We group all 3900 customers by segment and look at promo rate, spend, rating and tenure for each. Sorting by promo_rate_pct 
puts the most loyal segment at top and the most discount-driven at bottom.

 Result:
 Organic Loyalists (0% promo) and High-Value Retainers (0% promo) never use discounts. Promo Hunters (100% promo) ONLY buy on 
discounts.

 Organic Loyalists have the highest rating (4.38) and longest tenure (37.5 purchases) the clearest sign of genuine loyalty.
 High-Value Retainers spend the most ($77.07) and never use discounts either, but their lower rating (3.33) and shorter 
tenure (31.0) means they didn't fully qualify as Loyal_A.

loyal_a_count is 600 — all of them are in Organic Loyalist, by 
design (Loyal_A is checked first when building segments).

 loyal_b_count is 343 in At-Risk and 15 in Emerging Loyalist — 
both of these segments still have a 10%+ promo rate. This 
proves Loyal_B (revenue-based definition) doesn't capture real 
 loyalty — even "Loyal_B" customers are partly discount dependent.

 Conclusion: Organic Loyalists are the brand's genuinely loyal customers. Promo Hunters are the opposite 100% deal dependent.*/

 

--KEY QUESTION 2 — What behavioral patterns predict high customer value?

SELECT
    value_tier,
    COUNT(*)                                        AS total_customers,
    ROUND(AVG(purchase_amount_usd), 2)             AS avg_spend,
    ROUND(AVG(freq_score), 1)                      AS avg_frequency,
    ROUND(AVG(previous_purchases), 1)              AS avg_tenure,
    ROUND(AVG((is_promo_buyer::boolean)::int::numeric) * 100, 1)  AS promo_rate_pct,
    ROUND(AVG(review_rating), 2)                   AS avg_rating,
    COUNT(CASE WHEN segment = 'Organic Loyalist' 
               THEN 1 END)                         AS organic_loyalists
FROM customers
GROUP BY value_tier
ORDER BY avg_spend ASC;

/* Q2: What behavioral patterns predict high customer value?

Logic: We look at each value tier (Low to Premium) and check if 
spend, frequency, tenure and promo rate follow a clear pattern. 
If Premium customers consistently show certain behaviors, those 
behaviors predict high value.

Result:

As tier goes from Low to Premium:

Spend     : $41 → $57 → $65 → $76
Frequency : 7.2 → 11.6 → 18.2 → 32.8 visits/year
Tenure    : 14.4 → 23.1 → 28.7 → 35.2 purchases
Organic Loyalists : 48 → 110 → 191 → 251

Spend, frequency, tenure and organic loyalist count all increase 
clearly and consistently as tier goes up. Frequency shows the 
biggest jump Premium customers buy 4.5x more often than Low.

But promo_rate_pct stays almost flat (~43-44%) across all tiers, 
even Premium. This means even the brand's highest spending 
customers are still discount dependent at the same rate as Low 
tier customers.

Conclusion: Frequency and tenure are the strongest predictors of 
customer value not spend alone, and definitely not promo 
behavior, which doesn't change across tiers at all.
*/



-- Q3: Which geographies and demographics are commercially underlevered?


--PART A: Geography
SELECT
    location,
    COUNT(*)                                                            AS total_customers,
    ROUND(AVG(purchase_amount_usd), 2)                                 AS avg_spend,
    ROUND(AVG(CASE WHEN is_promo_buyer = 'True' THEN 1 ELSE 0 END) 
          * 100, 1)                                                     AS promo_rate_pct,
    COUNT(CASE WHEN segment = 'Organic Loyalist' THEN 1 END)           AS organic_loyalists,
    ROUND(COUNT(CASE WHEN segment = 'Organic Loyalist' THEN 1 END)::numeric 
          / COUNT(*) * 100, 1)                                          AS organic_loyalist_rate_pct,
    ROUND(SUM(discount_cost), 2)                                       AS total_discount_cost
FROM customers
GROUP BY location
ORDER BY organic_loyalist_rate_pct DESC, promo_rate_pct ASC
LIMIT 10;





--PART B: Demographics

SELECT
    age_band,
    COUNT(*)                                                            AS total_customers,
    ROUND(AVG(purchase_amount_usd), 2)                                 AS avg_spend,
    ROUND(AVG(CASE WHEN is_promo_buyer = 'True' THEN 1 ELSE 0 END) 
          * 100, 1)                                                     AS promo_rate_pct,
    COUNT(CASE WHEN segment = 'Organic Loyalist' THEN 1 END)           AS organic_loyalists,
    ROUND(AVG(review_rating), 2)                                       AS avg_rating
FROM customers
GROUP BY age_band
ORDER BY promo_rate_pct ASC, avg_spend DESC;
/* Q3
PART A: Geography:Which states are commercially underlevered?

Logic: We rank states by organic_loyalist_rate_pct (% of that 
state's customers who are Organic Loyalists) and promo_rate_pct.
A high loyalist rate + low promo rate = strong organic demand.

Result:

Alaska is the strongest state highest loyalist rate (23.6%) 
AND highest avg spend ($67.60) among the top states.

Alabama, Georgia and Montana form the next tier (20.8-22.5% 
loyalist rate). Montana has the largest loyal base (20 
loyalists out of 96 customers).

Kansas has the lowest promo rate (23.8%) but low spend ($54.56) 
and a lower loyalist rate (19%) these customers don't need 
discounts but haven't become high-value yet.

Conclusion: Alaska, Alabama, Georgia and Montana are the 
strongest organic demand states high loyalty AND high spend. 
Kansas is a different opportunity low promo dependency but 
needs to be converted into higher spend.
*/



--KEY QUESTION 4 — How should the brand restructure its promo strategy?

SELECT
    segment,
    COUNT(*)                                                        AS total_customers,
    ROUND(AVG(CASE WHEN is_promo_buyer = 'True' THEN 1 ELSE 0 END) 
          * 100, 1)                                                 AS promo_rate_pct,
    ROUND(AVG(purchase_amount_usd), 2)                             AS avg_spend,
    ROUND(SUM(discount_cost), 2)                                   AS total_discount_cost,
    ROUND(AVG(discount_cost), 2)                                   AS avg_discount_cost,
    ROUND(SUM(discount_cost) / SUM(purchase_amount_usd) * 100, 1) AS discount_as_pct_revenue
FROM customers
GROUP BY segment
ORDER BY total_discount_cost DESC;


/*
Logic: We look at total_discount_cost and discount_as_pct_revenue 
per segment to see where the brand is losing the most margin to 
discounts, and how risky/safe it is to cut discounts for each.

Result:

Promo Hunters are the most dangerous segment 100% promo rate 
and 20% of their revenue is given away as discounts. Their avg 
spend is only $48 the brand is discounting its cheapest 
buyers the most. Stopping discounts here saves margin without 
losing valuable customers.

At-Risk Customers are the biggest absolute drain $11,219.20 
in discount costs — but only 42.3% are promo buyers. The other 
57.7% already buy without discounts, so a gradual reduction is 
safer here rather than a full stop.

Organic Loyalists and High-Value Retainers cost $0 in discounts 
these segments must NEVER be discounted. They prove the brand 
can retain customers without promotions.

Priority order for promo sunset:

1. Promo Hunters first  — 100% promo, lowest spend, 20% revenue loss
2. At-Risk second       — gradual reduction, majority don't need discounts
3. Never touch          — Organic Loyalists and High-Value Retainers
*/



-- Q5: What does the brand's ideal customer profile look like, and how can it acquire more of them?

SELECT
    'gender' AS dimension, gender AS value,
    COUNT(*) AS customers,
    ROUND(AVG(loyal_a::numeric)*100, 1) AS loyalist_rate_pct,
    ROUND(AVG(loyal_a::numeric) / (SELECT AVG(loyal_a::numeric) FROM customers), 2) AS lift
FROM customers GROUP BY gender

UNION ALL

SELECT
    'subscription', subscription_status::text,
    COUNT(*),
    ROUND(AVG(loyal_a::numeric)*100, 1),
    ROUND(AVG(loyal_a::numeric) / (SELECT AVG(loyal_a::numeric) FROM customers), 2)
FROM customers GROUP BY subscription_status

UNION ALL

SELECT
    'category', category,
    COUNT(*),
    ROUND(AVG(loyal_a::numeric)*100, 1),
    ROUND(AVG(loyal_a::numeric) / (SELECT AVG(loyal_a::numeric) FROM customers), 2)
FROM customers GROUP BY category

UNION ALL

SELECT
    'payment_method', payment_method,
    COUNT(*),
    ROUND(AVG(loyal_a::numeric)*100, 1),
    ROUND(AVG(loyal_a::numeric) / (SELECT AVG(loyal_a::numeric) FROM customers), 2)
FROM customers GROUP BY payment_method

UNION ALL

SELECT
    'shipping_type', shipping_type,
    COUNT(*),
    ROUND(AVG(loyal_a::numeric)*100, 1),
    ROUND(AVG(loyal_a::numeric) / (SELECT AVG(loyal_a::numeric) FROM customers), 2)
FROM customers GROUP BY shipping_type

ORDER BY dimension, lift DESC;
/*

Logic: For every customer attribute (gender, subscription, 
category, payment method, shipping type), we compare the 
loyalist rate within that group vs the overall loyalist rate 
(15.4%). This is called "lift":

Lift > 1 = this group has MORE loyalists than average
Lift = 1 = no difference, no signal
Lift < 1 = this group has FEWER loyalists than average

Result:

Category, payment method, and shipping type ALL have lift 
between 0.90 and 1.13 basically no signal. None of these 
attributes meaningfully predict loyalty.

Only TWO attributes show real signal:

Gender :Female customers have 1.65x lift (25.4% loyalist rate 
vs 10.7% for males). Females are the strongest predictor of 
loyalty in the entire dataset. But females are only 1,248 out 
of 3,900 customers (32%) a small base with a strong signal.

Subscription: This is the most surprising finding. Customers 
WITHOUT a subscription (status 0) have a 21.1% loyalist rate 
(1.37x lift). Customers WITH a subscription (status 1) have 
0.0% loyalist rate (0.00x lift) literally ZERO loyal 
customers out of 1,053 subscribers.

Conclusion:

1. HIGHEST PRIORITY — Fix the subscription program. A 0.00 lift 
   on a program meant to drive loyalty is a product design 
   failure, not a targeting problem. It needs to be rebuilt 
   around non-discount value (early access, styling perks) 
   before spending any acquisition budget on it.

2. SECOND PRIORITY — Acquire more female customers. At 1.65x 
   lift, gender is the strongest lever available. Females are 
   only 32% of the base — even modest growth here compounds 
   directly into a higher overall loyalist rate.
*/









