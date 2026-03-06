# Analysis Scripts Documentation

## Script Overview

This repository contains Power BI dashboard files and supporting documentation for sales performance analysis. While the primary analysis is conducted within Power BI, this section provides the underlying data queries and transformation logic.

## Data Extraction Scripts

### SQL Data Source Queries
```sql
-- Core Sales Data Extraction
SELECT 
    s.SaleID,
    s.SaleDate,
    s.SalesAmount,
    s.Profit,
    s.Quantity,
    s.SalesType,
    s.PaymentMode,
    p.ProductID,
    p.ProductName,
    p.CategoryID,
    c.CategoryName,
    ch.ChannelID,
    ch.ChannelName,
    cu.CustomerID,
    cu.CustomerType
FROM SalesTransactions s
JOIN Products p ON s.ProductID = p.ProductID
JOIN Categories c ON p.CategoryID = c.CategoryID
JOIN Channels ch ON s.ChannelID = ch.ChannelID
JOIN Customers cu ON s.CustomerID = cu.CustomerID
WHERE s.SaleDate BETWEEN '2021-01-01' AND '2022-12-31'
  AND s.SalesAmount > 0
  AND s.Profit IS NOT NULL
ORDER BY s.SaleDate;
```

### Monthly Performance Aggregation
```sql
-- Monthly Sales Performance
SELECT 
    DATEPART(YEAR, SaleDate) AS SaleYear,
    DATEPART(MONTH, SaleDate) AS SaleMonth,
    DATENAME(MONTH, SaleDate) AS MonthName,
    ChannelName,
    COUNT(*) AS TransactionCount,
    SUM(SalesAmount) AS TotalSales,
    SUM(Profit) AS TotalProfit,
    AVG(SalesAmount) AS AvgTransactionValue,
    SUM(Quantity) AS TotalQuantity,
    SUM(Profit) * 100.0 / SUM(SalesAmount) AS ProfitMargin
FROM SalesView
WHERE SaleDate BETWEEN '2021-01-01' AND '2022-12-31'
GROUP BY 
    DATEPART(YEAR, SaleDate),
    DATEPART(MONTH, SaleDate),
    DATENAME(MONTH, SaleDate),
    ChannelName
ORDER BY SaleYear, SaleMonth, ChannelName;
```

### Product Performance Analysis
```sql
-- Product Performance Ranking
WITH ProductMetrics AS (
    SELECT 
        p.ProductID,
        p.ProductName,
        c.CategoryName,
        COUNT(*) AS TransactionCount,
        SUM(s.SalesAmount) AS TotalSales,
        SUM(s.Profit) AS TotalProfit,
        SUM(s.Quantity) AS TotalQuantity,
        SUM(s.Profit) * 100.0 / SUM(s.SalesAmount) AS ProfitMargin,
        SUM(s.SalesAmount) * 100.0 / SUM(SUM(s.SalesAmount)) OVER () AS RevenueShare
    FROM SalesTransactions s
    JOIN Products p ON s.ProductID = p.ProductID
    JOIN Categories c ON p.CategoryID = c.CategoryID
    WHERE s.SaleDate BETWEEN '2021-01-01' AND '2022-12-31'
    GROUP BY p.ProductID, p.ProductName, c.CategoryName
)
SELECT 
    *,
    RANK() OVER (ORDER BY TotalSales DESC) AS SalesRank,
    RANK() OVER (ORDER BY TotalProfit DESC) AS ProfitRank,
    RANK() OVER (ORDER BY ProfitMargin DESC) AS MarginRank
FROM ProductMetrics
ORDER BY TotalSales DESC;
```

### Category Analysis Query
```sql
-- Category Performance Summary
SELECT 
    c.CategoryID,
    c.CategoryName,
    COUNT(DISTINCT p.ProductID) AS ProductCount,
    COUNT(*) AS TransactionCount,
    SUM(s.SalesAmount) AS TotalSales,
    SUM(s.Profit) AS TotalProfit,
    AVG(s.SalesAmount) AS AvgTransactionValue,
    SUM(s.Profit) * 100.0 / SUM(s.SalesAmount) AS ProfitMargin,
    SUM(s.SalesAmount) * 100.0 / SUM(SUM(s.SalesAmount)) OVER () AS RevenueShare,
    -- Year-over-Year Growth
    SUM(CASE WHEN YEAR(s.SaleDate) = 2022 THEN s.SalesAmount ELSE 0 END) AS Sales2022,
    SUM(CASE WHEN YEAR(s.SaleDate) = 2021 THEN s.SalesAmount ELSE 0 END) AS Sales2021,
    (SUM(CASE WHEN YEAR(s.SaleDate) = 2022 THEN s.SalesAmount ELSE 0 END) * 100.0 / 
     SUM(CASE WHEN YEAR(s.SaleDate) = 2021 THEN s.SalesAmount ELSE 0 END)) - 100 AS YoYGrowth
FROM SalesTransactions s
JOIN Products p ON s.ProductID = p.ProductID
JOIN Categories c ON p.CategoryID = c.CategoryID
WHERE s.SaleDate BETWEEN '2021-01-01' AND '2022-12-31'
GROUP BY c.CategoryID, c.CategoryName
ORDER BY TotalSales DESC;
```

## Power Query Transformations

### Data Cleaning Script
```powerquery
// Main Data Transformation
let
    Source = Sql.Database("ServerName", "DatabaseName"),
    SalesData = Source{[Schema="dbo",Item="SalesTransactions"]}[Data],
    
    // Step 1: Remove invalid records
    FilteredData = Table.SelectRows(SalesData, 
        each not Text.IsNull([SalesAmount]) and 
             not Text.IsNull([Profit]) and 
             [SalesAmount] > 0),
    
    // Step 2: Add calculated columns
    EnrichedData = Table.AddColumn(FilteredData, "ProfitMargin", each 
        [Profit] / [SalesAmount] * 100, type number),
    
    // Step 3: Date dimension
    DateEnriched = Table.AddColumn(EnrichedData, "Year", 
        each Date.Year([SaleDate]), type number),
    
    // Step 4: Quarter dimension
    QuarterEnriched = Table.AddColumn(DateEnriched, "Quarter", 
        each "Q" & Text.From(Date.QuarterOfYear([SaleDate])), type text),
    
    // Step 5: Month dimension
    MonthEnriched = Table.AddColumn(QuarterEnriched, "MonthName", 
        each Date.ToText([SaleDate], "MMMM"), type text),
    
    // Step 6: Date formatting
    FinalData = Table.AddColumn(MonthEnriched, "DateKey", 
        each Date.ToText([SaleDate], "yyyy-MM-dd"), type text)
in
    FinalData
```

### Product Hierarchy Script
```powerquery
// Product Category Hierarchy
let
    Source = Sql.Database("ServerName", "DatabaseName"),
    ProductData = Source{[Schema="dbo",Item="Products"]}[Data],
    
    // Add category information
    WithCategory = Table.NestedJoin(
        ProductData, 
        {"CategoryID"}, 
        Categories, 
        {"CategoryID"}, 
        "CategoryData",
        JoinKind.LeftOuter
    ),
    
    // Expand category information
    ExpandedCategory = Table.ExpandTableColumn(
        WithCategory, 
        "CategoryData", 
        {"CategoryName"}, 
        {"CategoryName"}
    ),
    
    // Add product performance metrics
    WithMetrics = Table.AddColumn(ExpandedCategory, "ProductRank", 
        each 0, type number),
    
    // Sort by product name
    SortedProducts = Table.Sort(WithMetrics, {{"ProductName", Order.Ascending}})
in
    SortedProducts
```

## Business Logic Implementation

### Revenue Recognition Logic
```sql
-- Revenue Recognition Rules
CREATE VIEW RevenueRecognition AS
SELECT 
    SaleID,
    SaleDate,
    SalesAmount,
    Profit,
    CASE 
        WHEN SalesType = 'Online' THEN 'Digital Revenue'
        WHEN SalesType = 'Retail' THEN 'Store Revenue'
        WHEN SalesType = 'Wholesale' THEN 'B2B Revenue'
        ELSE 'Other Revenue'
    END AS RevenueCategory,
    CASE 
        WHEN PaymentMode = 'Credit Card' THEN 'Electronic Payment'
        WHEN PaymentMode = 'Cash' THEN 'Cash Payment'
        WHEN PaymentMode = 'Bank Transfer' THEN 'Bank Payment'
        ELSE 'Other Payment'
    END AS PaymentCategory,
    -- Revenue classification for reporting
    CASE 
        WHEN SalesAmount > 1000 THEN 'High Value Transaction'
        WHEN SalesAmount > 500 THEN 'Medium Value Transaction'
        ELSE 'Standard Transaction'
    END AS TransactionClass
FROM SalesTransactions
WHERE SaleDate BETWEEN '2021-01-01' AND '2022-12-31';
```

### Channel Performance Logic
```sql
-- Channel Performance Metrics
CREATE VIEW ChannelPerformance AS
SELECT 
    ch.ChannelID,
    ch.ChannelName,
    COUNT(DISTINCT s.SaleID) AS TransactionCount,
    COUNT(DISTINCT s.CustomerID) AS UniqueCustomers,
    SUM(s.SalesAmount) AS TotalRevenue,
    SUM(s.Profit) AS TotalProfit,
    AVG(s.SalesAmount) AS AvgTransactionValue,
    SUM(s.Profit) * 100.0 / SUM(s.SalesAmount) AS ProfitMargin,
    -- Channel efficiency metrics
    SUM(s.SalesAmount) / COUNT(DISTINCT s.CustomerID) AS RevenuePerCustomer,
    SUM(s.Quantity) / COUNT(DISTINCT s.SaleID) AS AvgItemsPerTransaction,
    -- Growth metrics
    SUM(CASE WHEN YEAR(s.SaleDate) = 2022 THEN s.SalesAmount ELSE 0 END) AS Revenue2022,
    SUM(CASE WHEN YEAR(s.SaleDate) = 2021 THEN s.SalesAmount ELSE 0 END) AS Revenue2021,
    (SUM(CASE WHEN YEAR(s.SaleDate) = 2022 THEN s.SalesAmount ELSE 0 END) * 100.0 / 
     NULLIF(SUM(CASE WHEN YEAR(s.SaleDate) = 2021 THEN s.SalesAmount ELSE 0 END), 0)) - 100 AS YoYGrowth
FROM Channels ch
LEFT JOIN SalesTransactions s ON ch.ChannelID = s.ChannelID
WHERE s.SaleDate BETWEEN '2021-01-01' AND '2022-12-31'
GROUP BY ch.ChannelID, ch.ChannelName
ORDER BY TotalRevenue DESC;
```

## Data Quality Validation

### Quality Check Scripts
```sql
-- Data Quality Validation
SELECT 
    'Total Records' AS Metric,
    COUNT(*) AS Value
FROM SalesTransactions
WHERE SaleDate BETWEEN '2021-01-01' AND '2022-12-31'

UNION ALL

SELECT 
    'Records with Null Sales Amount' AS Metric,
    COUNT(*) AS Value
FROM SalesTransactions
WHERE SalesAmount IS NULL AND SaleDate BETWEEN '2021-01-01' AND '2022-12-31'

UNION ALL

SELECT 
    'Records with Negative Sales Amount' AS Metric,
    COUNT(*) AS Value
FROM SalesTransactions
WHERE SalesAmount < 0 AND SaleDate BETWEEN '2021-01-01' AND '2022-12-31'

UNION ALL

SELECT 
    'Records with Profit > Sales Amount' AS Metric,
    COUNT(*) AS Value
FROM SalesTransactions
WHERE Profit > SalesAmount AND SaleDate BETWEEN '2021-01-01' AND '2022-12-31';
```

### Performance Monitoring Queries
```sql
-- Dashboard Performance Metrics
DECLARE @StartDate DATE = '2021-01-01';
DECLARE @EndDate DATE = '2022-12-31';

-- Overall performance summary
SELECT 
    'Total Revenue' AS KPI,
    SUM(SalesAmount) AS Value,
    '$' + FORMAT(SUM(SalesAmount), 'N0') AS FormattedValue
FROM SalesTransactions
WHERE SaleDate BETWEEN @StartDate AND @EndDate

UNION ALL

SELECT 
    'Total Profit' AS KPI,
    SUM(Profit) AS Value,
    '$' + FORMAT(SUM(Profit), 'N0') AS FormattedValue
FROM SalesTransactions
WHERE SaleDate BETWEEN @StartDate AND @EndDate

UNION ALL

SELECT 
    'Profit Margin' AS KPI,
    AVG(Profit / SalesAmount) * 100 AS Value,
    FORMAT(AVG(Profit / SalesAmount) * 100, 'N1') + '%' AS FormattedValue
FROM SalesTransactions
WHERE SaleDate BETWEEN @StartDate AND @EndDate

UNION ALL

SELECT 
    'Total Transactions' AS KPI,
    COUNT(*) AS Value,
    FORMAT(COUNT(*), 'N0') AS FormattedValue
FROM SalesTransactions
WHERE SaleDate BETWEEN @StartDate AND @EndDate;
```

## Usage Instructions

### Running the Analysis
```bash
# 1. Set up database connection
# Configure SQL Server connection in Power BI

# 2. Execute data extraction queries
# Run the provided SQL scripts to extract data

# 3. Import data into Power BI
# Use Power Query to connect to database
# Apply transformation scripts

# 4. Build dashboard
# Use the provided DAX measures
# Create visuals according to specifications

# 5. Validate data
# Run quality check queries
# Verify dashboard accuracy
```

### Custom Analysis Template
```sql
-- Template for custom time period analysis
DECLARE @AnalysisStartDate DATE = '2022-01-01';
DECLARE @AnalysisEndDate DATE = '2022-03-31';

SELECT 
    -- Time dimensions
    DATEPART(YEAR, s.SaleDate) AS Year,
    DATEPART(QUARTER, s.SaleDate) AS Quarter,
    DATEPART(MONTH, s.SaleDate) AS Month,
    
    -- Channel dimensions
    ch.ChannelName,
    
    -- Performance metrics
    COUNT(*) AS TransactionCount,
    SUM(s.SalesAmount) AS TotalSales,
    SUM(s.Profit) AS TotalProfit,
    AVG(s.SalesAmount) AS AvgTransactionValue,
    SUM(s.Profit) * 100.0 / SUM(s.SalesAmount) AS ProfitMargin,
    
    -- Comparative metrics
    SUM(s.SalesAmount) * 100.0 / 
        (SELECT SUM(SalesAmount) FROM SalesTransactions 
         WHERE SaleDate BETWEEN @AnalysisStartDate AND @AnalysisEndDate) AS RevenueShare
FROM SalesTransactions s
JOIN Channels ch ON s.ChannelID = ch.ChannelID
WHERE s.SaleDate BETWEEN @AnalysisStartDate AND @AnalysisEndDate
GROUP BY 
    DATEPART(YEAR, s.SaleDate),
    DATEPART(QUARTER, s.SaleDate),
    DATEPART(MONTH, s.SaleDate),
    ch.ChannelName
ORDER BY Year, Quarter, Month, TotalSales DESC;
```

## Troubleshooting

### Common Issues and Solutions
1. **Data Connection Problems**
   - Verify SQL Server credentials
   - Check network connectivity
   - Validate database permissions

2. **Slow Performance**
   - Optimize SQL queries with proper indexes
   - Reduce data volume with appropriate filters
   - Review Power BI data model relationships

3. **Incorrect Calculations**
   - Validate DAX measure syntax
   - Check filter context in calculations
   - Verify data type compatibility

4. **Visual Display Issues**
   - Review data formatting settings
   - Check visual configuration
   - Validate data relationships

### Performance Optimization Tips
```sql
-- Index recommendations for optimal performance
CREATE INDEX IX_SalesTransactions_Date ON SalesTransactions(SaleDate);
CREATE INDEX IX_SalesTransactions_Channel ON SalesTransactions(ChannelID);
CREATE INDEX IX_SalesTransactions_Product ON SalesTransactions(ProductID);
CREATE INDEX IX_SalesTransactions_Customer ON SalesTransactions(CustomerID);

-- Partitioning strategy for large datasets
-- Consider partitioning by date for improved query performance
```

## Extension Ideas

### Advanced Analytics
- **Forecasting:** Implement time series forecasting for future sales
- **Cohort Analysis:** Customer cohort performance tracking
- **Market Basket Analysis:** Product association analysis
- **Customer Lifetime Value:** CLV calculation and segmentation

### Automation Opportunities
- **Automated Reporting:** Scheduled report generation
- **Alert System:** Performance threshold alerts
- **Data Refresh Automation:** Automated data pipeline
- **Dashboard Updates:** Dynamic content updates
