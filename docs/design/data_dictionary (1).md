# Data dictionary (logical design)

**Student:** NAYIHIKI Nshuti Honore (ID 29683) · **Course:** INSY 8311 · **Phase II**

Logical level only: no Oracle data types or sizes (they are added in Phase IV). **Required** = the column must always have a value. **Unique** = no two rows may share a value. Keys: PK primary key, FK foreign key. The ERD is in `erd.png`; the steps that produced these tables are in `normalization.md`.

The design has **11 tables**. Tables with no raw source: AUDIT_LOG (whole table), and the columns DELIVERY_ORDER.status, VEHICLE_TYPE.max_active_orders, SERVICE_CITY.city_name, plus the surrogate keys.

## SERVICE_CITY

Delivery city, taken from the first letters of the rider code.

| Column | Key | Meaning | Required | Unique |
|---|---|---|---|---|
| city_code | PK | Short code of the city (e.g. INDO, BANG, MUM). 22 codes in the data. | Yes | Yes |
| city_name |  | Full city name. Not in the raw file; filled in later where known. | No | No |

## RIDER

A delivery rider, identified by the rider code in the data.

| Column | Key | Meaning | Required | Unique |
|---|---|---|---|---|
| rider_id | PK | Surrogate key of the rider. | Yes | Yes |
| rider_code |  | Code from the raw file (e.g. INDORES13DEL02). | Yes | Yes |
| city_code | FK | City the rider works in (parent: SERVICE_CITY). | Yes | No |

## VEHICLE_TYPE

A kind of vehicle and how many orders it may carry at once.

| Column | Key | Meaning | Required | Unique |
|---|---|---|---|---|
| vehicle_type_id | PK | Surrogate key of the vehicle type. | Yes | Yes |
| type_name |  | motorcycle, scooter, electric_scooter or bicycle. | Yes | Yes |
| max_active_orders |  | Largest number of open orders a rider on this vehicle may hold. Not in the raw file; the trigger rule reads it. Values set in Phase IV. | Yes | No |

## RESTAURANT

A restaurant, known only by its location in the data.

| Column | Key | Meaning | Required | Unique |
|---|---|---|---|---|
| restaurant_id | PK | Surrogate key of the restaurant. | Yes | Yes |
| latitude |  | Latitude of the restaurant. The pair (latitude, longitude) is unique. | Yes | Yes (with longitude) |
| longitude |  | Longitude of the restaurant. | Yes | Yes (with latitude) |

## AREA_TYPE

Kind of area the order was delivered in (raw column `City`).

| Column | Key | Meaning | Required | Unique |
|---|---|---|---|---|
| area_type_id | PK | Surrogate key. | Yes | Yes |
| area_name |  | Metropolitan, Urban or Semi-Urban. | Yes | Yes |

## WEATHER_CONDITION

Weather at the time of the order.

| Column | Key | Meaning | Required | Unique |
|---|---|---|---|---|
| weather_id | PK | Surrogate key. | Yes | Yes |
| weather_name |  | Sunny, Cloudy, Fog, Stormy, Windy or Sandstorms. | Yes | Yes |

## TRAFFIC_LEVEL

Road traffic level at the time of the order.

| Column | Key | Meaning | Required | Unique |
|---|---|---|---|---|
| traffic_id | PK | Surrogate key. | Yes | Yes |
| level_name |  | Low, Medium, High or Jam. | Yes | Yes |

## ORDER_TYPE

What kind of food was ordered.

| Column | Key | Meaning | Required | Unique |
|---|---|---|---|---|
| order_type_id | PK | Surrogate key. | Yes | Yes |
| type_name |  | Snack, Meal, Drinks or Buffet. | Yes | Yes |

## DELIVERY_ORDER

One food-delivery order (one row of the raw file).

| Column | Key | Meaning | Required | Unique |
|---|---|---|---|---|
| order_id | PK | Order code from the raw `ID` (e.g. 0x4607). | Yes | Yes |
| restaurant_id | FK | Restaurant that prepares the order (parent: RESTAURANT). | Yes | No |
| order_type_id | FK | Kind of food ordered (parent: ORDER_TYPE). | Yes | No |
| area_type_id | FK | Area type of the delivery; empty when the raw value is NaN (parent: AREA_TYPE). | No | No |
| weather_id | FK | Weather at order time; empty when unknown (parent: WEATHER_CONDITION). | No | No |
| traffic_id | FK | Traffic at order time; empty when unknown (parent: TRAFFIC_LEVEL). | No | No |
| order_date |  | Date the order was placed. | Yes | No |
| time_ordered |  | Time the order was placed; empty when the raw value is NaN. | No | No |
| dest_latitude |  | Latitude of the delivery location. | Yes | No |
| dest_longitude |  | Longitude of the delivery location. | Yes | No |
| is_festival |  | Y or N: was it a festival day; empty when unknown. | No | No |
| status |  | NEW, CONFIRMED, REJECTED, ASSIGNED or DELIVERED. Not in the raw file; added for the dispatch process. | Yes | No |

## ORDER_ASSIGNMENT

Links an order to the rider and vehicle that deliver it (junction table).

| Column | Key | Meaning | Required | Unique |
|---|---|---|---|---|
| assignment_id | PK | Surrogate key of the assignment. | Yes | Yes |
| order_id | FK | The order being delivered (parent: DELIVERY_ORDER). | Yes | No |
| rider_id | FK | The rider delivering it (parent: RIDER). | Yes | No |
| vehicle_type_id | FK | Vehicle used for this delivery (parent: VEHICLE_TYPE). | Yes | No |
| vehicle_condition |  | Condition score of the vehicle, 0 to 3, as in the raw file. | Yes | No |
| rider_age |  | Rider age recorded with this delivery; empty when NaN. | No | No |
| rider_rating |  | Rider rating recorded with this delivery; empty when NaN. | No | No |
| picked_up_time |  | Time the rider picked up the food; empty until pickup. | No | No |
| other_deliveries |  | Number of other deliveries carried at the same time (raw `multiple_deliveries`); empty when NaN. | No | No |
| time_taken_min |  | Minutes the delivery took; empty until delivered. | No | No |

## AUDIT_LOG

Record of every assignment attempt, written by the capacity trigger.

| Column | Key | Meaning | Required | Unique |
|---|---|---|---|---|
| log_id | PK | Surrogate key of the log entry. | Yes | Yes |
| order_id | FK | Order the attempt was for (parent: DELIVERY_ORDER). | Yes | No |
| rider_id | FK | Rider the attempt was for (parent: RIDER). | Yes | No |
| outcome |  | ALLOWED or DENIED. | Yes | No |
| reason |  | Short reason, e.g. rider at capacity. | No | No |
| logged_at |  | Date and time of the attempt. | Yes | No |

## Relationships (crow's foot notation)

Cardinality is written parent : child. `1` = exactly one, `0..1` = zero or one, `0..*` = zero or many. The child column holds the foreign key.

| Parent | Child | Relationship | Parent side | Child side | Foreign key | In words |
|---|---|---|---|---|---|---|
| SERVICE_CITY | RIDER | service_city has rider | 1 | 0..* | RIDER.city_code | A rider works in exactly one city; a city may have no riders yet or many. |
| RESTAURANT | DELIVERY_ORDER | restaurant prepares delivery_order | 1 | 0..* | DELIVERY_ORDER.restaurant_id | An order comes from exactly one restaurant; a restaurant may have no orders or many. |
| ORDER_TYPE | DELIVERY_ORDER | order_type classifies delivery_order | 1 | 0..* | DELIVERY_ORDER.order_type_id | An order has exactly one type; a type may be used by many orders. |
| AREA_TYPE | DELIVERY_ORDER | area_type describes delivery_order | 0..1 | 0..* | DELIVERY_ORDER.area_type_id | An order has at most one area type (unknown in 1,200 rows); an area type is used by many orders. |
| WEATHER_CONDITION | DELIVERY_ORDER | weather_condition describes delivery_order | 0..1 | 0..* | DELIVERY_ORDER.weather_id | An order has at most one weather record (unknown in 616 rows). |
| TRAFFIC_LEVEL | DELIVERY_ORDER | traffic_level describes delivery_order | 0..1 | 0..* | DELIVERY_ORDER.traffic_id | An order has at most one traffic level (unknown in 601 rows). |
| DELIVERY_ORDER | ORDER_ASSIGNMENT | delivery_order is delivered through order_assignment | 1 | 0..* | ORDER_ASSIGNMENT.order_id | An assignment belongs to exactly one order; an order has no assignment until a rider is chosen, and may have more than one if the first is cancelled. |
| RIDER | ORDER_ASSIGNMENT | rider carries out order_assignment | 1 | 0..* | ORDER_ASSIGNMENT.rider_id | An assignment has exactly one rider; a rider has many assignments over time. |
| VEHICLE_TYPE | ORDER_ASSIGNMENT | vehicle_type is used in order_assignment | 1 | 0..* | ORDER_ASSIGNMENT.vehicle_type_id | An assignment uses exactly one vehicle type; a type is used in many assignments. |
| DELIVERY_ORDER | AUDIT_LOG | delivery_order is checked in audit_log | 1 | 0..* | AUDIT_LOG.order_id | Each log entry is for one order; an order may have no entries or several attempts. |
| RIDER | AUDIT_LOG | rider is checked in audit_log | 1 | 0..* | AUDIT_LOG.rider_id | Each log entry is for one rider; a rider may have many entries. |

## Many-to-many relationships resolved by a junction table

- **RIDER ↔ DELIVERY_ORDER** (a rider delivers many orders; an order can have more than one rider if reassigned) → ORDER_ASSIGNMENT.
- **RIDER ↔ VEHICLE_TYPE** (a rider uses several vehicle types over time; the same type is used by many riders) → ORDER_ASSIGNMENT.
