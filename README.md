# Nike P&L Financial Dashboard 📊
<img width="1517" height="848" alt="image" src="https://github.com/user-attachments/assets/8cdcd3cf-ecf9-4550-8155-40278513a9bd" />

## 📋 Project Overview

An interactive Power BI dashboard demonstrating comprehensive P&L (Profit & Loss) statement development with advanced data modeling and financial analysis capabilities. This project showcases data transformation, DAX measure creation, and dynamic financial reporting techniques using a simulated retail dataset spanning 2017-2021.

> **Note**: This project uses fictional data for educational purposes to demonstrate Power BI capabilities in building financial dashboards. The data does not represent actual Nike financial information.

## 🎯 Project Objectives

- **P&L Statement Development**: Build a comprehensive, hierarchical Profit & Loss statement in Power BI
- **Data Manipulation & Cleaning**: Transform raw transaction data into structured financial statements
- **Advanced DAX Implementation**: Create complex measures for dynamic financial calculations
- **Segment Analysis**: Compare performance between Men's and Women's shoe revenue streams
- **Regional Insights**: Analyze revenue distribution across different geographic regions
- **Trend Analysis**: Track financial KPIs over a 5-year period (2017-2021)
- **Dynamic Reporting**: Enable flexible drill-down analysis with interactive slicers and conditional formatting

## 💡 Key Features

### Financial Metrics Dashboard
- **Total Revenue**: $269.9M
- **Gross Margin**: $122.9M (45.5%)
- **Year-over-Year Analysis**: 2017-2021 performance trends
- **Comprehensive P&L Statement**: Complete income statement with all major line items

### Interactive Components
- 📅 **Year Slicer**: Dynamic filtering between 2017-2021
- 🌎 **Regional Analysis**: Revenue breakdown by geographic location
- 👟 **Product Segmentation**: Men's vs Women's shoe revenue comparison
- 📊 **Visual KPIs**: Clear display of key financial metrics

## 🛠️ Technical Stack

- **Business Intelligence**: Microsoft Power BI Desktop
- **Data Modeling**: Star schema with fact and dimension tables
- **Calculations**: Advanced DAX expressions
- **Visualization**: Custom charts, KPIs, matrix tables, and trend analysis

## 📊 Data Model

The solution implements a **star schema** design optimized for financial reporting with the following structure:

### Dimension Tables

#### DimAcc (Account Dimension)
- **AccNum**: Account number (1001-9001)
- **Description**: Account description (e.g., "Mens Shoe Revenue", "Womens Shoe COGS")
- **Category**: Account category classification (Revenue, COGS, Operating Expenses, etc.)

**Sample Records**:
```
1001 | Mens Shoe Revenue        | Revenue
1002 | Womens Shoe Revenue      | Revenue
2001 | Mens Shoe COGS          | COGS
2002 | Womens Shoe COGS        | COGS
3001 | Employee Expenses       | Operating Expenses
5001 | Depreciation            | Depreciation & Amortization
8001 | Interest Expense        | Interest Expenses
9001 | Taxes                   | Tax Expenses
```

#### DimDate (Date Dimension)
- **Date**: Full date value
- **Date - Copy**: Duplicate date field for special relationships
- **Month**: Month name
- **Quarter**: Quarter (Q1-Q4)
- **Year**: Year (2017-2021)

**Coverage**: Daily granularity from January 1, 2017 to December 31, 2021

#### DimPLLineItems (P&L Line Items Hierarchy)
- **Line Item**: Major P&L categories (Revenue, COGS, Gross Margin, Operating Expenses, EBITDA, etc.)
- **LineItemType**: Classification (1=BaseValue, 2=Subtotal, 3=Subpercentage)
- **LineItemDesc**: Display type (BaseValue, Subtotal, Subpercentage)
- **LineOrder**: Sequence order for P&L presentation (1-12)

**P&L Structure**:
```
1  | Revenue                      | BaseValue      | 1
2  | COGS                        | BaseValue      | 2
3  | Gross Margin                | Subtotal       | 3
4  | Gross Margin %              | Subpercentage  | 4
5  | Operating Expenses          | BaseValue      | 5
6  | EBITDA                      | Subtotal       | 6
7  | Depreciation & Amortization | BaseValue      | 7
8  | EBIT                        | Subtotal       | 8
9  | Interest Expenses           | BaseValue      | 9
10 | EBT                         | Subtotal       | 10
11 | Tax Expenses                | BaseValue      | 11
12 | Net Income                  | Subtotal       | 12
```

#### DimStore (Store Location Dimension)
- **StoreID**: Unique store identifier (9180031-9180048+)
- **Store Type**: Store classification (General Store)
- **City**: Store city location
- **State**: U.S. state
- **Region**: Geographic region (Northeast, South, West, Midwest, Southeast)

**Sample Locations**:
```
9180031 | New York       | New York     | Northeast
9180032 | Houston        | Texas        | South
9180033 | San Francisco  | California   | West
9180034 | Los Angeles    | California   | West
9180035 | Chicago        | Illinois     | Midwest
9180041 | Miami          | Florida      | Southeast
```

### Fact Table

#### FactPLTran (P&L Transaction Fact)
- **PLTransID**: Unique transaction identifier
- **TransDate**: Transaction date
- **TransDescription**: Transaction description (e.g., "Mens Shoe Revenue")
- **Transaction Amount**: Dollar amount of transaction
- **AccNum**: Foreign key to DimAcc
- **StoreID**: Foreign key to DimStore

**Sample Transactions**:
```
1800 | 2017-01-01 | Mens Shoe Revenue | $24,116 | 1001 | 9180031
1813 | 2017-02-01 | Mens Shoe Revenue | $24,164 | 1001 | 9180031
1826 | 2017-03-01 | Mens Shoe Revenue | $24,889 | 1001 | 9180031
```

### Relationships
- **FactPLTran[AccNum]** ➝ **DimAcc[AccNum]** (Many-to-One)
- **FactPLTran[StoreID]** ➝ **DimStore[StoreID]** (Many-to-One)
- **FactPLTran[TransDate]** ➝ **DimDate[Date]** (Many-to-One)
- **Cross-filter direction**: Single direction for optimal performance
- **Cardinality**: Properly configured for star schema best practices

## 🧮 DAX Measures Library

### Core Financial Measures

#### 1. Sum Line Amounts (Base Measure)
```dax
Sum Line Amounts = SUM(FactPLTran[Transaction Amount])
```
Foundation measure that aggregates all transaction amounts from the fact table.

#### 2. Sum Revenue
```dax
Sum Revenue = 
CALCULATE(
    [Sum Line Amounts],
    DimPLLineItems[Line Item] = "Revenue"
)
```
Calculates total revenue by filtering transactions to revenue accounts only.

#### 3. SUM COGS (Cost of Goods Sold)
```dax
SUM COGS = 
CALCULATE(
    [Sum Line Amounts],
    DimPLLineItems[Line Item] = "COGS"
)
```
Isolates and sums all Cost of Goods Sold transactions.

#### 4. Sum Expenses
```dax
Sum Expenses = 
CALCULATE(
    [Sum Line Amounts] * -1,
    DimPLLineItems[Line Item] <> "Revenue"
)
```
Aggregates all non-revenue items (expenses) and converts to negative values for proper P&L presentation.

#### 5. Line Amount (Dynamic Calculation)
```dax
Line Amount = 
IF(
    SELECTEDVALUE(DimPLLineItems[Line Item]) = "Revenue",
    [Sum Revenue],
    [Sum Expenses]
)
```
Switches between revenue and expense calculations based on selected P&L line item.

### Profitability Measures

#### 6. Gross Profit
```dax
Gross Profit = [Sum Revenue] - [SUM COGS]
```
Calculates gross profit as the difference between revenue and cost of goods sold.

#### 7. Gross Margin (%)
```dax
Gross Margin = [Gross Profit] / [Sum Revenue]
```
Computes gross margin percentage for profitability analysis.

### Advanced Dynamic Measures

#### 8. PL Subtotal (Running Total Logic)
```dax
PL Subtotal = 
CALCULATE(
    [Sum Revenue] + [Sum Expenses],
    FILTER(
        ALL(DimPLLineItems),
        DimPLLineItems[LineOrder] < MAX(DimPLLineItems[LineOrder])
    )
)
```
**Key Feature**: Implements cascading subtotal logic for P&L structure
- Removes existing filter context with ALL()
- Calculates running sum up to current line item
- Enables proper EBITDA, EBIT, EBT, and Net Income calculations

#### 9. % Revenue (Percentage of Revenue)
```dax
% Revenue = 
[PL Subtotal] / 
CALCULATE(
    [Sum Revenue],
    FILTER(
        ALL(DimPLLineItems),
        DimPLLineItems[Line Item] = "Revenue"
    )
)
```
Calculates each line item as a percentage of total revenue, useful for vertical analysis.

#### 10. PLAmt (Master Display Logic)
```dax
PLAmt = 
VAR MainLineItemCheck = [Filter Test]
RETURN
SWITCH(
    TRUE(), 
    SELECTEDVALUE(DimPLLineItems[LineItemDesc]) = "BaseValue", 
        [Line Amount],
    SELECTEDVALUE(DimPLLineItems[LineItemDesc]) = "Subtotal" && MainLineItemCheck, 
        [PL Subtotal],
    SELECTEDVALUE(DimPLLineItems[LineItemDesc]) = "Subpercentage" && MainLineItemCheck, 
        [% Revenue]
)
```
**Critical Measure**: Master switching logic that determines which calculation to display
- **BaseValue**: Shows raw line amounts (Revenue, COGS, Operating Expenses)
- **Subtotal**: Shows calculated subtotals (Gross Margin, EBITDA, EBIT, Net Income)
- **Subpercentage**: Shows percentages (Gross Margin %, margins as % of revenue)

#### 11. USD (Conditional Formatting)
```dax
USD = 
SWITCH(
    TRUE(), 
    SELECTEDVALUE(DimPLLineItems[LineItemDesc]) = "BaseValue" && NOT ISBLANK([PLAmt]), 
        FORMAT([PLAmt], "$0,0"),
    SELECTEDVALUE(DimPLLineItems[LineItemDesc]) = "Subtotal" && NOT ISBLANK([PLAmt]), 
        FORMAT([PLAmt], "$0,0"),
    SELECTEDVALUE(DimPLLineItems[LineItemDesc]) = "Subpercentage" && NOT ISBLANK([PLAmt]), 
        FORMAT([PLAmt], "0.0%")
)
```
**Formatting Measure**: Dynamically formats values based on line item type
- Currency format ($) for dollar amounts
- Percentage format (%) for ratio calculations
- Handles blank values gracefully

### Measure Relationships
```
Sum Line Amounts (Base)
    ├── Sum Revenue
    ├── SUM COGS
    ├── Sum Expenses
    │   ├── Line Amount
    │   └── PL Subtotal
    │       ├── % Revenue
    │       └── PLAmt ──→ USD (Display)
    └── Gross Profit
        └── Gross Margin
```

## 📈 Dashboard Components

### 1. Executive Summary Section
- High-level KPI cards showing total revenue and gross margin
- Gross margin percentage indicator
- Year range selector for temporal analysis

### 2. Detailed P&L Matrix
- Hierarchical view of all income statement line items
- Multi-year comparison (2017-2021)
- Expandable categories for deeper analysis
- Color-coded performance indicators

### 3. Regional Performance
- Donut chart visualization showing revenue by region
- Geographic distribution insights
- Regional performance comparison

### 4. Product Line Analysis
- Bar chart comparing Men's vs Women's shoe revenue
- 5-year trend visualization
- Product segment performance tracking

## 🎨 Design Highlights

- **Professional Branding**: Nike-themed color scheme with logo integration
- **Clean Layout**: Intuitive navigation and information hierarchy
- **Responsive Design**: Optimized for various screen sizes
- **Consistent Styling**: Uniform formatting across all visualizations

## 📁 Repository Structure

```
nike-pl-dashboard/
│
├── 📁 Data/
│   ├── DimAcc.xlsx                 # Account dimension
│   ├── DimDate.xlsx                # Date dimension
│   ├── DimPLLineItems.xlsx         # P&L structure
│   ├── DimStore.xlsx               # Store locations
│   └── FactPLTran.xlsx             # Transaction data
│
├── 📁 PowerBI/
│   └── Nike_PL_Dashboard.pbix      # Power BI project file
│
└── 📄 README.md                     # Project documentation
```

> **Note**: DAX measures are stored within the `.pbix` file itself. Power BI does not require separate DAX files in the repository, as all measures, calculations, and data model configurations are embedded in the Power BI Desktop file.

## 🔮 Next Steps & Future Enhancements

### Planned Improvements
- [ ] **What-If Analysis**: Add parameter sliders for scenario modeling and forecasting
- [ ] **Variance Analysis**: Implement budget vs actual comparisons with variance calculations
- [ ] **Drill-Through Pages**: Create detailed transaction-level pages for deeper investigation
- [ ] **Mobile Layout**: Optimize dashboard for mobile viewing and touch interactions
- [ ] **Bookmarks**: Add saved views for different stakeholder perspectives
- [ ] **Advanced Visualizations**: Incorporate waterfall charts for P&L flow analysis
- [ ] **Row-Level Security**: Implement RLS for region-specific data access
- [ ] **Incremental Refresh**: Configure for handling larger datasets efficiently

### Skills to Explore
- Power BI Service deployment and sharing
- Integration with Excel for dynamic reporting
- Custom visuals from AppSource
- Power Automate for scheduled refreshes
- Python/R integration for advanced analytics

## 📊 Key Insights

- **Revenue Growth**: Track Nike's revenue trajectory from 2017 to 2021
- **Margin Analysis**: Gross margin consistently maintained around 45-46%
- **Regional Performance**: Identify top-performing geographic regions
- **Product Mix**: Understand revenue split between product categories
- **Cost Structure**: Analyze operating expense trends and efficiency metrics

## 🔄 Data Transformation & Cleaning

### Key Transformation Steps

#### 1. Date Dimension Creation
- Generated continuous date table covering entire analysis period (2017-2021)
- Created calculated columns for Month, Quarter, and Year
- Established proper date hierarchies for drill-down analysis
- Marked as date table for time intelligence functions

#### 2. P&L Hierarchy Structure
- Designed LineOrder column to maintain proper P&L sequence
- Created LineItemType classification system:
  - Type 1: Base values (Revenue, COGS, Operating Expenses)
  - Type 2: Calculated subtotals (Gross Margin, EBITDA, EBIT, Net Income)
  - Type 3: Percentage calculations (Gross Margin %)
- Implemented LineItemDesc for display logic control

#### 3. Account Dimension Cleansing
- Standardized account descriptions for consistency
- Created category groupings for aggregation
- Mapped account numbers to appropriate P&L line items
- Validated account number uniqueness

#### 4. Store Dimension Enhancement
- Geocoded store locations to regions (Northeast, South, Midwest, West, Southeast)
- Standardized city and state naming conventions
- Assigned store type classifications
- Created unique StoreID identifiers

#### 5. Fact Table Processing
- Validated transaction amount data types
- Ensured proper date formatting for relationships
- Handled null values in transaction descriptions
- Verified referential integrity with dimension tables

#### 6. Relationship Optimization
- Established one-to-many cardinality from dimensions to fact
- Set single-direction cross-filtering for performance
- Validated relationship keys for data integrity
- Optimized for star schema query patterns

### Data Quality Checks
- ✅ No orphaned records in fact table
- ✅ All dates within expected range (2017-2021)
- ✅ Transaction amounts validated (no unexpected nulls)
- ✅ Store and account references properly mapped
- ✅ P&L line items maintain proper hierarchy sequence

## 📚 Skills Demonstrated

### Data Modeling
- ⭐ **Star Schema Design**: Implemented industry-standard dimensional modeling
- ⭐ **Relationship Management**: Configured proper cardinality and cross-filtering
- ⭐ **Data Normalization**: Separated dimensions from facts for optimal querying
- ⭐ **Hierarchies**: Built drill-down structures for date and P&L line items

### DAX Expertise
- ⭐ **Context Transition**: Using CALCULATE with filter modifications
- ⭐ **SWITCH Functions**: Complex conditional logic for dynamic calculations
- ⭐ **ALL & FILTER**: Manipulating filter context for subtotal logic
- ⭐ **SELECTEDVALUE**: Single-value extraction for branching logic
- ⭐ **Aggregation Functions**: SUM, FORMAT, and mathematical operations
- ⭐ **Running Totals**: Implementing cascading P&L subtotal calculations
- ⭐ **Variables**: Using VAR for code readability and performance

### Data Transformation
- ⭐ **ETL Processes**: Raw data cleaning and structuring
- ⭐ **Calculated Columns**: Creating derived attributes in dimension tables
- ⭐ **Data Type Management**: Proper formatting for dates, currency, and text
- ⭐ **Quality Assurance**: Validation checks for data integrity
- ⭐ **Date Intelligence**: Calendar table creation and time-based hierarchies

### Financial Analysis
- ⭐ **P&L Statement Construction**: Building hierarchical income statements
- ⭐ **Financial KPIs**: Revenue, COGS, margins, EBITDA, net income
- ⭐ **Vertical Analysis**: Percentage of revenue calculations
- ⭐ **Profitability Metrics**: Gross margin and operating margin analysis
- ⭐ **Trend Analysis**: Multi-year financial performance tracking

### Visualization & Design
- ⭐ **Dashboard Layout**: Creating intuitive and professional interfaces
- ⭐ **Conditional Formatting**: Dynamic styling based on measure values
- ⭐ **Interactive Elements**: Slicers, filters, and drill-through capabilities
- ⭐ **Chart Selection**: Choosing appropriate visualizations for data types
- ⭐ **Color Theory**: Consistent branding and visual hierarchy
- ⭐ **User Experience**: Designing for ease of navigation and insights discovery

### Business Intelligence
- ⭐ **Requirements Gathering**: Translating business needs to technical solutions
- ⭐ **KPI Development**: Defining and implementing key performance indicators
- ⭐ **Documentation**: Creating comprehensive technical and user documentation
- ⭐ **Performance Optimization**: Query efficiency and dashboard responsiveness

## 🙏 Acknowledgments

- Case study provided by **Career Principles**
- Project created for **educational and portfolio demonstration purposes**
- Data is **fictional** and does not represent actual Nike financial information
- Power BI community for best practices and inspiration
- Focus on demonstrating **P&L statement construction**, **data manipulation**, and **data cleaning techniques**

---
