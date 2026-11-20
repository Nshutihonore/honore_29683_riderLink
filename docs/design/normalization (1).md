# Normalization: from the raw CSV to Third Normal Form

**Student:** NAYIHIKI Nshuti Honore (ID 29683) · **Course:** INSY 8311 · **Phase II**
Source: `data/raw/train.csv` (45,593 rows, 20 columns). Every row shown below is a real row from that file (the CSV line number is given). Values are shown trimmed of extra spaces so the tables stay readable; the raw file still has the spaces.

## 0. Entities found

The CSV is flat, so it has no nested objects. I found the entities by asking which columns describe the same thing:

| Entity | Evidence in the data |
|---|---|
| ORDER (DELIVERY_ORDER) | one row per `ID`; date, time, location, weather, traffic and festival describe the order |
| RESTAURANT | only latitude and longitude identify it (388 real locations, many orders each) |
| RIDER | rider code repeats across orders (1,320 codes, 5 to 67 orders each) |
| SERVICE_CITY | the first letters of the rider code (e.g. `INDO`, `BANG`): 22 values |
| VEHICLE_TYPE | 4 values; capacity is a new column for the project rule |
| ORDER_ASSIGNMENT | the link between an order and the rider and vehicle that delivered it |
| Lookups: ORDER_TYPE, WEATHER_CONDITION, TRAFFIC_LEVEL, AREA_TYPE | repeated text values (4, 6, 4 and 3 distinct values) |
| AUDIT_LOG | not in the raw data; written by the capacity trigger (project design from Phase 0) |

**Many-to-many relationships.** A rider delivers many orders, and an order could be given to more than one rider over time (if the first assignment is cancelled), so RIDER and DELIVERY_ORDER are many-to-many. In this file every order has exactly one rider, but the design must allow more. The junction table ORDER_ASSIGNMENT resolves it. The same table also resolves RIDER and VEHICLE_TYPE: 1,320 of the 1,320 rider codes appear on more than one vehicle type, so a vehicle type is not fixed for a rider.

**Lists.** The raw file has no list columns. The nearest thing is `multiple_deliveries`, a count of other deliveries carried at the same time. I keep it as a number on the assignment, because the raw file does not say which orders those were.

## 1. UNF (unnormalised form)

One wide row per order, 20 columns. The relation is `RAW(ID, Delivery_person_ID, Delivery_person_Age, ... , Time_taken(min))`. The rows below are the real rows I follow through every step (15 of the 20 columns are shown).

| csv line | ID | Delivery_person_ID | Delivery_person_Age | Delivery_person_Ratings | Restaurant_latitude | Restaurant_longitude | Order_Date | Time_Orderd | Time_Order_picked | Weatherconditions | Road_traffic_density | Type_of_vehicle | Vehicle_condition | multiple_deliveries | Time_taken(min) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 2 | 0x4607 | INDORES13DEL02 | 37 | 4.9 | 22.745049 | 75.892471 | 19-03-2022 | 11:30:00 | 11:45:00 | conditions Sunny | High | motorcycle | 2 | 0 | (min) 24 |
| 839 | 0x179b | INDORES13DEL02 | 25 | 4.2 | 22.745049 | 75.892471 | 05-04-2022 | 08:20:00 | 08:25:00 | conditions Windy | Low | motorcycle | 0 | 1 | (min) 29 |
| 259 | 0x2391 | INDORES13DEL03 | 33 | 4.5 | 22.745049 | 75.892471 | 10-03-2022 | 22:50:00 | 23:05:00 | conditions Stormy | Low | scooter | 2 | 0 | (min) 17 |
| 44 | 0x4f0 | MUMRES17DEL02 | NaN | NaN | 19.121999 | 72.908493 | 06-04-2022 | NaN | 18:35:00 | conditions Cloudy | Medium | scooter | 1 | 1 | (min) 33 |

The first two rows show one problem: rider `INDORES13DEL02` appears in both with different ages (37 and 25). The first three rows show another: they have the same restaurant coordinates, so the restaurant is repeated in every order it serves. The `Delivery_person_ID` value also packs three facts into one string.

## 2. First Normal Form (1NF)

**Rule:** every column holds one atomic value, there are no repeating groups, and every row is identified by a key.

**What changes and why**
- *Non-atomic value:* `Delivery_person_ID` (e.g. `INDORES13DEL02`) holds three facts (city code `INDO`, a number `13` after `RES`, and a number `02` after `DEL`). I split it into `city_code`, `res_no` and `del_no`, keeping `rider_code` as the whole code. The Kaggle page does not explain the two numbers, so I only record them.
- *Labels mixed into values:* `conditions ` in the weather column and `(min) ` in the time-taken column are removed so each cell holds one value.
- *Missing values:* the text `NaN` becomes a real NULL.
- *Repeating groups:* none, because the CSV is already flat (one row per order). There is no list to unnest.
- *Key:* `order_id` (the raw `ID`, unique in all 45,593 rows).

Key: **(order_id)**

| order_id | rider_code | city_code | res_no | del_no | age | weather | time_ordered | time_taken_min |
|---|---|---|---|---|---|---|---|---|
| 0x4607 | INDORES13DEL02 | INDO | 13 | 02 | 37 | Sunny | 11:30:00 | 24 |
| 0x179b | INDORES13DEL02 | INDO | 13 | 02 | 25 | Windy | 08:20:00 | 29 |
| 0x2391 | INDORES13DEL03 | INDO | 13 | 03 | 33 | Stormy | 22:50:00 | 17 |
| 0x4f0 | MUMRES17DEL02 | MUM | 17 | 02 | NULL | Cloudy | NULL | 33 |

## 3. Second Normal Form (2NF)

**Rule:** no attribute may depend on only part of a composite key.

A table whose key is one column is automatically in 2NF, so the single-column key of the 1NF table above hides the issue. The data answers a real question, though: *which rider delivered which order*. The key of that fact is the pair **(order_id, rider_code)**, because an order can have more than one rider over time and a rider has many orders. With that key, two kinds of partial dependency appear:

1. **Partial dependency on `order_id` only:** order date, time ordered, restaurant location, weather, traffic, order type, area type and festival describe the order, whoever delivers it. They move to **DELIVERY_ORDER**.
2. **Partial dependency on `rider_code` only:** `city_code` is part of the rider code and describes the rider only. It moves to **RIDER**.
3. **What stays in ORDER_ASSIGNMENT (depends on the whole pair):** vehicle type, vehicle condition, rider age, rider rating, pickup time, other deliveries carried and time taken. I tested whether age, rating and vehicle type depend on the rider alone: **they do not.** 1,320 of 1,320 rider codes appear with more than one age (median 17), and 1320 appear with more than one vehicle type. In this dataset they are recorded per delivery, so they stay with the assignment. Phase IV will treat them as values observed at delivery time.

**DELIVERY_ORDER** (key: order_id)

| order_id | order_date | time_ordered | restaurant_lat | restaurant_long | weather | traffic | order_type | area_type | festival |
|---|---|---|---|---|---|---|---|---|---|
| 0x4607 | 19-03-2022 | 11:30:00 | 22.745049 | 75.892471 | Sunny | High | Snack | Urban | No |
| 0x179b | 05-04-2022 | 08:20:00 | 22.745049 | 75.892471 | Windy | Low | Buffet | Metropolitian | No |
| 0x2391 | 10-03-2022 | 22:50:00 | 22.745049 | 75.892471 | Stormy | Low | Buffet | Metropolitian | No |
| 0x4f0 | 06-04-2022 | NULL | 19.121999 | 72.908493 | Cloudy | Medium | Drinks | Metropolitian | No |

**RIDER** (key: rider_code)

| rider_code | city_code |
|---|---|
| INDORES13DEL02 | INDO |
| INDORES13DEL03 | INDO |
| MUMRES17DEL02 | MUM |

**ORDER_ASSIGNMENT** (key: order_id + rider_code)

| order_id | rider_code | vehicle_type | vehicle_condition | rider_age | rider_rating | picked_up_time | other_deliveries | time_taken_min |
|---|---|---|---|---|---|---|---|---|
| 0x4607 | INDORES13DEL02 | motorcycle | 2 | 37 | 4.9 | 11:45:00 | 0 | 24 |
| 0x179b | INDORES13DEL02 | motorcycle | 0 | 25 | 4.2 | 08:25:00 | 1 | 29 |
| 0x2391 | INDORES13DEL03 | scooter | 2 | 33 | 4.5 | 23:05:00 | 0 | 17 |
| 0x4f0 | MUMRES17DEL02 | scooter | 1 | NULL | NULL | 18:35:00 | 1 | 33 |

## 4. Third Normal Form (3NF)

**Rule:** no non-key attribute may depend on another non-key attribute (no transitive dependency).

Each split below names the dependency removed:

| # | Transitive dependency removed (one sentence) | New table |
|---|---|---|
| 1 | `order_id → restaurant → its latitude and longitude`: the coordinates describe the restaurant, not the order, and one restaurant serves many orders, so they move out and the order keeps only `restaurant_id`. | RESTAURANT |
| 2 | `order_id → weather_name`: the weather text repeats in thousands of orders, so it becomes a lookup row and the order keeps `weather_id`. | WEATHER_CONDITION |
| 3 | `order_id → traffic level text`: same reason, 4 values repeated in all orders. | TRAFFIC_LEVEL |
| 4 | `order_id → order type text`: 4 values repeated in all orders. | ORDER_TYPE |
| 5 | `order_id → area type text` (the raw column named `City`, with values Metropolitian, Urban, Semi-Urban): 3 values repeated in all orders. | AREA_TYPE |
| 6 | `rider_code → city_code → city_name`: the city name depends on the city code, not on the rider, so the code and name move to a table of their own. | SERVICE_CITY |
| 7 | `assignment → vehicle type text → max_active_orders`: the capacity limit depends on the vehicle type, not on the assignment, so the type and its limit live in one place. This is also where the capacity rule of the trigger looks up its limit. | VEHICLE_TYPE |

The keys shown (`restaurant_id`, `rider_id`, and so on) are surrogate numbers that I assign in Phase IV; the numbers below only illustrate.

**DELIVERY_ORDER** (key: order_id)

| order_id | restaurant_id | order_type_id | area_type_id | weather_id | traffic_id | order_date | is_festival |
|---|---|---|---|---|---|---|---|
| 0x4607 | 1 | 4 | 3 | 5 | 3 | 19-03-2022 | N |
| 0x179b | 1 | 1 | 1 | 6 | 1 | 05-04-2022 | N |
| 0x2391 | 1 | 1 | 1 | 4 | 1 | 10-03-2022 | N |
| 0x4f0 | 39 | 2 | 1 | 1 | 2 | 06-04-2022 | N |

**RESTAURANT** (key: restaurant_id; the first three orders share one restaurant, so it is stored once)

| restaurant_id | latitude | longitude |
|---|---|---|
| 1 | 22.745049 | 75.892471 |
| 39 | 19.121999 | 72.908493 |

**RIDER** (key: rider_id; unique: rider_code)

| rider_id | rider_code | city_code |
|---|---|---|
| 1 | INDORES13DEL02 | INDO |
| 224 | INDORES13DEL03 | INDO |
| 42 | MUMRES17DEL02 | MUM |

**ORDER_ASSIGNMENT** (key: assignment_id; the pair order_id + rider_id is kept unique while an assignment is open)

| assignment_id | order_id | rider_id | vehicle_type_id | vehicle_condition | rider_age | rider_rating | picked_up_time | other_deliveries | time_taken_min |
|---|---|---|---|---|---|---|---|---|---|
| 1 | 0x4607 | 1 | 3 | 2 | 37 | 4.9 | 11:45:00 | 0 | 24 |
| 2 | 0x179b | 1 | 3 | 0 | 25 | 4.2 | 08:25:00 | 1 | 29 |
| 3 | 0x2391 | 224 | 4 | 2 | 33 | 4.5 | 23:05:00 | 0 | 17 |
| 4 | 0x4f0 | 42 | 4 | 1 | NULL | NULL | 18:35:00 | 1 | 33 |

**WEATHER_CONDITION** (6 rows) and **VEHICLE_TYPE** (4 rows) as examples of the lookup tables:

| weather_id | weather_name |
|---|---|
| 1 | Cloudy |
| 2 | Fog |
| 3 | Sandstorms |
| 4 | Stormy |
| 5 | Sunny |
| 6 | Windy |

| vehicle_type_id | type_name | max_active_orders |
|---|---|---|
| 1 | bicycle | (set in Phase IV) |
| 2 | electric_scooter | (set in Phase IV) |
| 3 | motorcycle | (set in Phase IV) |
| 4 | scooter | (set in Phase IV) |

TRAFFIC_LEVEL (Low, Medium, High, Jam), ORDER_TYPE (Buffet, Drinks, Meal, Snack) and AREA_TYPE (Metropolitian, Semi-Urban, Urban) follow the same pattern.

**Final set: 11 tables.** SERVICE_CITY, RIDER, VEHICLE_TYPE, RESTAURANT, AREA_TYPE, WEATHER_CONDITION, TRAFFIC_LEVEL, ORDER_TYPE, DELIVERY_ORDER, ORDER_ASSIGNMENT, AUDIT_LOG. Check against 3NF: every non-key column in each table depends on the key, the whole key and nothing but the key.

## 5. Field mapping and cleaning checklist (every raw field)

| # | Raw field | Target table.column | Conversion needed in Phase IV |
|---|---|---|---|
| 1 | ID | DELIVERY_ORDER.order_id | trim the trailing space; keep as text (values like `0x4607`) |
| 2 | Delivery_person_ID | RIDER.rider_code and RIDER.city_code | trim; city_code = the letters before `RES`; link to SERVICE_CITY |
| 3 | Delivery_person_Age | ORDER_ASSIGNMENT.rider_age | text → NUMBER; `NaN` → NULL (1,854 rows); review the 15s (38) and 50s (53) |
| 4 | Delivery_person_Ratings | ORDER_ASSIGNMENT.rider_rating | text → NUMBER; `NaN` → NULL (1,908); 53 values above 5 to review |
| 5 | Restaurant_latitude | RESTAURANT.latitude | text → NUMBER; 0 → no restaurant (3,640 rows); negative → take the absolute value (431 rows) |
| 6 | Restaurant_longitude | RESTAURANT.longitude | same rules (162 negative rows) |
| 7 | Delivery_location_latitude | DELIVERY_ORDER.dest_latitude | text → NUMBER |
| 8 | Delivery_location_longitude | DELIVERY_ORDER.dest_longitude | text → NUMBER |
| 9 | Order_Date | DELIVERY_ORDER.order_date | text `DD-MM-YYYY` → DATE |
| 10 | Time_Orderd | DELIVERY_ORDER.time_ordered | text → time on the order date; `NaN` → NULL (1,731). The column name is misspelt in the file |
| 11 | Time_Order_picked | ORDER_ASSIGNMENT.picked_up_time | text → time on the order date; if earlier than the order time, add one day (831 rows) |
| 12 | Weatherconditions | WEATHER_CONDITION.weather_name | remove `conditions `; `NaN` → NULL (616) |
| 13 | Road_traffic_density | TRAFFIC_LEVEL.level_name | trim; `NaN` → NULL (601) |
| 14 | Vehicle_condition | ORDER_ASSIGNMENT.vehicle_condition | text → NUMBER (0 to 3) |
| 15 | Type_of_order | ORDER_TYPE.type_name | trim |
| 16 | Type_of_vehicle | VEHICLE_TYPE.type_name | trim |
| 17 | multiple_deliveries | ORDER_ASSIGNMENT.other_deliveries | text → NUMBER; `NaN` → NULL (993). My reading: other deliveries carried at the same time (check the Kaggle description) |
| 18 | Festival | DELIVERY_ORDER.is_festival | `Yes`/`No` → Y/N; `NaN` → NULL (228) |
| 19 | City | AREA_TYPE.area_name | trim; correct `Metropolitian` to `Metropolitan`; `NaN` → NULL (1,200). The values are area types, not city names |
| 20 | Time_taken(min) | ORDER_ASSIGNMENT.time_taken_min | remove `(min) `; text → NUMBER |

**Dropped fields:** none. **Columns that do not come from the raw file** (added by the project): DELIVERY_ORDER.status, VEHICLE_TYPE.max_active_orders, SERVICE_CITY.city_name, all surrogate keys, and the whole AUDIT_LOG table.

**Observation for the capacity rule.** The highest `multiple_deliveries` value is 3 for motorcycle, scooter and electric scooter, and 2 for bicycle. This is evidence to help choose `max_active_orders` per vehicle type in Phase IV; the final values are my design decision.

## 6. Class demonstration (5 minutes): follow one record

Take CSV line 2, order `0x4607`:

| Raw value | Ends up in |
|---|---|
| `0x4607` | DELIVERY_ORDER.order_id |
| `INDORES13DEL02` | RIDER.rider_code; `INDO` → RIDER.city_code → SERVICE_CITY |
| 37 / 4.9 | ORDER_ASSIGNMENT.rider_age / rider_rating |
| 22.745049, 75.892471 | RESTAURANT.latitude / longitude (via DELIVERY_ORDER.restaurant_id) |
| 22.765049, 75.912471 | DELIVERY_ORDER.dest_latitude / dest_longitude |
| 19-03-2022 / 11:30:00 | DELIVERY_ORDER.order_date / time_ordered |
| 11:45:00 | ORDER_ASSIGNMENT.picked_up_time |
| `conditions Sunny` | WEATHER_CONDITION (Sunny) via weather_id |
| `High` | TRAFFIC_LEVEL (High) via traffic_id |
| 2 | ORDER_ASSIGNMENT.vehicle_condition |
| `Snack` | ORDER_TYPE (Snack) via order_type_id |
| `motorcycle` | VEHICLE_TYPE (motorcycle) via vehicle_type_id |
| 0 | ORDER_ASSIGNMENT.other_deliveries |
| `No` | DELIVERY_ORDER.is_festival = N |
| `Urban` | AREA_TYPE (Urban) via area_type_id |
| `(min) 24` | ORDER_ASSIGNMENT.time_taken_min = 24 |
