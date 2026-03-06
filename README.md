# 📊 Sales Performance Analytics Dashboard

## 📊 Problem Statement
A retail organization lacked comprehensive visibility into sales performance across multiple channels, products, and categories. Management needed actionable insights to optimize sales strategies, identify growth opportunities, and make data-driven decisions to increase profitability and market share.

## 🗄️ Dataset Overview
**Source:** Retail sales transaction database  
**Size:** Multi-table relational database with sales, product, and customer data  
**Time Period:** 2021-2022 (2-year comparative analysis)  
**Geography:** Multi-region retail operations  

### Key Features:
- **Sales Transactions:** Revenue, profit margins, sales trends
- **Product Hierarchy:** Product categories and individual product performance
- **Channel Analysis:** Online, direct, and wholesaler sales channels
- **Temporal Data:** Monthly, quarterly, and yearly performance metrics
- **Customer Segments:** Different customer types and buying patterns

## 🔬 Approach & Methodology

### 1. Data Integration & Modeling
- **Multi-Source Integration:** Combined sales, product, and customer databases
- **Data Modeling:** Created star schema for optimal Power BI performance
- **KPI Framework:** Developed 20+ key performance indicators
- **Data Validation:** Ensured data quality and consistency across sources

### 2. Business Intelligence Development
- **Multi-Page Dashboard:** 4 focused analytical modules
- **Interactive Slicers:** Year, quarter, channel, and category filters
- **Drill-Down Capabilities:** From category to product-level analysis
- **Automated Refresh:** Scheduled data updates for real-time insights

### 3. Advanced Analytics Implementation
- **Year-over-Year Analysis:** Comparative performance metrics
- **Channel Mix Optimization:** Revenue contribution analysis
- **Product Performance:** Top-selling products and growth opportunities
- **Profitability Analysis:** Margin optimization insights

## 🛠️ Tools & Technologies

| Category | Tools | Purpose |
|----------|-------|---------|
| **Business Intelligence** | Power BI Desktop | Interactive dashboard development |
| **Data Modeling** | DAX, Power Query | Data transformation and measures |
| **Visualization** | Power BI Visuals | Charts, KPIs, and interactive elements |
| **Database** | SQL Server | Data source and query optimization |
| **Documentation** | Markdown | Technical documentation |

## 💡 Key Insights & Business Impact

### Revenue Growth Performance
- **Year-over-Year Growth:** 14% sales increase from 2021 to 2022
- **Channel Optimization:** Online channel contributes 52% of total revenue
- **Profit Margins:** 17% gross profit margin with optimization opportunities
- **Seasonal Patterns:** Identified peak sales periods for inventory planning

### Product & Category Analysis
- **Top Performers:** Category04 leads with $95K sales revenue
- **Product Leaders:** Product41 and Product30 are highest-selling items
- **Growth Opportunities:** Mid-performing products identified for bundling strategies
- **Category Performance:** Category02 and Category05 show strong contribution

### Strategic Business Recommendations
- **Channel Focus:** Optimize direct and wholesaler channels for growth
- **Product Strategy:** Develop bundling opportunities for mid-tier products
- **Pricing Optimization:** Review pricing strategy based on 17% margin analysis
- **Inventory Planning:** Use seasonal patterns for better stock management

## � Visualizations & Dashboards

### Dashboard Modules
1. **Overview Dashboard:** Total sales, profit, and KPI metrics
2. **Sales Performance:** Monthly trends and payment mode analysis
3. **Product Analysis:** Top-selling products with market share
4. **Category Analysis:** Category-wise performance and top performers

### Key Visual Elements
- **KPI Cards:** Revenue, profit, and margin indicators with trend arrows
- **Trend Analysis:** Year-over-year growth charts with monthly breakdowns
- **Channel Distribution:** Revenue contribution by sales channel
- **Product Rankings:** Top products with percentage share analysis

## 🚀 How to Run This Project

### Prerequisites
```bash
# Install Power BI Desktop (free version)
# Ensure SQL Server access for data source
# Clone repository for dashboard files
```

### Setup Instructions
```bash
# 1. Clone repository
git clone https://github.com/harshmeena9977-ops/sales-performance-powerbi-dashboard-
cd sales-performance-powerbi-dashboard-

# 2. Open Power BI Dashboard
# Open "Sales Performance Dashboard.pbix" in Power BI Desktop

# 3. Configure Data Source
# Update database connection settings in Power Query
# Refresh data to load latest information

# 4. Explore Interactive Features
# Use slicers for year, quarter, and channel filtering
# Navigate between 4 dashboard pages
# Export reports using built-in Power BI features
```

### Data Refresh Process
```sql
-- Sample SQL queries for data extraction
-- Monthly sales trends
SELECT 
    DATEPART(month, SaleDate) as Month,
    DATEPART(year, SaleDate) as Year,
    SUM(SalesAmount) as TotalSales,
    SUM(Profit) as TotalProfit
FROM SalesTransactions
GROUP BY DATEPART(month, SaleDate), DATEPART(year, SaleDate)
ORDER BY Year, Month;

-- Channel performance
SELECT 
    SalesChannel,
    SUM(SalesAmount) as Revenue,
    COUNT(*) as TransactionCount,
    SUM(SalesAmount) * 100.0 / SUM(SUM(SalesAmount)) OVER () as RevenueShare
FROM SalesTransactions
GROUP BY SalesChannel
ORDER BY Revenue DESC;
```

## 📋 Project Structure
```
├── README.md                              # This file
├── Sales Performance Dashboard.pbix       # Main Power BI dashboard
├── Overview.png.png                       # Dashboard overview screenshot
├── Sales-Performance.png.png              # Sales performance analysis
├── Product-Analysis.png.png               # Product performance view
├── Category-Analysis.png.png              # Category analysis view
└── business_insights.md.txt               # Detailed business insights
```

## 🏆 Resume Achievement

**Senior Data Analyst** | Sales Performance Analytics Dashboard  
*Developed comprehensive 4-page Power BI dashboard analyzing multi-channel sales performance, identifying 14% YoY growth opportunities and channel optimization strategies that enabled data-driven revenue growth decisions*

## 📞 Contact & Repository
- **GitHub:** [harshmeena9977-ops/sales-performance-powerbi-dashboard-](https://github.com/harshmeena9977-ops/sales-performance-powerbi-dashboard-)
- **Live Dashboard:** Available in Sales Performance Dashboard.pbix
- **Data Source:** Retail sales transaction database
- **Business Insights:** Detailed analysis in business_insights.md.txt

---

*Last Updated: March 2026 | Project Version: 2.0 (Recruiter-Ready)*
