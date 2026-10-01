# Product Analyst Intern Assignment

## Task 1 - What doesn't match?

I compared the API documentation with the three API responses.

### 1\. status has an undocumented value

- **Docs:** status can be pending, shipped, delivered, or cancelled.
- **Actual:** orders_page1.json -> ord_1003.status is refunded.
- **Impact:** Yes. A system built using the documented list may not know how to handle this order.

### 2\. customer.email can be null

- **Docs:** customer.email is a string and is always present.
- **Actual:** orders_page2.json -> ord_1005.customer.email is null.
- **Impact:** Yes. Code that expects a string may fail or reject the order.

### 3\. ord_1006 uses a different money format

- **Docs:** All money fields are integers in the smallest currency unit. For example, \$54.70 is returned as 5470.
- **Actual:** orders_page2.json -> ord_1006 has subtotal: 44.0, tax: 3.63, shipping: 5.99, and total: 53.62.
- **Impact:** Yes, and this is serious. A customer could divide all totals by 100 as the docs say and get the wrong amount for this order.

### 4\. Pagination says there is no next page when there is one

- **Docs:** has_more should be used to decide whether to request another page.
- **Actual:** orders_page1.json has has_more: false, but it also has next_cursor: "cur_8f2a19bd". The candidate pack shows that this cursor was used to get orders_page2.json.
- **Impact:** Yes. A client following has_more would stop after page 1 and miss orders.

### 5\. Missing order returns 200 instead of 404

- **Docs:** GET /v1/orders/{id} returns 404 if the order does not exist.
- **Actual:** order_ord_9999.json is for a non-existing order and the README shows HTTP 200; the response is {"order": null}.
- **Impact:** Yes. A client may treat the request as successful and may not handle the missing order correctly.

### Most serious issue

I think the money format issue is the most serious because it can directly cause wrong financial calculations. The API tells customers to treat values as cents, but ord_1006 looks like dollars. This is especially important because the customer complaint is about revenue reconciliation.

## Task 2 - Total revenue

I used the total field and excluded ord_1003 because its status is refunded. I am assuming it was fully refunded; the data does not give the refund amount, so this is an assumption.

For ord_1006, I treated 53.62 as dollars because its parts add up correctly:

44.00 + 3.63 + 5.99 = 53.62

The other orders use cents as documented.

- ord_1001 = \$54.70
- ord_1002 = \$23.81
- ord_1004 = \$68.10
- ord_1005 = \$25.47
- ord_1006 = \$53.62

**Total revenue = \$225.70**

I would confirm the money format and refund rule with the API team before using this number for production reporting.

## Task 3A - Reply to Priya

**Subject: Revenue reconciliation **

Hi Priya,

I found two data issues that can explain the difference in the revenue report.

Most orders return money in cents, as documented, but ord_1006 returns its amounts as dollar values. Also, ord_1003 is marked as refunded.

Using the non-refunded orders and treating ord_1006 as \$53.62, I get total revenue of **$225.70**.

The money format is inconsistent, so I would confirm the expected format with the API team before using the API data for financial reporting.

Best,  
Syed Sadab Alam

## Task 3B - Bug report

### Money fields returned in the wrong unit

**What to look at:** GET /v1/orders, order ord_1006

**Actual:** subtotal = 44.0, tax = 3.63, shipping = 5.99, total = 53.62.

**Expected:** Following the API documentation, these should be integer values in the smallest currency unit: 4400, 363, 599, and 5362.

**Impact:** Clients following the documentation can calculate the wrong order and revenue amounts.

**Suggested fix:** Return all monetary fields using the documented smallest-unit integer format, and add a regression test for ord_1006-type values.
