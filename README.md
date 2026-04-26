# 🍗 Church's Chicken Payroll ETL

![Azure](https://img.shields.io/badge/Azure-Data%20Factory-blue)
![Databricks](https://img.shields.io/badge/Apache-Spark-orange)
![Python](https://img.shields.io/badge/Python-3.10-green)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-yellow)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

**Built with Microsoft Azure**

End-to-end payroll ETL pipeline combining **SynergySuite** and **FocusPOS** data sources into a unified Target Output following **Medallion Architecture** (Bronze → Silver → Gold).

---

## 🛠️ Tech Stack

| Category | Technology |
|----------|-----------|
| Cloud Platform | Microsoft Azure |
| Data Storage | Azure Data Lake Gen2 |
| Pipeline Orchestration | Azure Data Factory |
| Data Processing | Azure Databricks (Python) |
| Secret Management | Azure Key Vault |
| Database | Azure SQL Database |
| Visualization | Power BI |
| Version Control | GitHub |
| Language | Python 3.10 |
| Libraries | pandas, azure-storage-blob, pymssql, openpyxl, logging |

---

## 🏗️ Architecture

**Medallion Architecture: Bronze → Silver → Gold**

```
┌─────────────────────────────────────────────────────────────┐
│                     SOURCE SYSTEMS                          │
│  SynergySuite (Google Sheets)    FocusPOS (Google Sheets)   │
└──────────────────┬──────────────────────┬───────────────────┘
                   │                      │
                   ▼                      ▼
┌─────────────────────────────────────────────────────────────┐
│              LANDING ZONE (Azure Blob Storage)              │
│         SynergySuite Input.xlsx | FocusPOS Input.xlsx       │
└──────────────────┬──────────────────────────────────────────┘
                   │  ADF Copy Activities
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                    BRONZE LAYER                             │
│                    Raw Excel files                          │
└──────────────────┬──────────────────────────────────────────┘
                   │  Databricks Notebooks 1 & 2
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                    SILVER LAYER                             │
│     SynergySuite_Silver.csv | FocusPOS_Silver.csv           │
└──────────────────┬──────────────────────────────────────────┘
                   │  Databricks Notebook 3
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                     GOLD LAYER                              │
│                  merged_payroll.csv                         │
└──────────────────┬──────────────────────────────────────────┘
                   │  ADF Copy Activity
                   ▼
┌─────────────────────────────────────────────────────────────┐
│              SERVING LAYER (Azure SQL Database)             │
│                  dbo.target_output                          │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                    VISUALIZATION                            │
│                Power BI Dashboard                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 📥 Data Sources

### 1. SynergySuite Input
Labor and payroll report containing employee shift data, hours worked, labor costs and tips across 6 store locations. Structure: Store → Employee → Position → Shift rows.

### 2. FocusPOS Input
POS payroll report containing employee time and attendance records with clock-in/out times across 1 store location. Structure: Store → Employee → Shift rows.

---

## 🎯 Target Output Columns

| Column | Type | Description |
|--------|------|-------------|
| ID | INT | Employee Payroll ID |
| Employee_name | VARCHAR(100) | Full employee name |
| StoreID | VARCHAR(100) | Store number |
| Week | INT | Relative week number (1 = first week of pay period) |
| E_Regular_hours | FLOAT | Regular hours worked (rounded to 2 decimals) |
| E_overtime_hours | FLOAT | Overtime hours worked (rounded to 2 decimals) |
| E_PTO_Pay_Hours | VARCHAR(20) | PTO hours (empty if not applicable) |
| E_Sick_Pay_Hours | VARCHAR(20) | Sick pay hours (empty if not applicable) |
| E_Back_Pay_Hours | VARCHAR(20) | Back pay hours (empty if not applicable) |
| E_Back_Pay_Dollars | VARCHAR(20) | Back pay amount (empty if not applicable) |
| E_Tips_Dollars | FLOAT | Tips earned (rounded to 2 decimals) |
| #week_end_date | VARCHAR(20) | Week ending date (M/D/YYYY format) |

---

## 🔄 Pipeline Flow

```
PL_Churchs_Payroll_ETL
│
├── [Copy]     Copy_Synergy_Landing_To_Bronze
├── [Copy]     Copy_Focus_Landing_To_Bronze
│                        ↓
├── [Notebook] Run_Synergy_Bronze_Silver
├── [Notebook] Run_Focus_Bronze_Silver
│                        ↓
├── [Notebook] Run_Silver_Merge
│                        ↓
└── [Copy]     Copy_Gold_To_SQL
```

---

## 🔑 Key Challenges Solved

1. **Hierarchical report parsing** — SynergySuite has nested Store → Employee → Position → Shift structure with no standard headers
2. **Name format standardization** — Employee names normalized across both sources
3. **Date format normalization** — week_end_date formatted as M/D/YYYY
4. **Store ID mapping** — Extracted numeric store ID from full store name (e.g. FC3607 - S MAIN ST → 3607)
5. **Tips column consolidation** — Combined Non_Cash_Tips and Declared_Tips into single E_Tips_Dollars column
6. **SSN exclusion (PII)** — ID1 field (SSN) excluded, only ID2 (payroll ID) used
7. **Week number derivation** — Calculated relative week number (Week 1 = first week of pay period) from shift dates
8. **Employee ID reconciliation** — Removed system placeholder rows (Taker, Order with ID dash)
9. **Decimal precision** — Rounded Regular hours, OT hours and Tips to 2 decimal places
10. **FocusPOS OT calculation** — Split total hours into Regular (capped at 8 hrs/shift) and OT (hours over 8) since FocusPOS has no separate OT column
11. **Auto column detection** — Built dynamic column position discovery that automatically maps Excel columns without hardcoding positions, making the pipeline resilient to source format changes

---

## ⏰ Pipeline Schedule

The pipeline is designed to support automated scheduling. If connected to live data sources, it can be triggered every **Monday at 6:00 AM** using ADF schedule trigger `TR_Churchs_Payroll_Weekly` to process the previous week's payroll automatically.

---

## 🔐 Security

- All credentials stored in **Azure Key Vault**
- Databricks secret scope linked to Key Vault
- No passwords or keys hardcoded in any notebook or pipeline
- Storage account key accessed via `dbutils.secrets.get()`
- No sensitive data committed to GitHub

---

## 📊 Live Dashboard

📈 **[View Live Power BI Dashboard](https://app.powerbi.com/links/SAjRehjDjm?ctid=593fa49a-b093-407c-9f67-1f50b77cb432&pbi_source=linkShare&bookmarkGuid=026d33f5-d24f-4118-bc5c-621927cda3dc)**

---

## 📂 Repository Structure

```
churchs-chicken-payroll-etl/
│
├── 📁 notebooks/                          
│   ├── synergysuite_bronze_silver.ipynb   # Bronze → Silver (SynergySuite)
│   ├── focuspos_bronze_silver.ipynb       # Bronze → Silver (FocusPOS)
│   └── silver_merge.ipynb                # Silver → Gold merge
│
├── 📁 pipeline/                           
│   └── PL_Churchs_Payroll_ETL.json
│
├── 📁 dataset/                            
│
├── 📁 linkedService/                      
│
├── 📁 dashboard/                          
│   ├── Churchs_Payroll_Dashboard.pdf
│   └── README.md
│
├── README.md
└── publish_config.json
```

---

## 🚀 How to Run

1. Ensure Databricks cluster **payroll-cluster** is running
2. Upload source Excel files to `landing` container in `churchspayrollstorage`:
   - `SynergySuite Input.xlsx`
   - `FocusPOS Input.xlsx`
3. Trigger ADF pipeline `PL_Churchs_Payroll_ETL`
4. Pipeline automatically copies files to bronze, runs ETL notebooks, merges silver to gold, and loads to SQL
5. Refresh the Power BI dashboard

---

## 👤 Author

**Ajay Kumar Reddy Poreddy**
MS Data Science — University of Houston, Clear Lake
🔗 [GitHub](https://github.com/reddy-rbg)

---

## 📄 License

This project is proprietary and intended for portfolio and demonstration purposes only.
Unauthorized copying, modification, or distribution of this code is not permitted.
© 2026 Ajay Kumar Reddy Poreddy. All rights reserved.# 🍗 Church's Chicken Payroll ETL

![Azure](https://img.shields.io/badge/Azure-Data%20Factory-blue)
![Databricks](https://img.shields.io/badge/Apache-Spark-orange)
![Python](https://img.shields.io/badge/Python-3.10-green)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-yellow)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

**Built with Microsoft Azure**

End-to-end payroll ETL pipeline combining **SynergySuite** and **FocusPOS** data sources into a unified Target Output following **Medallion Architecture** (Bronze → Silver → Gold).

---

## 🛠️ Tech Stack

| Category | Technology |
|----------|-----------|
| Cloud Platform | Microsoft Azure |
| Data Storage | Azure Data Lake Gen2 |
| Pipeline Orchestration | Azure Data Factory |
| Data Processing | Azure Databricks (Python) |
| Secret Management | Azure Key Vault |
| Database | Azure SQL Database |
| Visualization | Power BI |
| Version Control | GitHub |
| Language | Python 3.10 |
| Libraries | pandas, azure-storage-blob, pymssql, openpyxl |

---

## 🏗️ Architecture

**Medallion Architecture: Bronze → Silver → Gold**

```
┌─────────────────────────────────────────────────────────────┐
│                     SOURCE SYSTEMS                          │
│  SynergySuite (Google Sheets)    FocusPOS (Google Sheets)   │
└──────────────────┬──────────────────────┬───────────────────┘
                   │                      │
                   ▼                      ▼
┌─────────────────────────────────────────────────────────────┐
│              LANDING ZONE (Azure Blob Storage)              │
│         SynergySuite Input.xlsx | FocusPOS Input.xlsx       │
└──────────────────┬──────────────────────────────────────────┘
                   │  ADF Copy Activities
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                    BRONZE LAYER                             │
│                    Raw Excel files                          │
└──────────────────┬──────────────────────────────────────────┘
                   │  Databricks Notebooks 1 & 2
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                    SILVER LAYER                             │
│     SynergySuite_Silver.csv | FocusPOS_Silver.csv           │
└──────────────────┬──────────────────────────────────────────┘
                   │  Databricks Notebook 3
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                     GOLD LAYER                              │
│                  merged_payroll.csv                         │
└──────────────────┬──────────────────────────────────────────┘
                   │  ADF Copy Activity
                   ▼
┌─────────────────────────────────────────────────────────────┐
│              SERVING LAYER (Azure SQL Database)             │
│                  dbo.target_output                          │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                    VISUALIZATION                            │
│                Power BI Dashboard                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 📥 Data Sources

### 1. SynergySuite Input
Labor and payroll report containing employee shift data, hours worked, labor costs and tips across 6 store locations. Structure: Store → Employee → Position → Shift rows.

### 2. FocusPOS Input
POS payroll report containing employee time and attendance records with clock-in/out times across 1 store location. Structure: Store → Employee → Shift rows.

---

## 🎯 Target Output Columns

| Column | Type | Description |
|--------|------|-------------|
| ID | INT | Employee Payroll ID |
| Employee_name | VARCHAR(100) | Full employee name |
| StoreID | VARCHAR(100) | Store number |
| Week | INT | Relative week number (1 = first week of pay period) |
| E_Regular_hours | FLOAT | Regular hours worked (rounded to 2 decimals) |
| E_overtime_hours | FLOAT | Overtime hours worked (rounded to 2 decimals) |
| E_PTO_Pay_Hours | VARCHAR(20) | PTO hours (empty if not applicable) |
| E_Sick_Pay_Hours | VARCHAR(20) | Sick pay hours (empty if not applicable) |
| E_Back_Pay_Hours | VARCHAR(20) | Back pay hours (empty if not applicable) |
| E_Back_Pay_Dollars | VARCHAR(20) | Back pay amount (empty if not applicable) |
| E_Tips_Dollars | FLOAT | Tips earned (rounded to 2 decimals) |
| #week_end_date | VARCHAR(20) | Week ending date (M/D/YYYY format) |

---

## 🔄 Pipeline Flow

```
PL_Churchs_Payroll_ETL
│
├── [Copy]     Copy_Synergy_Landing_To_Bronze
├── [Copy]     Copy_Focus_Landing_To_Bronze
│                        ↓
├── [Notebook] Run_Synergy_Bronze_Silver
├── [Notebook] Run_Focus_Bronze_Silver
│                        ↓
├── [Notebook] Run_Silver_Merge
│                        ↓
└── [Copy]     Copy_Gold_To_SQL
```

---

## 🔑 Key Challenges Solved

1. **Hierarchical report parsing** — SynergySuite has nested Store → Employee → Position → Shift structure with no standard headers
2. **Name format standardization** — Employee names normalized across both sources
3. **Date format normalization** — week_end_date formatted as M/D/YYYY
4. **Store ID mapping** — Extracted numeric store ID from full store name (e.g. FC3607 - S MAIN ST → 3607)
5. **Tips column consolidation** — Combined Non_Cash_Tips and Declared_Tips into single E_Tips_Dollars column
6. **SSN exclusion (PII)** — ID1 field (SSN) excluded, only ID2 (payroll ID) used
7. **Week number derivation** — Calculated relative week number (Week 1 = first week of pay period) from shift dates
8. **Employee ID reconciliation** — Removed system placeholder rows (Taker, Order with ID dash)
9. **Decimal precision** — Rounded Regular hours, OT hours and Tips to 2 decimal places
10. **FocusPOS OT calculation** — Split total hours into Regular (capped at 8 hrs/shift) and OT (hours over 8) since FocusPOS has no separate OT column

---

## ⏰ Pipeline Schedule

The pipeline is designed to support automated scheduling. If connected to live data sources, it can be triggered every **Monday at 6:00 AM** using ADF schedule trigger `TR_Churchs_Payroll_Weekly` to process the previous week's payroll automatically.

---

## 🔐 Security

- All credentials stored in **Azure Key Vault**
- Databricks secret scope linked to Key Vault
- No passwords or keys hardcoded in any notebook or pipeline
- Storage account key accessed via `dbutils.secrets.get()`
- No sensitive data committed to GitHub

---

## 📊 Live Dashboard

📈 **[View Live Power BI Dashboard](https://app.powerbi.com/links/SAjRehjDjm?ctid=593fa49a-b093-407c-9f67-1f50b77cb432&pbi_source=linkShare&bookmarkGuid=026d33f5-d24f-4118-bc5c-621927cda3dc)**

---

## 📂 Repository Structure

```
churchs-chicken-payroll-etl/
│
├── 📁 notebooks/                          
│   ├── synergysuite_bronze_silver.ipynb   
│   ├── focuspos_bronze_silver.ipynb       
│   └── silver_merge.ipynb                
│
├── 📁 pipeline/                           
│   └── PL_Churchs_Payroll_ETL.json
│
├── 📁 dataset/                            
│
├── 📁 linkedService/                      
│
├── 📁 dashboard/                          
│   ├── Churchs_Payroll_Dashboard.pdf
│   └── README.md
│
├── README.md
└── publish_config.json
```

---

## 🚀 How to Run

1. Upload source Excel files to the `landing` container in `churchspayrollstorage`
2. Trigger ADF pipeline `PL_Churchs_Payroll_ETL`
3. Pipeline automatically copies files to bronze, runs ETL notebooks, merges silver to gold, and loads to SQL
4. Refresh the Power BI dashboard

---

## 👤 Author

Ajay Kumar Reddy Poreddy

🔗 [GitHub](https://github.com/reddy-rbg)

---

## 📄 License

This project is proprietary and intended for portfolio and demonstration purposes only.
Unauthorized copying, modification, or distribution of this code is not permitted.
© 2026 Ajay Kumar Reddy Poreddy. All rights reserved.
