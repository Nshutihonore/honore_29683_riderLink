# RiderLink: Order Tracking and Rider Assignment for Food Delivery in Kigali

- **Student:** NAYIHIKI Nshuti Honore
- **Student ID:** 29683
- **Course:** INSY 8311: Database Development with PL/SQL
- **Instructor:** Eric Maniraguha
- **University:** Adventist University of Central Africa (AUCA)
- **Tools:** Oracle Database, SQL Developer, Oracle APEX
- **Repository:** honore_29683_riderLink
- **Current phase:** Phase II (Data Sourcing, Normalization and Logical Design)

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
- **File used:** `data/raw/train.csv` (the Kaggle `train.csv` file, unchanged). Details: [dataset card](data/dataset_card.md).
- **Size:** 45,593 delivery records from 11 February to 6 April 2022, with 1,320 rider codes, 388 real restaurant locations (705 coordinate pairs as written, including 3,640 rows with 0,0) and four vehicle types (bicycle, electric scooter, motorcycle, scooter).
- **Main fields:** delivery person ID, age and ratings, restaurant and delivery coordinates, order date and pickup time, weather, road traffic density, vehicle condition, type of order, type of vehicle, festival indicator, city, time taken (minutes).
- **Licence and privacy:** licence: [copy the License line from the Kaggle Data Card]. Rider IDs in the dataset are codes, not names; no real person is identified.
- **Data quality:** some columns contain the text "NaN", trailing spaces, prefixes such as "conditions " in the weather column and "(min) " in the time column. They are counted in the dataset card and cleaned in Phase IV.
- **Adaptation for Kigali:** the data comes from Indian cities, so it is used as realistic order and rider activity. Restaurants are derived from distinct restaurant coordinates, and a rider capacity (`max_active_orders`) is assigned per vehicle type. This is a design decision, not a field in the original data. Currency and area names in the application are set for Kigali (RWF, Gasabo, Kicukiro, Nyarugenge).

The raw file goes in `data/raw/` and is never edited.

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
docs/business_process/  Phase I diagram (PDF, .drawio) and explanation
data/raw/           raw dataset file (never edited)
data/dataset_card.md  source, licence, size and data-quality counts
docs/design/        normalization.md, erd.png, data_dictionary.md
innovation/         APEX app documentation and screenshots
```

## Phase I – Business Process Modeling

**Process modelled:** order-to-delivery with capacity-controlled rider assignment. A customer places an order, the dispatcher enters it in the APEX app, the restaurant accepts or rejects it, the dispatcher assigns a rider, and the database checks the rider's load. If the rider is at capacity, a trigger blocks the assignment and logs the attempt; otherwise the rider collects and delivers the food and the order is closed. Actors: customer, dispatcher, restaurant, rider, operations manager, APEX app and Oracle database.

**Tool used:** draw.io (diagrams.net), BPMN-style swimlane diagram.

**Published diagram (draw.io link):** [Open the RiderLink business process diagram](https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=RiderLink_business_process.drawio&dark=auto#R%3Cmxfile%3E%3Cdiagram%20id%3D%22riderlink-phase1%22%20name%3D%22Order-to-Delivery%20with%20Rider%20Assignment%22%3E7Z1bd5s6FoB%2FjVdzHtqFxP3R8SXNTJJmbJ9z2qcuxSgxEwwewE1yfv1IXGwQOMGyRIjrPjRYXIz3t%2FeWtCVt9dTB8vkiRKvFdeBgrwcV57mnDnsQAhvY5A8teclKoK2kJQ%2Bh62Rl24Kp%2Bw%2FOCvPL1q6Do9KFcRB4sbsqF84D38fzuFSGwjB4Kl92H3jlb12hB1wpmM6RVy3923XiRf7DoLI98RW7D4v8q4GanVmi%2FOqsIFogJ3gqFKmjnjoIgyBOj5bPA%2BxR8eWCSe8b7zi7ebMQ%2B3GTG2I3Jr%2BqclP2nCh%2ByX9zjJ%2FJufNFvPRIASCHyHMffHLs4Xt65hcOY5fIqJ8VL13HoTefPy3cGE9XaE6f9ERUgpTdB36ccSUvmH3OvgxknweBF4TJl6sAQkPtk%2FL0tX4hb5291oRoQnjl%2Bo89ch4q5%2BvI9XEUkcPbMJinR5kGDsjht5Bc%2FjkOPg%2Bx55I3fiFlTy5hApUBIq%2Foxi%2BfB%2BS7w8DzsENKk%2BeTv%2F0oIj9rSQUElbPz2%2BsbKp4nd%2BkhH1NddtFDiJZ%2FZC9IZIGfC6LMhH%2BBgyWOk29dFPRDh%2BltTwVlgnqmIdljcoXJbMbKtDnT5IfNg7esyUGGux49WscLIuCD2Ifpj%2BCGD7QqbE3X%2B4ZWB%2Fum%2F%2BPy6%2BW%2FL0npTbRYxy45%2BBr4QUgJ9Aawd26Sg0tyqQJtw1JLpTfTH%2BSPpQJQLL5doIjefVksnBFnkuqTE6KnL24giqpulqECVRWOlSqk0oRqGKx9B9O7lB2gCsDvXc8rMBprY3N8TsojYiqPuHDGVvpaf7Q5k%2FtH8EWvI8ol2I03LdqLZb1qL7ngD5Wsh%2B6wJ128g%2FHQGg72Fm%2BdF80NTX3FqxI34P5DziEve9sKp8E6ionAQmG8XvduomjVVIdiQY36YzC23ssO9P3tACoiDUG6fDtnCEM3WqF4vkgaBWffQjSnzSelfzv6ToW6WvG1AOpQvm4iwkDCo64rIIeNqLpIG5Eu387ZyMYqhihGd2n76uz2ikhs%2Bp8rPvuow%2Fi6fQiDqJ7qEEa0htA6RLp8O2cfExzFaB0i2pdspa4QBkw76roCaBz9CqF1hXT5ds8W0viKIFpv9CpEsdKPu0qA%2B5uBZYs0A%2Bny7ZwZfFvhEMVu4CfxSuSjB16rqIH3RiRRBLq8p42dSuS8yjBYh%2FPsqmkFK31CLmM%2F8GkMsUy6QJVlVGsxhAFU6RnsO30a%2FCfFd14wf0yLxq6XPw4%2Fu%2FH35EEK%2BZcV%2FKBf%2B0XfFgyfsxdJPrzkH3wise%2FJtfnNtKR8Ny3Z3p58eilpRIzCB5xJa7aDdkhsJHZ%2FlcV8EDvIx25WtckOwCux%2BsHSfBOeXoGncMGDbcFTOeHBE7yd8NS24Gmc8NQuwjvQbYqCpx0IL7mV%2FGL0UrhgFbh%2BHBWefEsLtnWqZpUrVa087EoO0idu1WLzas00RefUFO0INUVQBXvRWgVr8MG7%2BBgVrHKQjwacZq63Bc%2FktDz9BG8XvFFrlmcdk%2BV1w23OzLbg2ZyWZ3YRXjcsb2a0BS%2Bfq7c3PeNEbyc9qzV6nPGcmdVFeh1xnHZr9HgjOnYX6XUkKACU1vDxxnRyn9stft2wvovWInKAM6pz0cmQXDfozUBrXQbAG2oBnew0dKTlAlqLqQLOaMsMdDKqqhxif%2BADtl044y3d9J5dabu0V%2FlxRlxm4Bj5mdBUBfCbHFr5cY1pAGAw82cMZi0Ze4PCTrgxBI%2BCgO6EhBwULZJrQfbhFsUxDuniJ1IBKVqd%2FumaMTL7Zf0LVtgvqZ8iTv0g0b8Pq34Wq0zZ4qhd2mez2lq%2B%2FnDlg5wRrUkn24XdaNdPWquY8ofsTU98vfQ%2BruNd2v2D9vhyxrwk8D0a65yBQ0dZ%2BRoem%2Fnq%2BRRFdhH7WzdAUxfs%2B7lDch9jGPiduiWtjUZB3plW4GMMR70Lv%2Bv2ZqhyxuSuT02vnfRGrUXkIGdEbnCqmnfTa6thNd2NqoDF89xVRGm8tQ6jsGIC9navmGDXa1gjY2DX8huZQxVWaMNkvUaeleSKLiq5DSKXrrCggIM4Dpa9atqSOFj1alZnbLMfKE8oaboocUD%2BC0LexUtqZhKFVRp5Uf6QvEGbNWlAHhg9BObsFcOrW1kD3iaKwnkJKLOeL%2FlXB041NVMbV8HtoSQRTdjjP1wl%2BYeG2rZkkgm6NonMrUd%2FSg4vyfIyXwRBsko5LKzMpOmCyC9fUtrId%2Bj%2FjhMmGYU4gJuwAhyosEQcalqJuG0LAA5PwEd%2BnJhuZq0KcQGbhA1J2efE2fayvF8t0YWqCLx7Lp4%2BRrwhfnAjQvhnwaDJaddBMd6YrutHhGhBCc5uRn%2FzpSLgYK1amgDWey4OPkbWEzzHtHGz5ejgGLne1kc7eO460oxYA2WwhggjvmhWKS%2BC5d06atDEqlBUB7UhxbHdt6C%2Bo%2BlUx7FJgr6dTS7s0bZ10xbXtwwvms%2FxKqb6POZialRXwxrl1bAGmytB0wXY6p4rmI%2FRVqe46G9JuypeUzOdjP41GsxGQ1kmqltlE9UVAb531MhEZfeCxuPBcAjrIJ%2BbYAAGFcjUkD3GENmMjayhbjI3Nk3vWdYOo74F5qT1cq4NIf4vnlOzltN3MnKd2KzGgwJs2jzZdD9xxwWQaaVLjDtpXK1CTKRDHtKjSXUDPrwNbNzUJVTD%2BcKD35nvDp89%2BHYzvpxcy3PaLFAhTjtfi%2FA7A%2F3LxU9UBNh3UrNM2G6by2GW3vgpCB%2B9ANGiMydLcUiO7wIUOtK6SSx1MV1i%2B0R9ij2a9pwIsoA4BT733PljL89nTXGj5Ohncpk01DaUgTpf7%2FA7s55gtDHkT1ECb%2BGm6RfnWUbzDXw0j9Pec2rq7aAWEvzIZ%2FcedR857QuzDe3NYEUF%2FVXqsNOudQG3tL7ydh%2BDnK2I3FGbhRO%2FsxnPSL8syfmlJKOPSQ293XKA3kCMllbgDvbdZGcCOmdsueJLIdnAjAE0ZAQxwSliTapn6oM%2F5RXz%2FdrzPpG%2FSxxFycYjFPYqraaRH8RprumQO1EiD2xB9XMzr33UsCd4TtrcNeYclfpa%2Fen08uKG9LAKpo484tg5gyRNoCu2hF7X5OTMC8MUqLhtTNYCp1vLJK1zmSESthVmivDek5M9w2zjoA2%2BdCbIfDtDJOtY41WQxMSczUZDkqxYUyWQHpxIF6xYpqFW%2BImY9jFqhO80XtF0vCIzYmkDFsAwYEkJBM32Og1DwmsUPpbcMKmAlysPJ5NDzvLjn%2Fl5aREwYKhSmtinUQs48NLZe6%2B3sIejq8u%2FRhN5gxgVwkLa09en9jSpiX%2Bl4xiFYYpNAyuf90PDIp9L7e2kF9USaxsIYD1qFBzpZLUtaqr1puoNE%2BbyalyLmSNg558PAXj3cwZ%2FyhyUUJUqL6Ab%2BkCp48WYcnUzg7prX9vkANRaJxeiPFFEAREs25iqlwkRRkIIqSdCkgiRCkIQIZmVXh0hZhlJ64QGsgjlO8oKJqSdbEgQIR2WbcgQ5OX01gmNDB0o70joT0mEDLtMSLPF2JBxIiSIkGXIIWSdvJwkQqLackA5IRKECChMj0hUWwGA1hkda3MOqKYkRu13W4%2BWEVTk1EaEUdvdomNtMABgWnIYnRp1whiZmhQ7um6%2FOjraJgOLyBYx4uH9bLbPNPktMUOh4RBwHUxWiIzwNV3vGxojfFgn7MrKZA7R53IuiJ7Zf1ezyg5MBwKWrRLJNxpO7KjkbwJRwlaZNcJ6WdiGYggRdqMFSh0V9g8sTLVZaTPxFkNXhUgbfGSvIk23SXOV2bpSETCnn4r7I7uSTUYgEROwayAAZk9uoDBKD%2FOhlUMpNFq13FEK8nwM6QAwWm8IcjIf2aeXZk8Ibr8AhSVgMJWqKsjvNFoA2lEC2ZziECOHb8ZwjeTNsuAtxtOYiiVE7q8ldfwocj9snmeN7G1G6XWFUXpNjNLnX%2F0hhV%2BYm5nO6mpP%2FqYiptsEP3LbcoVDN3DceS%2BfdSVL%2BmZF%2BiK0Hz9g37kLnpsQKCeL3XuV8I6JkbbS1%2FqjHVGciri5pJuLslijWkxMgMlkBRQhNWoi3tiNPSxXxYvqrO6IctXNXqzI9yp5Y1E6zMiYGSMjMhYR8YqaiHY773O3jnImyhWjo7AmYGiVpKexGmqK0NAobk8zax3oNMt%2FhH8lTWcxomOcp2nIEF0jm26ieFyThttSO2BL0Tv83no3Sqa%2By9Q6CKWoXSPBlQdg5K5fkKaJennauSrHA8bvrYl9mlkmSTUjRmxMn1VTNBlie2ikhpukMsITyIjRurxJUox2wbL4Kv7PENHnf3hvrRviuRvRdSZytE5XpRjrfSajRtthvLr1hbzNLurFt9%2B2T5VtmthUu0Bhl6qnu39kt21FWn2S9daT0r0yKk%2Fq7b%2F3E%2BH13no%2Bxf9bYz%2FZteA%2ByfUkRt3ZkKzBjCmL0nfYkr5L3VdPikVYQJRFWGabFgHf2yIu%2FfsgXKJkkaFgowBlo7BMKUZxxzEHSeTUyu5Po2Q3EyaCFzFEdxc3krxE3R2EONmagkdftaq%2BMo0WoGhyFJZj7qnICXLdnwwH813qRCus5IG1NxV2Qt7Zxb%2FkqSzMZ%2FgIVlkoUWUbTLvt%2FhRbrdrDEaOyjSQvUWX%2FXDkyfaymy%2FGxHMkxGyssE4eVqrBDWQprqJJ8bCPJSw1lZCmJ6EuTXxPOF65Ej2sYUjyuH8SSRwmlDoQPUUwzpt8h0vWkGXiXae4olAY33aQsXiDa7YqDdZIhP15QaMTZoDsU4S%2FkmDxIOXNqcf5BZUUfQuRE%2Fl9HSSLfBQ5xnvOE5gpNcv%2BG9ObHdLMN0tXZfBNaO27y%2FSFyvS98%2BlEzlGwwoYDKMKe9%2F4xn8jEMyA8tdDMJr8V1QIRDCv8P%3C%2Fdiagram%3E%3C%2Fmxfile%3E)

**Files:**
- [Diagram and supporting tables (PDF)](docs/business_process/business_process_diagram.pdf)
- [One-page explanation (PDF)](docs/business_process/business_process_explanation.pdf)
- [Editable diagram (.drawio)](docs/business_process/RiderLink_business_process.drawio)

## Phase II – Data Sourcing, Normalization and Logical Design

- **Raw data and dataset card:** [data/raw/train.csv](data/raw/train.csv) and [data/dataset_card.md](data/dataset_card.md) (source, licence, size, quality counts).
- **Normalization (UNF → 1NF → 2NF → 3NF) with real rows and the field mapping:** [docs/design/normalization.md](docs/design/normalization.md)
- **Logical ERD (crow's foot):** [docs/design/erd.png](docs/design/erd.png)
- **Data dictionary:** [docs/design/data_dictionary.md](docs/design/data_dictionary.md)

The design has 11 tables: SERVICE_CITY, RIDER, VEHICLE_TYPE, RESTAURANT, AREA_TYPE, WEATHER_CONDITION, TRAFFIC_LEVEL, ORDER_TYPE, DELIVERY_ORDER, ORDER_ASSIGNMENT and AUDIT_LOG.

## 8. References

Verify each reference by reading it before submission; remove any you have not read.

- Christie, N., & Ward, H. (2023). Delivering hot food on motorcycles: A mixed method study of the impact of business model on rider behaviour and safety. *Safety Science, 158*, Article 105991. https://doi.org/10.1016/j.ssci.2022.105991
- Garg, A., Ayaan, M., Parekh, S., & Udandarao, V. (2025). *Food delivery time prediction in Indian cities using machine learning models*. arXiv. https://doi.org/10.48550/arXiv.2503.15177
- Yalçınkaya, E., & Areta Hızıroğlu, O. (2024). A comparative analysis of machine learning models for time prediction in food delivery operations. *Artificial Intelligence Theory and Applications, 4*(1), 43-56.
- Malik, G. (n.d.). *Food delivery dataset* [Data set]. Kaggle. https://www.kaggle.com/datasets/gauravmalik26/food-delivery-dataset
- Wearetech Africa. (2025, December 17). *Vuba Vuba emerges as Rwanda's leading local delivery platform*. https://www.wearetech.africa/en/fils-uk/solutions/vuba-vuba-emerges-as-rwanda-s-leading-local-delivery-platform
- Oracle. (n.d.). *Database PL/SQL language reference*. Oracle Docs. https://docs.oracle.com
