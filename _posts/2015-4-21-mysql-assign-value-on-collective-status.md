---
layout: post
title: Migration Tips - MySQL - Assign Value on Collective Status of All
---

Background
==========

- Order has 4 line_items
  - 3 are shippable line_items
  - 1 is service item so no shipping needed
- Logically, Order has 2 Packages for Processing Purposes
  - first, package has 3 shippable line_items
  - second, package has 1 item whose processing is complete as nothing
    more to be done
- ShippablePackage will be having shipping_status to track how far we
  are we have reached with processing
  - Collected All 3 items
  - Wrapped in Package to Ship
  - Picked by Driver to deliver to Customer
  - Customer Accepted / Rejected
- Shippable Package has status which depends on line_items status
  - Shippable Package need to be processed till it has atleast one
    non-cancelled item
  - Shippable Package will be considered Processed
    - if it is Delivered
    - if all items are cancelled, nothing to process
- Typically, Shippable Package 
  - has status (Released / Cancelled)
  - has shipping_status (Collected / Wrapped / Picked / Delivered)

Problem Statement
=================

Migration on existing orders need to be done

- [1] Shippable and NonShippable Package should be created in cancelled state, if all its items are
  cancelled else released
- Copy the shipping status from orders to Shippable Package
- NonShippable Package would not have any shipping_status

Constraint
==========

Migration to be done using MySQL INSERT / UPDATE statements

Approach
========

Lets start with Shippable Package Creation / Migration First

- First, we need to find shippable line_items

```sql
SELECT *
FROM line_items
WHERE line_items.shippable = 1 
```

- Now, we need to ensure that only *ONE* shippable package is created
  for one order with multiple shippable line_items
 
```sql
SELECT *
FROM line_items
WHERE line_items.shippable = 1 
GROUP by line_items.order_id
```

- Now, INSERT Query is simple to create a Shippable Package

```sql
INSERT INTO packages(`package_number`, `status`, `shipping_status`) (
  SELECT orders.order_number, <what to put here - refer [1]>,
orders.shipping_status 
  FROM line_items
  INNER JOIN orders ON orders.id = line_items.order_id
  WHERE line_items.shippable = 1 
  GROUP by line_items.order_id
)
```

- `status` is tricky as we will put `cancelled` only when all line_items
  belonging to package are cancelled else released
  - We have at our advantage that `line_items.status` cancelled status
    is stored as integer 2 in DB and 0 when bought
  - You must be wondering how this solves or helps confirm the fact that
    all line_items are cancelled
  - Answer is Simple Math .. What !!
  - Here is how,
    - Suppose you have 2 line_items
    - Both are cancelled, implying value 2 in status column
    - Now, if you do a SUM(status) = 2 + 2 = 4
    - Also, we can count the rows COUNT(*) and multiplying it by
      cancelled value i.e 2 we get 2 * 2 = 4
    - SUM(status) = COUNT(*) * CANCELLED_INTEGER_VALUE
    - When above calculates to true, it implies that Infact all
      line_items are cancelled else not 
    - BINGO !!!

```sql
INSERT INTO packages(`package_number`, `status`, `shipping_status`) (
  SELECT orders.order_number, IF(SUM(line_items.status) = COUNT(*) * 2,
"cancelled", "released"), orders.shipping_status 
  FROM line_items
  INNER JOIN orders ON orders.id = line_items.order_id
  WHERE line_items.shippable = 1 
  GROUP by line_items.order_id
)
```

SIMPLE MATH help solved our problem easily and execute it using MySQL
which looked like a problem to be solved in some high level language
like Ruby, Java etc. having power of loops and handling every item
one-by-one.

Non-Shippable Package Migration will use the same trick with one
exception that shipping_status will be `none` as nothing to ship.

*Always enter the territory you want to conquer, reign will be yours :)*
