# Dataset Card: DairyMonitor Individual Cow Milk Production Dataset

## 1. Overview & Identification
* **Project Name:** DairyMonitor — Individual Cow Milk Production Monitoring and Productivity Analysis System
* **Course:** INSY 8311 – Database Development with PL/SQL (AUCA)
* **Student:** HIRWA Langton (ID: 20252SEN087)
* **Raw Dataset Path:** `data/raw/dairy_production_raw.csv`
* **Format & Encoding:** Comma-Separated Values (`.csv`), UTF-8 encoding, comma-delimited, header row included.

---

## 2. Dataset Source & Provenance
* **Domain Context:** East African smallholder and commercial dairy farm operations (modeled on Rwandan dairy cooperative standards in Kigali/Eastern Province).
* **Collection Scope:** Daily operational logs capturing individual lactating cattle performance, morning and evening milking yields, daily feed ration allocations, veterinary health status, and cooperative milk deliveries.
* **License & Usage:** Synthetic/curated academic dataset compiled strictly for database normalization, PL/SQL automation, and Oracle APEX analytics under the AUCA Capstone Project guidelines. The raw file (`data/raw/dairy_production_raw.csv`) is preserved unaltered as the single source of truth prior to relational decomposition.

---

## 3. Dataset Structure & Column Descriptions
The unnormalized flat dataset (`dairy_production_raw.csv`) consolidates farm, animal, feeding, milking, and market supply attributes into a single transactional log:

| Column Name | Data Type | Unit / Format | Description |
| :--- | :--- | :--- | :--- |
| `record_id` | Integer | Numeric ID | Unique sequential identifier for each raw log entry |
| `log_date` | Date | `YYYY-MM-DD` | Calendar date of the feeding and milking observation |
| `farm_id` | String | `FRM-XXX` | Unique code identifying the dairy farm unit |
| `farm_name` | String | Text | Registered name of the dairy farm |
| `farm_location` | String | Text | District or sector where the farm operates |
| `cow_ear_tag` | String | `RW-COW-XXXX` | Unique national/farm ear tag identifier for the cow |
| `cow_name` | String | Text | Assigned name of the individual cow |
| `breed` | String | Categorical | Cattle breed (e.g., *Friesian*, *Jersey*, *Ayrshire*, *Crossbreed*) |
| `birth_date` | Date | `YYYY-MM-DD` | Date of birth used to compute cow age and lactation stage |
| `lactation_stage` | String | Categorical | Current physiological stage (*Early*, *Mid*, *Late*, *Dry*) |
| `health_status` | String | Categorical | Clinical status on log date (*Healthy*, *Under Treatment*, *Alert*) |
| `feed_type` | String | Categorical | Primary feed ration administered (e.g., *Dairy Silage*, *Maize Bran*, *Napier Grass*, *Concentrate*) |
| `feed_unit_cost` | Decimal | RWF / kg | Unit cost per kilogram of the administered feed |
| `feed_qty_kg` | Decimal | Kilograms (`kg`) | Total weight of feed consumed by the cow on `log_date` |
| `morning_yield_l` | Decimal | Liters (`L`) | Volume of milk harvested during the morning milking session |
| `evening_yield_l` | Decimal | Liters (`L`) | Volume of milk harvested during the evening milking session |
| `cooperative_name` | String | Text | Receiving milk collection center (MCC) or dairy cooperative |
| `supplied_qty_l` | Decimal | Liters (`L`) | Volume of milk dispatched to the cooperative for sale |
| `price_per_liter` | Decimal | RWF / L | Farm-gate price paid per liter by the cooperative |

---

## 4. Data Quality & Integrity Audit
Before loading the dataset into the 3NF Oracle schema (`i_20252sen087_langton_dairy_db`), the following quality checks and business rules are enforced:

1. **Completeness & Null Handling:**
   * Primary keys (`cow_ear_tag`, `farm_id`, `log_date`) contain `0%` missing values.
   * For cows in the `Dry` lactation stage, `morning_yield_l` and `evening_yield_l` are explicitly recorded as `0.00` rather than `NULL`.
2. **Domain & Range Validation:**
   * `morning_yield_l >= 0` and `evening_yield_l >= 0` (no negative milk volumes).
   * `feed_qty_kg > 0` for all active feeding records.
   * `supplied_qty_l <= (morning_yield_l + evening_yield_l)` to prevent overselling beyond harvested farm volume.
3. **Elimination of Derived Columns in Raw Storage:**
   * Derived metrics such as `total_daily_yield_l` (`morning_yield_l + evening_yield_l`), `feed_efficiency_ratio` (`total_daily_yield_l / feed_qty_kg`), and `gross_revenue_rwf` (`supplied_qty_l * price_per_liter`) are intentionally excluded from raw storage and computed dynamically via Oracle PL/SQL triggers and stored functions.

---

## 5. Target 3NF Schema Mapping
During ETL and database population, the flat CSV columns decompose cleanly into the seven normalized tables of the **DairyMonitor** database:

* **`FARMS`**: `farm_id`, `farm_name`, `farm_location`
* **`COWS`**: `cow_ear_tag`, `farm_id`, `cow_name`, `breed`, `birth_date`, `lactation_stage`, `health_status`
* **`FEED_TYPES`**: `feed_type_id`, `feed_type`, `feed_unit_cost`
* **`FEED_RECORDS`**: `feed_record_id`, `cow_ear_tag`, `feed_type_id`, `log_date`, `feed_qty_kg`
* **`MILK_PRODUCTION_RECORDS`**: `prod_id`, `cow_ear_tag`, `log_date`, `morning_yield_l`, `evening_yield_l`, `total_daily_yield_l` *(trigger-populated)*
* **`COOPERATIVES`**: `coop_id`, `cooperative_name`
* **`MARKET_SUPPLY_RECORDS`**: `supply_id`, `farm_id`, `coop_id`, `log_date`, `supplied_qty_l`, `price_per_liter`