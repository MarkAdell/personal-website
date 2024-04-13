---
title: A Practical Example of Using SQL Self Joins
date: 2024-01-03
tags: ["SQL"]
ShowToc: false
---

## Problem

In this article, I am going to share a scenario that I recently faced at work and made use of SQL self joins.

Our company operates a food ordering app. We noticed that some customers add items to their basket, and for one or more hours, do not proceed to create an order. To address this, we decided to send these customers a push notification to encourage them to complete the ordering process. And that was my task. In this article, we will focus on how to retrieve these customers.

## Solution

We have a relational table called `customer_events` where we store various events created by our app customers, such as: `'opened_app'`, `'added_items_to_basket'`, `'created_order'`, `'closed_app'`, and so on.

> While Time-Series Databases could be a more suitable solution for this type of data, given our specific scenario and scale, we decided to stick with the traditional relational database.

Following is a sample of the `customer_events` table:

| id | customer_id | event_type | event_timestamp |
|----|-------------|------------|-----------------|
| 1 | 20 | opened_app | 2023-01-01 11:00:00 |
| 2 | 30 | added_items_to_basket | 2023-01-01 12:00:00 |
| 3 | 30 | closed_app | 2023-01-01 12:10:00 |
| 4 | 40 | added_items_to_basket | 2023-01-01 13:00:00 |
| 5 | 40 | created_order | 2023-01-01 13:30:00 |
| 6 | 50 | added_items_to_basket | 2023-01-01 15:00:00 |
| 7 | 60 | added_items_to_basket | 2023-01-01 16:30:00 |

We have events from 5 customers. Only the `customer_id` of `40` added items to their basket and then created an order. The others did not, so we want to retrieve these customers to send them a push notification, which are `customer_id` of `30`, `50`, and `60`.

Before writing non-straightforward SQL queries, I like to use plain text to describe exactly what I want to achieve, in a way that would help me translate that into the query. Let's see what I would write for our case:

"I want to retrieve customer IDs that their `event_type` = `'added_items_to_basket'` and their `event_timestamp` is one or more hours ago, excluding those for whom there exists a row with the same `customer_id` and `event_name` = `'created_order'` and a greater `event_timestamp`"

Let's start with the easy part of the query, which is selecting all customers who added items to their basket at least one hour ago:

```sql
SELECT
    customer_id
FROM
    customer_events
WHERE
    event_type = 'added_items_to_basket'
    AND (EXTRACT(EPOCH FROM (NOW() - event_timestamp)) / 3600) >= 1;
```

Now, let's update the query to exclude those customers who created an order after they added items to the basket. We can do this using self joins:

```sql
SELECT
    a.customer_id
FROM
    customer_events a
LEFT JOIN
    customer_events b
    ON a.customer_id = b.customer_id
    AND b.event_type = 'created_order'
    AND b.event_timestamp > a.event_timestamp
WHERE
    a.event_type = 'added_items_to_basket'
    AND (EXTRACT(EPOCH FROM (NOW() - a.event_timestamp)) / 3600) >= 1
    AND b.customer_id IS NULL;
```

To the untrained eye, this query might look intimidating, but it's actually quite simple. Let's break it down.

We performed a `LEFT JOIN` on the `customer_events` table (a) with itself (b). This retrieves all rows from the left table (a) along with their matching rows from the right table (b) based on the specified join conditions. In the right side, all columns will be `NULL` if there are no subsequent `'created_order'` event for the customer on the left side, and will have values otherwise. We use the `b.customer_id IS NULL` condition to only get customers where there are no subsequent `'created_order'` event.

Query Output:

| customer_id |
|-------------|
| 30 |
| 50 |
| 60 |

If you are not sure what the output of the self join looks like before filtering using `b.customer_id IS NULL`, you may run [this](https://www.db-fiddle.com/f/wtS6yTrzRXp9dbfgfRBQcn/1) example I prepared on [db-fiddle](https://www.db-fiddle.com/).

It is worth noting that this is a simplified version of the actual query. There are other cases that we need to handle, such as ensuring that the order belongs to the specified basket, or only taking the last event into account. However, this is beyond the scope of this article.

Now, I want you to go back and read the plain text we wrote for this query, and then check the query again. You will find that it does exactly what we described.

## Another Approach

There is an alternative way we can write this query to achieve the same result, using sub-queries:

```sql
SELECT
    a.customer_id
FROM
    customer_events a
WHERE
    a.event_type = 'added_items_to_basket'
    AND (EXTRACT(EPOCH FROM (NOW() - a.event_timestamp)) / 3600) >= 1
    AND NOT EXISTS (
        SELECT 1
        FROM customer_events b
        WHERE a.customer_id = b.customer_id
        AND b.event_type = 'created_order'
        AND b.event_timestamp > a.event_timestamp
    );
```

Which approach to use? We have two factors to consider: readability and performance.

In terms of readability, it is a personal preference. For me, I find both equally readable.

In terms of performance, it depends on many factors. It is always advisable to check the execution plans of your queries.

# Conclusion

Whenever you find yourself in a situation where you want to exclude certain rows from your result based on conditions that involve other rows in the same table, SQL self joins can come in handy.

I hope you found this article helpful. Please let me know if you have any feedback. Happy querying!
