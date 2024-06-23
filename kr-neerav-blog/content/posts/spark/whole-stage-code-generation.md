+++
title = 'Whole Stage Code Generation'
date = 2024-04-27T20:46:19-07:00
draft = true
tags = ["Spark Internals"]
categories = ["Spark"]
+++

## Background
When we submit a SQL to database the execution steps are 1) create an abstract syntax tree 2) create a logical plan 3) create an optimized logical plan 4) select a physical plan 5) execute the physical plan. The execution follows an iterator model. The physical plan ,which consists of algerabic expressions, is executed by calling next function which generates the next tuple by recursively calling next in the AST. This model is CPU heavy. The iterator model is simple and allows for arbitary combination of algebraic operators. This model worked well when the query processing was dominated by I/O. However with increase in memory capacity of computers, utilization of CPU cycles also plays a role in query processing time.

## Challeges with iterator model
When executing a query the next function is called for each tuple (millions of time) which results in the algebric expression being interpreted each time. There is also context switch involved as data is swapped in/out of registors multiple times. 
An approach that is used to address the above limitations is to batch records before passing to functions. However this results in materializing intermediate records which has impact of performance. 

## Whole Stage Codegen
The solution to this is to compile all operations in a spark stage into a single java function which is compiled once. This improves the query performance. 
## References
* Databricks blog link
* Paper link
