# RiderLink: Order Tracking and Rider Assignment for Food Delivery in Kigali

- **Student:** NAYIHIKI Nshuti Honore
- **Student ID:** 29683
- **Course:** INSY 8311: Database Development with PL/SQL
- **Instructor:** Eric Maniraguha
- **University:** Adventist University of Central Africa (AUCA)
- **Tools:** Oracle Database, SQL Developer, Oracle APEX
- **Repository:** honore_29683_riderLink
- **Current phase:** Phase 0 (research, data source, proposal)

---

## 1. Problem statement

Online food delivery in Kigali runs on motorbike riders. When orders arrive faster than a dispatcher can track them, riders are assigned by hand, often through chat groups or memory, and nothing in the system stops one rider from being given more active orders than they can safely carry. Overloaded riders deliver late, ride under time pressure and carry unstable loads, which hurts customers, restaurants and the riders themselves.

This project builds an Oracle database that records restaurants, orders and riders, and enforces the rule **"a rider who is already at capacity cannot be assigned another order"** inside the database itself, so the rule holds no matter which tool a dispatcher uses.

## 2. Objectives

**Main objective:** Design and build an Oracle database with PL/SQL that records food-delivery orders and rider assignments and prevents over-capacity assignments.

**Specific objectives:**
1. Model the order-to-delivery business process and normalize real delivery data to Third Normal Form (3NF).
2. Build PL/SQL procedures, functions, cursors and a package to register orders, assign riders, complete deliveries and report rider workload.
3. Enforce the capacity rule and an operating-hours restriction with triggers, and record every attempted and denied action in an audit log.
4. Provide an Oracle APEX dispatch application that lets a dispatcher see rider load and assign orders by calling the PL/SQL procedures.

## 3. Data source

- **Dataset:** Food Delivery Dataset (Gaurav Malik), Kaggle: https://www.kaggle.com/datasets/gauravmalik26/food-delivery-dataset
- **Size:** about 45,000 delivery records (confirm exact row count after download).
- **Main fields:** delivery person ID, age and ratings, restaurant and delivery coordinates, order date and pickup time, weather, road traffic density, vehicle condition, type of order, type of vehicle, festival indicator, city, time taken (minutes).
- **Licence and privacy:** confirm the licence allows educational use on the Kaggle page before loading. Rider IDs in the dataset are codes, not names; no real person is identified.
- **Adaptation for Kigali:** the data comes from Indian cities, so it is used as realistic order and rider activity. Restaurants are derived from distinct restaurant coordinates, and a rider capacity (`max_active_orders`) is assigned per vehicle type. This is a design decision, not a field in the original data. Currency and area names in the application are set for Kigali (RWF, Gasabo, Kicukiro, Nyarugenge).

Raw files go in `data/`.

## 4. Innovation (Oracle APEX dispatch application)

An Oracle APEX application connected to the live Oracle database:
- a **dispatch board** showing each rider's current active orders against their capacity;
- an **assign-rider form** that calls the `assign_rider` procedure;
- an **order entry page** that calls `register_order`;
- an **audit view** of denied assignments.

Business rules stay in PL/SQL. APEX only shows the data and calls the procedures; the trigger blocks over-capacity assignments even if someone bypasses the app. Documentation, run instructions and screenshots go in `innovation/`.

## 5. Planned tables (to be refined in Phase II)

| Table | Purpose |
|-------|---------|
| `vehicle_type` | Vehicle types and `max_active_orders` |
| `rider` | Rider code, age, rating, vehicle type, status |
| `restaurant` | Restaurant derived from location, area |
| `weather_condition`, `traffic_level` | Lookup tables for order conditions |
| `delivery_order` | Order, restaurant, order time, delivery location, conditions, status |
| `order_assignment` | Rider assigned to an order, assigned/completed times, status |
| `audit_log` | User, action, table, time, outcome (allowed or denied) |

## 6. Planned PL/SQL features

| Feature | Where it is used |
|---------|------------------|
| Function `get_active_load(rider_id)` | Counts a rider's current active orders |
| Function `get_rider_capacity(rider_id)` | Returns capacity from the rider's vehicle type |
| Procedure `register_order` | Inserts a new order with validation |
| Procedure `assign_rider` | Assigns a rider; raises a custom exception if over capacity |
| Procedure `complete_delivery` | Closes an assignment and frees capacity |
| Explicit cursor | Loops over riders to produce a workload and late-delivery report |
| Package `dispatch_pkg` | Groups the procedures, functions and exceptions |
| Exception handling | Custom errors for full rider, unknown order, inactive rider |
| Trigger (compound) on `order_assignment` | Blocks assigning a rider who is already at capacity |
| Restriction trigger | Blocks assignment changes outside operating hours (rule to be confirmed with the instructor) |
| Audit trigger and `audit_log` | Records allowed and denied actions per user |

## 7. Repository structure

```
docs/research/      research log
docs/proposal/      proposal
docs/presentation/  slides
data/               raw dataset files
innovation/         APEX app documentation and screenshots
```

## 8. References

Verify each reference by reading it before submission; remove any you have not read.

- Christie, N., & Ward, H. (2023). Delivering hot food on motorcycles: A mixed method study of the impact of business model on rider behaviour and safety. *Safety Science, 158*, Article 105991. https://doi.org/10.1016/j.ssci.2022.105991
- Garg, A., Ayaan, M., Parekh, S., & Udandarao, V. (2025). *Food delivery time prediction in Indian cities using machine learning models*. arXiv. https://doi.org/10.48550/arXiv.2503.15177
- Yalçınkaya, E., & Areta Hızıroğlu, O. (2024). A comparative analysis of machine learning models for time prediction in food delivery operations. *Artificial Intelligence Theory and Applications, 4*(1), 43-56.
- Malik, G. (n.d.). *Food delivery dataset* [Data set]. Kaggle. https://www.kaggle.com/datasets/gauravmalik26/food-delivery-dataset
- Wearetech Africa. (2025, December 17). *Vuba Vuba emerges as Rwanda's leading local delivery platform*. https://www.wearetech.africa/en/fils-uk/solutions/vuba-vuba-emerges-as-rwanda-s-leading-local-delivery-platform
- Oracle. (n.d.). *Database PL/SQL language reference*. Oracle Docs. https://docs.oracle.com
