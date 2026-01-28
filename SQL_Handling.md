SQL Query Handling Notes with Examples

1. Using CTEs (Common Table Expressions)

CTEs allow you to build intermediate result sets that can be reused in the main query.

Example: Find clients whose total claims exceed 50% of their income

WITH claim_totals AS (
SELECT client_id, SUM(claim_amt) AS total_claims
FROM claim
GROUP BY client_id
)
SELECT cl.id, cl.first_name, cl.last_name, cl.income, ct.total_claims
FROM client cl
JOIN claim_totals ct ON cl.id = ct.client_id
WHERE ct.total_claims > 0.5 \* cl.income;

2. Window Functions with QUALIFY

Window functions let you rank or order rows within partitions. QUALIFY filters based on those results.

Example: Highest claim per car

SELECT id, car_id, claim_amt,
RANK() OVER (PARTITION BY car_id ORDER BY claim_amt DESC) AS rank
FROM claim
QUALIFY rank = 1;

3. LEFT JOIN vs RIGHT JOIN vs FULL OUTER JOIN

LEFT JOIN: Keep all rows from the left table, even if no match.

RIGHT JOIN: Keep all rows from the right table, even if no match.

FULL OUTER JOIN: Keep all rows from both tables, matched or not.

Example: Clients and Claims

-- LEFT JOIN: All clients, claims if they exist
SELECT cl.first_name, cl.last_name, clm.claim_amt
FROM client cl
LEFT JOIN claim clm ON cl.id = clm.client_id;

-- RIGHT JOIN: All claims, clients if they exist
SELECT cl.first_name, cl.last_name, clm.claim_amt
FROM client cl
RIGHT JOIN claim clm ON cl.id = clm.client_id;

-- FULL OUTER JOIN: All clients and all claims
SELECT cl.first_name, cl.last_name, clm.claim_amt
FROM client cl
FULL OUTER JOIN claim clm ON cl.id = clm.client_id;

4. Aggregations and Summaries

Aggregations help profile data.

Example: Average claim amount per car type

SELECT ca.car_type, AVG(clm.claim_amt) AS avg_claim
FROM car ca
JOIN claim clm ON ca.id = clm.car_id
GROUP BY ca.car_type;

Example: Clients with no claims

SELECT cl.first_name, cl.last_name
FROM client cl
LEFT JOIN claim clm ON cl.id = clm.client_id
WHERE clm.id IS NULL;

5. Risk Analysis Queries

Example: Identify heavy hitters (claims much larger than average)

SELECT car_id, claim_amt
FROM claim
WHERE claim_amt > 10 \* (SELECT AVG(claim_amt) FROM claim);

Interpretation: A claim 10× higher than average indicates tail risk and potential catastrophic exposure.

6. Practical Tips

Use CTEs for clarity when building multi-step queries.

Use QUALIFY (DuckDB, BigQuery, Snowflake) to filter window functions directly.

Use LEFT JOIN when your analysis is client-centric, RIGHT JOIN when claim-centric, and FULL OUTER JOIN for audits.

Always check for outliers in insurance data — they drive risk management decisions.

7. Additional Advanced SQL Techniques and Best Practices

7.1 Window Functions for Running Totals and Moving Averages

Window functions can calculate running totals, moving averages, and other cumulative metrics.

Example: Running total of claims per client

SELECT client_id, claim_date, claim_amt,
SUM(claim_amt) OVER (PARTITION BY client_id ORDER BY claim_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM claim;

7.2 Recursive Queries

Recursive CTEs allow querying hierarchical or tree-structured data.

Example: Retrieve all subordinates of a manager

WITH RECURSIVE subordinates AS (
  SELECT employee_id, manager_id, name 
  FROM employees 
  WHERE manager_id IS NULL -- top-level manager
  UNION ALL 
  SELECT e.employee_id, e.manager_id, e.name 
  FROM employees e 
  INNER JOIN subordinates s ON e.manager_id = s.employee_id
) 
SELECT * FROM subordinates;

7.3 Using CASE Statements for Conditional Logic

CASE statements enable conditional transformations within queries.

Example: Categorize claims by amount

SELECT claim_id, claim_amt,
CASE
WHEN claim_amt < 1000 THEN 'Low'
WHEN claim_amt BETWEEN 1000 AND 5000 THEN 'Medium'
ELSE 'High'
END AS claim_category
FROM claim;

7.4 Query Optimization Tips

Use indexes on columns frequently used in WHERE, JOIN, and ORDER BY clauses.

Avoid SELECT \*; specify only needed columns.

Use EXPLAIN plans to analyze query performance.

Filter early to reduce dataset size.

Use LIMIT to restrict result sets when appropriate.

7.5 Stored Procedures and Automation

Stored procedures encapsulate reusable SQL logic for automation and consistency.

Example: Stored procedure to update claim status

CREATE PROCEDURE UpdateClaimStatus()
BEGIN
UPDATE claim
SET status = 'Reviewed'
WHERE claim_date < CURRENT_DATE - INTERVAL '30' DAY;
END;

7.6 Security Best Practices

Use parameterized queries to prevent SQL injection.

Limit user permissions to only necessary operations.

Encrypt sensitive data at rest and in transit.

8. Additional Enhancements and Domain-Specific Examples

8.1 Fraud Detection Queries

Example: Detect claims with suspiciously high frequency per client

SELECT client*id, COUNT(*) AS claim*count
FROM claim
WHERE claim_date > CURRENT_DATE - INTERVAL '30' DAY
GROUP BY client_id
HAVING COUNT(*) > 5;

8.2 Claim Lifecycle Tracking

Example: Track claims by status over time

SELECT claim_status, COUNT(\*) AS count, DATE_TRUNC('month', claim_date) AS month
FROM claim
GROUP BY claim_status, month
ORDER BY month;

8.3 Visualization and Dashboard Query Patterns

Example: Monthly claim amounts for dashboard visualization

SELECT DATE_TRUNC('month', claim_date) AS month, SUM(claim_amt) AS total_claims
FROM claim
GROUP BY month
ORDER BY month;

8.4 Common Pitfalls and Debugging Tips

Check for NULL values in JOIN keys.

Verify data types match in JOIN conditions.

Use LIMIT to test queries on smaller datasets.

Use descriptive aliases for readability.

8.5 Integration with BI Tools and Data Pipelines

Use parameterized queries for dynamic dashboards.

Schedule query refreshes during off-peak hours.

Cache results for frequently accessed reports.

8.6 Performance Benchmarking Queries

Example: Compare query execution times

EXPLAIN ANALYZE
SELECT \* FROM claim WHERE claim_amt > 10000;

This page now includes a comprehensive set of SQL query handling techniques, best practices, domain-specific examples, and practical tips for performance, security, and integration.
