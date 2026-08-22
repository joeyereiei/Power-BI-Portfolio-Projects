# DAX Measures Reference

Key measures used across the report, grouped by purpose. All reference `EcommerceOrders` and `Date Table` directly — there is no dimension table in this model (see README > Project overview).

## Core KPIs

```dax

Total Revenue = SUM(EcommerceOrders[Order_Amount])
Total Profit = SUM(EcommerceOrders[Profit_Amount])
Total COGS = Total COGS = [Total Revenue] - [Total Profit]
Profit Margin % = DIVIDE([Total Profit], [Total Revenue])
Total Orders = DISTINCTCOUNT(EcommerceOrders[Order_ID])
AOV = DIVIDE(EcommerceOrders[Total Revenue],EcommerceOrders[Total Orders])
High-Value Order Rate = 
DIVIDE([High-Value Orders],[Total Orders],0)
Total Discount Given = SUM(EcommerceOrders[Discount_Amount])
Total Discount Tax Amount = SUM(EcommerceOrders[Tax_Amount])
Return Rate % = DIVIDE(CALCULATE(COUNTROWS(EcommerceOrders), EcommerceOrders[Returned] = "YES"),COUNTROWS(EcommerceOrders))
Cancelled Rate % = DIVIDE(CALCULATE(COUNTROWS(EcommerceOrders), EcommerceOrders[Order_Status] = "Cancelled"),COUNTROWS(EcommerceOrders))

```

## All-slicer-safe YTD comparison

Prevents `N/A` when a Year/Month slicer is set to "All":

```dax
Revenue Card =
IF(
    HASONEVALUE('Date Table'[Year]),
    TOTALYTD(SUM(EcommerceOrders[Order_Amount]), 'Date Table'[Date]),
    SUM(EcommerceOrders[Order_Amount])
)
```

## YoY growth display pattern (value + formatted text + conditional colour)

**Note:** `Date Table` must be marked as the model's official Date table (Modeling > Mark as Date table) for `DATEADD` and the Forecast analytics feature to work correctly.

Three measures working together to give a KPI card a directional arrow, coloured text, and a safe fallback when multiple years are selected — rather than one measure trying to do everything at once.

**1. The underlying number**, guarded against the "All years selected" case with `HASONEVALUE`:

```dax
YoY Value_Revenue =
VAR IsSingleYear = HASONEVALUE('Date Table'[Year])
VAR CY = [Total Revenue]
VAR PY = CALCULATE([Total Revenue], DATEADD('Date Table'[Date], -1, YEAR))
RETURN
    IF(NOT IsSingleYear, BLANK(), DIVIDE(CY - PY, PY))
```

**2. The display text**, built on top of the same logic, adding a ▲/▼ unicode arrow and an `"N/A"` fallback:

```dax
YoY % =
VAR IsSingleYear = HASONEVALUE('Date Table'[Year])
VAR CY = [Total Revenue]
VAR PY = CALCULATE([Total Revenue], DATEADD('Date Table'[Date], -1, YEAR))
VAR perc = DIVIDE(CY - PY, PY)
RETURN
    IF(
        NOT IsSingleYear, "N/A",
        SWITCH(
            TRUE(),
            ISBLANK(PY) || PY = 0, "N/A",
            perc > 0, UNICHAR(11165) & " " & FORMAT(perc, "0.0%"),
            perc < 0, UNICHAR(11167) & " " & FORMAT(ABS(perc), "0.0%"),
            "0.0%"
        )
    )
```

**3. The conditional format colour**, applied via Format pane > Conditional formatting > Font colour > fx, referencing this measure:

```dax
CF YoY =
SWITCH(
    TRUE(),
    [YoY Value_Revenue] > 0, "#00B050",
    [YoY Value_Revenue] < 0, "#FF0000",
    "#808080"
)
```

**Why three separate measures instead of one:** each one has a single job — `YoY Value_Revenue` is the number other measures or visuals can safely do math on, `YoY %` is purely for what the person reading the card sees, and `CF YoY` is purely for colour. Keeping them separate means the colour logic and the text logic can't drift out of sync with each other, since both read from the same underlying value measure rather than recalculating `CY`/`PY` independently.

**Possible simplification:** `YoY %` currently recalculates `CY`/`PY` from scratch rather than referencing `[YoY Value_Revenue]` directly (i.e. `VAR perc = [YoY Value_Revenue]`). Referencing the existing measure would remove the duplicated logic and guarantee the display text and the conditional colour can never disagree, even if the YoY calculation is changed later.

### Adapting the pattern to other metrics

The three measures above are just `[Total Revenue]` wrapped in the same reusable shape. To reuse it for any other base measure, swap out `[Total Revenue]` for the new measure and rename the three variants consistently — the `HASONEVALUE`, `DATEADD`, and `SWITCH` logic stays identical.

**Example — same pattern applied to Profit:**

```dax
YoY Value_Profit =
VAR IsSingleYear = HASONEVALUE('Date Table'[Year])
VAR CY = [Total Profit]
VAR PY = CALCULATE([Total Profit], DATEADD('Date Table'[Date], -1, YEAR))
RETURN
    IF(NOT IsSingleYear, BLANK(), DIVIDE(CY - PY, PY))

YoY % Profit =
VAR IsSingleYear = HASONEVALUE('Date Table'[Year])
VAR perc = [YoY Value_Profit]
RETURN
    IF(
        NOT IsSingleYear, "N/A",
        SWITCH(
            TRUE(),
            ISBLANK(perc), "N/A",
            perc > 0, UNICHAR(11165) & " " & FORMAT(perc, "0.0%"),
            perc < 0, UNICHAR(11167) & " " & FORMAT(ABS(perc), "0.0%"),
            "0.0%"
        )
    )

CF YoY Profit =
SWITCH(
    TRUE(),
    [YoY Value_Profit] > 0, "#00B050",
    [YoY Value_Profit] < 0, "#FF0000",
    "#808080"
)
```

Note `YoY % Profit` above already applies the DRY fix suggested earlier — it references `[YoY Value_Profit]` instead of recalculating `CY`/`PY` a second time.

**Same swap works for `Total Orders`, `AOV`, or any other additive measure** — just rename `_Revenue`/`_Profit` to match (e.g. `YoY Value_Orders`, `YoY % Orders`, `CF YoY Orders`) and point `CY`/`PY` at the new base measure. A consistent naming convention like `YoY Value_<Metric>` makes it easy to find all three related measures together in the Fields pane, since Power BI sorts measures alphabetically.

### Adapting the pattern to a different time grain (MoM instead of YoY)

Same trio again, but comparing month-over-month instead of year-over-year — only two things change: the `HASONEVALUE` check moves from `Year` to `Month_Start_Date` (so it only evaluates when a single month is in context, not a single year), and `DATEADD(..., -1, YEAR)` becomes `DATEADD(..., -1, MONTH)`.

```dax
MoM Value_Revenue =
VAR IsSingleMonth = HASONEVALUE('Date Table'[Month_Start_Date])
VAR CM = [Total Revenue]
VAR PM = CALCULATE([Total Revenue], DATEADD('Date Table'[Date], -1, MONTH))
RETURN
    IF(NOT IsSingleMonth, BLANK(), DIVIDE(CM - PM, PM))

MoM % =
VAR IsSingleMonth = HASONEVALUE('Date Table'[Month_Start_Date])
VAR perc = [MoM Value_Revenue]
RETURN
    IF(
        NOT IsSingleMonth, "N/A",
        SWITCH(
            TRUE(),
            ISBLANK(perc), "N/A",
            perc > 0, UNICHAR(11165) & " " & FORMAT(perc, "0.0%"),
            perc < 0, UNICHAR(11167) & " " & FORMAT(ABS(perc), "0.0%"),
            "0.0%"
        )
    )

CF MoM =
SWITCH(
    TRUE(),
    [MoM Value_Revenue] > 0, "#00B050",
    [MoM Value_Revenue] < 0, "#FF0000",
    "#808080"
)
```

Combine freely with the metric swap above — e.g. `MoM Value_Profit` / `MoM % Profit` / `CF MoM Profit` needs both changes (base measure *and* time grain) applied at once, but the shape of all three measures never changes.
