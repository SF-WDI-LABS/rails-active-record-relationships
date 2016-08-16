## Tables

Our goals is to create database tables to model the relationship between `drivers`, `licenses`, and `vehicles` (with foreign keys in the correct places!).

Our tables should be structured as follows:

#### `drivers` table
| id | first_name | last_name | birth_date | height | weight | hair | eyes | sex | created_at | updated_at |
|:---|------------|-----------|------------|--------|----|------|------|-----|------------|------------|
| 1 | "nathan" | "a" | "11/03/2016" | 78 | 160 | "brown" | "hazel" | "M" | "05/04/2016" | "05/04/2016" |
| 2 | "justin" | "c" | "07/11/2016" | 69 | 170 | "black" | "brown" | "M" | "05/04/2016" | "05/04/2016" |

#### `licenses` table
| id | driver_id | license_number | expiration | donor | created_at | updated_at |
|:---|-----------|----------------|------------|-------|------------|------------|
| 22 | 1 | "F89ARST4" | "11/03/2020" | false | "05/04/2016" | "05/04/2016" |
| 23 | 2 | "NCC1701D" | "07/11/2020" | true | "05/04/2016" | "05/04/2016" |

#### `vehicles` table
| id | make | model | year | license_plate| created_at | updated_at |
|:---|------|-------|------|---------------|-----------|------------|
| 8 | "Tesla" | "S" | 2015 | "e13krc" | "05/04/2016" | "05/04/2016" |
| 9 | "Schwinn" | "Sports Tourer" | 1979 | null | "05/04/2016" | "05/04/2016" |

#### `drivers_vehicles` table
| id | driver_id | vehicle_id |
|:---|-----------|------------|
| 11 | 1 | 8 |
| 12 | 1 | 9 |
| 13 | 2 | 8 |

## Some Example SQL Queries

* Grab driver #1
  * `SELECT * from drivers WHERE id=1;` 
* Grab driver #1's license
  * `SELECT * from licenses WHERE driver_id=1 LIMIT 1;`
* Grab driver #1's vehicles
  * `SELECT * FROM vehicles INNER JOIN drivers_vehicles ON vehicles.id = drivers_vehicles.vehicle_id WHERE drivers_vehicles.driver_id = 1;`
 
