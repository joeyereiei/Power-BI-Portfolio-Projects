# Repository Structure

```
ecommerce-sales-dashboard/
├── README.md                          # Project overview, data model, findings — start here
├── STRUCTURE.md                       # This file
│
├── powerbi/
│   └── EcommerceDashboard.pbix        # The Power BI report: Overview, Sales Analysis,
│                                       # and Product & Marketing pages
│
├── data/
│   └── ecommerce_orders_dataset.csv   # Source dataset (30,000 orders, 41 columns)
│                                       # See README > Data Source for provenance
│
│
├── docs/
│   └── dax_measures.md                # Reference list of key DAX measures used in the report
│                                       
│
└── screenshots/
    └── README.md                      # Placeholder — add exported page screenshots here
                                        # (overview.png, sales-analysis.png, product-marketing.png)
```

## Notes on this structure

- **`powerbi/`** holds the single source of truth for the report itself. All DAX and modelling decisions described in the README live inside this file.
