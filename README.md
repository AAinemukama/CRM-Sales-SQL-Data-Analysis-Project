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
### Deals Still Not Concluded by Each Team
--Which team still has the most deals still not concluded?
```
select manager, deal_stage, count(deal_stage) as count
from SALES_pip
where deal_stage like 'Engaging' and close_date is null
group by manager,deal_stage 
order by count desc
```
--Which team has the highest number of prospects?
select manager, deal_stage, count(deal_stage) as count
from SALES_pip
where deal_stage like 'Prospecting' and close_date is null and engage_date is null
group by manager,deal_stage 
order by count desc

-- now we want to look at the above numbers side by side
select manager, deal_stage, count(deal_stage) as count
from SALES_pip
group by manager,deal_stage
order by manager, count desc

---What product type has had the most successfull(won) deaals?
select product, count(product) as Count
from SALES_pip
where deal_stage like 'Won' and close_date is not null
group by product
order by count desc

--number of won product sales opportunities of each sales manager	
select manager, product, count(product) as count
from SALES_pip
where engage_date is not null and close_date is not null and deal_stage like 'Won'
group by manager, product
order by manager,count  desc

select * from SALES_pip

--How many sales were closed above the sales price per team?
select manager, count(close_value) as count
from SALES_pip
where sales_price < close_value and close_date is not null
group by manager
order by count desc

--How many sales were closed below sales perice for each team?
select manager, count(close_value) as count
from SALES_pip
where sales_price > close_value and close_date is not null
group by manager
order by count desc

--How has the count of won deals changed over time?
select engage_date, count(deal_stage) as count
from SALES_pip
where close_date is not null and deal_stage like 'Won'
group by engage_date, deal_stage
order by engage_date

--How has the count of lost deals changed overtime?
select engage_date, count(deal_stage) as count
from SALES_pip
where close_date is not null and deal_stage like 'Lost'
group by engage_date, deal_stage
order by engage_date

---Which regional office generated the highest revenue?
select regional_office, sum(close_value) as Revenue
from SALES_pip
where close_date is not null
group by regional_office
order by Revenue desc

--which teams generated the most revenue within each regional office?

select regional_office,manager, sum(close_value) as Revenue
from SALES_pip
where close_date is not null
group by regional_office,manager
order by Revenue desc

--Who are the top 5 sales agents overall?
select top 5 sales_agent, regional_office, manager, sum(close_value) as Revenue
from SALES_pip
where close_date is not null	
group by sales_agent,regional_office,manager
order by Revenue desc


--Who are the top 5 sales agents who had a close value higher than the sales price?
select top 5 sales_agent, regional_office, manager, sum(close_value) as Revenue
from SALES_pip
where close_date is not null and sales_price < close_value	
group by sales_agent,regional_office,manager
order by Revenue desc

---What is the general performance of each sales agent? How much Revenue was brough in by each agent?
select sales_agent, regional_office, manager, sum(close_value) as Revenue
from SALES_pip
where close_date is not null	
group by sales_agent,regional_office,manager
order by Revenue desc
