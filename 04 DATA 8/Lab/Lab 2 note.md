---
title: Lab 2 note
date: 2022-09-05 周一
tags:
  - table
  - operation

mindmap-plugin: basic
---
# Lab 2 note

## summary

|          Name |                        Example |                                                      Purpose |
| ------------: | -----------------------------: | -----------------------------------------------------------: |
|        `sort` |                `tbl.sort("N")` |    Create a copy of a table sorted by the values in a column |
|       `where` | `tbl.where("N", are.above(2))` | Create a copy of a table with only the rows that match some *predicate* |
|    `num_rows` |                 `tbl.num_rows` |                        Compute the number of rows in a table |
| `num_columns` |              `tbl.num_columns` |                     Compute the number of columns in a table |
|      `select` |              `tbl.select("N")` |       Create a copy of a table with only some of the columns |
|        `drop` |                `tbl.drop("N")` |         Create a copy of a table without some of the columns |

## predicate in 'where'

|                 Predicate |                          Example |                                                       Result |
| ------------------------: | -------------------------------: | -----------------------------------------------------------: |
|            `are.equal_to` |               `are.equal_to(50)` |                            Find rows with values equal to 50 |
|        `are.not_equal_to` |           `are.not_equal_to(50)` |                        Find rows with values not equal to 50 |
|               `are.above` |                  `are.above(50)` |            Find rows with values above (and not equal to) 50 |
|   `are.above_or_equal_to` |      `are.above_or_equal_to(50)` |                Find rows with values above 50 or equal to 50 |
|               `are.below` |                  `are.below(50)` |                               Find rows with values below 50 |
|             `are.between` |             `are.between(2, 10)` |       Find rows with values above or equal to 2 and below 10 |
| `are.between_or_equal_to` | `are.between_or_equal_to(2, 10)` | Find rows with values above or equal to 2 and below or equal to 10 |%%%%