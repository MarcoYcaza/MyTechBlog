---
title: "SQL Window Functions Explained (Sample)"
date: 2026-10-14
description: "Compare each row with its group without collapsing the result."
tags: [SQL, Window Functions, Analytics]
categories: [Programming]
draft: false
---

> Sample article: replace or remove this post when you publish your own work.

Window functions calculate a value over a set of related rows while keeping each input row. Unlike `GROUP BY`, they do not collapse the result into one row per group.

## Rank orders within each customer

```sql
SELECT customer_id, order_id, amount,
       ROW_NUMBER() OVER (
         PARTITION BY customer_id
         ORDER BY amount DESC, order_id
       ) AS order_rank
FROM orders;
```

For one customer, this might produce:

| customer_id | order_id | amount | order_rank |
| --- | --- | ---: | ---: |
| 42 | 103 | 120 | 1 |
| 42 | 101 | 80 | 2 |
| 42 | 102 | 80 | 3 |

The extra `order_id` makes ties deterministic. Use `RANK()` or `DENSE_RANK()` when tied amounts should share a rank, and omit the tie-breaker from the window ordering.

## Compute a running total

```sql
SELECT customer_id, order_id, amount,
       SUM(amount) OVER (
         PARTITION BY customer_id ORDER BY order_id
         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM orders;
```

The explicit `ROWS` frame makes the running total's boundaries clear. SQL dialects can differ in default frames and supported syntax.
