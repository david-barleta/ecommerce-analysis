# ecommerce-analysis

## Executive Summary



## Project Background
Olist was founded on February 2015 with the mission to empower small and medium-sized businesses in Brazil by providing an online platform for selling their items.


## Data Structure & Initial Checks


Although the timeframe of the orders starts from September 2016, the data for 2016 only contains 3 months: September, October, and December. September & December only contain 1-5 orders, while November is completely missing. Hence, the 2016 data can't be reliably used for trend analysis, as the data effectively starts in January 2017 for meaningful insights. 

### Data Model
![[data_model.png]]

The final data model in Power BI is shown above. The main fact table is the `fact_order_items` table, while the `fact_orders`, which is a "factless" fact table, serves a bridge table that connects the records in the `fact_order_items` table to the other dimension tables. 

Two summary tables were also created: `dim_seller_segments`, which groups sellers by the average review scores for their products, and `dim_customer_segments`, which groups customers by their order count.


### Query Structure
![[queries_structure.png]]

The query structure (shown above) is divided into three groups:
1. Raw group: 
2. Transformed group – transformations done:
	- 
3. Model group:  The tables were categorized into either a fact table or a dimension table.


## Insights Deep Dive
--- 
### Executive Overview

**Q1: How has total revenue trended over time, and is the business growing?**
- Total revenue reached 13.2M across the entire timeframe of the dataset (late 2016 to August 2018), with a total of 96.48K delivered orders at an average order value of 137.
- The platform grew 8x in 2017, with revenue scaling from 111.80K in January 2017 to 987.77K in November 2017. The revenue spike in November 2017, which is aligned with the Black Friday sale, also marked the peak of the business's revenue.
- By 2018, the monthly revenue stabilized in the 820K to 977K range through August, which suggests that the business's phase had transitioned from high early growth to a mature, steadier phase. 
- The full total revenue in 2017 was 5.96M, while the total revenue in 2018 (only up to August) was 7.22M. This implies an annualized **run rate** of roughly 10.8M, which would indicate a 81% year-over-year growth from 2017 if the pace of monthly revenue held through December 2018.
- The revenue trend also shows a drop in June for both 2017 and 2018, with a 12%-14% decline in revenue and 9%-12% decline in orders from May to June, although this is followed by a partial recovery in July. The AOV barely moves during this period, which suggests that this is driven by order volume and likely a seasonal pattern.


**Q2: How many orders were placed per month, and does order volume track with revenue?** 
- Order volume moved almost perfectly in sync with the revenue, which suggests that the business's revenue growth was primarily driven by order volume, not by customers spending more per order.
- During the 2017 growth phase, monthly orders scaled from 750 in January to a peak of 7.29K in November, which mirrored in the revenue growth in the same period.
- The average order value was somewhat fluctuating in early 2017 during the business's early growth, ranging from 123 in July to 151 in January, but stabilized in the 125 to 145 range by late 2017 and held there through August 2018. This suggests the platform's product mix and pricing had matured by then, with no meaningful drift in either direction. ***(revise this part)***
- Similar to the revenue trend in 2018, order volume also plateaued in 2018, with the monthly orders settled in the 6K to 7K range. Since the two metrics moved together, AOV remained stable instead of increasing, which suggests that the business hasn't yet found a different strategy to grow revenue independently of order volume. 

**Q3: Which product categories generate the most revenue and have the highest order volume?**
**Q4: What is the average order value, and how does it vary by product category?**
- The product category rankings shift depending on the whether they are sorted by revenue or items sold. 
- By items sold, Bed Bath Table is the top selling product with 10.9K delivered orders, ahead of Health Beauty at 9.4K These are followed by Sports Leisure (8.4K), Furniture Decor (8.1K), and Computers Accessories (7.6K). These five categories account for majority of the total order volume, while the long tail of the product category list contains categories with less than 200 items sold across the entire timeframe. This distribution represents a typical online marketplace platform where order volume is concentrated on a small number of popular categories.
- By revenue, the ranking is different. Health and Beauty moves to the top at 1.23M, followed by Watches Gifts at 1.17M, Bed Bath Table at 1.02M, Sports Leisure at 955K, and Computer Accessories at 889K.
- The most notable shift is Watches Gifts, which jumped to 2nd in revenue despite ranking 7th in items sold. This is driven by the category's inherently higher price points. With individual item prices ranging from 8.99 to 3,999.90, and a median of 129, the category contains a mix of affordable gift items and genuine watches that skew the AOV to 212 per order. It's worth noting that the product type itself drives higher prices, which is what increases its AOV rather than a particular customer spending behavior.
- The total AOV across all products is at 138, but it greatly varies by category. For instance, Computers tops the list at 1.23K per order, followed by Small Appliances Home Oven and Coffee at 632, and Home Appliances at 484. At the bottom are Home Comfort (32), Flowers (39), and Diapers And Hygiene (58), which are all below the total AOV.
- Understanding what drives a category's AOV matters before drawing conclusions from it. A high AOV can mean two different things: either the products in that category are consistently expensive, or a small number of highly priced items are pulling the average upwards while the usual item prices stay at average prices.

### Operational Insights

**Q5: What is the average delivery time, and how has it changed over time?** 
- The average delivery days fluctuated throughout the timeframe of the data. It started at 12.75 days in January 2017, worsened to 16.88 days in February 2018, and then improved to 7.66 days by August 2018. The downward trend starting from February 2018 suggests that that 
- Two spikes in the trend are worth highlighting. The first occurred in November and December 2017, where the average delivery days rose up to 15.07 and 15.31, respectively. This coincided with the Black Friday volume surge and subsequent holiday season. The platform processed its highest order volumes in 2017 during the said months, so the logistics clearly struggled to keep up with the demand. 
- The second, larger spike happened in February and March 2018, reaching 16.88 and 16.24 days, respectively. This is a bit difficult to attribute to a single event since order volumes in the said months were not unusually high. This suggests a logistics or carrier capacity issue rather than a problem with keeping up with the demand.
- As for the average fulfillment days, it stayed consistently low throughout the entire timeframe, ranging between 2.6 and 4.0 days across all months. This shows that the platform sellers were processing and fulfilling orders reliably, and the bottleneck can be entirely attributed on the carrier delivery side, not on the seller side.


**Q6: How often do orders arrive later than the estimated delivery date (on time vs. late orders), and how fast do sellers complete the fulfillment of orders?** 

**Q7: What is the freight cost as a percentage of product revenue by category, and which categories are most freight-intensive?**

**Q8: How does order volume and revenue vary by state, and which states are underserved?**



### Sellers & Customers

**Q9: Is revenue concentrated among a small number of sellers, and what's the risk of seller dependency?**

**Q10: What's the distribution of customer review scores?**

**Q12: How does delivery time affect the review scores?**

**Q13: How do review scores affect seller revenue performance?**

**Q15: How many customers are repeat buyers vs. (one-time buyers), and what share of revenue do they contribute?** 



## Recommendations
---
Based on the insights and findings above, the following are recommendations for the sales team:

- Specific observation that is related to a recommended action. **Recommended or general guidance based on this observation.**

- The slowdown in monthly growth rate going into 2018 for both revenue and order volume, while not necessarily a red flag, signals that the high early growth phase of the business is over. Hence, the business should investigate whether the plateau reflects market saturation in the existing regions in existing regions, increased competition, delivery capacity constraints, or seller supply limitations. Accordingly, in relation to the ratio of one-time customers to returning customers, the business should also explore customer retention-focused strategies rather than pure acquisition.
- As the business also experiences seasonality in sales, the business should explore potential promotional events, as well as effectively plan logistics capacity and seller readiness ahead of high-demand periods. 
- The strong correlation between revenue and order volume, while healthy in the short term, reveals a strategic gap: there is no visible growth for AOV yet. Hence, the business should explore strategies like bundling, upselling, or curated product recommendations that could increase basket size, considering that AOV has been flat for over a year.
- For the products Bed Bath Table and Health Beauty which rank near the top on both dimensions (revenue and items sold), the business should focus on maintaining seller quality and logistics reliability since any problem that would arise in these two categories would have a huge impact on the business's overall order metrics.
- For categories where high AOV reflects consistently high product prices, growing seller supply is the most direct revenue lever. These categories already attract transactions worth significantly more per order, so adding more seller supply translates predictable into higher revenue without proportionally growing operational costs. **(revise this part)**

Q5
- The high delivery times on February to March 2018 should be investigated. Was it caused by 



## Assumptions and Caveats
--- 
Multiple assumptions were made throughout the analysis to manage challenges with the data. These assumptions and caveats are given below:

- Assumption 1: Total Revenue only considers orders which have been delivered. This is in accordance with the accounting principle that revenue can only be recognized once the items have been delivered.
- Assumption 2: 


# Limitations
---
- While Olist launched in February 2015, the business only shared data starting from late 2016. Hence, this analysis doesn't cover the company's full operating history starting from its inception. For 2016, the source dataset only contains 3 months of data, which is not meaningful enough for trend analysis on its own. **(to be revised)**


# Recommendations
---
- Explore highest rated vs. lowest rated product categories
- 