# HIRWA-Langton_20252SEN087_DairyMonitor
A dairy farming management system that helps small- and medium-scale farmers record individual cows' feed quantities and milk production, monitor productivity, and track milk supplied to the market.
**Student:** HIRWA Langton  
**Student ID:** 20252SEN087  
**Institution:** Adventist University of Central Africa (AUCA)  
**Course:** Database Development with PL/SQL
# DairyMonitor 🐄

## Individual Cow Milk Production Monitoring and Productivity Analysis System for Small- and Medium-Scale Dairy Farmers

**Student Name:** HIRWA Langton  
**Student ID:** 20252SEN087  
**Institution:** Adventist University of Central Africa (AUCA)  
**Course:** Database Development with PL/SQL  
**Academic Year:** 2026–2027  

---

## 1. Project Overview
**DairyMonitor** is a relational database and analytical management system designed specifically for **small- and medium-scale dairy farmers**. In many smallholder and mid-sized operations, herd performance is tracked at the aggregate farm level rather than per animal, masking low-yielding cows and inefficient feeding practices. 

This system provides a centralized, structured platform to capture daily animal-level metrics—linking nutritional intake directly to milking output and commercial sales. By automating daily yield calculations and feed-to-milk efficiency ratios through Oracle PL/SQL, DairyMonitor empowers farmers to make data-driven decisions regarding herd nutrition, culling, breeding, and market supply planning.

---

## 2. Problem Statement
Small- and medium-scale dairy farmers typically rely on paper notebooks, chalkboards, or disconnected spreadsheets to manage daily farm operations. This manual approach creates critical operational bottlenecks:

* **Loss of Cow-Level Visibility:** Recording bulk milk totals hides underperforming cows that consume expensive feed without generating proportional milk yields.
* **Inaccurate Feed-to-Milk Tracking:** Without linking daily feed rations (e.g., concentrates, forage, supplements) to individual morning and evening milk yields, farmers cannot calculate true feed conversion efficiency or detect early signs of illness/mastitis through sudden yield drops.
* **Calculation & Transcription Errors:** Manually summing morning and evening milking sessions, tracking spoiled/rejected milk, and reconciling farm output against milk delivered to collection centers or local buyers frequently results in arithmetic errors and missing records.
* **Lack of Historical & Trend Analysis:** Paper logs make it difficult to compare lactation curves across months, evaluate seasonal feed impacts, or generate reliable production summaries for agricultural loans and cooperative audits.

---

## 3. Main Objective
To design and implement an automated, Oracle-based relational database system powered by PL/SQL that tracks individual cow feed intake, milking yields, and market deliveries—enabling small- and medium-scale dairy farmers to evaluate animal-level productivity, minimize feed waste, and optimize farm profitability.

---

## 4. Specific Objectives
* **Herd & Farm Registry:** Maintain structured profiles for farmers, farm locations, and individual cows (including breed, tag number, date of birth, and lactation status).
* **Nutritional Intake Logging:** Record daily feed types, nutritional categories, and exact quantities (in kg) allocated to each cow.
* **Milking Session Capture:** Accurately log separate **morning** and **evening** milk yields (in liters) per cow on a daily basis.
* **Automated Daily Yield Computation:** Automatically compute total daily milk production per cow and calculate productivity metrics (such as milk-to-feed ratios) using PL/SQL triggers and functions.
* **Market Supply & Distribution Tracking:** Track quantities of milk supplied to buyers, cooperatives, or local markets, alongside unit prices, rejections, and farm-retained milk.
* **Productivity & Analytical Reporting:** Generate actionable herd performance rankings, low-yield alerts, feed cost-versus-output summaries, and periodic production reports.

---

## 5. Technologies
| Component | Technology | Role in System |
| :--- | :--- | :--- |
| **Database Engine** | Oracle Database (19c / 21c / XE) | Relational storage, referential integrity, and ACID-compliant transactions |
| **Backend Logic** | Oracle PL/SQL | Automated calculations, business rule enforcement, triggers, and reporting procedures |
| **User Interface** | Oracle APEX | Interactive web dashboard for data entry, cow performance charts, and farm analytics |
| **Version Control** | Git & GitHub | Source code management, SQL script versioning, and project documentation |

---

## 6. Database Design
The relational schema is normalized to **Third Normal Form (3NF)** to eliminate data redundancy and ensure strict referential integrity across farm entities:

1. **`FARMERS`**: Stores farmer profile details (`farmer_id`, full name, contact phone, national ID/registration code, address).
2. **`FARMS`**: Links to `FARMERS` (`farm_id`, `farmer_id`, farm name, district/sector location, herd capacity).
3. **`COWS`**: Links to `FARMS` (`cow_id`, `farm_id`, ear tag number, cow name, breed, date of birth, lactation stage, health/active status).
4. **`FEED_TYPES`**: Lookup catalog of available feeds (`feed_type_id`, feed name, category [Forage/Concentrate/Mineral], unit cost per kg).
5. **`FEED_RECORDS`**: Daily feeding logs per animal (`feed_record_id`, `cow_id`, `feed_type_id`, feeding date, quantity_kg, feeding_session).
6. **`MILK_PRODUCTION_RECORDS`**: Daily milking logs per animal (`production_id`, `cow_id`, production_date, morning_yield_liters, evening_yield_liters, `total_daily_liters`, milk_quality_notes).
7. **`MARKET_SUPPLY_RECORDS`**: Commercial sales and distribution logs (`supply_id`, `farm_id`, supply_date, buyer_name, quantity_supplied_liters, unit_price, total_revenue, rejected_liters).

---

## 7. PL/SQL Features
To enforce farm business rules and automate analytical workflows inside the database layer, the project implements:

* **Stored Procedures:**
  * `sp_register_cow`: Validates herd capacity before onboarding a new cow to a farm.
  * `sp_record_daily_milking`: Inserts or updates morning/evening milk entries and logs anomalies.
  * `sp_generate_herd_productivity_report`: Compiles periodic yield, feed consumption, and net margin summaries per cow.
* **Stored Functions:**
  * `fn_calculate_daily_yield(p_cow_id, p_date)`: Returns total liters produced by a specific cow on a given date.
  * `fn_feed_efficiency_ratio(p_cow_id, p_start_date, p_end_date)`: Computes liters of milk produced per kilogram of concentrate/feed consumed.
* **Database Triggers:**
  * `trg_calc_total_daily_milk`: `BEFORE INSERT OR UPDATE` trigger on `MILK_PRODUCTION_RECORDS` that automatically computes `total_daily_liters = NVL(morning_yield_liters, 0) + NVL(evening_yield_liters, 0)`.
  * `trg_validate_market_supply`: Prevents recording market deliveries that exceed available farm milk production for that date.
  * `trg_low_yield_alert`: Flags a warning log when a cow's daily yield drops by more than 20% compared to its 7-day rolling average.
* **Explicit Cursors:** Used to iterate across active milking cows within a farm to generate ranked productivity leaderboards and batch monthly statements.
* **Exception Handling & Data Validation:** Custom user-defined exceptions (e.g., `e_negative_yield`, `e_inactive_cow_milking`, `e_supply_exceeds_production`) paired with `CHECK` constraints to prevent impossible agricultural entries.

---

## 8. Innovation
**DairyMonitor** bridges enterprise-grade Oracle database automation with accessible farm management through an **Oracle APEX Interactive Dashboard**:

* **Cow-Level Scorecard:** Visualizes each cow's morning vs. evening yield trends, lifetime lactation curve, and feed-to-milk efficiency score.
* **Feed-to-Yield Correlation Charts:** Interactive bar and line graphs comparing feed costs against milk revenue per animal, helping farmers immediately identify high-performing vs. loss-making cows.
* **Automated Health & Drop Alerts:** Highlights sudden production dips on the dashboard home screen so farmers can intervene early with veterinary checks or ration adjustments.
* **Market Reconciliation Tracker:** Real-time gauge comparing total farm milk harvested against milk sold to cooperatives/buyers and milk retained for home/calf use.

---

## 9. Data Source and References
* **Candidate Reference Dataset:** Mendeley Data — *Dairy Cattle Milk Production and Feeding Dataset* (`https://data.mendeley.com/datasets/kd5bwkrwrx`)
* **Dataset Verification Checklist (Pre-Implementation):**
  * [ ] Confirm dataset license terms (e.g., CC BY 4.0) for academic and prototype testing use.
  * [ ] Verify total record count, observation frequency (daily vs. monthly), and completeness.
  * [ ] Map dataset variables (cow ID, parity/lactation stage, feed intake, milk yield) to the `COWS`, `FEED_RECORDS`, and `MILK_PRODUCTION_RECORDS` tables for realistic data seeding and stress testing.

---

## 10. Project Status & Roadmap
**Current Status:** `Phase 0 – Project Definition, Scope Analysis, and Repository Registration`

### Implementation Milestones
* [x] **Phase 0:** Problem analysis, system architecture definition, and GitHub repository setup.
* [ ] **Phase 1:** Entity-Relationship Diagram (ERD) modeling and DDL table creation scripts with constraints.
* [ ] **Phase 2:** Dataset verification, cleaning, and initial DML data population scripts.
* [ ] **Phase 3:** Implementation of PL/SQL triggers, functions, stored procedures, cursors, and exception handling.
* [ ] **Phase 4:** Oracle APEX dashboard development, analytical visualizations, and end-to-end testing.

### Future Improvements
* **Mobile/SMS Offline Data Entry:** Enable farmers or farmhands to submit daily morning/evening yields via lightweight mobile forms or SMS gateways in areas with limited connectivity.
* **Breeding & Veterinary Module:** Expand the schema to track artificial insemination (AI) dates, calving intervals, vaccination schedules, and mastitis treatment withdrawal periods.
* **Predictive Yield Forecasting:** Integrate Oracle Machine Learning (OML) to forecast seasonal milk supply for dairy cooperatives.
