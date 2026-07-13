# Business Question 1 — Top 10 Customers by Lifetime Revenue

### 🎯 Objective
Identify the Top 10 customers generating the highest lifetime revenue.

### 💼 Why does this matter?
High-value customers drive a significant share of revenue. Identifying them helps improve retention, loyalty programs, and personalized marketing.

### 🧠 Approach
- Join `customers` and `orders`
- Aggregate revenue using `SUM(amount)`
- Group by customer
- Sort by revenue
- Return Top 10

### 💻 SQL

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS full_name,
    SUM(o.amount) AS lifetime_revenue
FROM `flipkart.customers` c
JOIN `flipkart.orders` o
ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.first_name,
    c.last_name
ORDER BY lifetime_revenue DESC
LIMIT 10;
```

### 💡 Business Recommendation

Create loyalty programs and personalized offers for the highest-value customers to maximize retention and long-term revenue.

---

# Business Question 2 — Top 10 Customers by Number of Orders

### 🎯 Objective
Identify the Top 10 customers who have placed the highest number of orders.

### 💼 Why does this matter?
Customers who place orders frequently are generally more engaged and loyal. Identifying these customers helps Flipkart improve customer retention, increase repeat purchases, and build long-term relationships.

### 🧠 Approach
- Join `customers` and `orders`
- Count the total number of orders placed by each customer using `COUNT(*)`
- Group by customer
- Sort by the number of orders in descending order
- Return the Top 10 customers

### 💻 SQL

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS full_name,
    COUNT(*) AS no_of_orders
FROM `flipkart.customers` c
JOIN `flipkart.orders` o
ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.first_name,
    c.last_name
ORDER BY no_of_orders DESC
LIMIT 10;
```

### 💡 Business Recommendation

Reward highly engaged customers through loyalty programs, exclusive offers, early access to sales, and personalized product recommendations to encourage continued repeat purchases.

---

# Business Question 3 — Top 10 Customers by Average Order Value (AOV)

### 🎯 Objective
Calculate the Average Order Value (AOV) for each customer and identify the Top 10 customers with the highest Average Order Value.

### 💼 Why does this matter?
Average Order Value (AOV) measures how much a customer spends on average per purchase. Customers with a high AOV tend to purchase premium products and generate higher revenue per transaction, making them valuable targets for personalized marketing and premium loyalty programs.

### 🧠 Approach
- Join `customers` and `orders` using `customer_id`
- Calculate the average order amount for each customer using `AVG(amount)`
- Round the AOV to two decimal places using `ROUND()`
- Group the results by customer
- Sort customers by AOV in descending order
- Return the Top 10 customers

### 💻 SQL

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS fullname,
    ROUND(AVG(o.amount), 2) AS AOV
FROM `flipkart.customers` c
JOIN `flipkart.orders` o
ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.first_name,
    c.last_name
ORDER BY AOV DESC
LIMIT 10;
```

### 💡 Business Recommendation

Customers with a high Average Order Value should be targeted through premium membership programs, personalized product recommendations, exclusive product launches, and high-value promotional campaigns. Increasing retention among these customers can significantly improve overall profitability.

---

# Business Question 4 — Customer Scorecard

### 🎯 Objective

Create a customer scorecard displaying each customer's Lifetime Revenue, Number of Orders, and Average Order Value (AOV), and identify the Top 10 customers by lifetime revenue.

### 💼 Why does this matter?

A single metric does not provide a complete picture of customer value. Combining revenue, purchase frequency, and average order value enables Flipkart to identify its most valuable customers and make informed decisions around loyalty programs, customer retention, and personalized marketing.

### 🧠 Approach

- Join `customers` and `orders` using `customer_id`
- Calculate Lifetime Revenue using `SUM(amount)`
- Calculate Number of Orders using `COUNT(*)`
- Calculate Average Order Value using `AVG(amount)`
- Round AOV to two decimal places
- Group the results by customer
- Sort customers by Lifetime Revenue
- Return the Top 10 customers

### 💻 SQL

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    SUM(o.amount) AS lifetime_revenue,
    COUNT(*) AS number_of_orders,
    ROUND(AVG(o.amount), 2) AS average_order_value
FROM `flipkart.customers` c
JOIN `flipkart.orders` o
    ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.first_name,
    c.last_name
ORDER BY lifetime_revenue DESC
LIMIT 10;
```

### 💡 Business Recommendation

Use this customer scorecard to identify Flipkart's highest-value customers and prioritize them for premium loyalty programs, personalized offers, exclusive product launches, and retention campaigns. Rather than relying on a single metric, this scorecard provides a holistic view of customer value, enabling better business decisions.

---

# Business Question 5 — Customer Segmentation Based on Lifetime Revenue

### 🎯 Objective
Segment customers into High Value, Medium Value, and Low Value categories based on their Lifetime Revenue.

### 💼 Why does this matter?
Not all customers contribute equally to business growth. Customer segmentation enables Flipkart to design targeted marketing campaigns, optimize loyalty programs, allocate resources effectively, and improve customer retention by treating customers according to the value they generate.

### 🧠 Approach
- Join `customers` and `orders` using `customer_id`
- Calculate the Lifetime Revenue for each customer using `SUM(amount)`
- Use a `CASE` statement to classify customers into different value segments
- Group the results by customer
- Sort customers by Lifetime Revenue

### 💻 SQL

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    SUM(o.amount) AS lifetime_revenue,
    CASE
        WHEN SUM(o.amount) > 50000 THEN 'High Value'
        WHEN SUM(o.amount) BETWEEN 20000 AND 50000 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS customer_segment
FROM `flipkart.customers` c
JOIN `flipkart.orders` o
    ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.first_name,
    c.last_name
ORDER BY lifetime_revenue DESC;
```

### 💡 Business Recommendation

Customer segmentation should be the foundation of Flipkart's CRM strategy. High-value customers can be rewarded with premium loyalty benefits, medium-value customers can be encouraged to increase their spending through personalized promotions, and low-value customers can be targeted with re-engagement campaigns to improve retention and purchase frequency.

---

# Business Question 6 — Customers Spending Above Average

### 🎯 Objective
Identify customers whose Lifetime Revenue is greater than the average Lifetime Revenue of all customers.

### 💼 Why does this matter?
Rather than using fixed revenue thresholds, comparing customers against the overall average provides a dynamic benchmark. This helps Flipkart identify above-average customers who are ideal candidates for premium services, loyalty programs, and targeted marketing campaigns.

### 🧠 Approach
- Calculate the Lifetime Revenue for each customer.
- Store the aggregated results in a Common Table Expression (CTE).
- Calculate the average Lifetime Revenue from the CTE.
- Return only customers whose Lifetime Revenue is greater than the overall average.

### 💻 SQL

```sql
WITH customer_revenue AS (
    SELECT
        c.customer_id,
        CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
        SUM(o.amount) AS lifetime_revenue
    FROM `flipkart.customers` c
    JOIN `flipkart.orders` o
        ON c.customer_id = o.customer_id
    GROUP BY
        c.customer_id,
        c.first_name,
        c.last_name
)

SELECT *
FROM customer_revenue
WHERE lifetime_revenue >
(
    SELECT AVG(lifetime_revenue)
    FROM customer_revenue
)
ORDER BY lifetime_revenue DESC;
```

### 💡 Business Recommendation

Customers spending above the average Lifetime Revenue should be prioritized for premium loyalty programs, exclusive offers, and personalized marketing campaigns. Since the benchmark is calculated dynamically from customer data, this approach automatically adapts as customer spending patterns evolve.

---

# Business Question 7 — Rank Customers by Lifetime Revenue

### 🎯 Objective
Rank all customers based on their Lifetime Revenue from highest to lowest.

### 💼 Why does this matter?
Ranking customers provides a complete view of customer performance instead of just identifying the Top 10. It helps Flipkart recognize VIP customers, design tiered loyalty programs, and prioritize high-value customers for retention and personalized marketing initiatives.

### 🧠 Approach
- Join `customers` and `orders` using `customer_id`
- Calculate Lifetime Revenue for each customer using `SUM(amount)`
- Store the aggregated results in a Common Table Expression (CTE)
- Use the `RANK()` window function to assign a rank based on Lifetime Revenue in descending order
- Display each customer's rank alongside their revenue

### 💻 SQL

```sql
WITH customer_revenue AS (
    SELECT
        c.customer_id,
        CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
        SUM(o.amount) AS lifetime_revenue
    FROM `flipkart.customers` c
    JOIN `flipkart.orders` o
        ON c.customer_id = o.customer_id
    GROUP BY
        c.customer_id,
        c.first_name,
        c.last_name
)

SELECT *,
       RANK() OVER (ORDER BY lifetime_revenue DESC) AS customer_rank
FROM customer_revenue;
```

### 💡 Business Recommendation

Customer rankings can be used to create tiered loyalty programs such as Platinum, Gold, and Silver memberships. High-ranking customers should receive exclusive benefits, personalized recommendations, and premium customer support to maximize retention and long-term profitability.

---

