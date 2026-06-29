## Business Problem

A growing retail company in Rwanda wants to analyse its sales transactions to understand:
- Which customers generate the most revenue
- Which product categories perform best
- How revenue trends change month over month
- How to segment customers for targeted marketing campaigns

This database provides the analytical foundation to answer those questions using advanced SQL — specifically **Common Table Expressions (CTEs)** and **Window Functions**.

---

## Database Schema

The system uses **three related tables**:

| Table | Description |
|-------|-------------|
| `customers` | Customer profiles (name, email, city, country) |
| `products` | Product catalogue (name, category, price, stock) |
| `sales` | Transaction records linking customers to products |

### Relationships

- `sales.customer_id` → `customers.customer_id` (FK)
- `sales.product_id` → `products.product_id` (FK)

---

## ER Diagram

```
┌──────────────────┐        ┌─────────────────────────────────────────┐
│   customers      │        │   sales                                 │
│──────────────────│        │─────────────────────────────────────────│
│ PK customer_id   │◄──┐    │ PK sale_id                              │
│    full_name     │   └────│ FK customer_id                          │
│    email         │        │ FK product_id ──────────────────────┐   │
│    city          │        │    sale_date                        │   │
│    country       │        │    quantity                         │   │
│    created_at    │        │    unit_price                       │   │
└──────────────────┘        │    discount_pct                     │   │
                            │    total_amount                     │   │
                            └─────────────────────────────────────┘   │
                                                                       │
┌──────────────────┐                                                   │
│   products       │◄──────────────────────────────────────────────────┘
│──────────────────│
│ PK product_id    │
│    product_name  │
│    category      │
│    unit_price    │
│    stock_qty     │
└──────────────────┘
```

---

## SQL Files

| File | Contents |
|------|----------|
| `01_schema_and_data.sql` | CREATE TABLE statements + INSERT sample data |
| `02_part_A_CTEs.sql` | All 5 CTE implementations |
| `03_part_B_window_functions.sql` | All window function queries |

---

## Part A: CTE Implementations

### CTE 1 — Simple CTE: Customer Revenue Tiers

**Purpose:** Calculate total spending per customer and classify them as Bronze / Silver / Gold / Platinum.

```sql
WITH customer_revenue AS (
    SELECT
        s.customer_id,
        c.full_name,
        c.city,
        SUM(s.total_amount) AS total_spent
    FROM sales s
    JOIN customers c ON c.customer_id = s.customer_id
    GROUP BY s.customer_id, c.full_name, c.city
)
SELECT
    customer_id, full_name, city, total_spent,
    CASE
        WHEN total_spent >= 1000000 THEN 'Platinum'
        WHEN total_spent >=  500000 THEN 'Gold'
        WHEN total_spent >=  100000 THEN 'Silver'
        ELSE                             'Bronze'
    END AS customer_tier
FROM customer_revenue
ORDER BY total_spent DESC;
```

**Business Value:** Sales reps can quickly prioritise high-value accounts for relationship management.

---

### CTE 2 — Multiple CTEs: Category Revenue Share

**Purpose:** Use two CTEs — one for per-category revenue, one for the grand total — to compute each category's percentage share.

**Business Value:** Informs buying and marketing decisions by showing which product lines drive the most income.

---

### CTE 3 — Recursive CTE: Monthly Sales Calendar

**Purpose:** Recursively generate every month from January to June 2024, then LEFT JOIN sales to ensure months with zero sales still appear.

**Business Value:** Prevents gaps in reports; highlights slow periods that may need promotional activity.

---

### CTE 4 — CTE with Aggregation: Best Product per Category

**Purpose:** Aggregate revenue per product, then use RANK() inside a CTE to pick the number-one product in each category.

**Business Value:** Guides inventory restocking — never run out of your category's top seller.

---

### CTE 5 — CTE with JOIN: High-Value Repeat Buyers

**Purpose:** Identify customers who have ordered more than once AND spent above the average customer total.

**Business Value:** These customers are the best candidates for loyalty rewards, referral programmes, or premium account status.

---

## Part B: Window Function Implementations

### Section 1 — Ranking Functions

| Function | Query Goal |
|----------|-----------|
| `ROW_NUMBER()` | Unique sequential number for every sale (for pagination / audit) |
| `RANK()` | Customer spending rank; ties share a rank, gap follows |
| `DENSE_RANK()` | Same but no gaps, so "Top 3" always returns exactly 3 |
| `PERCENT_RANK()` | Percentile position of each sale amount in the dataset |

---

### Section 2 — Aggregate Window Functions

| Function | Query Goal |
|----------|-----------|
| `SUM() OVER()` | Running total of all revenue ordered by date |
| `AVG() OVER()` | Each sale compared to its product-category average |
| `MIN() / MAX() OVER()` | Customer's cheapest and most expensive purchase alongside each row |
| `MAX() OVER()` | Best revenue day within each calendar month |

---

### Section 3 — Navigation Functions

| Function | Query Goal |
|----------|-----------|
| `LAG()` | Month-over-month revenue growth percentage |
| `LEAD()` | Next scheduled purchase date and amount per customer |

---

### Section 4 — Distribution Functions

| Function | Query Goal |
|----------|-----------|
| `NTILE(4)` | Divide customers into revenue quartiles (Top 25%, Upper-Mid, etc.) |
| `CUME_DIST()` | Cumulative distribution percentage of individual sale amounts |

---

## Analysis and Findings

### Descriptive Analysis — What happened?

- **30 sales transactions** were recorded from November 2023 to June 2024.
- **Electronics** was the highest-revenue category, driven by Laptop Pro 15 sales.
- Customer **#11 (Leo Mugisha)** generated the single highest transaction (2 × Laptop Pro 15 = RWF 1,530,000).
- Monthly revenue was highest in **March 2024** and dipped in January 2024.

### Diagnostic Analysis — Why did it happen?

- Electronics dominates because laptops and printers carry high unit prices, not high volumes.
- The January dip likely reflects post-holiday spending contraction — a common seasonal pattern.
- Repeat buyers (Alice, Bob, Clara, David, Eva) account for a disproportionate share of revenue, confirming that retaining existing customers is more valuable than acquiring new ones.

### Prescriptive Analysis — What actions should be taken?

1. **Loyalty Programme:** Enrol Platinum/Gold customers (NTILE quartile 1) in a rewards scheme to increase order frequency.
2. **January Promotion:** Run a January sale on high-margin categories (Electronics) to counter the seasonal dip.
3. **Category Expansion:** Stationery has the most transactions but lowest revenue — consider bundling it with Electronics accessories to raise average order value.
4. **Stock Priority:** Maintain at least 30 units of Laptop Pro 15 at all times; it is both the top revenue driver and a high-reorder item.

---

## References

- MySQL 8.0 Documentation — Window Functions: https://dev.mysql.com/doc/refman/8.0/en/window-functions.html
- MySQL 8.0 Documentation — WITH (Common Table Expressions): https://dev.mysql.com/doc/refman/8.0/en/with.html
- Course slides: C11665 — DPR400210 Database Programming (June 2026)

---

## Academic Integrity Statement

I confirm that this submission is entirely my own original work. I have not copied code from classmates, online repositories, or any other source without proper attribution. I understand that academic misconduct will be handled according to UNILAK university regulations.
