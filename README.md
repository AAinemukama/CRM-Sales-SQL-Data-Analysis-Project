# Sales Team Performance Analysis
## Project Overview
This project aims to analyze sales team performance by examining the rate at which teams win or lose sales leads. Using SQL queries on a sales pipeline database, we investigate various aspects of sales team performance, including revenue generation, deal closure rates, and regional performance.

## Project Key Objectives
- Analyze revenue generation by team and manager
- Compare won vs. lost deal rates for each team
- Evaluate ongoing deals and prospect numbers
- Assess product performance in successful deals
- Examine pricing strategies (deals closed above/below sales price)
- Track deal trends over time
- Compare regional office performance
- Identify top-performing sales agents
- Evaluate team efficiency in closing deals above sales price
  
## Data source
- Github

## Data Structure
The analysis is based on three main tables:

- products
- sales_pipeline
- sales_teams
These tables are joined to create a comprehensive SALES_pip table for analysis.

## Key Analyses

- Revenue by Team
- Won and Lost Deals by Manager
- Ongoing Deals and Prospects by Team
- Product Performance
- Pricing Strategy (Deals Closed Above/Below Sales Price)
- Deal Trends Over Time
- Regional Office Performance
- Top Performing Sales Agents

## SQL Queries used 
### Data Extraction
#### From Products Table
```sql
select * from products
```
#### From Salees Pipeline Table
```sql
select * from sales_pipeline
```
### From Sales Team Table
```sql
select * from sales_teams
```

### Data Joining and Preparation
#### Joining the 3 tables to create a comprehensive dataset for analysis.
```sql
select sp.*, st.*, p.*
from sales_pipeline sp
left join sales_teams st on sp.sales_agent=st.sales_agent
left join products p on sp.product=p.product
```

#### Creating a New Table (SALES_pip) for further manipulation
```sql
SELECT sp.*, st.sales_agent as salesagent_team, st.manager,st.regional_office, p.product as pdt, p.sales_price, p.series
INTO SALES_pip
FROM sales_pipeline sp
LEFT JOIN sales_teams st ON sp.sales_agent = st.sales_agent
LEFT JOIN products p ON sp.product = p.product
```
#### Removing Duplicate Columns
```sql
alter table sales_pip
drop column salesagent_team,pdt
```

#### Standardizing the dates
```sql
update SALES_pip
set engage_date = cast(engage_date as date)
```

### DATA ANALYSIS

#### Revenue collected by each team from won deals
```sql
select manager, sum(close_value) as Revenue
from SALES_pip
where close_date is not null and deal_stage like 'Won'
group by manager
order by Revenue desc
```

#### Sales Opportunities won by each sales Manager
--How many sales opportunities have been won for each sales manager? Whose team won the most deals?
```sql
select manager, deal_stage, count(deal_stage) as count
from SALES_pip
where deal_stage like 'Won' and close_date is not null
group by manager,deal_stage
order by count desc
```
#### Sales Opportunities Lost by Each Sales Manager
--whose team lost the most deals?
```sql
select manager, deal_stage, count(deal_stage) as count
from SALES_pip
where deal_stage like 'Lost' and close_date is not null
group by manager,deal_stage
order by count desc
```
#### Deals Still Not Concluded by Each Team
--Which team still has the most deals still not concluded?
```sql
select manager, deal_stage, count(deal_stage) as count
from SALES_pip
where deal_stage like 'Engaging' and close_date is null
group by manager,deal_stage 
order by count desc
```
#### Number of Prospects by Each Team
--Which team has the highest number of prospects?
```sql
select manager, deal_stage, count(deal_stage) as count
from SALES_pip
where deal_stage like 'Prospecting' and close_date is null and engage_date is null
group by manager,deal_stage 
order by count desc
```
#### Side-by-Side Comparison of Deal Stages
```sql
select manager, deal_stage, count(deal_stage) as count
from SALES_pip
group by manager,deal_stage
order by manager, count desc
```
#### Most Successful Product Types
--What product type has had the most successfull(won) deaals?
```sql
select product, count(product) as Count
from SALES_pip
where deal_stage like 'Won' and close_date is not null
group by product
order by count desc
```
#### Number of Won Product Sales Opportunities by Each Sales Manager
```sql
select manager, product, count(product) as count
from SALES_pip
where engage_date is not null and close_date is not null and deal_stage like 'Won'
group by manager, product
order by manager,count  desc
```

#### Sales Closed Above Sales Price per Team
--How many sales were closed above the sales price per team?
```sql
select manager, count(close_value) as count
from SALES_pip
where sales_price < close_value and close_date is not null
group by manager
order by count desc
```

#### Sales Closed Below Sales Price per Team
--How many sales were closed below sales perice for each team?
```sql
select manager, count(close_value) as count
from SALES_pip
where sales_price > close_value and close_date is not null
group by manager
order by count desc
```

#### Count of Won Deals Over Time
--How has the count of won deals changed over time?
```sql
select engage_date, count(deal_stage) as count
from SALES_pip
where close_date is not null and deal_stage like 'Won'
group by engage_date, deal_stage
order by engage_date
```

#### Count of Lost Deals Over Time
--How has the count of lost deals changed overtime?
```sql
select engage_date, count(deal_stage) as count
from SALES_pip
where close_date is not null and deal_stage like 'Lost'
group by engage_date, deal_stage
order by engage_date
```

#### Highest Revenue by Regional Office
---Which regional office generated the highest revenue?
```sql
select regional_office, sum(close_value) as Revenue
from SALES_pip
where close_date is not null
group by regional_office
order by Revenue desc
```

#### Highest Revenue by each team per Regional Office
--which teams generated the most revenue within each regional office?
```sql
select regional_office,manager, sum(close_value) as Revenue
from SALES_pip
where close_date is not null
group by regional_office,manager
order by Revenue desc
```

#### Top 5 Sales Agents Overall
--Who are the top 5 sales agents overall?
```sql
select top 5 sales_agent, regional_office, manager, sum(close_value) as Revenue
from SALES_pip
where close_date is not null	
group by sales_agent,regional_office,manager
order by Revenue desc
```
#### Top 5 Sales Agents with Close Value Higher than Sales Price
--Who are the top 5 sales agents who had a close value higher than the sales price?  It provides insights into the performance of individual sales agents and highlights those who have successfully negotiated higher deal values.
```sql
select top 5 sales_agent, regional_office, manager, sum(close_value) as Revenue
from SALES_pip
where close_date is not null and sales_price < close_value	
group by sales_agent,regional_office,manager
order by Revenue desc
```
Understanding which sales agents consistently close deals above the sales price can help in recognizing top performers, setting benchmarks for other agents, and developing targeted training programs to improve negotiation skills across the team.

#### General Performance of Each Sales Agent
---What is the general performance of each sales agent? How much Revenue was brough in by each agent? It provides insights into the contributions of individual sales agents to the overall sales performance.
```sql
select sales_agent, regional_office, manager, sum(close_value) as Revenue
from SALES_pip
where close_date is not null	
group by sales_agent,regional_office,manager
order by Revenue desc
```
