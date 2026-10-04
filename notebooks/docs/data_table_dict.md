| Column | Meaning | Type Expected | Analyst Notes |
|---|---|---|---|
| `INVDate` | transaction date | datetime | currently text |
| `INVNumber` | invoice identifier | string | partially corrupted |
| `Outlet` | outlet identifier | string | preserve exactly |
| `Outlet Name` | customer/outlet name | text | may need normalization |
| `SKUCode` | product identifier | string | likely usable |
| `ProductName` | product description | text | check consistency |
| `GSV` | gross sales value | numeric | locale parsing needed |
| `Discount` | discount value | numeric | usually negative |
| `Tax` | tax | numeric | validate relationship |
| `Net` | final sales amount | numeric | validate formula |