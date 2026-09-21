# 🏥 Healthcare Analytics Dashboard

**Turning patient records into actionable insights — a Power BI analytics project.**

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-Measures-0B3D3D?style=flat)
![Status](https://img.shields.io/badge/Status-Complete-1B998B?style=flat)

---

## 📌 Overview

Hospitals and clinics generate patient, appointment, treatment and billing data across disconnected systems. Without a unified view, it's hard to spot operational issues — like rising no-shows or a billing shortfall — in time to act.

This project consolidates **five operational datasets** into a single **Power BI** model with a governed star-schema design, DAX-driven KPIs, and a 3-page interactive report covering patient flow, doctor performance, and revenue health.

## 🎯 Objective

Build a relational Power BI data model connecting **Patients, Doctors, Appointments, Treatments and Billing** into a single source of truth — enabling DAX-driven KPIs and a 3-page interactive dashboard.

| Step | Description |
|---|---|
| **01 · Ingest** | Import 5 source tables — Patients, Doctors, Appointments, Treatments, Billing — covering 200 appointment records |
| **02 · Model** | Relate the tables into a star schema via `patient_id`, `doctor_id`, `appointment_id` and `treatment_id` |
| **03 · Report** | Build DAX measures and a 3-page report: Executive Dashboard, Doctors' Performance, and Financials |

## 🗂️ Dataset

**Range:** January 2023 – December 2023

| Table | Records | Description |
|---|---|---|
| `patients` | 50 rows | Demographics, registration date, insurance provider |
| `doctors` | 10 rows | Specialization, hospital branch, years of experience |
| `appointments` | 200 rows | Visit date/time, reason for visit, status *(fact table)* |
| `treatments` | 200 rows | Treatment type, cost, treatment date |
| `billing` | 200 rows | Bill amount, payment method, payment status |

Covers **3 hospital branches** (Central Hospital, Westside Clinic, Eastside Clinic) and **3 specializations** (Pediatrics, Dermatology, Oncology).

## 🧩 Data Model

`appointments` sits at the center as the fact table, linking `patients` and `doctors` dimensions and cascading into `treatments` → `billing` for a complete patient-to-payment view.

![Data Model](screenshots/data_model.png)

**Enrichment (calculated columns):** `Age`, `Age Group`, `Month` (appointment month), `Experience Level` (doctor tenure band) — added for segmentation and correct chronological/category sorting.

## 🧮 Key DAX Measures

<details>
<summary><strong>No-Show Rate</strong></summary>

```DAX
No Show Rate =
DIVIDE(
    CALCULATE(COUNTROWS(appointments), appointments[status] = "No-show"),
    COUNTROWS(appointments)
)
```
</details>

<details>
<summary><strong>Cancellation Rate</strong></summary>

```DAX
Cancellation Rate =
DIVIDE(
    CALCULATE(COUNTROWS(appointments), appointments[status] = "Cancelled"),
    COUNTROWS(appointments)
)
```
</details>

<details>
<summary><strong>Completion Rate</strong></summary>

```DAX
Completion Rate =
DIVIDE(
    CALCULATE(COUNTROWS(appointments), appointments[status] = "Completed"),
    COUNTROWS(appointments)
)
```
</details>

<details>
<summary><strong>Patient Retention</strong></summary>

```DAX
Patient Retention =
VAR RetainedPatients =
    CALCULATE(
        DISTINCTCOUNT(appointments[patient_id]),
        FILTER(
            VALUES(appointments[patient_id]),
            CALCULATE(COUNTROWS(appointments)) > 1
        )
    )
VAR TotalPatients = DISTINCTCOUNT(appointments[patient_id])
RETURN
    DIVIDE(RetainedPatients, TotalPatients)
```
</details>

<details>
<summary><strong>Total Billed / Total Collected / Collection Rate</strong></summary>

```DAX
Total Billed Amount = SUM(billing[amount])

Amount Collected =
CALCULATE(SUM(billing[amount]), billing[payment_status] = "Paid")

Collection Rate = DIVIDE([Amount Collected], [Total Billed Amount])
```
</details>

<details>
<summary><strong>Average Treatment Cost</strong></summary>

```DAX
Avg Treatment Cost = AVERAGE(treatments[cost])
```
</details>

## 📊 Dashboard Pages

Three report pages moving from patient-level detail to financial performance — **22 visuals in total**.

### 1️⃣ Executive Dashboard *(12 visuals)*

KPI cards, appointment-trend area chart, status pie & donut, and a Month slicer.

![Executive Dashboard](screenshots/executive_dashboard.png)

| KPI | Value |
|---|---|
| Total Billed Amount | 551.25K |
| Amount Collected | 173.42K |
| Amount Due | 184.61K |
| Failed Amount | 193.21K |
| Total Appointments | 200 |
| Cancellation Rate | 25.50% |
| No-Show Rate | 26.00% |
| Completion Rate | 23.00% |

### 2️⃣ Doctors' Performance *(5 visuals)*

Cancellation/no-show stacked bar by doctor, doctor-wise revenue funnel, revenue & doctor count by specialization, appointment trend by doctor, and a Doctor ID slicer.

![Doctors' Performance](screenshots/doctors_performance.png)

- **Top revenue doctor:** Sarah Taylor (82.70K)
- **Highest combined risk (cancellation + no-show):** Sarah Smith (58.82%)
- **Revenue by specialization:** Pediatrics (258.94K) > Dermatology (202.71K) > Oncology (89.60K)

### 3️⃣ Financials *(5 visuals)*

Billing by payment method, billed amount by doctor experience, billed-vs-collected trend, billed amount by age group, and a doctor billing detail table.

![Financials](screenshots/financials.png)

- **Payment method mix:** Credit Card (36.53%), Insurance (33.04%), Cash (30.42%)
- **Highest-billing age group:** 18–35 (237.95K, 31.5% of total)

## 📈 Key Insights

- No-shows (26%) and cancellations (25.5%) together account for **51.5%** of all bookings — more than double the completed-visit rate (23%).
- Only **31.5%** of ₹5.51L billed has been collected; **35%** is marked Failed and **33.5%** is Pending — more revenue is unresolved than confirmed.
- Patient retention is a strength — every active patient returned for a repeat visit.
- Doctor workload is uneven: Sarah Taylor handled 29 appointments vs. Robert Davis's 13, a 2x+ spread.
- MRI is the costliest treatment type (avg ₹3,225), ~27% above the cheapest, ECG (avg ₹2,532).
- Appointment volume is seasonal — April peaks at 25 bookings, September dips to 11.
- Clinical breadth is narrow: only 3 specializations across 10 doctors and 3 branches.

## ✅ Business Decisions

1. **Reduce no-shows** — launch automated SMS/email reminders 24–48 hours before appointments.
2. **Recover lost capacity** — introduce buffer/overbooking slots to offset no-shows and cancellations.
3. **Prioritize collections** — chase the Failed bucket first (highest recovery value), then Pending, largest balances first.
4. **Rebalance doctor workload** — shift scheduling/referrals from over-loaded doctors toward under-utilized ones.
5. **Audit MRI pricing** — review vendor/equipment costs behind its cost premium.
6. **Align staffing to seasonality** — plan for the April peak and September trough.
7. **Re-engage dormant patients** — outreach campaign for registered patients with zero bookings.

## 🛠️ Tools & Tech Stack

- **Power BI Desktop** — data modeling, relationships, report design
- **Power Query** — importing and shaping the 5 source tables
- **DAX** — `CALCULATE`, `DIVIDE`, `FILTER` patterns for rate and retention KPIs
- **Star Schema Modeling** — Appointments fact table linked to Patient, Doctor, Treatment and Billing dimensions

## 📁 Repository Structure

```
├── HealthCare_Analytics.pbix          # Power BI report file
├── data/
│   ├── patients.csv
│   ├── doctors.csv
│   ├── appointments.csv
│   ├── treatments.csv
│   └── billing.csv
├── screenshots/
│   ├── data_model.png
│   ├── executive_dashboard.png
│   ├── doctors_performance.png
│   └── financials.png
└── README.md
```

## 🚀 How to Use

1. Clone this repository.
2. Open `HealthCare_Analytics.pbix` in **Power BI Desktop**.
3. If prompted, update the data source paths to point to the `data/` folder.
4. Explore the three report pages via the tabs at the bottom of the report canvas.

## 👤 Author

**[Your Name]**
*Business Intelligence & Analytics (Power BI) Project*

## 📄 License

This project is for educational/portfolio purposes. Dataset is synthetic/sample data.
