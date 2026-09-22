# Build Guide

How the dashboard is built, from three raw tables to one interactive screen. Follow these steps to rebuild it from `data/coffeeOrdersData.xlsx`.

## 1. Turn the raw ranges into Excel Tables

Select each range and press Ctrl+T (Insert > Table), then name them on the Table Design tab:

- `orders` to `Orders`
- `customers` to `Customers`
- `products` to `Products`

Tables make formulas readable (`Customers[Customer ID]` instead of `A2:A1001`) and let PivotTables grow automatically when rows are added.

## 2. Enrich the Orders table

Add these columns to `Orders`. Because it is a Table, one formula fills the whole column.

**Customer attributes (from Customers):**

```excel
Customer Name  =XLOOKUP([@[Customer ID]], Customers[Customer ID], Customers[Customer Name], "", 0)
Email          =XLOOKUP([@[Customer ID]], Customers[Customer ID], Customers[Email], "", 0)
Country        =XLOOKUP([@[Customer ID]], Customers[Customer ID], Customers[Country], "", 0)
Loyalty Card   =XLOOKUP([@[Customer ID]], Customers[Customer ID], Customers[Loyalty Card], "", 0)
```

**Product attributes (from Products):**

```excel
Coffee Type  =XLOOKUP([@[Product ID]], Products[Product ID], Products[Coffee Type], "", 0)
Roast Type   =XLOOKUP([@[Product ID]], Products[Product ID], Products[Roast Type], "", 0)
Size         =XLOOKUP([@[Product ID]], Products[Product ID], Products[Size], "", 0)
Unit Price   =XLOOKUP([@[Product ID]], Products[Product ID], Products[Unit Price], "", 0)
```

**Derived columns:**

```excel
Sales             =[@[Unit Price]] * [@Quantity]
Coffee Type Name  =SWITCH([@[Coffee Type]], "Ara","Arabica", "Rob","Robusta", "Exc","Excelsa", "Lib","Liberica", "Unknown")
Roast Type Name   =SWITCH([@[Roast Type]], "L","Light", "M","Medium", "D","Dark", "Unknown")
```

These are the optimized versions. See `formula-optimizations.md` for how they differ from the first build.

## 3. Build the PivotTables

Create three PivotTables from the `Orders` table (Insert > PivotTable). Put each on its own sheet.

1. **Total Sales Over Time**
   - Rows: Order Date (grouped by Year, then Month)
   - Columns: Coffee Type Name
   - Values: Sum of Sales

2. **Sales by Country**
   - Rows: Country
   - Values: Sum of Sales
   - Sort descending

3. **Top 5 Customers**
   - Rows: Customer Name
   - Values: Sum of Sales
   - Sort descending, then Value Filters > Top 10 set to Top 5

## 4. Add the charts

- Total Sales Over Time: Line chart (a line per coffee type)
- Sales by Country: Bar or column chart
- Top 5 Customers: Bar chart

Remove chart clutter: delete gridlines you do not need, keep titles short, and format the value axis as currency.

## 5. Wire up the interactivity

Select any PivotTable, then PivotTable Analyze > Insert Slicer and Insert Timeline.

- Slicers: Roast Type, Size, Loyalty Card
- Timeline: Order Date

Then use **Report Connections** on each slicer and the timeline to connect it to all three PivotTables, so one click filters the whole dashboard. This shared filtering is the core of the dashboard.

## 6. Assemble the Dashboard sheet

- Create a `Dashboard` sheet and move (cut and paste) the three charts, slicers, and timeline onto it.
- Add a title banner and arrange the visuals on a grid.
- Hide the helper pivot sheets, or group them, so viewers see only the dashboard.
- Set a clean, consistent color theme and number formatting.

## Requirements

- Excel 2016 or later for slicers and the timeline.
- `XLOOKUP` requires Microsoft 365 or Excel 2021. On older versions, use the `INDEX`/`MATCH` fallback shown in `formula-optimizations.md`.
