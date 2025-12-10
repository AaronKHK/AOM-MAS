# asset_meta_og

**Table Purpose / Objective**  
`asset_meta_og` stores the **master metadata** for each asset in the AOM system.  
It serves as the single source of truth for asset identification, specifications, OEM details, electrical ratings, system hierarchy, site location, and references to associated timeseries, inventory, and alert tables.  
Each row represents **one physical asset**.

**Primary Key**  
- `asset_id`


## Column Definitions

| Column Name         | Data Type      | Nullable | Description                                                                               |
|---------------------|----------------|----------|-------------------------------------------------------------------------------------------|
| `asset_id`          | varchar(max)   | NO       | Unique identifier for the asset. Used as the primary key and join key for related tables. |
| `asset_name`        | varchar(max)   | NO       | Human-readable asset name.                                                                |
| `asset_type`        | varchar(max)   | NO       | High-level category of the asset (e.g., pump, motor, HX, compressor).                     |
| `maker`             | varchar(max)   | YES      | OEM / manufacturer of the asset.                                                          |
| `asset_model`       | varchar(max)   | YES      | OEM model name or number.                                                                 |
| `rated_power_kw`    | bigint         | YES      | Rated electrical power of the asset in kilowatts.                                         |
| `rated_voltage_v`   | bigint         | YES      | Rated operating voltage in volts.                                                         |
| `sub_system`        | varchar(max)   | YES      | Sub-system this asset belongs to (e.g., WTS-01, CDU-01).                                  |
| `system`            | varchar(max)   | YES      | Higher-level system (e.g., Water Treatment System, Crude Distillation Unit).              |
| `location`          | varchar(max)   | YES      | Physical site or plant location of the asset.                                             |
| `ts_data_table`     | varchar(max)   | YES      | Name of the timeseries data table that stores telemetry for this asset.                   |
| `inv_data_table`    | varchar(max)   | YES      | Name of the inventory table containing spare parts and material information.              |
| `alert_data_table`  | varchar(max)   | YES      | Name of the alert/event table storing alarms and recommendations for this asset.          |


#

---
# asset_pump_std_og

**Table Purpose / Objective**  
`asset_pump_std_og` stores the **standardized pump operational timeseries data** for each asset.  
This includes vibration, temperature, electrical load, and speed readings captured at regular intervals.  
Each row represents **one telemetry sample** for a given pump.

**Primary Key**  
- `asset_id`, `timestamp`


## Column Definitions

| Column Name                  | Data Type    | Nullable | Description                                                                                   |
|------------------------------|--------------|----------|-----------------------------------------------------------------------------------------------|
| `timestamp`                  | datetime     | NO       | Timestamp of the telemetry reading. Represents the exact moment the sensor data was captured. |
| `asset_id`                   | varchar(max) | NO       | Identifier of the asset. Used as the join key to `asset_meta_og`.                             |
| `pump_de_overall_vibration`  | float        | YES      | Overall vibration at the **Drive-End (DE)** bearing housing, measured in mm/s RMS.            |
| `pump_de_skin_temperature`   | float        | YES      | Bearing housing surface temperature at the **DE**, measured in °C.                            |
| `pump_nde_overall_vibration` | float        | YES      | Overall vibration at the **Non-Drive-End (NDE)** bearing housing, measured in mm/s RMS.       |
| `pump_nde_skin_temperature`  | float        | YES      | Bearing housing surface temperature at the **NDE**, measured in °C.                           |
| `pump_operating_power`       | float        | YES      | Pump operating power in kW.                                                                   |
| `pump_operating_current`     | float        | YES      | Pump operating current in Amperes.                                                            |
| `pump_operating_speed`       | bigint       | YES      | Pump/motor rotational speed in RPM. Typically ~2900 RPM for 2-pole 50Hz motors.               |

#

---

# asset_param_og

**Table Purpose / Objective**  
`asset_param_og` stores the **parameter catalog** for each timeseries data table.  
It documents the parameter ID, description, and engineering unit so that signals in
telemetry tables (e.g., `asset_pump_std_og`, `asset_hx_std_og`) can be interpreted consistently.

**Primary Key (logical)**  
- ts_data_table`, `parameter_id`

---

## Column Definitions

| Column Name     | Data Type    | Nullable | Description                                                                                 |
|-----------------|--------------|----------|---------------------------------------------------------------------------------------------|
| `ts_data_table` | varchar(max) | NO       | Name of the timeseries table this parameter belongs to (e.g., `asset_pump_std_og`).         |
| `parameter_id`  | varchar(max) | NO       | Identifier of the parameter/signal (e.g., `pump_de_overall_vibration`, `hx_tube_out_flow`). |
| `description`   | varchar(max) | YES      | Human-readable description of the parameter.                                                |
| `unit`          | varchar(max) | YES      | Engineering unit of measure (e.g., `mm/s RMS`, `°C`, `kW`, `m³/h`, `%`).                    |

#

---

# asset_inv_og

**Table Purpose / Objective**  
`asset_inv_og` stores **inventory and spare parts data** grouped by asset and part categories.  
It tracks current stock levels, min/max stock policies, lead time, and basic part information
to support maintenance planning and spare management.

**Primary Key**  
- `asset_category`, `part_number`


## Column Definitions

| Column Name        | Data Type    | Nullable | Description                                                                                |
|--------------------|--------------|----------|--------------------------------------------------------------------------------------------|
| `asset_category`   | varchar(max) | NO       | High-level asset group this part is associated with. Format: {asset_type}-{maker}-{power}. |
| `asset_count`      | bigint       | YES      | Number of assets in this category that may use this part.                                  |
| `asset_type`       | varchar(max) | YES      | More specific asset type if applicable (e.g., `horizontal_pump`, `shell_tube_hx`).         |
| `part_category`    | varchar(max) | YES      | Category of part (e.g., `bearing`, `mechanical_seal`, `coupling`, `electrical`).           |
| `part_name`        | varchar(max) | NO       | Descriptive name of the part.                                                              |
| `item_type`        | varchar(max) | YES      | Type classification, e.g., `spare`, `consumable`.                                          |
| `part_number`      | varchar(max) | NO       | OEM or internal part number / material code.                                               |
| `unit`             | varchar(max) | YES      | Stock-keeping unit of measure (e.g., `pc`, `set`, `kit`).                                  |
| `stock`            | bigint       | NO       | Current on-hand quantity in store.                                                         |
| `min_stock`        | bigint       | YES      | Minimum stock level before re-ordering.                                                    |
| `max_stock`        | bigint       | YES      | Target maximum stock level.                                                                |
| `lead_time_week`   | bigint       | YES      | Procurement lead time in weeks.                                                            |
| `maker`            | varchar(max) | YES      | OEM / manufacturer of the part.                                                            |
| `update_timestamp` | datetime     | YES      | Timestamp of the last stock/record update.                                                 |

#

---

# asset_hx_std_og

**Table Purpose / Objective**  
`asset_hx_std_og` stores **standardized operating timeseries data** for shell-and-tube heat exchangers.  
It captures shell/tube temperatures, flow, differential pressure, and valve position at each timestamp.

**Primary Key (logical)**  
- `asset_id`, `timestamp`

## Column Definitions

| Column Name                | Data Type    | Nullable | Description                                                                       |
|----------------------------|--------------|----------|-----------------------------------------------------------------------------------|
| `timestamp`                | datetime     | NO       | Timestamp of the telemetry reading.                                               |
| `asset_id`                 | varchar(max) | NO       | Identifier of the heat exchanger asset (joins to `asset_meta_og`).                |
| `hx_shell_in_temperature`  | float        | YES      | Shell-side inlet temperature (e.g., hot crude in), typically in °C.               |
| `hx_shell_out_temperature` | float        | YES      | Shell-side outlet temperature, in °C.                                             |
| `hx_tube_in_temperature`   | float        | YES      | Tube-side inlet temperature, in °C.                                               |
| `hx_tube_out_temperature`  | float        | YES      | Tube-side outlet temperature, in °C.                                              |
| `hx_tube_out_flow`         | float        | YES      | Tube-side outlet flow, typically in m³/h or similar volumetric flow unit.         |
| `hx_tube_diff_pressure`    | float        | YES      | Tube-side differential pressure across the exchanger, typically in kPa or bar.    |
| `hx_tube_valve_opening`    | float        | YES      | Control valve opening for the tube-side flow, expressed as a percentage (0–100%). |

#

---

# asset_threshold_og

**Table Purpose / Objective**  
`asset_threshold_og` stores **parameter-specific warning and alert thresholds** for each asset.  
These thresholds are used by monitoring/alerting logic to determine when a parameter is in warning or alarm state.

**Primary Key (logical)**  
- `asset_id`, `parameter_id`

## Column Definitions

| Column Name    | Data Type    | Nullable | Description                                                                                          |
|----------------|--------------|----------|------------------------------------------------------------------------------------------------------|
| `asset_id`     | varchar(max) | NO       | Identifier of the asset the threshold applies to.                                                    |
| `parameter_id` | varchar(max) | NO       | Parameter identifier (matches `asset_param_og.parameter_id` and the signal name in timeseries data). |
| `warning`      | float        | YES      | Warning threshold value; exceeding this may trigger early warning / amber status.                    |
| `alert`        | float        | YES      | Alert / alarm threshold value; exceeding this may trigger red alarm or trip recommendation.          |

#

---

# asset_wo_og

**Table Purpose / Objective**  
`asset_wo_og` stores **maintenance work orders** associated with assets.  
It tracks creation date, planned/actual maintenance date, responsible person, and current status.

**Primary Key (logical)**  
- `wo_no`


## Column Definitions

| Column Name        | Data Type    | Nullable | Description                                                                               |
|--------------------|--------------|----------|-------------------------------------------------------------------------------------------|
| `asset_id`         | varchar(max) | NO       | Identifier of the asset the work order relates to.                                        |
| `create_date`      | datetime     | YES      | Date and time when the work order was created.                                            |
| `wo_no`            | bigint       | NO       | Work order number (unique identifier for the maintenance job).                            |
| `description`      | varchar(max) | YES      | Short description of the maintenance task or issue.                                       |
| `maintenance_date` | datetime     | YES      | Planned or actual maintenance execution date/time.                                        |
| `maintenance_by`   | varchar(max) | YES      | Name or identifier of the technician, contractor, or team performing the maintenance.     |
| `status`           | varchar(max) | YES      | Current status of the work order (e.g., `Open`, `In Progress`, `Completed`, `Cancelled`). |
