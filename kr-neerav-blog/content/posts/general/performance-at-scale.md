+++
title = 'Performance at Scale'
date = 2024-05-13T06:50:05-07:00
draft = true
tags = ["Performance"]
categories = ["General"]
+++

## Background
Customers expect good performance from products/applications at all times. This means designing them to be performant at normal workload and at spike.

## Tenet for designing for performance
The way to achieve performance at scale is to reduce the number of steps in critical paths of the application. In software engineering this means using static assessts, minimizing the queries to database, reduce the number of hops across network, limit data access to memory instead of disk etc. In data engineering it could mean caching frequently used results, minimizing the number of joins in SQL, precomputed/materialized datasets etc. These techniques help keep the perfomance of the application good even at scale by reducing the dependencies/bottlenecks.

## References
* https://blog.bytebytego.com/p/black-friday-flash-sale
* https://blog.bytebytego.com/p/low-latency-stock-exchange
