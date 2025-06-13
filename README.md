
# Loan Default Risk Analysis Dashboard (Power BI Project)

### Dashboard Link : https://app.powerbi.com/links/zZNKbPSJPV?ctid=8dd408f1-7776-48ac-ad65-fa5487f17aef&pbi_source=linkShare
## Problem Statement
In the financial sector, understanding patterns and factors leading to loan defaults is critical for minimizing risk and optimizing lending strategies. The objective of this project is to analyze a historical loan dataset to uncover insights into borrower profiles, loan characteristics, and demographic trends that contribute to loan defaults. The project also aims to visualize the year-over-year (YoY) trends, borrower risk segmentation, and financial performance using Power BI. These insights can help financial institutions identify high-risk applicants, improve credit assessment strategies, and enhance portfolio risk management.

---
##  Dataset Description
The dataset contains historical loan application records with the following features:

- **Demographics**: Age, Marital Status, Education, Has Dependents
- **Financials**: Income, Loan Amount, Credit Score, DTI Ratio
- **Employment**: Employment Type, Months Employed, Has Mortgage, Has Co-signer
- **Loan Info**: Loan Term, Interest Rate, Loan Purpose
- **Target Variable**: Default (True/False)
- **Date**: Loan Issued Date (`Loan Date (DD/MM/YYYY)`)

---

## Tools & Technologies

- **SQL Server**: Data storage
- **Power BI Dataflows Gen1**: ETL and transformation
- **Power BI Desktop**: Dashboard creation and DAX calculations
- **DAX Functions**: `CALCULATE()`, `FILTER()`, `PARALLELPERIOD()`, `SAMEPERIODLASTYEAR()`, `DIVIDE()`, `IF()`, `SWITCH()`, `SUMX()`, `AVERAGEX()`etc.

---

## Steps Followed
1. **Data Loading**:
   - Imported the dataset from SQL Server into Power BI using Dataflow Gen1.

2. **Data Transformation**:
   - Cleaned and transformed data using Power Query.
   - Parsed and reformatted `Loan Date (DD/MM/YYYY)` into proper Date format.
   - Created a custom **Date Table** and marked it as a date table in Power BI.

3. **Feature Engineering**:
   - Created custom age bins: Teenagers, Adults, Middle-Aged Adults, Senior Citizens.
   - Created Credit Score bins based on the credit score of each loan ID: Very Low, Low, Medium, High.
   - Created custom flags (e.g., High Credit Score, Has Mortgage, Has Dependents).

5. **Data Visualization**:
   - Designed and published 3 dashboards: Loan Overview, Demographics, and Risk Matrix.
   - Used visuals like Ribbon Charts, Decomposition Trees, Line Charts, and Stacked Columns.

6. **Analysis & Interpretation**:
   - Interpreted results to uncover borrower behavior, risk factors, and macro trends.


## Dashboards Created

### 1. Loan Default & Overview
This dashboard provides a high-level view of loan distribution and default trends.

Visuals Included:

- Loan Amount by Loan Purpose

- Average Income by Employment Type

- Default % by Employment Type

- Average Loan Amount by Age Group

- Default Rate by Year

![Image](https://github.com/user-attachments/assets/1b3a3235-6a62-4fe7-be3d-d50ad8ebbc37)

#### Feature Engineering:

- Age Grouping: Borrowers were segmented into four age-based categories:
   
           AgeGroup = IF(Loan_default[Age] <= 19, "Teenage",
           IF(Loan_default[Age] <= 39, "Adults",
           IF(Loan_default[Age] <= 59, "Middle Aged Adults", "Senior Citizens")))


- Credit Score Binning: Created a Credit Score bin based on the borrowers' credit scores.

         CreditScoreBins = IF(Loan_default[CreditScore]<=400, "Very Low", IF(Loan_default[CreditScore]<=450, "Low", 
         IF(Loan_default[CreditScore]<=650, "Medium", "High")))

- Income Bracket Classification: Created an Income Bracket for borrowers based on their annual income:

         Income Bracket = SWITCH(TRUE(), Loan_default[LoanAmount]<30000, "Low Income", 
         Loan_default[LoanAmount]>=30000 && Loan_default[LoanAmount]<60000, "Medium Income", 
         Loan_default[LoanAmount]>=60000, "High Income")

#### DAX Measures Used:

Used the following formulae to form the charts in the first Dashboard.

      -- Average Income by Employment Type
      AverageIncomeByEmployment = CALCULATE(AVERAGE(Loan_default[LoanAmount]),ALLEXCEPT(Loan_default, Loan_default[EmploymentType])) 


      -- Average Loan by Age Group
      AverageLoanByAge = AVERAGEX(VALUES(Loan_default[AgeGroup]), AVERAGE(Loan_default[LoanAmount]))


      -- Default Rate by Employment Type
      DefaultRateByEmploymentType = 
         VAR TotalRecordCount=COUNTROWS(ALL(Loan_default))
         VAR DefaultCases=COUNTROWS(FILTER(Loan_default,Loan_default[Default]=TRUE()))
      RETURN
         CALCULATE(DIVIDE(DefaultCases, TotalRecordCount), ALLEXCEPT(Loan_default, Loan_default[EmploymentType]))


      -- Default Rate by Year
      DefaultRateByYear = 
         VAR totalloans=CALCULATE(COUNTROWS(Loan_default),  ALLEXCEPT(Loan_default, Loan_default[Year]))
         VAR defaultLoans=CALCULATE(COUNTROWS(FILTER(Loan_default, Loan_default[Default]=TRUE())), ALLEXCEPT(Loan_default, Loan_default[Year]))
      RETURN 
         DIVIDE(defaultLoans, totalloans)


      -- Loan Amount by Purpose
      LoanAmountByPurpose = SUMX(FILTER(Loan_default, NOT(ISBLANK(Loan_default[LoanAmount]))), Loan_default[LoanAmount])
### 2. Loan Applicant Demographics & Profile
This dashboard explores borrower characteristics and how they influence loan trends.

#### Visuals Included:
- Median Loan by Credit Score Bins
- Avg Loan for High Credit by Age Group & Marital Status
- Total Loan of Adults vs Credit Score Bins
- Total Loan for Middle-Aged Adults by HasMortgage and HasDependents
- Total Loan by Education Type

![Image](https://github.com/user-attachments/assets/ad70b3d9-17d2-40a5-89f0-f7505dc0c42b)

#### DAX Measures Used:
      -- Average Loan for High Credit Borrowers
      AverageLoanAmount(HighCredit) = AVERAGEX(FILTER(Loan_default, Loan_default[CreditScoreBins]="High"), 
      AVERAGE(Loan_default[LoanAmount]))

      
      -- Median Loan by Credit Score Bins
      MedianByCreditScoreBins = MEDIANX(Loan_default, Loan_default[LoanAmount])
 
      
      -- Total Loan for Middle-Aged Adults
      TotalLoan(Middle-AgeAdults) = SUMX(FILTER(Loan_default, Loan_default[AgeGroup]="Middle Aged Adults"),
       Loan_default[LoanAmount])

      
      -- Total Loan for Adults by Credit Score Bin
      TotalLoanByCreditCategoryBins = CALCULATE(SUM(Loan_default[LoanAmount]), Loan_default[AgeGroup]="Adults", 
      ALLEXCEPT(Loan_default, Loan_default[CreditScoreBins], Loan_default[AgeGroup], Loan_default[Age], Loan_default[CreditScore]))

      -- Total Loans by Education Type
      TotalLoansByEducation = COUNTROWS(FILTER(Loan_default, NOT(ISBLANK(Loan_default[LoanID]))))

### 3. Financial Risk Matrix
This dashboard highlights financial risk indicators and trends across years and demographics.

#### Visuals Included:
- YoY Change in Total Loan Amount (%)
- YoY Change in Defaulted Loans (%)
- YTD Loan Amount by Credit Score & Marital Status (Ribbon Chart)
- Decomposition Tree: Total Loan by Income Bracket & Employment Type

![Image](https://github.com/user-attachments/assets/e011f10f-3bb8-4a18-8ead-4bca534576b0)


#### DAX Measures Used:
      -- YoY Change in Defaulted Loans
      YOY Default Loans Change = 
         VAR CY=CALCULATE(COUNTROWS(FILTER(Loan_default, Loan_default[Default]=TRUE())))
         VAR PY = CALCULATE(COUNTROWS(Loan_default), Loan_default[Default]=TRUE(),
                  PARALLELPERIOD(Loan_default[Loan_Date_DD_MM_YYYY].[Date], -1, YEAR))
      RETURN
         DIVIDE(CY-PY, PY,0)

      
      -- YoY Change in Loan Amount
      YOY_LoanAmountChange = 
         VAR CurrentYear=SUM(Loan_default[LoanAmount])
         VAR Previous_Year=CALCULATE(SUM(Loan_default[LoanAmount]), 
                           SAMEPERIODLASTYEAR(Loan_default[Loan_Date_DD_MM_YYYY]))
      RETURN
         DIVIDE((CurrentYear-Previous_Year), Previous_Year)


      -- YTD Loan Amount
      YTD LoanAmount = CALCULATE(SUM(Loan_default[LoanAmount]), 
                       DATESYTD(Loan_default[Loan_Date_DD_MM_YYYY].[Date]))


### Insights

- Loan Default & Overview
Loan Purpose Trends: Home loans and business loans accounted for the highest loan amounts, suggesting these are the primary reasons borrowers seek credit.
![Image](https://github.com/user-attachments/assets/906a4ba9-62fa-4401-a8b4-aff0392ace2a)

Income & Employment: Self-employed borrowers had lower average incomes but showed a moderately higher default percentage compared to salaried individuals.

Age-Based Risk: Senior citizens tend to take higher average loans, while Middle Aged adults and Adults showed a relatively higher default rate.

![Image](https://github.com/user-attachments/assets/8cb7e5d7-814e-447a-aa18-a0eea11526cc)

Yearly Default Pattern: An increasing trend in default rates was observed in the most recent years, indicating a rising risk exposure over time.

![Image](https://github.com/user-attachments/assets/999dd56f-3fb8-428e-8719-a13a51111b2c)

![Image](https://github.com/user-attachments/assets/65c5f7af-6d84-41b4-80ba-2560ce6b72e4)


## How to Use

Connect Power BI to SQL Server or use the `.pbix` file (shared).


## Future Improvements

- Integrate predictive models (Python, Azure ML) to forecast defaults.
- Build a risk scoring system based on historical trends.
- Add macroeconomic indicators like interest rate trends or unemployment.













     
