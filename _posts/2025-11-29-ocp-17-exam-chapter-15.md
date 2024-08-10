---
title: OCP 17 Exam - chapter 15 notes - JDBC
categories: [ Book notes ]
tags: [ ocp17, java ]
---

- Java provides JDBC interfaces, while database libraries provides implementations - except implementation of
  `DriverManager`, which is used to create connection (Q.1)
- if autocommit is enabled (default), then commit happens after every statement (Q.6)
- if nothing to **rollback** (changes has been commited), then rollback doesn't throw any Exception (Q.6)
- **rollback** will throw if you try to rollback to already rollbacked savepoint (after savepoint used to rollback), so
  order matters. Also you cannot rollback to the same savepoint twice (Q.15)
- `CallableStatement` requires `{}` for every statement, e.g. `{?= call my_databse_function(?) }`
- `CallableStatement`s `OUT` param should be registered, e.g. `registerOutParameter(1, Types.INTEGER)` (Q.14)
- be aware that JDBC code might throw `SqlException`, which is checked, so if not declared is compilation error (Q.10)
- be aware of missing `ResultSet#next` (moves query cursor), which might throw SqlException (Q.18)
- be aware of missing parameter - `prepareStatement()` requires SQL (Q.20)

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
