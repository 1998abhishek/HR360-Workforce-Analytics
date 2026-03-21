# HR360 — Workforce Intelligence Platform

## Overview

End-to-end HR analytics solution on
Microsoft Fabric analysing 1,470 employee
records to identify attrition drivers,
predict flight risk and quantify the
$9.5M annual cost of employee turnover.

## Architecture

Bronze (Raw CSV)
→ Silver (43 engineered columns)
→ Gold (4 business ready tables)
→ Semantic Model (Direct Lake)
→ Power BI Report (5 pages)

## Tech Stack

Microsoft Fabric | Power BI | DAX
Delta Lake | Dataflow Gen2
Medallion Architecture | Direct Lake
Row Level Security | Pipeline Automation

## Key Findings

* Overtime workers leave at 3× average
  → Estimated saving: $2.1M annually
* Sales attrition 20.6% — highest dept
* Reducing to 10% saves $4.3M per year
* $9.5M annual cost of attrition identified
* High risk employees identified by name

## Report Pages

Page 1: Executive Overview
Page 2: Attrition Deep Dive
Page 3: Risk Dashboard ← showpiece
Page 4: Compensation & Pay Equity
Page 5: Performance & Development

## DAX Measures (30 total)

Includes: RANKX, SWITCH(TRUE()) ,
Rolling 3M Average , What-If Parameters ,
Time Intelligence , Cost of Attrition

List of all 30 DAX Measures :

HEADCOUNT MEASURES

Measure 1 — Total Employees

Total Employees = 
COUNTROWS(gold_fact_employees)

Measure 2 — Active Employees

Active Employees = 
CALCULATE(
    COUNTROWS(gold_fact_employees),
    gold_fact_employees[Attrition] = "No"
)

Measure 3 — Total Attritions

Total Attritions = 
CALCULATE(
    COUNTROWS(gold_fact_employees),
    gold_fact_employees[Attrition] = "Yes"
)

Measure 4 — Total Job Roles

Total Job Roles = 
DISTINCTCOUNT(gold_fact_employees[Job_Role])

ATTRITION MEASURES

Measure 5 — Attrition Rate

Attrition Rate = 
DIVIDE(
    [Total Attritions],
    [Total Employees],
    0
)

Measure 6 — Attrition Rate %

Attrition Rate % = 
FORMAT([Attrition Rate], "0.00%")
Measure 7 — Attrition Rate PY
Attrition Rate PY = 
CALCULATE(
    [Attrition Rate],
    SAMEPERIODLASTYEAR(dim_date[Date])
)

Measure 8 — Attrition YoY Change

Attrition YoY Change = 
[Attrition Rate] - [Attrition Rate PY]
Measure 9 — Avg Tenure Leavers
Avg Tenure Leavers = 
CALCULATE(
    AVERAGE(gold_fact_employees[Years_At_Company]),
    gold_fact_employees[Attrition] = "Yes"
)

COMPENSATION MEASURES

Measure 10 — Avg Monthly Salary

Avg Monthly Salary = 
AVERAGE(gold_fact_employees[Monthly_Income])

Measure 11 — Avg Annual Salary

Avg Annual Salary = 
[Avg Monthly Salary] * 12

Measure 12 — Total Salary Cost

Total Salary Cost = 
SUMX(
    gold_fact_employees,
    gold_fact_employees[Monthly_Income] * 12
)

Measure 13 — Gender Pay Gap

Gender Pay Gap = 
VAR MaleSalary = 
    CALCULATE(
        AVERAGE(gold_fact_employees[Monthly_Income]),
        gold_fact_employees[Gender] = "Male"
    )
VAR FemaleSalary = 
    CALCULATE(
        AVERAGE(gold_fact_employees[Monthly_Income]),
        gold_fact_employees[Gender] = "Female"
    )
RETURN
    DIVIDE(
        MaleSalary - FemaleSalary,
        MaleSalary,
        0
    )

PERFORMANCE MEASURES

Measure 14 — Avg Performance Rating

Avg Performance Rating = 
AVERAGE(gold_fact_employees[Performance_Rating])
Measure 15 — High Performers
High Performers = 
CALCULATE(
    COUNTROWS(gold_fact_employees),
    gold_fact_employees[Performance_Rating] = 4
)

Measure 16 — High Performer %

High Performer % = 
DIVIDE(
    [High Performers],
    [Total Employees],
    0
)

RISK MEASURES

Measure 17 — High Risk Employees

High Risk Employees = 
CALCULATE(
    COUNTROWS(gold_attrition_risk),
    gold_attrition_risk[Risk_Category] = "High Risk"
)

Measure 18 — High Risk %

High Risk % = 
DIVIDE(
    [High Risk Employees],
    [Active Employees],
    0
)

SATISFACTION MEASURE

Measure 19 — Avg Job Satisfaction

Avg Job Satisfaction = 
AVERAGE(gold_fact_employees[Job_Satisfaction])

DYNAMIC MEASURE

Measure 20 — Last Refreshed

Last Refreshed = 
"Last refreshed: " & 
FORMAT(NOW(), "DD MMM YYYY HH:MM")

10 ADVANCED MEASURES

FINANCIAL MEASURES

Measure 21 — Cost of Attrition

Cost of Attrition = 
VAR AvgSalary = 
    AVERAGE(gold_fact_employees[Monthly_Income])
VAR AnnualSalary = 
    AvgSalary * 12
VAR ReplacementCost = 
    AnnualSalary * 0.5
RETURN
    [Total Attritions] * ReplacementCost
    
Measure 22 — Attrition Index

Attrition Index = 
VAR CompanyAvgAttrition = 
    CALCULATE(
        [Attrition Rate],
        ALL(gold_fact_employees)
    )
VAR CurrentAttrition = 
    [Attrition Rate]
RETURN
    ROUND(
        DIVIDE(
            CurrentAttrition,
            CompanyAvgAttrition,
            0
        ) * 100,
    0)

LABEL MEASURES

Measure 23 — Attrition Status Label

Attrition Status = 
VAR Rate = [Attrition Rate]
RETURN
    SWITCH(
        TRUE(),
        Rate > 0.20, "🔴 Critical — Above 20%",
        Rate > 0.15, "🟡 Warning — Above 15%",
        Rate > 0.10, "🟢 Acceptable — Above 10%",
                     "✅ Excellent — Below 10%"
    )
    
Measure 24 — Overtime Attrition Rate

Overtime Attrition Rate = 
CALCULATE(
    [Attrition Rate],
    gold_fact_employees[Over_Time] = "Yes"
)

Measure 25 — Dynamic Report Title

Dynamic Report Title = 
VAR SelectedDept = 
    SELECTEDVALUE(
        gold_fact_employees[Department],
        "All Departments"
    )
VAR SelectedGender = 
    SELECTEDVALUE(
        gold_fact_employees[Gender],
        "All Genders"
    )
RETURN
    "HR360 Analytics  |  " &
    SelectedDept & "  |  " &
    SelectedGender

RANKING MEASURE

Measure 26 — Dept Attrition Rank

Dept Attrition Rank = 
IF(
    ISINSCOPE(gold_fact_employees[Department]),
    RANKX(
        ALL(gold_fact_employees[Department]),
        [Attrition Rate],
        ,
        DESC,
        DENSE
    ),
    BLANK()
)

TIME INTELLIGENCE MEASURES

Measure 27 — Attrition YTD

Attrition YTD = 
TOTALYTD(
    [Total Attritions],
    dim_date[Date]
)

Measure 28 — Rolling 3M Attrition Rate

Rolling 3M Attrition Rate = 
VAR LastDate = 
    LASTDATE(dim_date[Date])
VAR DateRange = 
    DATESINPERIOD(
        dim_date[Date],
        LastDate,
        -3,
        MONTH
    )
RETURN
    CALCULATE(
        [Attrition Rate],
        DateRange
    )

WHAT-IF MEASURES

Measure 29 — Projected Attritions

Projected Attritions = 
ROUND(
    [Active Employees] *
    'Attrition_Scenario'[Attrition_Scenario],
    0
)

Measure 30 — Projected Cost Saving

Projected Cost Saving = 
VAR CurrentCost = 
    [Cost of Attrition]
VAR ProjectedAttritions = 
    [Projected Attritions]
VAR AvgAnnualSalary = 
    AVERAGE(gold_fact_employees[Monthly_Income]) * 12
VAR ProjectedCost = 
    ProjectedAttritions * AvgAnnualSalary * 0.5
RETURN
    IF(
        CurrentCost > ProjectedCost,
        CurrentCost - ProjectedCost,
        0
    )

## Live Report

[ https://app.powerbi.com/Redirect?action=OpenApp&appId=ecd74a22-0d27-435c-93a8-981f64e91d6d&ctid=fb29d8c3-7084-47da-a03f-f93b80073804&experience=power-bi ] ✅

## Row Level Security

Dynamic RLS using ( USERPRINCIPALNAME()
+ LOOKUPVALUE ) against dim_roles table
Department managers see only their data.

## Pipeline

Automated daily refresh at 7AM IST
Bronze → Silver → Gold sequence
Failure email notifications are configured.

## Contact

LinkedIn: [ www.linkedin.com/in/abhishek-b-85351a367 ]

______________________________________________________________________________________Thank you_______________________________________________________________________________________




