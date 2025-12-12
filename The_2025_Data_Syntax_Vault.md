# The 2025 Data Syntax Vault
### SQL + Regex + Pandas + Python – 150+ Snippets You Actually Use Daily
Instant copy-paste. No fluff. Last updated December 2025.

Pay-what-you-want ($7+). Thank you for supporting indie makers ❤️

---

## 1. SQL – The 50 Queries Every Analyst Uses

```sql
-- 1. Basic SELECT + filtering
SELECT * FROM users WHERE created_at >= '2025-01-01';

-- 2. INNER JOIN
SELECT u.name, o.amount 
FROM users u 
INNER JOIN orders o ON u.id = o.user_id;

-- 3. LEFT JOIN + IS NULL (find users with no orders)
SELECT u.name 
FROM users u 
LEFT JOIN orders o ON u.id = o.user_id 
WHERE o.user_id IS NULL;

-- 4. Multiple JOINs
SELECT c.name, p.title, o.amount 
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN products p ON o.product_id = p.id;

-- 5. GROUP BY + HAVING
SELECT DATE(created_at) AS day, COUNT(*) AS signups
FROM users 
WHERE created_at >= '2025-01-01'
GROUP BY DATE(created_at)
HAVING COUNT(*) > 100;

-- 6. Window: ROW_NUMBER()
SELECT *, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) AS rn
FROM events;

-- 7. Window: RANK() vs DENSE_RANK()
SELECT user_id, score,
       RANK() OVER (ORDER BY score DESC) AS rank,
       DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM scores;

-- 8. Running total
SELECT date, revenue,
       SUM(revenue) OVER (ORDER BY date ROWS UNBOUNDED PRECEDING) AS running_total
FROM daily_revenue;

-- 9. LAG / LEAD (previous/next value)
SELECT date, revenue,
       LAG(revenue) OVER (ORDER BY date) AS prev_day
FROM daily_revenue;

-- 10. Date formatting & extraction
SELECT 
  DATE_TRUNC('month', created_at) AS month,
  EXTRACT(YEAR FROM created_at) AS year,
  TO_CHAR(created_at, 'YYYY-MM-DD') AS iso_date
FROM users;

-- 11. Pivot (PostgreSQL)
SELECT * FROM crosstab(
  'SELECT country, product, SUM(amount) FROM sales GROUP BY 1,2 ORDER BY 1,2',
  $$  VALUES ('Shoes'), ('Hats'), ('Jackets')  $$
) AS ct(country text, shoes numeric, hats numeric, jackets numeric);

-- 12. CTE + recursion (explode dates)
WITH RECURSIVE dates AS (
  SELECT '2025-01-01'::date AS day
  UNION ALL
  SELECT day + 1 FROM dates WHERE day < '2025-12-31'
) SELECT * FROM dates;

-- 13. UPSERT (PostgreSQL)
INSERT INTO users (id, email, name)
VALUES (123, 'new@email.com', 'John')
ON CONFLICT (id) DO UPDATE SET email = EXCLUDED.email;

-- 14. JSON extraction (PostgreSQL)
SELECT data->>'name' AS name, data->'address'->>'city' AS city
FROM users_json;

-- 15–50. (Full version has 35 more common patterns – pivots, cohort queries, percentile, etc.)


Email (strict)          ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$
Email (lax)             \S+@\S+\.\S+
US Phone                ^$$ ?(\d{3}) $$?[- ]?(\d{3})[- ]?(\d{4})$
International Phone     ^\+?[\d\s\-$$  $$]{10,20}$
URL (with http(s))      https?://[^\s]+
URL (no protocol)       (www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_\+.~#?&//=]*)
IPv4                    \b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b
IPv6                    ([0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}
Date YYYY-MM-DD         \b\d{4}-\d{2}-\d{2}\b
Date MM/DD/YYYY         \b\d{1,2}/\d{1,2}/\d{4}\b
Time 24h                \b([01]?[0-9]|2[0-3]):[0-5][0-9]\b
Credit Card (spaces)    \b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b
US ZIP Code             \b\d{5}(-\d{4})?\b
Password (8+ chars, letter + number)  ^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d@$$ !%*#?&]{8,} $$
Hex Color               ^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$
UUID                    [0-9a-f]{8}-[0-9a-f]{4}-[4][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}
Bitcoin Address         [13][a-km-zA-HJ-NP-Z1-9]{25,34}
Extract numbers only    \d+
Remove all non-digits   \D+
Find duplicated words   \b(\w+)\s+\1\b
(Full version has 60 more – logs, JWT, HTML tags, SQL injection patterns, etc.)


# Read & write
df = pd.read_csv('file.csv', parse_dates=['date'])
df.to_parquet('fast.parquet', index=False)

# Quick look
df.head(10); df.info(); df.describe()
df.isnull().sum().sort_values(ascending=False)

# Filter
df[df['age'] > 30]
df.query('age > 30 and country == "US"')

# Multiple conditions
df[df['col'].isin(['A', 'B', 'C'])]

# Drop duplicates
df.drop_duplicates(subset=['email'], keep='first')

# Fill missing
df['age'].fillna(df['age'].median(), inplace=True)
df.fillna({'age': 0, 'name': 'Unknown'})

# Group + agg multiple
df.groupby('country').agg({'revenue': 'sum', 'user_id': 'nunique'})

# Pivot
df.pivot_table(values='revenue', index='country', columns='product', aggfunc='sum', fill_value=0)

# Melt (unpivot)
df.melt(id_vars=['date'], value_vars=['sales_us', 'sales_eu'])

# Date magic
df['month'] = df['date'].dt.to_period('M')
df['dow'] = df['date'].dt.day_name()

# Rolling
df['revenue_7d'] = df['revenue'].rolling(7).mean()

# Resample time series
df.set_index('date').resample('W').sum()

# Top N per group
df[df['rank'] = df.groupby('country')['revenue'].rank(method='first', ascending=False) <= 10]

# (Full version has 35 more – merge tricks, memory optimization, apply vs vectorized, etc.)


# Strip whitespace everywhere
df = df.apply(lambda x: x.str.strip() if x.dtype == "object" else x)

# Lowercase all string columns
df[df.select_dtypes(include='object').columns] = df.select_dtypes(include='object').apply(lambda x: x.str.lower())

# Replace multiple values at once
df['status'].replace({'yes': True, 'no': False, 'Y': True, 'N': False}, inplace=True)

# Fast category conversion (saves 90% memory)
for col in ['country', 'product', 'source']:
    df[col] = df[col].astype('category')

# One-line dedupe + keep last
df = df.sort_values('timestamp').drop_duplicates('user_id', keep='last')

# Explode list column
df.explode('tags')

# Memory usage before/after
df.memory_usage(deep=True).sum() / 1e9  # GB