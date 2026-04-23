# Church's Chicken Payroll ETL
## Built with Microsoft Azure

---

## Project Overview
End-to-end payroll ETL pipeline
combining SynergySuite and FocusPOS
data sources into a unified Target Output
following Medallion Architecture

---

## Tech Stack
- Azure Data Lake Gen2
- Azure Data Factory
- ADF Data Flows
- Azure SQL Database
- Azure Key Vault
- Power BI Desktop
- GitHub

---

## Architecture
Medallion Architecture
Bronze → Silver → Gold

---

## Data Sources
1. SynergySuite Input
   Labor and payroll report

2. FocusPOS Input
   POS payroll report

---

## Target Output Columns
- ID
- Employee_name
- StoreID
- Week
- E_Regular_hours
- E_overtime_hours
- E_PTO_Pay_Hours
- E_Sick_Pay_Hours
- E_Back_Pay_Hours
- E_Back_Pay_Dollars
- E_Tips_Dollars
- week_end_date

---

## Key Challenges Solved
1. Hierarchical report parsing
2. Name format standardization
3. Date format normalization
4. Store ID mapping
5. Tips column consolidation
6. SSN exclusion PII
7. Week number derivation
8. Employee ID reconciliation

---

## Pipeline Schedule
Runs every Monday at 6 AM
Automated weekly payroll ETL

---

## Security
All credentials are stored in
Azure Key Vault
No passwords in code
No sensitive data in GitHub
