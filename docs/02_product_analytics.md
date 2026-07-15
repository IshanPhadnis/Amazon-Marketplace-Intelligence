# Business Question 1 — Top 10 Revenue-Generating Products

### 🎯 Objective
Identify the Top 10 products generating the highest revenue.

### 💼 Why does this matter?
High-revenue products are key revenue drivers. Identifying them helps Flipkart optimize inventory planning, prioritize marketing efforts, and ensure product availability.

### 🧠 Approach
- Join `products` and `order_items` using `product_id`
- Calculate revenue as `quantity × price`
- Aggregate revenue using `SUM()`
- Group by product
- Sort by revenue in descending order
- Return the Top 10 products

### 💻 SQL

```sql
SELECT
    p.product_id,
    p.category,
    p.product_type,
    SUM(oi.quantity * oi.price) AS revenue
FROM flipkart.order_items oi
JOIN flipkart.products p
ON oi.product_id = p.product_id
GROUP BY
    p.product_id,
    p.category,
    p.product_type
ORDER BY revenue DESC
LIMIT 10;
```

### 💡 Business Recommendation

Prioritize high-revenue products by maintaining sufficient inventory, featuring them in promotional campaigns, and monitoring demand to minimize stock-outs.

---


# Business Question 2 — Top 10 Best-Selling Products by Quantity Sold

### 🎯 Objective

Identify the Top 10 best-selling products based on the total quantity sold.

### 💼 Why does this matter?

Best-selling products indicate customer demand and purchasing behavior. Identifying these products helps Flipkart:

- Optimize inventory levels
- Prevent stock-outs
- Improve demand forecasting
- Prioritize marketing campaigns
- Negotiate better pricing with suppliers

---

## ✅ Solution 1 — Production Approach (Recommended)

### 🧠 Approach

The business only asked for the **Top 10 products**.

Since rankings are not required, the simplest and most efficient solution is to:

- Join `products` and `order_items`
- Calculate the total quantity sold using `SUM(quantity)`
- Group by product
- Sort by quantity sold
- Return the first 10 rows using `LIMIT`

### 💻 SQL

```sql
SELECT
    p.product_id,
    p.product_type,
    SUM(oi.quantity) AS total_quantity
FROM flipkart.order_items oi
JOIN flipkart.products p
ON oi.product_id = p.product_id
GROUP BY
    p.product_id,
    p.product_type
ORDER BY total_quantity DESC
LIMIT 10;
```

### 💡 Why this is the preferred solution

This query is:

- Easier to read
- Faster to execute
- Simpler to maintain
- Exactly matches the business requirement

Whenever the business simply asks for the "Top N" records, this is the preferred approach.

---

## ✅ Solution 2 — Using Window Functions (Learning Approach)

### 🧠 Approach

Instead of limiting the results directly, assign a rank to every product based on the quantity sold.

This approach is useful when the business wants:

- Product rankings
- Leaderboards
- To include ties
- Reporting dashboards

### 💻 SQL

```sql
WITH best_selling AS (
    SELECT
        p.product_id,
        p.product_type,
        SUM(oi.quantity) AS total_quantity
    FROM flipkart.order_items oi
    JOIN flipkart.products p
        ON oi.product_id = p.product_id
    GROUP BY
        p.product_id,
        p.product_type
),

ranked_products AS (
    SELECT
        *,
        DENSE_RANK() OVER (ORDER BY total_quantity DESC) AS product_rank
    FROM best_selling
)

SELECT *
FROM ranked_products
WHERE product_rank <= 10;
```

### 💡 Why use DENSE_RANK()?

Unlike `LIMIT`, `DENSE_RANK()` handles ties fairly.

Example:

| Product | Quantity | Rank |
|---------|---------:|-----:|
| Laptop | 120 | 1 |
| Mobile | 115 | 2 |
| Speaker | 100 | 3 |
| Sandals | 95 | 4 |
| Shoes | 95 | 4 |
| Watch | 95 | 4 |

If multiple products have the same quantity sold, they receive the same rank.

As a result, filtering with:

```sql
WHERE product_rank <= 10
```

may return **more than 10 rows**, ensuring that tied products are not excluded unfairly.

---

## 🏆 Which approach should you use?

For this business question, **Solution 1 is the better choice** because the stakeholder only requested the Top 10 products.

Use **Solution 2** when rankings or tied positions are important, such as leaderboards, executive reports, or performance analysis.

---

### 💡 Business Recommendation

Ensure the highest-selling products are consistently available in inventory, prioritize them in promotional campaigns, and monitor demand trends to minimize stock-outs during peak sales periods.

---

# Business Question 3 — Product Category Generating the Highest Revenue

### 🎯 Objective

Identify the product category that generates the highest total revenue.

### 💼 Why does this matter?

Understanding which product category contributes the most revenue helps Flipkart:

- Allocate marketing budgets effectively
- Prioritize inventory investments
- Focus on high-performing categories
- Improve category-level business strategy

### 🧠 Approach

- Join `products` and `order_items` using `product_id`
- Calculate revenue as `quantity × price`
- Aggregate revenue by category
- Sort categories by total revenue
- Return the highest revenue-generating category

### 💻 SQL

```sql
SELECT
    p.category,
    SUM(oi.quantity * oi.price) AS total_revenue
FROM flipkart.order_items oi
JOIN flipkart.products p
ON oi.product_id = p.product_id
GROUP BY
    p.category
ORDER BY total_revenue DESC
LIMIT 1;
```

### 💡 Business Recommendation

Prioritize the highest-performing category by ensuring adequate inventory, increasing marketing investment, and expanding the product assortment to maximize revenue growth.

---

# Business Question 4 — Highest Revenue-Generating Product in Each Category

### 🎯 Objective

Identify the highest revenue-generating product within each product category.

### 💼 Why does this matter?

Identifying the top-performing product in each category helps Flipkart:

- Identify category leaders
- Prioritize inventory for flagship products
- Allocate marketing budgets effectively
- Benchmark other products within the same category

### 🧠 Approach

- Join `products` and `order_items` using `product_id`
- Calculate product revenue as `quantity × price`
- Aggregate revenue for each product
- Use `ROW_NUMBER()` with `PARTITION BY category` to rank products within each category
- Return only the highest revenue-generating product from each category

### 💻 SQL

```sql
WITH product_revenue AS (
    SELECT
        p.category,
        p.product_id,
        p.product_type,
        SUM(o.quantity * o.price) AS revenue
    FROM `flipkart.products` p
    JOIN `flipkart.order_items` o
        ON p.product_id = o.product_id
    GROUP BY
        p.category,
        p.product_id,
        p.product_type
),

ranked_products AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY category
            ORDER BY revenue DESC
        ) AS rn
    FROM product_revenue
)

SELECT
    category,
    product_id,
    product_type,
    revenue
FROM ranked_products
WHERE rn = 1;
```

### 💡 Business Recommendation

Use the highest-performing product in each category as a flagship product for promotions, ensure consistent inventory availability, and analyze the factors contributing to its success to improve the performance of other products in the same category.

---

# Business Question 5 — Products with the Highest Return Rate

### 🎯 Objective

Identify the products with the highest return rate based on actual transaction data.

### 💼 Why does this matter?

Products with high return rates can negatively impact profitability, customer satisfaction, and operational costs. Identifying them helps Flipkart:

- Detect quality issues
- Improve supplier performance
- Reduce refund costs
- Improve customer experience

### 🧠 Approach

- Join `products` with `order_items` using `product_id`
- Left join `return_status` using `order_id`
- Count total orders for each product
- Count returned orders for each product
- Calculate the return rate as:

  **(Returned Orders / Total Orders) × 100**

- Rank products by return rate

### 💻 SQL

```sql
SELECT
    p.product_id,
    p.product_type,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    COUNT(DISTINCT rs.order_id) AS returned_orders,
    ROUND(
        COUNT(DISTINCT rs.order_id) * 100.0 /
        COUNT(DISTINCT oi.order_id),
        2
    ) AS return_rate
FROM flipkart.products p
JOIN flipkart.order_items oi
    ON p.product_id = oi.product_id
LEFT JOIN flipkart.return_status rs
    ON oi.order_id = rs.order_id
GROUP BY
    p.product_id,
    p.product_type
ORDER BY return_rate DESC;
```

### 💡 Business Recommendation

Investigate products with the highest return rates to identify root causes such as product quality, inaccurate descriptions, supplier issues, or shipping damage. Reducing return rates can improve customer satisfaction while lowering logistics and refund costs.

---

# Business Question 6 — High Revenue Products with Low Customer Ratings

### 🎯 Objective

Identify products that generate high revenue despite having below-average customer ratings.

### 💼 Why does this matter?

Products with strong sales but poor ratings may indicate underlying quality or customer experience issues. Identifying them helps Flipkart:

- Detect products at risk of losing future sales
- Improve product quality
- Reduce negative customer reviews
- Improve long-term customer satisfaction

### 🧠 Approach

- Calculate the average rating across all products
- Join `products` and `order_items` using `product_id`
- Calculate revenue as `quantity × price`
- Filter products whose rating is below the overall average
- Sort products by revenue in descending order

### 💻 SQL

```sql
WITH avg_rating AS (
    SELECT AVG(average_rating) AS avg_rating
    FROM flipkart.products
)

SELECT
    p.product_id,
    p.product_type,
    SUM(o.quantity * o.price) AS revenue,
    p.average_rating
FROM flipkart.products p
JOIN flipkart.order_items o
    ON p.product_id = o.product_id
WHERE p.average_rating < (
    SELECT avg_rating
    FROM avg_rating
)
GROUP BY
    p.product_id,
    p.product_type,
    p.average_rating
ORDER BY revenue DESC;
```

### 💡 Business Recommendation

Investigate products generating high revenue despite poor customer ratings. Improve product quality, packaging, or descriptions before customer dissatisfaction begins to impact future sales and brand reputation.

---

# Business Question 7 — Product Categories with the Highest Average Customer Ratings

### 🎯 Objective

Identify the product categories with the highest average customer ratings.

### 💼 Why does this matter?

Highly rated product categories indicate strong customer satisfaction and product quality. Identifying these categories helps Flipkart:

- Invest more in high-performing categories
- Identify best-performing suppliers
- Improve lower-rated categories
- Enhance customer trust and retention

### 🧠 Approach

- Calculate the average customer rating for each category
- Group products by category
- Sort categories by average rating in descending order

### 💻 SQL

```sql
SELECT
    category,
    ROUND(AVG(average_rating), 2) AS avg_customer_rating
FROM flipkart.products
GROUP BY category
ORDER BY avg_customer_rating DESC;
```

### 💡 Business Recommendation

Focus marketing and inventory investments on highly rated categories while analyzing lower-rated categories to identify opportunities for product quality improvements, better supplier management, or enhanced customer experience.

---

# Business Question 8 — Products Priced Above the Average Product Price

### 🎯 Objective

Identify products priced above the average product price.

### 💼 Why does this matter?

Premium-priced products contribute to revenue and profit but may require different pricing and marketing strategies. Identifying these products helps Flipkart:

- Analyze premium product performance
- Evaluate pricing strategies
- Design targeted marketing campaigns
- Monitor premium inventory

### 🧠 Approach

- Calculate the average product price using a subquery
- Compare each product's price with the overall average
- Return products priced above the average

### 💻 SQL

```sql
SELECT
    product_id,
    product_type,
    price
FROM flipkart.products
WHERE price > (
    SELECT AVG(price)
    FROM flipkart.products
);
```

### 💡 Business Recommendation

Analyze whether premium-priced products justify their pricing through strong customer ratings and sales performance. High-priced products with poor ratings may require pricing or quality improvements.

---

# Business Question 9 — Products That Have Never Been Ordered

### 🎯 Objective

Identify products that have never been ordered by customers.

### 💼 Why does this matter?

Products that have never been purchased may indicate poor demand, ineffective pricing, lack of visibility, or inventory issues. Identifying these products helps Flipkart:

- Reduce dead inventory
- Improve product recommendations
- Optimize inventory costs
- Re-evaluate pricing and marketing strategies

### 🧠 Approach

- Start with the `products` table to include all products
- Left join `order_items` using `product_id`
- Identify products with no matching order records
- Filter using `WHERE order_id IS NULL`

### 💻 SQL

```sql
SELECT
    p.product_id,
    p.product_type,
    p.category
FROM flipkart.products p
LEFT JOIN flipkart.order_items oi
ON p.product_id = oi.product_id
WHERE oi.order_id IS NULL;
```

### 💡 Business Recommendation

Review products that have never been ordered to determine whether they should be promoted, repriced, bundled with other products, or discontinued to reduce inventory holding costs.

---

# Business Question 10 — Categories Contributing to 80% of Total Revenue (Pareto Analysis)

### 🎯 Objective

Identify the product categories that contribute to approximately 80% of Flipkart's total revenue.

### 💼 Why does this matter?

In many businesses, a small number of categories generate the majority of revenue (Pareto Principle or the 80/20 Rule). Identifying these categories helps Flipkart:

- Prioritize marketing investments
- Allocate inventory efficiently
- Focus business strategy on high-impact categories
- Improve revenue forecasting and planning

### 🧠 Approach

- Calculate total revenue for each product category.
- Sort categories by revenue in descending order.
- Calculate the cumulative (running) revenue.
- Calculate the total marketplace revenue.
- Compute the cumulative revenue percentage.
- Identify the categories contributing to approximately 80% of total revenue.

### 💻 SQL

```sql
WITH category_revenue AS (
    SELECT
        p.category,
        SUM(oi.quantity * oi.price) AS revenue
    FROM flipkart.products p
    JOIN flipkart.order_items oi
        ON p.product_id = oi.product_id
    GROUP BY
        p.category
),

running_category_revenue AS (
    SELECT
        category,
        revenue,
        SUM(revenue) OVER (
            ORDER BY revenue DESC
        ) AS running_revenue,
        SUM(revenue) OVER () AS total_revenue
    FROM category_revenue
),

running_total_percentage AS (
    SELECT
        *,
        ROUND((running_revenue / total_revenue) * 100, 2) AS cumulative_percentage
    FROM running_category_revenue
)

SELECT *
FROM running_total_percentage
WHERE cumulative_percentage <= 80;
```

### 📝 Explanation

This analysis uses the **Pareto Principle (80/20 Rule)**, which states that a relatively small number of categories often contribute to the majority of revenue.

The solution is built in three stages:

1. **Category Revenue**
   - Calculate total revenue for each category.

2. **Running Revenue**
   - Sort categories by revenue in descending order.
   - Calculate a cumulative (running) revenue using a window function.
   - Calculate the overall marketplace revenue.

3. **Cumulative Percentage**
   - Divide the running revenue by the total revenue to determine how much of the total revenue has been accumulated after each category.

For example:

| Category | Revenue | Running Revenue | Total Revenue | Cumulative % |
|----------|---------:|----------------:|--------------:|-------------:|
| Electronics | 15,756,072 | 15,756,072 | 20,436,608 | 77.10 |
| Fashion | 2,100,000 | 17,856,072 | 20,436,608 | 87.38 |

In this dataset, **Electronics alone contributes 77.1% of Flipkart's revenue**, which is why it is returned by the query.

> **Note:** In real-world analytics, stakeholders may also want to include the first category that causes the cumulative percentage to exceed 80% (Fashion in this example), since Electronics alone does not actually reach the 80% threshold. The exact implementation depends on the business requirement.

### 💡 Business Recommendation

Since Electronics contributes the majority of marketplace revenue, Flipkart should:

- Prioritize inventory availability for Electronics.
- Allocate a larger share of the marketing budget to this category.
- Monitor category performance closely, as any decline could significantly impact overall revenue.
- Continue optimizing lower-performing categories to diversify revenue sources and reduce business risk.

---

# Business Question 11 — Product Categories with the Highest Average Revenue per Order

### 🎯 Objective

Identify which product categories generate the highest average revenue per order.

### 💼 Why does this matter?

Average Revenue per Order helps measure how much customers spend whenever they purchase products from a particular category. Categories with a higher average revenue per order often represent premium products or higher customer spending.

This metric helps Flipkart:

- Identify premium-performing categories
- Optimize pricing and promotional strategies
- Allocate marketing budgets effectively
- Prioritize high-value product categories

### 🧠 Approach

- Calculate total revenue generated by each category.
- Count the total number of orders for each category.
- Divide total revenue by total orders to calculate the average revenue per order.
- Rank categories from highest to lowest average revenue.

### 💻 SQL

```sql
WITH metrics_revenue AS (
    SELECT
        p.category,
        SUM(oi.price * oi.quantity) AS total_revenue,
        COUNT(oi.order_id) AS total_orders
    FROM flipkart.products p
    JOIN flipkart.order_items oi
        ON p.product_id = oi.product_id
    GROUP BY p.category
)

SELECT
    category,
    total_revenue,
    total_orders,
    ROUND(SAFE_DIVIDE(total_revenue, total_orders), 2) AS avg_revenue_per_order
FROM metrics_revenue
ORDER BY avg_revenue_per_order DESC;
```

### 📝 Explanation

This analysis first aggregates the total revenue and total number of orders for each product category.

It then calculates:

> **Average Revenue per Order = Total Revenue ÷ Total Orders**

Unlike Average Selling Price (ASP), this metric measures the average value generated whenever a customer places an order within a category.

Finally, the categories are ranked in descending order to identify which categories generate the highest revenue per purchase.

### 💡 Business Recommendation

Categories with a high average revenue per order represent high-value purchases. Flipkart can focus on these categories by:

- Promoting premium products
- Cross-selling complementary products
- Improving inventory availability
- Prioritizing marketing campaigns for high-value customers

---

# Business Question 12 — Product Categories Generating High Revenue but High Return Rates

### 🎯 Objective

Identify product categories that generate above-average revenue while also having above-average return rates.

### 💼 Why does this matter?

A category may generate significant revenue but still reduce overall profitability if customers frequently return its products.

This analysis helps Flipkart:

- Identify high-risk, high-revenue categories
- Investigate product quality issues
- Improve supplier performance
- Reduce return-related operational costs
- Increase overall profitability

### 🧠 Approach

- Calculate the total revenue generated by each product category.
- Calculate the average return rate for each category.
- Compute the average revenue across all categories.
- Compute the average return rate across all categories.
- Return only categories where both revenue and return rate are above their respective averages.

### 💻 SQL

```sql
WITH category_metrics AS (
    SELECT
        p.category,
        SUM(oi.price * oi.quantity) AS total_revenue,
        ROUND(AVG(p.return_rate), 2) AS avg_return_rate
    FROM flipkart.products p
    JOIN flipkart.order_items oi
        ON p.product_id = oi.product_id
    GROUP BY p.category
)

SELECT *
FROM category_metrics
WHERE total_revenue > (
    SELECT AVG(total_revenue)
    FROM category_metrics
)
AND avg_return_rate > (
    SELECT AVG(avg_return_rate)
    FROM category_metrics
)
ORDER BY total_revenue DESC,
         avg_return_rate DESC;
```

### 📝 Explanation

This analysis first calculates two key metrics for every product category:

- **Total Revenue** generated by the category.
- **Average Return Rate** of products within that category.

Instead of simply sorting the categories, the query compares each category against the **overall average** for both metrics.

Only categories satisfying **both** conditions are returned:

- Revenue is greater than the average category revenue.
- Return rate is greater than the average category return rate.

This highlights categories that contribute significantly to sales but may also be reducing profitability because of excessive returns.

### 💡 Business Recommendation

High-revenue categories with high return rates should be prioritized for investigation. Flipkart should:

- Audit suppliers and product quality.
- Analyze customer return reasons.
- Improve product descriptions and images.
- Strengthen quality control before shipment.
- Monitor these categories closely to reduce return-related losses while maintaining revenue.

---

# Business Question 13 — Product Categories with the Highest Profit Potential

### 🎯 Objective

Estimate the profit potential of each product category by combining revenue generation with return rate performance.

> **Note:** Since the dataset does not contain product cost or profit information, a custom **Profit Potential Score** is used as a proxy instead of actual profit.

### 💼 Why does this matter?

High revenue alone does not guarantee profitability. Categories with excessive product returns increase logistics costs, refund expenses, and operational losses.

This analysis helps Flipkart:

- Identify the most profitable-looking categories
- Balance revenue with operational risk
- Prioritize marketing and inventory investments
- Detect categories requiring quality improvements

### 🧠 Approach

- Calculate total revenue for each category.
- Calculate the average return rate for each category.
- Convert the return rate into a decimal.
- Estimate the Profit Potential Score using:

> **Profit Potential Score = Revenue × (1 − Return Rate)**

Higher scores indicate categories with strong revenue generation and relatively lower return rates.

### 💻 SQL

```sql
WITH cte AS (
    SELECT
        p.category,
        ROUND(AVG(p.return_rate), 2) / 100 AS avg_rr,
        SUM(oi.price * oi.quantity) AS total_revenue
    FROM flipkart.products p
    JOIN flipkart.order_items oi
        ON p.product_id = oi.product_id
    GROUP BY p.category
)

SELECT
    *,
    total_revenue * (1 - avg_rr) AS profit_potential_score
FROM cte
ORDER BY profit_potential_score DESC;
```

### 📝 Explanation

Since actual product costs are unavailable, true profit cannot be calculated.

Instead, a proxy metric is created by adjusting revenue based on the average return rate.

For example:

- Category Revenue = ₹10,00,000
- Average Return Rate = 10%

Profit Potential Score:

```
₹10,00,000 × (1 − 0.10) = ₹9,00,000
```

Categories with higher revenue and lower return rates receive a higher score, making them more attractive from a business perspective.

### 💡 Business Recommendation

Flipkart should prioritize categories with the highest Profit Potential Score by:

- Increasing inventory availability
- Allocating larger marketing budgets
- Expanding product assortment
- Monitoring return rates to maintain profitability

> **Future Improvement:** If product cost data becomes available, replace the Profit Potential Score with actual **Profit** or **Profit Margin** calculations for a more accurate financial analysis.

---

# Business Question 14 — Underrated Product Categories

### 🎯 Objective

Identify product categories that generate above-average revenue despite having below-average customer ratings.

### 💼 Why does this matter?

Some product categories continue to generate strong sales even though customers rate them poorly. These categories represent opportunities for Flipkart to significantly improve profitability by addressing customer pain points.

This analysis helps Flipkart:

- Identify hidden quality issues
- Improve customer satisfaction
- Reduce future return rates
- Increase repeat purchases
- Prioritize product quality improvements

### 🧠 Approach

- Calculate the total revenue generated by each category.
- Calculate the average customer rating for each category.
- Compute the overall average revenue across all categories.
- Compute the overall average customer rating across all categories.
- Return only categories where:
  - Revenue is **above** the average category revenue.
  - Customer rating is **below** the average category rating.

### 💻 SQL

```sql
WITH category_metrics AS (
    SELECT
        p.category,
        SUM(oi.price * oi.quantity) AS total_revenue,
        ROUND(AVG(p.average_rating), 2) AS avg_rating
    FROM flipkart.products p
    JOIN flipkart.order_items oi
        ON p.product_id = oi.product_id
    GROUP BY p.category
)

SELECT *
FROM category_metrics
WHERE total_revenue > (
    SELECT AVG(total_revenue)
    FROM category_metrics
)
AND avg_rating < (
    SELECT AVG(avg_rating)
    FROM category_metrics
)
ORDER BY total_revenue DESC;
```

### 📝 Explanation

This analysis compares each product category against the marketplace average.

A category is considered **underrated** if:

- It generates **higher-than-average revenue**, indicating strong customer demand.
- It has a **lower-than-average customer rating**, indicating customer dissatisfaction.

These categories often have significant growth potential because improving customer satisfaction could further increase sales while reducing complaints and returns.

### 💡 Business Recommendation

Flipkart should prioritize these categories for improvement by:

- Investigating product quality issues.
- Improving product descriptions and images.
- Working with suppliers to enhance product quality.
- Monitoring customer reviews to identify recurring complaints.
- Launching quality improvement initiatives before expanding marketing spend.

---

# Business Question 15 — Premium Products with Low Demand

### 🎯 Objective

Identify products that are priced above their category's average price but have below-average sales quantity within the same category.

### 💼 Why does this matter?

Premium-priced products with weak demand may indicate pricing issues, poor marketing, or weak product positioning.

This analysis helps Flipkart:

- Identify overpriced products
- Optimize pricing strategies
- Improve promotional campaigns
- Decide whether products should be repositioned or discontinued

### 🧠 Approach

- Calculate the total quantity sold for each product.
- Compute the average product price within each category.
- Compute the average sales quantity within each category.
- Return products where:
  - Product price is above the category average price.
  - Total quantity sold is below the category average quantity sold.

### 💻 SQL

```sql
WITH total_quantity_product AS (
    SELECT
        p.product_id,
        p.product_type,
        p.category,
        p.price,
        SUM(oi.quantity) AS total_quantity
    FROM flipkart.products p
    JOIN flipkart.order_items oi
        ON p.product_id = oi.product_id
    GROUP BY
        p.product_id,
        p.product_type,
        p.category,
        p.price
),

cte AS (
    SELECT
        product_id,
        product_type,
        category,
        price,
        total_quantity,
        ROUND(AVG(price) OVER (PARTITION BY category), 2) AS avg_price_category,
        ROUND(AVG(total_quantity) OVER (PARTITION BY category), 2) AS avg_quant_category
    FROM total_quantity_product
)

SELECT
    product_id,
    product_type,
    category,
    price,
    total_quantity,
    avg_price_category,
    avg_quant_category
FROM cte
WHERE price > avg_price_category
  AND total_quantity < avg_quant_category;
```

### 📝 Explanation

The analysis is performed in two stages.

First, the total quantity sold is calculated for every product.

Next, window functions are used to calculate:

- The average product price within each category.
- The average sales quantity within each category.

Finally, products are filtered to identify those that are:

- Priced above their category's average price.
- Selling below their category's average sales quantity.

These products may be overpriced, poorly positioned, or insufficiently promoted.

### 💡 Business Recommendation

Flipkart should review these premium products by:

- Evaluating pricing strategies.
- Increasing promotional efforts.
- Bundling products with complementary items.
- Reviewing customer feedback.
- Considering price reductions if demand remains consistently low.

---
# Business Question 16 — Product Portfolio Matrix

## 🎯 Objective

Classify every product into one of four business segments based on its revenue performance and customer rating.

## 💼 Why does this matter?

Not every product should receive the same business strategy.

Some products sell exceptionally well and delight customers, while others generate revenue despite poor customer satisfaction. Some products have excellent ratings but low sales and may simply require better marketing.

This analysis helps Flipkart prioritize investments across its product portfolio.

---

## 🧠 Classification Logic

Each product is compared against two overall benchmarks:

- Average Product Revenue
- Average Product Rating

Products are classified into four categories:

| Revenue | Rating | Classification |
|----------|---------|----------------|
| High | High | ⭐ Star Product |
| High | Low | ⚠️ Improve Quality |
| Low | High | 📈 Growth Opportunity |
| Low | Low | ❌ Underperformer |

---

## 💻 SQL

```sql
WITH product_metrics AS (
    SELECT
        p.product_id,
        p.product_type,
        SUM(oi.price * oi.quantity) AS revenue,
        p.average_rating
    FROM flipkart.products p
    JOIN flipkart.order_items oi
        ON p.product_id = oi.product_id
    GROUP BY
        p.product_id,
        p.product_type,
        p.average_rating
),

benchmark AS (
    SELECT
        *,
        ROUND(AVG(revenue) OVER (), 2) AS avg_revenue,
        ROUND(AVG(average_rating) OVER (), 2) AS avg_product_rating
    FROM product_metrics
)

SELECT
    product_id,
    product_type,
    revenue,
    average_rating,
    CASE
        WHEN revenue > avg_revenue
             AND average_rating > avg_product_rating
            THEN '⭐ Star Product'

        WHEN revenue > avg_revenue
             AND average_rating <= avg_product_rating
            THEN '⚠️ Improve Quality'

        WHEN revenue <= avg_revenue
             AND average_rating > avg_product_rating
            THEN '📈 Growth Opportunity'

        ELSE '❌ Underperformer'
    END AS product_portfolio
FROM benchmark
ORDER BY revenue DESC;
```

---

## 📝 Explanation

The analysis is performed in two stages.

### Step 1

Calculate the revenue generated by each product.

Revenue is calculated as:

> **Revenue = Price × Quantity Sold**

Each row now represents one unique product.

### Step 2

Using window functions, calculate:

- Overall Average Product Revenue
- Overall Average Product Rating

Every product is then compared against these two benchmarks.

Finally, a `CASE` statement assigns every product to one of four business categories.

---

## 💡 Business Recommendations

### ⭐ Star Product

- Increase inventory
- Prioritize marketing campaigns
- Maintain supplier relationships
- Feature prominently on the homepage

---

### ⚠️ Improve Quality

- Investigate customer complaints
- Improve product quality
- Update product descriptions and images
- Reduce return rates

---

### 📈 Growth Opportunity

- Increase marketing exposure
- Recommend through personalized suggestions
- Bundle with popular products
- Improve discoverability

---

### ❌ Underperformer

- Review pricing strategy
- Evaluate supplier performance
- Consider promotional discounts
- Consider discontinuing consistently weak performers

---

## 📊 Business Impact

This Product Portfolio Matrix provides management with a simple framework for prioritizing investments, improving customer satisfaction, and maximizing revenue using data-driven product segmentation.
