# SalesAnalytics – Power BI project with Power Query and DAX

This folder is a Power BI Project (PBIP). The project loads sample sales data from CSV files with Power Query (M). It calculates the results with DAX. The data is not real. A script made it for practice.

## Contents

| Item | What it is |
|---|---|
| `SalesAnalytics.pbip` | The project file. Open this file in Power BI Desktop. |
| `SalesAnalytics.SemanticModel/` | The data model in TMDL text files: Power Query queries, tables, relationships and DAX. |
| `SalesAnalytics.Report/` | The report in PBIR JSON files: 2 pages and their visuals. |
| `Data/` | The source CSV files. `Data/Sales/` has 1 file for each year (2023–2025). |
| `.gitignore` | Tells Git not to save the local cache and settings files. |

## Before you start

- Power BI Desktop runs only on Windows. On a Mac, use a Windows computer or a Windows virtual machine (for example, Parallels Desktop).
- Install the latest version of Power BI Desktop.

## Open the project

1. Copy the full `power_bi` folder to `C:\power_bi` on the Windows computer.
2. Double-click `SalesAnalytics.pbip`.
3. Select **Home > Refresh**. Power BI loads the CSV files.
4. If you put the folder in a different location, do these steps before you refresh:
   1. Select **Home > Transform data > Edit parameters**.
   2. Set **DataFolder** to the full path of the `Data` folder. Example: `D:\Projects\power_bi\Data\`.
   3. Select **OK**, then select **Apply changes**.
5. Select **File > Save**. Power BI Desktop writes your changes back to the text files in this folder.

If Power BI Desktop shows an error about the TMDL format, do this: select **File > Options and settings > Options > Preview features**. Select **Store semantic model using TMDL format**. Restart Power BI Desktop and open the project again.

## Power Query (M)

Open the queries with **Home > Transform data**.

| Query | What it does |
|---|---|
| `DataFolder` | Parameter. The path of the `Data` folder. |
| `fnCleanText` | Custom function. It trims text, removes double spaces and uses Proper Case. |
| `Sales` | Reads all CSV files in `Data\Sales` and appends them. Changes empty discounts to 0. Removes rows with quantity 0 and duplicate order lines. Renames the columns. Adds the column `Line Amount`. |
| `Product` | Reads `Products.csv`, sets data types and trims text. |
| `Customer` | Reads `Customers.csv`. Cleans names and cities with `fnCleanText`. Changes `USA` and `UK` to full country names. |
| `Category` | References the `Product` query and keeps 1 row for each category. |
| `Date` | Makes 1 row for each day, for the full years in the sales data. Adds year, quarter, month and weekday columns. |
| `Target` | Reads `Targets.csv` (1 column for each month) and unpivots it to 1 row for each category and month. |

All type changes use the `en-US` culture. Thus the numbers in the CSV files load correctly on computers with other regional settings.

## Data model

```
Category 1 ──* Product 1 ──* Sales *── 1 Customer
Category 1 ──* Target  *──────────────────────┐
Date     1 ──* Sales                          │
Date     1 ──* Target (Month Start) ──────────┘
```

All relationships filter in one direction, from the "1" side to the "*" side. `Date` is marked as a date table.

## DAX

Calculated columns:

- `Sales[Line Cost]` = quantity × the unit cost of the related product (`RELATED`).
- `Product[Price Band]` = a price group (`SWITCH`).

Measures are in the table `_Measures`, in 5 display folders:

| Folder | Measures |
|---|---|
| 1. Sales | Sales Amount, Gross Sales, Discount Amount, Total Cost, Gross Profit, Gross Margin %, Units Sold, Orders, Avg Order Value |
| 2. Customers | Active Customers, New Customers |
| 3. Time Intelligence | Sales YTD (`TOTALYTD`), Sales PY (`SAMEPERIODLASTYEAR`), Sales YoY %, Sales Rolling 3M (`DATESINPERIOD`) |
| 4. Targets | Target Amount, Sales vs Target %, Target Variance |
| 5. Ranking and Share | Product Rank (`RANKX`), Sales % of Total (`ALLSELECTED`), Top Product (`TOPN`) |

Each measure has a description. You can see it when you put the pointer on the measure in the **Data** pane.

## Report

- **Sales Overview**: 4 cards, sales by month (this year and last year), sales and target by category, sales by product, sales by country.
- **Product and Customer Details**: a matrix of products by category with rank and share, cards, customers by segment, and a monthly table with YTD, rolling 3 months and target.

The Year and Category slicers are synchronized on the 2 pages.

## Check the numbers

After the refresh, compare the report with these values. Small differences in cents can occur because of rounding.

| Year | Sales Amount | Gross Margin % | Orders | Sales YoY % |
|---|---|---|---|---|
| 2023 | 376,121 | 55.4% | 1,511 | – |
| 2024 | 457,549 | 55.5% | 1,734 | +21.6% |
| 2025 | 494,214 | 55.2% | 1,864 | +8.0% |
| Total | 1,327,884 | 55.4% | 5,109 | |

Row counts: the CSV files have 9,681 sales rows. Power Query removes 34 rows with quantity 0 and 31 duplicate lines, so `Sales` has 9,616 rows. `Date` has 1,096 rows.

## How the files were checked

- All JSON files were checked against the Microsoft JSON schemas for PBIP and PBIR.
- All M code was checked with the Microsoft Power Query parser (syntax only).
- A script checked that each DAX column and measure reference, each relationship and each report field exists in the model.
- A script ran the same Power Query steps in Python to get the numbers above.
- The project was **not** opened in Power BI Desktop before delivery, and no DAX engine ran the measures. If Power BI Desktop shows an error, the message gives the file and the line.

## Edit the project

- In Power BI Desktop: make changes and save. The text files change.
- In a text editor (for example, VS Code with the TMDL extension): close Power BI Desktop first. Then edit the `.tmdl` and `.json` files and open the project again.
