# Olist E-commerce Analysis

Business analysis of the Olist Brazilian e-commerce public dataset (about 99k orders, spanning 2016 to 2018).
Tools: Python, DuckDB (SQL), pandas, matplotlib.

## Key findings and recommendations

Scope: 96,470 delivered orders and 93,350 customers (Sep 2016 to Aug 2018), worth R$13.2M in item revenue. Details, definitions and charts for each question are further down.

1. **Late delivery is the clearest driver of poor reviews.** Only 6.8% of orders arrive late, but they produce about a third (32.5%) of all 1 and 2 star reviews. The average score falls from 4.29 (on time) to 1.70 (more than 7 days late). This is an association rather than proven cause.
   *Suggested action:* treat late deliveries as the first lever for improving customer satisfaction.

2. **Late deliveries cluster in Rio de Janeiro and the northeast.** RJ and the nine northeastern states are about 22% of orders but about 40% of late orders. In RJ, 12.1% of orders are late versus 4.5% in São Paulo.
   *Suggested action:* start any carrier or fulfilment review with RJ and the northeast.

3. **Growth has come from new customers, not repeat buyers.** 97.0% of customers bought once, and 94.5% of revenue comes from one-time customers. About 30% of "repeat" buyers ordered again on the same day, so only around 2.1% of customers genuinely returned.
   *Suggested action:* retention is the largest untapped opportunity, so test post-purchase follow-ups for first-time buyers.

4. **Demand is strongly seasonal.** Black Friday (24 Nov 2017) brought R$150k in one day, 6.6x a typical day and about 15% of that month's revenue. Monthly revenue then levelled off at R$0.83 to 0.98M through 2018.
   *Suggested action:* plan stock and delivery capacity ahead of peak promotion days.

5. **Revenue is broad-based, and card instalments go with bigger orders.** The top 10 categories make up 62% of revenue, and watches_gifts earns its rank on high spend per order (about R$212 vs R$110 for bed_bath_table). Credit card is used in 77% of orders, and card orders have the highest typical value (median R$109), with 3.5 instalments on average.
   *Suggested action:* keep instalment options prominent at checkout, since card orders are larger (association, not proven cause).

## Business questions
1. Which product categories generate the most revenue, and how has revenue trended month by month?
2. How late are deliveries compared with the estimated date, and does lateness hurt review scores?
3. Which states have the most customers, and where is delivery slowest?
4. How many customers buy more than once? (In progress)
5. Which payment methods are used most, and how do they differ in order value? (In progress)

## Definitions and data notes
- **Revenue** = sum of item price, excluding freight. Freight is a shipping charge, not product sales.
- Revenue and delivery metrics use **delivered** orders only. Canceled, unavailable and in-progress orders are excluded.
- Customers are counted by `customer_unique_id` (one per person), not `customer_id` (one per order). Customer counts by state include all customers, whether or not their orders were delivered.
- State-level delivery statistics only include states with at least 100 delivered orders (AC, AP and RR are excluded).
- Currency is Brazilian reais (R$).
- Reviews are deduplicated to one per order (814 duplicate review IDs in the raw data).
- Monthly trends use full months only (Jan 2017 to Aug 2018). There are no delivered orders in Nov 2016 or after Aug 2018.
- Lateness is measured in calendar days against the estimated delivery date. 646 delivered orders (0.7%) have no review and are excluded from the review analysis.
- Order value in Q5 is the total the customer paid (items plus freight), so it is higher than item revenue. One delivered order has no payment record and is excluded from Q5.

## Steps to reproduce
1. Clone the repo and create an environment (Python 3.12 recommended):
   ```
   git clone https://github.com/eQuaEnoo/olist-ecommerce-analysis.git
   cd olist-ecommerce-analysis
   python -m venv .venv
   .venv\Scripts\activate
   pip install -r requirements.txt
   ```
2. Download the **Brazilian E-Commerce Public Dataset by Olist** from Kaggle and place the 9 CSV files in `data/raw/`.
3. Open `notebooks/01-explore.ipynb`, select the `.venv` kernel and run all cells in order. The first cell builds a local DuckDB database from the CSVs, and the rest run the cleaning and analysis.

The raw data and the database file are not stored in this repo (see `.gitignore`).

## Findings

### Q1: Revenue by category and over time
- Revenue is spread across many categories: the top 5 make up about 40% of revenue and the top 10 about 62%.
- health_beauty (R$1.23M, 9.3%) and watches_gifts (R$1.17M, 8.8%) lead. watches_gifts earns its place with far fewer orders than bed_bath_table (5,493 vs 9,272), meaning a higher spend per order (about R$212 vs R$110).
- Monthly revenue grew from about R$112k in Jan 2017 to R$648k in Oct 2017, then jumped 52% to R$988k in Nov 2017. Daily data shows why: on Black Friday (24 Nov 2017) revenue hit R$150k, 6.6x the median day in 1 to 20 Nov (R$23k) and about 15% of the whole month. Revenue stayed elevated in the days after and mostly above early-November levels through December, so the rise reflects the Black Friday spike plus a general seasonal lift. Monthly revenue then settled around R$0.83 to 0.98M through 2018.
- Jan to Aug 2018 revenue was about 2.4x the same months of 2017 (from a small early-2017 base).

![Monthly revenue](reports/monthly_revenue.png)
![Top categories](reports/top_categories.png)
![Daily revenue around Black Friday](reports/black_friday_daily.png)

### Q2: Delivery performance and reviews
- Most orders arrive early: the median delivered order arrived 12 days before the estimated date, and only 6.8% of 96,470 delivered orders were late (by 10.6 days on average).
- Lateness hurts reviews sharply. On-time orders average 4.29 stars with 9.3% rated 1 or 2. Orders 1 to 3 days late average 3.29 (32.1% low), and orders more than 7 days late average 1.70 (79.3% low).
- Late orders are 6.7% of reviewed orders but account for about a third (32.5%) of all 1 and 2 star reviews, so the small late tail drives a large share of customer dissatisfaction.
- This is an association, not proof of cause: late orders may also have other problems, but the score drops at every step of lateness.

![Delivery lateness vs review scores](reports/delivery_vs_reviews.png)

### Q3: Customers and delivery by state
- Customers are heavily concentrated: São Paulo (SP) has 41.9% of unique customers, followed by Rio de Janeiro (RJ, 12.9%) and Minas Gerais (MG, 11.7%). The top 3 states hold 66.5% of customers and the top 10 hold 90.4%.
- Delivery speed follows geography: the median is 7.2 days in SP, about 10 days in MG and PR, and 16 to 26 days across the north and northeast (AM 25.9, AL 22.3, PA 21.1, MA 19.2).
- Slow is not the same as late. AM is the slowest state but only 2.8% of its orders were late (small sample: 145 orders), while AL is both slow and late (21.4% late).
- Late deliveries are not just a big-state problem. SP and RJ together make up about 55% of delivered orders and about half (51%) of late orders, roughly in line with their size. The difference is within the pair: SP does better than its size (42% of orders, 28% of late orders), while RJ does worse (13% of orders, 23% of late orders; 12.1% of its orders are late vs 4.5% in SP) and has lower reviews (3.97 vs 4.25).
- States with higher late rates have lower average reviews, consistent with Q2: across the 24 states, the correlation between late rate and average review score is -0.91. The lowest-rated states (MA 3.83, AL 3.85, PA and SE 3.91) all have late rates above 11%. This is a state-level pattern, so it supports the order-level result in Q2 rather than replacing it.
- The nine northeastern states (MA, PI, CE, RN, PB, PE, AL, SE, BA) are about 9% of delivered orders but about 18% of late orders. Together with RJ, they make up about 22% of orders but about 40% of late orders, so they are where reducing late deliveries would most likely improve customer satisfaction.

![Delivery time by state](reports/delivery_by_state.png)
![Late deliveries vs reviews by state](reports/late_vs_reviews_by_state.png)

### Q4: Repeat customers
- 97.0% of customers (90,549 of 93,350 with a delivered order) bought only once. 2.8% bought twice (2,573) and 0.2% three or more times (228), so only 3.0% are repeat buyers.
- About 30% of those repeat buyers (29.6%) placed their second order on the same day as their first, which looks like a split purchase rather than a return visit. Excluding them, roughly 2.1% of customers came back on a later day. Half of repeat buyers placed their second order within 30 days of their first (median: 29 days).
- Repeat buyers are a small but valuable group: they are 3.0% of customers but 5.5% of revenue (R$728k of R$13.2M), and spent R$260 each on average versus R$138 for one-time customers, because they placed two or more orders.
- 94.5% of revenue comes from customers who bought once, which suggests growth in 2017 and 2018 came from acquiring new customers rather than from repeat purchases. Retention is the clearest opportunity in this dataset.
- Caveats: the data covers about two years, so recent customers had little time to buy again, and `customer_unique_id` is the dataset's own identifier, so a person using different details would count as separate customers. The true repeat rate may be somewhat higher than 3.0%.

### Q5: Payment methods
- Credit card dominates: it is used in 77.0% of delivered orders and accounts for 78.5% of the R$15.4M paid. Boleto (a Brazilian bank-slip payment) is a distant second (19.9% of orders, 18.0% of value), followed by voucher (3.8%) and debit card (1.5%). Order percentages add to more than 100% because 2.3% of orders (2,182) were paid with more than one method.
- Those mixed-payment orders are almost all a credit card combined with a voucher: vouchers appear in 3,679 orders, but only 1,498 were paid with a voucher alone.
- Among orders paid with a single method, credit card orders are the largest: median R$109 (average R$166), versus R$94 (R$144) for boleto, R$90 (R$140) for debit card and R$70 (R$105) for voucher. Credit card customers paid in 3.5 instalments on average, while the other methods are single payments.
- Averages sit well above medians for every method because a small number of large orders pull the mean up, so the medians are the better guide to a typical order.
- Order value here includes freight (R$15.4M paid overall versus R$13.2M of item revenue), so it is not directly comparable with the revenue figures above.

![Payment methods](reports/payment_methods.png)
