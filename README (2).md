# Coffee Sales Dashboard (Excel)

An interactive Excel dashboard that turns three raw tables of coffee-bean orders into a single-screen sales report, built with structured tables, lookup formulas, PivotTables, charts, slicers, and a timeline. All native Excel, no add-ins.

![Made with Excel](https://img.shields.io/badge/Built%20with-Microsoft%20Excel-217346)
![Skills](https://img.shields.io/badge/Skills-PivotTables%20%7C%20XLOOKUP%20%7C%20Slicers-blue)
![License](https://img.shields.io/badge/License-MIT-green)

---

## Overview

A fictional coffee retailer sells roasted beans to customers across the United States, Ireland, and the United Kingdom. The raw data lives in three disconnected tables (orders, customers, products). On its own it answers nothing: the orders table has no names, no country, no prices, and no sales figures.

This project connects those tables and builds a dashboard that answers the questions a sales manager actually asks:

- How much are total sales, and how are they trending over time?
- Which countries drive revenue?
- Who are the most valuable customers?
- How do coffee type, roast, size, and loyalty membership affect sales?

The dashboard is one screen with three visuals and four interactive controls, so a non-technical stakeholder can slice the numbers without touching a formula.

## Dashboard

![Coffee Sales Dashboard](images/dashboard.png)

> The preview above is a screenshot of the live workbook. Excel slicers, the timeline, and PivotCharts only render inside Excel, so open [`dashboard/Coffee_Sales_Dashboard.xlsx`](dashboard/Coffee_Sales_Dashboard.xlsx) to interact with it.

## Key figures

Computed from the 1,000 order lines in the dataset (Jan 2019 to Aug 2022):

| Metric | Value |
| --- | --- |
| Total sales | $45,134 |
| Total bags sold | 3,551 |
| Orders | 957 |
| Customers | 913 |
| Countries | 3 |

**Sales by country:** United States $35,639 (79%), Ireland $6,697 (15%), United Kingdom $2,799 (6%).

**Sales by coffee type:** Exclesa $12,306, Liberica $12,054, Arabisa $11,768, Robusta $9,005. Sales are spread fairly evenly across the four types. (The labels `Exclesa` and `Arabisa` come from the source mapping; the optimization notes flag these as misspellings of Excelsa and Arabica.)

**Sales by year:** 2019 $12,187, 2020 $12,118, 2021 $13,766, 2022 $7,063 (partial year, ends August).

## How it is built

The workbook is a three-layer model: raw data, an enrichment layer, and a presentation layer.

1. **Raw tables.** `orders`, `customers`, and `products` are stored as structured Excel Tables so formulas and PivotTables expand automatically as rows are added.
2. **Enrichment (the `Orders` table).** The orders table is widened with lookup columns that pull customer and product attributes into each order line, then derive sales and readable labels:
   - Customer Name, Email, Country, and Loyalty Card are pulled from `customers` with `XLOOKUP`.
   - Coffee Type, Roast Type, Size, and Unit Price are pulled from `products`.
   - Sales is `Unit Price * Quantity`.
   - Coffee and roast codes are mapped to full names (Rob to Robusta, M to Medium, and so on).
3. **Presentation.** Three PivotTables feed three charts:
   - Total Sales Over Time (line chart, by month and coffee type)
   - Sales by Country (bar chart)
   - Top 5 Customers (bar chart)
   Four controls filter every visual at once: slicers for Roast Type, Size, and Loyalty Card, plus a Timeline on Order Date.

Full step-by-step in [`docs/build-guide.md`](docs/build-guide.md).

## Skills demonstrated

- Data modeling across multiple related tables
- Structured Excel Tables and named references
- Lookup formulas: `XLOOKUP`, `INDEX`/`MATCH`, nested `IF`
- PivotTables and PivotCharts
- Interactive slicers and a timeline wired to multiple pivots
- Dashboard layout, formatting, and storytelling for a non-technical audience

## What was optimized

The original build worked but mixed several lookup styles and repeated some calculations. The optimization pass, applied in the workbook, standardizes the enrichment on `XLOOKUP`, removes a double lookup in the Email column, replaces the product-column `INDEX`/`MATCH` pairs with `XLOOKUP`, and swaps nested `IF` label mapping for `SWITCH`. Only the orders-sheet formulas changed; the PivotTables, charts, slicers, timeline, and layout are untouched and the results are identical. The before-and-after formulas, with the reasoning for each change, are in [`docs/formula-optimizations.md`](docs/formula-optimizations.md).

## Insights and recommendations

- **The United States is the business.** It is 79% of sales. Ireland and the UK are small by comparison, so growth efforts and inventory should be weighted heavily to the US, while Ireland is the more promising of the two smaller markets.
- **No single coffee type dominates.** The four types are within a few percent of each other, so the range is balanced and no type should be cut on sales volume alone.
- **Loyalty customers are not outspending non-members.** Loyalty-card holders account for less total sales than non-members, which suggests the loyalty program is not yet driving incremental spend and is worth reviewing.
- **2022 is a partial year.** It ends in August, so the apparent drop is a data-coverage effect, not necessarily a decline. Compare like-for-like months before concluding a downturn.

## Repository structure

```
coffee-sales-dashboard/
├── README.md
├── LICENSE
├── .gitignore
├── data/
│   └── coffeeOrdersData.xlsx           # raw source: orders, customers, products
├── dashboard/
│   └── Coffee_Sales_Dashboard.xlsx     # the interactive dashboard
├── docs/
│   ├── data-dictionary.md              # every field, in all three tables
│   ├── build-guide.md                  # how the dashboard is built, step by step
│   └── formula-optimizations.md        # original vs optimized formulas
└── images/
    └── dashboard.png                   # dashboard screenshot
```

## How to use

1. Download or clone the repository.
2. Open `dashboard/Coffee_Sales_Dashboard.xlsx` in Excel 2016 or later (slicers and timeline need a modern build; `XLOOKUP` needs Microsoft 365 or Excel 2021).
3. Use the slicers and timeline to filter. To rebuild from scratch, start from `data/coffeeOrdersData.xlsx` and follow the build guide.

## Data source

The sample dataset is the widely used "Coffee Sales" practice dataset (orders, customers, products). It is fictional and used here for portfolio purposes.

## License

Released under the [MIT License](LICENSE).
