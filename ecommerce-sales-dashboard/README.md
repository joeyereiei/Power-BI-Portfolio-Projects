# E-Commerce Sales Analytics Dashboard

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-217346?style=flat&logo=microsoft&logoColor=white)

A Power BI business intelligence project analysing approximately 30,000 e-commerce orders across sales performance, profitability, product performance, marketing activity, and revenue forecasting.

## Project overview

The dashboard was developed to help stakeholders monitor commercial performance and identify practical opportunities across customers, products, discounting, and marketing channels.

Before producing insights, the underlying identifiers were tested as potential master-data keys. That investigation found that **`Customer_ID`** and **`Product_ID`** were not consistently associated with a single set of descriptive attributes across orders. Using them as conventional dimension keys therefore changed the results shown in visuals.

An early dimensional-model test, for example, materially understated Electronics' revenue share—by roughly three times in one visual. Rather than force unreliable identifiers into a star schema, the final report uses a two-table model: a **`EcommerceOrder`** table and a dedicated **`Date table`**. This retains the order-time attributes as recorded and supports reliable time intelligence.

## Data Source

[E-commerce Orders Dataset 2026 | SCRA](https://www.kaggle.com/datasets/mmumairkhattak/e-commerce-orders-dataset-2026-scra) — Kaggle, by user `mmumairkhattak`. Synthetic data, 30,000 rows, 41 columns, spanning 2023–2026.

## Data model

| Table | Purpose |
|---|---|
| `EcommerceOrders` | Transaction-level source of fact: order details, customer and product attributes as recorded at purchase, financial measures, discounts, and marketing fields. |
| `Date table` | Dedicated calendar table used for date filtering, period comparisons, and time-intelligence measures. |

**Relationship:** `Date Table[Date]` → `EcommerceOrders[Order_Date]`, single-direction, many-to-one.

<img width="838" height="461" alt="Screenshot 2026-08-22 151519" src="https://github.com/user-attachments/assets/d8441da3-e194-4b58-afd7-0f354788fd1e" />

## Data Cleaning & Transformation

- Built a **continuous calendar table** (`Date Table`) spanning the full order date range.
- Added a **`Month_Start_Date`** column (Date type) so Power BI's built-in Forecast analytics and in order of time sorting would work.
- Added **`Day_of_week`** and **`Customer Age Bin`** grouping columns to support time-based and demographic analysis. **`Day_of_week`** is paired with **`Day_of_week_sort`** to ensure weekdays appear in order.
- Created a **`Discount Band`** grouping column to categorise discount levels into ranges (e.g., 0%, 1–10%, 11–20%, 21–30%, and 30%+) for comparing AOV and discount behaviour across orders.

## Dashboard Coverage

- **Executive performance:** revenue, profit, orders, profit margin, AOV, and period trends.
- **Sales analysis:** revenue, profit, COGS, high-value orders, peak days, peak seasons, and forecasting.
- **Product analysis:** category performance, product contribution, profitability, discount bands, and high-value orders.
- **Marketing analysis:** traffic sources, payment methods, device types, and discount activity.
- **Discount analysis:** discount levels, total discount given, average discount per order, and AOV by discount band.

## DAX and KPIs

The report uses DAX measures for core KPIs and time-intelligence analysis, including revenue, profit, AOV, profit margin, order metrics, discounts, and high-value orders. Detailed calculations are documented in [`powerbi/dax/README.md`](powerbi/dax/README.md).

Core KPIs include:

| KPI | Value |
|---|---|
| Total Revenue | £11.37M |
| Total Profit | £2.22M |
| Total COGS | £9.15M |
| Profit Margin % | 19.49% |
| Total Orders | 30,000 |
| Total Units Sold | 92,420 |
| AOV | £379.00 |
| High-Value Order Rate | 25.00% |
| Total Discount Given | £1.48M |
| Total Discount Tax Amount | £874.31K |
| Return Rate % | 10.11% |
| Cancelled Rate % | 5.90% |

## Key insights

- Electronics dominates revenue but not profitability, Fashion has the highest order volume, and Beauty has the highest profit margin.
- Higher discounts are associated with lower AOV
- High-value orders are driven by product category, not customer segment
- Wednesday and summer are the strongest revenue periods
- No single marketing channel or payment method stands out

## Repository Structure

See [`STRUCTURE.md`](./STRUCTURE.md) for a full folder-by-folder breakdown.

## Author

Hataipon Pengkumma
