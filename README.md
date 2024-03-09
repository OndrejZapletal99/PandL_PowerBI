# P&L financial report PowerBI
- PowerBi:
  - PowerQuery
  - Dax
  - Visualisation
---

- ## Table of content
  - [1. Introduction](#1-introduction)
  - [2. Data import](#2-data-import)
  - [3. Tables and data modeling](#3-tables-and-data-modeling)
    - [3.1 Date table](#31-date-table)
    - [3.2 Parent/Child DimAccount](#32-parentchild-dimaccount)
    - [3.3 Parent/Child DimDepartmentGroup](#32-parentchild-dimaccount)
    - [3.4 Parent/Child DimOrganization](#34-parentchild-dimorganization)
    - [3.5 Amount FactFinance](#35-amount-factfinance)
    - [3.6 Data model](#36-data-model)


## 1. Introduction
- The AdventureWorks sample databases is open dataset with business data
- The original dataset is available to downloard [here](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms)
- For this report AdventureWorksDW2022 will be used.
## 2. Data import
- After uploading the database backup to SSMS, you can see many tables.
- The data could be prepared in SQL, but for the purpose of training it will be modified in PowerBI
- Use the SQL server connector to upload tables:
  - DimAccount
  - DimDepartmentGroup
  - DimOrganization
  - DimScenario
  - FactFinance
## 3. Tables and data modeling
### 3.1 Date table
- For almost every PowerBI data model is better to create a new Date/Calendar table.
```
Calendar table = 
VAR minDate =
    MIN ( FactFinance[Date])
VAR maxDate =
    MAX (FactFinance[Date] )
RETURN
    ADDCOLUMNS (
        CALENDAR ( minDate, maxDate ),
        "Year", YEAR ( [Date] ),
        "Month", FORMAT ( [Date], "MMMM" ),
        "Month number", MONTH ( [Date] ),
        "Calendar quarter", CONCATENATE ( "Q", QUARTER ( [Date] ) ),
        "Calendar week", WEEKNUM ( [Date], 2 ),
        "Day number", WEEKDAY ( [Date], 2 ),
        "Day", FORMAT ( [Date], "DDD" )
    )
```

- And for better connection between tables is better to create datekey column

```
Date INT = 
YEAR ( 'Date'[Date] ) * 10000
    + MONTH ( 'Date'[Date] ) * 100
    + DAY ( 'Date'[Date] )
```
### 3.2 Parent/Child DimAccount
1. First of all is necessary to create a Parent-cild path by DAX code:
```
Parent-child path =
PATH ( DimAccount[AccountKey], DimAccount[ParentAccountKey] )
```
2. Create all levels of hierarchy. In this case, there are 6 levels. Only the second value in the formula changes according to the level.
```
Level 1 = PATHITEM(DimAccount[Parent-child path],1,INTEGER)
```
3. Find name by level. Only the third value in the fomula changes according to the level.
 ```
Level 1 name =
LOOKUPVALUE (
    DimAccount[AccountDescription],
    DimAccount[AccountKey], DimAccount[Level 1]
)
```
4. Pathlength for P&L structure
```
Pathlenght = PATHLENGTH(DimAccount[Parent-child path])
```

### 3.3 Parent/Child DimDepartmentGroup
1. First of all is necessary to create a Parent-cild path by DAX code:
```
Parent-child path = PATH(DimDepartmentGroup[DepartmentGroupKey],DimDepartmentGroup[ParentDepartmentGroupKey])
```
1. Create all levels of hierarchy. In this case, there are 2 levels. Only the second value in the formula changes according to the level.
```
Level 1 =
PATHITEM ( DimDepartmentGroup[Parent-child path], 1, INTEGER )
```
1. Find name by level. Only the third value in the fomula changes according to the level.
```
Level 1 name =
LOOKUPVALUE (
    DimDepartmentGroup[DepartmentGroupName],
    DimDepartmentGroup[DepartmentGroupKey], DimDepartmentGroup[Level 1]
)
```
### 3.4 Parent/Child DimOrganization
1. First of all is necessary to create a Parent-cild path by DAX code:
```
Parent-child path = PATH(DimOrganization[OrganizationKey],DimOrganization[ParentOrganizationKey])
```
1. Create all levels of hierarchy. In this case, there are 4 levels. Only the second value in the formula changes according to the level.
```
Level 1 =
PATHITEM (DimOrganization[Parent-child path], 1, INTEGER )
```
1. Find name by level. Only the third value in the fomula changes according to the level.
```
Level 1 name = 
LOOKUPVALUE (
    DimOrganization[OrganizationName],
    DimOrganization[OrganizationKey], DimOrganization[Level 1]
)
```
### 3.5 Amount FactFinance
In this database all accounts have a plus sign-Column Amount in the Fact Finance table
- So two new columns must be created in the FactFinance table.

1. Operator sign to DimAccount table
``` 
Sign =
SWITCH (
    TRUE (),
    DimAccount[Operator] = "-", -1,
    DimAccount[AccountType] = "Expenditures", -1,
    1
)
``` 
1. Column with amnout and related sign
```
Amount with operator = RELATED(DimAccount[Sign]) * FactFinance[Amount]
```
### 3.6 Data model
![Data model](https://github.com/OndrejZapletal99/PandL_PowerBI/blob/main/Data_model.png)
## 4. P&L
- For simplicity, the adventureworks P&L structure will retained.
- The P&L structure is always similar but differs slightly in each company.
- In real financial controlling , another 1-2 tables would be added, in wich the P&L structure would be defined as well as which accounts belong where.
![First page]()

![Second page]()