---
Date: 2022-02-09
Author: Jimmy Briggs <jimmy.briggs@jimbrig.com>
Source: medium
Link: https://medium.com/p/51740bedb4b6
Tags: ["#Type/Highlight/Article"]
Aliases: ["Data Warehouse Tips and Best Practices I Wish I Knew Earlier", "Data Warehouse Tips and Best Practices I Wish I Knew Earlier"]
---
# Data Warehouse Tips and Best Practices I Wish I Knew Earlier

## Metadata
- Author: [[Michael]]
- Full Title: Data Warehouse Tips and Best Practices I Wish I Knew Earlier
- Category: #Type/Highlight/Article
- URL: https://medium.com/p/51740bedb4b6

## Highlights
- A data warehouse is a type of data management system that is designed to enable and support business intelligence (BI) activities, especially analytics. Data warehouses are solely intended to perform queries and analysis and often contain large amounts of historical data. - Oracle.com
- Simple — Straight-forward structure, easy to understand
- Fast — Queries should return results quickly
- Flexible — Analysis from any angle and breakdown
- Fact and Dimension Tables
- Keys and Normalization
- Other Fact Table Best Practices
- Other Dimension Table Best Practices
- Data involved in calculations should be in fact tables, and data involved in constraints, groups, and labels should be in dimensional tables. 
  - The Data Warehouse Toolkit
- Table Grain
- The grain is the level of detail captured in a single row. In this case, it is a single transaction in a particular store.
- Grain Do’s and Don’ts
- Do determine the grain of the fact table early
- Grain definition is a critical step as it affects all design decisions afterward. Always get commitment from your stakeholders when choosing the grain, to prevent costly reimplementation.
- Do store the lowest-level grain (realistically)
- There may be a need to aggregate the grain at a higher level (to reduce storage space or analyze at higher levels). This is alright but make sure to keep a copy of the lowest level grain data in another table, to maintain the flexibility for lower-level analysis.
- Do not mix the grain
- Storing facts at different aggregation levels may lead to potential errors when fact table measures are aggregated. We can avoid this by educating users to apply certain filters, but this violates the “simple to use” principle
- Star Schema
- Natural Keys
- Natural keys are unique keys used by the operational system outside the data warehouse. They are usually long codes that contain certain business meanings that are understood by the operational side.
- Surrogate Keys
- Surrogate keys are unique keys (usually integers) generated by the database, mapped to one or many natural keys. It contains no business meaning and is created for the sole purpose of database joins.
- Benefits of Surrogate Keys
- Buffer from operational changes
  If a natural key is modified for whatever reason, the value need only be updated in a single row of the dimension table, instead of a potentially huge number of rows on the fact table.
- Reduced space and improved performance
  Natural keys are often string datatypes that take up significantly more space than integers. Joins on string columns are also less efficient. When fact tables get into the millions/billions, space savings and performance become significant.
- Handling of null values
  Natural keys may occasionally have no values (nulls) due to errors in the operational system or other special cases. It is bad practice to have null keys in a fact table because of inconsistent query behavior when users join and filter on these keys. This can be solved by reserving a surrogate key to represent these cases.
- Integrate multiple source systems
  Duplicate natural keys found in more than one source system can be handled simply by assigning individual surrogate keys to each duplicate key.
- Normalization is the restructuring of a database to reduce data redundancy and improve data integrity. When we upgraded to the fact table to use surrogate keys, we were normalizing the fact table.
- This second level of branching creates what is known as a Snowflake schema (you can imagine more and more sub-branches eventually forming a complex snowflake). This type of schema is not recommended for Data Warehouses due to:
- Increased complexity for users
- Increased difficulty of maintenance for developers
- Potential performance reduction (hinders database optimizers)
- Measures or measurements are numerical values that are mean to be aggregated (e.g. sum, average, min, max).
- Unlike dimension keys, having null-valued measures are standard practice. It is okay because they are handled properly by SQL aggregation functions.
- Designers are sometimes tempted to unpivot this table into a long format, where there is a measure type dimension. This was also my natural tendency when I was starting as a data analyst.
- Measures can be grouped into 3 categories:
- Additive
  These measures can be summed up across any dimension.
  e.g. quantity, amount
- Semi-Additive
  These measures can only be summed up across some dimensions. Usually it is the time dimension that cannot be used.
  e.g. account balances
- Non-Additive
  These measures cannot be summed across any dimension. These are usually things like ratios, distinct counts, summarizations (e.g. mean, median).
- In many ways, the data warehouse is only as good as the dimension attributes; the analytic power of the DW/BI environment is directly proportional to the quality and depth of the dimension attributes.
  - The Data Warehouse Toolkit
- Sometimes dimensions are not related so there is no obvious way to group them up. In these cases, we can implement a Junk Dimension. The analogy is like junk drawers, where we dump loose items into one place. Let's understand how a junk dimension works:
- Casual/Status dimensions
- Date-related dimensions
- Degenerate dimensions
- Role-playing dimensions
- Slowly Changing Dimensions — Dimensional attributes may change over time, but just overwriting the old attributes will prevent analysis of any historical data that rely on them. We can solve these through traditional or new-age methods.
- Fact Tables Variants: Transaction, Periodic and Accumulating Snapshot — In this guide, I have only shown the transactional type of fact table.
- Enterprise Data Warehouse Bus Architecture and Opportunity/Stakeholder Matrix — These are useful frameworks to organize multiple user requirements, making it easier to formulate the best data warehouse design. The book covers these topics in great detail.
- The best practices recommended are not meant to be taken as gospel, so do assess if they suit your particular project’s requirements. When in doubt, remember the guiding principles of simple, fast and flexible.
- I highly recommend reading The Data Warehouse Toolkit yourself. It includes many real-world case studies and goes into more detail on the concepts and techniques I have highlighted, as well as other advanced ones I did not cover.
- Unlike an operational implementation where business users have no choice but to use the new system, DW/BI usage is sometimes optional. Business users will embrace the DW/BI system if it is the simple and fast source for actionable information. — The Data Warehouse Toolkit