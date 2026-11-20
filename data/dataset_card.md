# Dataset card: Food Delivery Dataset (RiderLink)

**Student:** NAYIHIKI Nshuti Honore (ID 29683) · **Course:** INSY 8311 · **Phase II**

## 1. About the dataset

| Item | Value |
|---|---|
| Name | Food Delivery Dataset |
| Source link | https://www.kaggle.com/datasets/gauravmalik26/food-delivery-dataset |
| Author on Kaggle | Gaurav Malik |
| Licence | **[copy the License line from the Kaggle Data Card before submission]** |
| Raw file | `data/raw/train.csv`, saved exactly as downloaded, never edited |
| Format | CSV, comma-separated, 1 header row, all values stored as text |
| Size | 45,593 data rows × 20 columns, about 7.8 MB |
| Date collected | **[write the date you downloaded it, for example 11 November 2026]** |
| Period covered by the data | 11 February 2022 to 06 April 2022 (44 different order dates) |
| Other files in the download | `test.csv` and `Sample_Submission.csv` were not used (they are for a prediction contest and are not needed for this design) |

**One record represents** one food-delivery order: the restaurant it came from, the delivery location, the rider who delivered it, the weather and traffic at the time, and the minutes the delivery took.

## 2. Personal data and anonymisation

The file has no names, phone numbers, ID numbers or addresses. Riders appear only as codes such as `INDORES13DEL02`, and places appear only as latitude and longitude. I did not add any personal data. The dataset is from Indian cities, not Kigali (see the project README).

## 3. Data quality report (counts only; nothing has been fixed)

All counts were computed from the raw file with a script. Percentages are of 45,593 rows.

### 3.1 Missing values (the text `NaN` stored as a value)

| Column | Rows with `NaN` | Share |
|---|---:|---:|
| Delivery_person_Age | 1,854 | 4.1% |
| Delivery_person_Ratings | 1,908 | 4.2% |
| Time_Orderd | 1,731 | 3.8% |
| Weatherconditions (as `conditions NaN`) | 616 | 1.4% |
| Road_traffic_density | 601 | 1.3% |
| multiple_deliveries | 993 | 2.2% |
| Festival | 228 | 0.5% |
| City | 1,200 | 2.6% |

Rows with at least one `NaN` in any column: **4,225** (9.3%). The other 12 columns have no `NaN`.

### 3.2 Duplicates

- Fully duplicated rows: **0**
- Duplicated order `ID` values: **0**

### 3.3 Values stored in the wrong form (text instead of number, date or time)

| Problem | Rows affected |
|---|---:|
| `Order_Date` stored as text in day-month-year form (e.g. `19-03-2022`), to become a DATE | 45,593 |
| `Time_taken(min)` stored as text with the prefix `(min) ` (e.g. `(min) 24`) | 45,593 |
| `Weatherconditions` stored with the prefix `conditions ` (e.g. `conditions Sunny`) | 45,593 |
| Age, rating, vehicle condition, multiple_deliveries and all four coordinates stored as text, to become numbers | 45,593 |
| `Time_Orderd` and `Time_Order_picked` hold a time only, with no date | 45,593 |

### 3.4 Extra spaces

Cells with a leading or trailing space: ID **45,593**, Delivery_person_ID **45,593**, Road_traffic_density **45,593**, Type_of_order **45,593**, Type_of_vehicle **45,593**, Festival **45,593**, City **45,593**. Age, ratings, Time_Orderd and multiple_deliveries have the space only where the value is `NaN` (1,854, 1,908, 1,731 and 993 cells).

### 3.5 Invalid or suspicious values

| Problem | Rows affected |
|---|---:|
| Restaurant latitude and longitude are both `0` (a missing location stored as zero) | 3,640 |
| Restaurant latitude is negative (the Indian cities are all north of the equator, so the sign looks wrong) | 431 |
| Restaurant longitude is negative (the cities are east of Greenwich, so the sign looks wrong) | 162 |
| Rider rating above 5 (the highest value is 6) | 53 |
| Rider rating of 1 (the lowest value; very different from the usual 4.x) | 38 |
| Rider age of 15 (under 18) | 38 |
| Rider age of 50 (the age after 39 jumps straight to 50, so it looks like a typing error) | 53 |
| Pickup time earlier than order time (all of these cross midnight, so they are not wrong, but the missing date must be handled) | 831 |
| Spelling `Metropolitian` for Metropolitan in column `City` | 34,093 |

### 3.6 Counts that shape the design

| Fact | Count |
|---|---:|
| Distinct rider codes | 1,320 |
| Distinct restaurant coordinate pairs as written | 705 |
| Distinct real restaurant locations (after ignoring the 0,0 rows and the minus signs) | 388 |
| Distinct city codes found at the start of the rider code | 22 |
| Distinct delivery locations | 4,373 |
| Rider codes that appear with more than one `Delivery_person_Age` | 1320 of 1320 (median 17 different ages per rider code) |
| Rider codes that appear with more than one vehicle type | 1320 of 1320 |
| Delivery time range | 10 to 54 minutes |
| Highest `multiple_deliveries` value | 3 |

The other categorical columns: weather has 6 real values plus `NaN`; traffic has 4 (Low, Medium, High, Jam) plus `NaN`; order type has 4 (Snack, Meal, Drinks, Buffet); vehicle type has 4 (motorcycle, scooter, electric_scooter, bicycle); `City` has 3 (Metropolitian, Urban, Semi-Urban) plus `NaN`.

## 4. What I will do about it

Nothing in Phase II. The cleaning checklist is in `docs/design/normalization.md`, section 5, and is carried out in Phase IV.
