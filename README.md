# LEGO Product Analysis Dashboard (1970–2025)

An end-to-end data analytics project that cleans and consolidates 55 years of LEGO product data, loads it into a MySQL database, and visualizes it through an interactive Power BI dashboard. The project tracks how LEGO's product catalog, ratings, packaging, and availability have evolved from 1970 to 2025.

## Dashboard Preview
![](https://github.com/mohitrawat-7/Lego_pro/blob/main/lego%20dashboard%20screenshot.png)

## Project Overview
 
LEGO has released thousands of sets across dozens of themes since 1970. This project answers a simple business question: **how has LEGO's product strategy shifted over five decades, and what does that mean for category, theme, and channel investment going forward?**
 
The pipeline follows a classic analytics workflow:
 
**Raw data (56 yearly CSVs) → Python/Pandas (cleaning & consolidation) → MySQL (storage) → Power BI (visualization & insight)**
 
## Tech Stack
 
| Layer | Tool |
|---|---|
| Data source | Brickset.com set data (yearly CSV exports, 1970–2025) |
| Data cleaning & transformation | Python (Pandas, glob, os) |
| Database | MySQL (via SQLAlchemy + PyMySQL) |
| Visualization | Power BI (.pbix) |
| Connectivity | Power BI → MySQL connector (DirectQuery/Import) |
 
## Data Pipeline
 
### 1. Ingestion
56 separate yearly CSV files (`1970.csv` through `2025.csv`) are read from a local directory and loaded into a dictionary of DataFrames using `glob` and `pandas.read_csv`.
 
### 2. Schema Reconciliation
Earlier years (e.g., 1970) and later years (e.g., 2025) have different column sets — later files include extra fields like `minifigs`, regional pricing (`US_retailPrice`, `UK_retailPrice`, `CA_retailPrice`, `DE_retailPrice`), and physical dimensions (`height`, `width`, `depth`, `weight`). An **inner join concatenation** (`pd.concat(..., join='inner')`) was used to retain only the columns common across all 56 years, since fields like product weight/dimensions add little analytical value compared to theme, category, and rating data.
 
### 3. Cleaning
- Dropped low-value/non-analytical columns: `pieces`, `bricksetURL`, `thumbnailURL`, `imageURL`
- Dropped rows missing a `themeGroup` (insufficient metadata for classification)
- Imputed missing `subtheme` values with `"Not Specified"` rather than dropping rows, to preserve sample size
- Final dataset: **21,544 LEGO sets** across **14 clean columns** (`setID`, `number`, `numberVariant`, `name`, `year`, `theme`, `themeGroup`, `subtheme`, `category`, `released`, `rating`, `reviewCount`, `packagingType`, `availability`)
### 4. Loading
The cleaned DataFrame is pushed into a MySQL database (`projects.lego`) using SQLAlchemy's `create_engine` and `to_sql`, making it queryable via standard SQL and ready for Power BI to connect to directly.
 
### 5. Visualization
Power BI connects to the MySQL `lego` table and builds out the interactive report described below.
 
## Dashboard Features
 
The Power BI report (`lego_dashboard.pbix`) includes:
 
- **KPI cards** — Average rating (3.8), count of categories (7), and total themes (139)
- **Sets Launched per Year** — area chart showing the explosive growth in annual set releases, especially post-2000
- **Avg. Ratings by Year** — trend line showing rating volatility in early decades versus stabilization in recent years
- **Sets by Category** — bar chart showing category distribution (dominated by "Normal" sets at ~8.5K)
- **Availability Channels** — bar chart breaking down sets by retail channel (Retail, Retail-limited, LEGO exclusive, Promotional, etc.)
- **Theme → Subtheme decomposition tree** — drill-down view (e.g., Star Wars → Magazine Gift / The Clone Wars / Episode III) for exploring set distribution within top themes
- **Slicers/filters** — Year range, Packaging Type, and Category, allowing users to dynamically filter the entire report
## Key Insights
 
- **Catalog growth is exponential, not linear.** Annual set releases stayed under 50/year for nearly three decades (1970s–1990s), then grew sharply after 2000, peaking at 500+ sets/year around 2015–2020, before a recent pullback.
- **"Normal" sets dominate the catalog** (~8.5K of ~9.3K analyzed sets), while Gear, Books, Extended, and Collection sets remain niche categories.
- **Retail is overwhelmingly the primary channel** (6.3K sets), more than 5x the next channel (Retail-limited, 1.1K), indicating heavy reliance on mainstream distribution over exclusivity-driven drops.
- **Ratings are volatile in early decades and have stabilized/improved in recent years**, suggesting more consistent quality control or review-base maturity as LEGO scaled.
- **Theme concentration is high within franchises** — e.g., within Star Wars, the Clone Wars and Episode III subthemes account for a disproportionate share of related sets, useful for licensing renewal decisions.

## Business Recommendations
 
1. **Right-size the post-peak catalog.** The decline in sets launched after ~2020 (from 500+/year down sharply) should be investigated against revenue and inventory data — if this is a deliberate move toward quality over quantity, double down on fewer, higher-rated, higher-margin sets rather than chasing volume.
2. **Diversify beyond "Normal" category dependence.** With ~91% of sets falling into a single category, LEGO carries concentration risk. Selectively scaling underrepresented but growing categories (e.g., Collection or Extended sets) could open new customer segments (collectors, adult builders) without cannibalizing the core line.
3. **Reduce over-reliance on standard retail.** Limited and exclusive channels (currently <15% combined of total sets) tend to drive urgency and premium pricing. A modest, deliberate increase in limited/exclusive drops could lift average order value and brand exclusivity without disrupting mass retail availability.
4. **Use early rating volatility as a quality-control benchmark.** Since ratings stabilized notably in recent years, that consistency should be treated as a baseline to protect — any new theme or category launch should be tracked against this standard before scaling production.
5. **Prioritize licensing renewals based on subtheme performance, not just theme.** The decomposition tree shows performance varies significantly within a single franchise (e.g., across Star Wars-era subthemes). Licensing/partnership decisions should be made at the subtheme level, not just the theme level, to avoid overpaying for underperforming pockets of a franchise.
 
 
Set-level data sourced from [Brickset](https://brickset.com), the comprehensive LEGO set database, covering official LEGO releases from 1970–2025.
