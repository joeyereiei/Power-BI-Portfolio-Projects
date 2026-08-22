# Repository Structure

```
ecommerce-sales-dashboard/
├── README.md                          # Project overview, data model, findings — start here
├── STRUCTURE.md                       # This file
│
├── powerbi/
│   └── EcommerceDashboard.pbix        # The Power BI report: Overview, Sales Analysis,
│                                       # and Product & Marketing pages
│   └── dax_measures.md                # Reference list of key DAX measures used in the report
    └── Screenshots.md                 # overview.png, sales-analysis.png, product-marketing.png
```

## Notes on this structure

- **`powerbi/`** holds the single source of truth for the report itself. All DAX and modelling decisions described in the README live inside this file.
