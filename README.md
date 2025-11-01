
# 🧩 Employee Performance Grading ETL Pipeline

## 📘 Overview

This project automates **employee performance grading** using data from multiple HR and productivity systems. It extracts data from multiple sources (attendance, JIRA, PTO, master, and suppression lists), transforms it according to defined logic, and produces final grading and negative performance reports.

The ETL pipeline helps **managers and analysts** evaluate employee productivity and office attendance patterns in a consistent, data-driven manner.

---

## 📂 Data Sources

| Table Name              | Description                                | Key Columns                                                                        |
| ----------------------- | ------------------------------------------ | ---------------------------------------------------------------------------------- |
| **Employee_Attendance** | Daily entry and exit logs                  | `Emp_Id`, `Emp_Name`, `Entry`, `Exit`                                              |
| **Employee_Jira**       | JIRA performance metrics                   | `Emp_Id`, `Emp_Name`, `Jira_Points`                                                |
| **Employee_PTO**        | Employee Paid Time Off records             | `Emp_Id`, `Emp_Name`, `Date`                                                       |
| **Employee_Master**     | Master employee metadata                   | `Emp_Id`, `Emp_Name`, `Emp_Manager`, `Emp_Lan`, `Emp_Age`, `Emp_Gender`, `Emp_Exp` |
| **Employee_Supression** | List of employees exempted from evaluation | `Emp_Id`, `Emp_Name`, `Emp_Manager`, `Approver`, `Status`                          |

---

## 🧮 Transformation & Logic Design

### 1. **RTO (Return to Office) Calculation**

* `RTO = Number of in-office days per week`
* **Condition:**

  * `RTO >= 3` → Acceptable
  * `RTO < 3` → Negative flag

### 2. **Score Calculation**

* If `RTO >= 3` → RTO Condition Score = 1
* Otherwise → RTO Condition Score = 0
* **Final Score** is rounded to **2 decimal places**

### 3. **Negative Table Conditions**

Employees are listed in **Negative_Table** if:

* `(Jira_Points / 200 * 100) <= 50`, **or**
* `RTO < 3`

🛑 **Exception:** Employees present in the `Employee_Supression` table are **excluded** from Negative_Table.

---

## 📊 Output Tables

### 🔹 **Negative_Table**

| Column        | Description             |
| ------------- | ----------------------- |
| `Emp_Id`      | Employee ID             |
| `Emp_Name`    | Employee Name           |
| `Emp_Manager` | Reporting Manager       |
| `Jira_Points` | Employee's JIRA Points  |
| `RTO`         | Days in office per week |

**Criteria:**
Includes employees with poor RTO or low Jira performance, excluding suppressed employees.

---

### 🔹 **Grade_Table**

| Column        | Description                      |
| ------------- | -------------------------------- |
| `Emp_Id`      | Employee ID                      |
| `Emp_Name`    | Employee Name                    |
| `Emp_Manager` | Reporting Manager                |
| `Jira_Points` | Employee's JIRA Points           |
| `RTO`         | Days in office per week          |
| `Grade_Score` | Computed final performance score |

**Formula:**
`Grade_Score = (RTO Score + (Jira_Points/200 * 100))`

---

## ⚙️ ETL Pipeline Stages

1. **Extract**

   * Pull data from:

     * SQL / CSV / API sources containing the above tables.
   * Load all datasets into a staging layer (e.g., Spark, Pandas, or SQL staging tables).

2. **Transform**

   * Clean and standardize employee identifiers (`Emp_Id`, `Emp_Name`).
   * Calculate RTO per employee using attendance data.
   * Compute total JIRA points from `Employee_Jira`.
   * Remove suppressed employees.
   * Apply grading and negative filters based on defined logic.

3. **Load**

   * Write transformed results into:

     * `Grade_Table` for overall performance metrics.
     * `Negative_Table` for flagged employees.

---

## 🧠 Business Insights

* **RTO vs Productivity:** Identify correlation between office attendance and Jira productivity.
* **Exception Handling:** Suppression mechanism allows fair evaluation during special cases (e.g., long-term PTO, approved remote work).
* **Automation-Ready:** Can be scheduled weekly using **AutoSys**, **Airflow**, or **Cron**.

---

## 🛠️ Tech Stack (Suggested)

* **Language:** Python (Pandas / PySpark)
* **Scheduler:** AutoSys / Apache Airflow
* **Storage:** SQL / Parquet / CSV
* **Visualization (Optional):** Power BI / Tableau / Streamlit Dashboard

---




