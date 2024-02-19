# 36.34. referential\_constraints

The view `referential_constraints` contains all referential (foreign key) constraints in the current database. Only those constraints are shown for which the current user has write access to the referencing table (by way of being the owner or having some privilege other than `SELECT`).

#### **Table 36.32. `referential_constraints` Columns**

| Name                        | Data Type        | Description                                                                                                                                      |
| --------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `constraint_catalog`        | `sql_identifier` | Name of the database containing the constraint (always the current database)                                                                     |
| `constraint_schema`         | `sql_identifier` | Name of the schema containing the constraint                                                                                                     |
| `constraint_name`           | `sql_identifier` | Name of the constraint                                                                                                                           |
| `unique_constraint_catalog` | `sql_identifier` | Name of the database that contains the unique or primary key constraint that the foreign key constraint references (always the current database) |
| `unique_constraint_schema`  | `sql_identifier` | Name of the schema that contains the unique or primary key constraint that the foreign key constraint references                                 |
| `unique_constraint_name`    | `sql_identifier` | Name of the unique or primary key constraint that the foreign key constraint references                                                          |
| `match_option`              | `character_data` | Match option of the foreign key constraint: `FULL`, `PARTIAL`, or `NONE`.                                                                        |
| `update_rule`               | `character_data` | Update rule of the foreign key constraint: `CASCADE`, `SET NULL`, `SET DEFAULT`, `RESTRICT`, or `NO ACTION`.                                     |
| `delete_rule`               | `character_data` | Delete rule of the foreign key constraint: `CASCADE`, `SET NULL`, `SET DEFAULT`, `RESTRICT`, or `NO ACTION`.                                     |
