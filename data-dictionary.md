# Data Dictionary

The source workbook `coffeeOrdersData.xlsx` has three tables. The dashboard joins them into one enriched `Orders` table.

## orders (1,000 rows)

The transaction log. In the raw file, only the first five columns are filled; the rest are added by the enrichment formulas.

| Column | Source | Description |
| --- | --- | --- |
| Order ID | raw | Order identifier (repeats across lines of the same order) |
| Order Date | raw | Date the order was placed |
| Customer ID | raw | Foreign key to `customers` |
| Product ID | raw | Foreign key to `products` (encodes coffee, roast, size) |
| Quantity | raw | Number of bags on the line |
| Customer Name | lookup | Pulled from `customers` |
| Email | lookup | Pulled from `customers` (blank shown as empty, not 0) |
| Country | lookup | Pulled from `customers` |
| Coffee Type | lookup | Code pulled from `products` (Ara, Rob, Exc, Lib) |
| Roast Type | lookup | Code pulled from `products` (L, M, D) |
| Size | lookup | Bag size in kg, pulled from `products` |
| Unit Price | lookup | Price per bag, pulled from `products` |
| Sales | derived | `Unit Price * Quantity` |
| Coffee Type Name | derived | Full coffee name from the code |
| Roast Type Name | derived | Full roast name from the code |
| Loyalty Card | lookup | Yes or No, pulled from `customers` |

## customers (1,000 rows)

| Column | Description |
| --- | --- |
| Customer ID | Primary key |
| Customer Name | Full name |
| Email | Email address (some blank) |
| Phone Number | Phone |
| Address Line 1 | Street address |
| City | City |
| Country | United States, Ireland, or United Kingdom |
| Postcode | Postal code |
| Loyalty Card | Yes or No |

## products (48 rows)

| Column | Description |
| --- | --- |
| Product ID | Primary key, format `Coffee-Roast-Size` (for example `A-L-0.2`) |
| Coffee Type | Ara, Rob, Exc, Lib |
| Roast Type | L (light), M (medium), D (dark) |
| Size | Bag size in kg (0.2, 0.5, 1.0, 2.5) |
| Unit Price | Price per bag |
| Price per 100g | Unit economics reference |
| Profit | Profit per bag |

## Code mappings

| Field | Code | Name |
| --- | --- | --- |
| Coffee Type | Ara | Arabica (labeled `Arabisa` in the source) |
| Coffee Type | Rob | Robusta |
| Coffee Type | Exc | Excelsa (labeled `Exclesa` in the source) |
| Coffee Type | Lib | Liberica |
| Roast Type | L | Light |
| Roast Type | M | Medium |
| Roast Type | D | Dark |
