# Formula Optimizations

The first build worked and produced correct numbers. This pass makes the enrichment layer consistent, easier to read, and cheaper to recalculate. Each change is shown as original versus optimized, with the reason.

The theme: standardize every lookup on `XLOOKUP`, calculate each value once, and give every mapping a default.

**These optimizations are applied in `dashboard/Coffee_Sales_Dashboard.xlsx`.** Only the formula text in the orders sheet was changed; the PivotTables, charts, slicers, timeline, layout, and every other sheet are untouched, and the results are identical. To keep the change minimal, the applied formulas use range references (for example `customers!$A$1:$A$1001`) rather than converting the `customers` and `products` ranges to Tables. The structured-reference versions shown below are the further ideal if those two ranges are also turned into Tables. The columns that were already optimal (Customer Name, Country, Loyalty Card, and Sales) were left as they were.

## 0. Make all three ranges Tables

Only `orders` was a structured Table in the original; `customers` and `products` were plain ranges referenced by fixed addresses like `customers!$A$1:$A$1001`. Converting them to Tables (`Customers`, `Products`) means formulas read by name and never break when rows are added or moved.

## 1. Customer lookups: keep XLOOKUP, use structured references

**Original**

```excel
=XLOOKUP(C2, customers!$A$1:$A$1001, customers!$B$1:$B$1001, , 0)
```

**Optimized**

```excel
=XLOOKUP([@[Customer ID]], Customers[Customer ID], Customers[Customer Name], "", 0)
```

Why: the lookup range started at row 1 (the header), which risks a header-to-header match and breaks if rows are inserted. Structured references are self-documenting and auto-expanding. Supplying `""` as the if-not-found argument avoids a `#N/A` on a missing customer.

## 2. Email: one lookup instead of two

**Original**

```excel
=IF(XLOOKUP(C2, customers!$A$1:$A$1001, customers!$C$1:$C$1001, , 0)=0, "",
    XLOOKUP(C2, customers!$A$1:$A$1001, customers!$C$1:$C$1001, , 0))
```

**Optimized**

```excel
=LET(e, XLOOKUP([@[Customer ID]], Customers[Customer ID], Customers[Email], "", 0),
     IF(e=0, "", e))
```

Why: the original runs the same `XLOOKUP` twice, once to test for a blank and once to return it, doubling the work on every row. `LET` computes it once, names it `e`, and reuses it. Same result, half the lookups.

## 3. Product lookups: XLOOKUP instead of double INDEX/MATCH

**Original** (one of four near-identical formulas)

```excel
=INDEX(products!$A$1:$G$49,
       MATCH(orders!$D2, products!$A$1:$A$49, 0),
       MATCH(orders!I$1, products!$A$1:$G$1, 0))
```

**Optimized**

```excel
Coffee Type  =XLOOKUP([@[Product ID]], Products[Product ID], Products[Coffee Type], "", 0)
Roast Type   =XLOOKUP([@[Product ID]], Products[Product ID], Products[Roast Type], "", 0)
Size         =XLOOKUP([@[Product ID]], Products[Product ID], Products[Size], "", 0)
Unit Price   =XLOOKUP([@[Product ID]], Products[Product ID], Products[Unit Price], "", 0)
```

Why: each original cell ran two `MATCH` calls (one for the row, one for the column). `XLOOKUP` against a named column does the same job with one search and reads far more clearly. It also matches the style used for the customer columns, so the whole table uses one lookup pattern instead of two.

## 4. Name mapping: SWITCH with a default

**Original**

```excel
Coffee Type Name  =IF(I2="Rob","Robusta", IF(I2="Exc","Exclesa", IF(I2="Ara","Arabisa", IF(I2="Lib","Liberica"))))
Roast Type Name   =IF(J2="M","Medium", IF(J2="L","Light", IF(J2="D","Dark")))
```

**Optimized**

```excel
Coffee Type Name  =SWITCH([@[Coffee Type]], "Ara","Arabica", "Rob","Robusta", "Exc","Excelsa", "Lib","Liberica", "Unknown")
Roast Type Name   =SWITCH([@[Roast Type]], "L","Light", "M","Medium", "D","Dark", "Unknown")
```

Why: the nested `IF` chains have no final value, so any unexpected code returns `FALSE` into the data. `SWITCH` is flat, reads top to bottom, and ends with an `"Unknown"` default so bad codes are visible instead of silently becoming `FALSE`. This also corrects two misspellings from the source, `Arabisa` to Arabica and `Exclesa` to Excelsa. Correcting them changes the labels shown on the PivotChart legend, so apply it knowingly.

## 5. Sales: structured reference

**Original**

```excel
=L2*E2
```

**Optimized**

```excel
=[@[Unit Price]] * [@Quantity]
```

Why: `L2*E2` breaks if columns are reordered. The structured version refers to the fields by name and survives reordering.

## Compatibility note

`XLOOKUP` and `LET` need Microsoft 365 or Excel 2021. On Excel 2019 or earlier, use `INDEX`/`MATCH`:

```excel
=IFERROR(INDEX(Customers[Customer Name], MATCH([@[Customer ID]], Customers[Customer ID], 0)), "")
```
