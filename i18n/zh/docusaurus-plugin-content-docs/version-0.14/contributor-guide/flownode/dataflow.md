---
keywords: [Dataflow, SQL 查询, 执行计划, 数据流, map, reduce]
description: 解释了 Dataflow 模块的核心计算功能，包括 SQL 查询转换、内部执行计划、数据流的触发运行和支持的操作。
---

# 数据流

Dataflow 模块（参见 `flow::compute` 模块）是 `flow` 的核心计算模块。
它接收 SQL 查询并将其转换为 `flow` 的内部执行计划。
然后，该执行计划被转化为实际的数据流，而数据流本质上是一个由带有输入和输出端口的函数组成的有向无环图（DAG）。
数据流会在需要时被触发运行。

目前该数据流只支持 `map`和 `reduce` 操作，未来将添加对 `join` 等操作的支持。

在内部，数据流使用 `tuple(row, time, diff)` 以行格式处理数据。
这里 `row` 表示实际传递的数据，可能包含多个 `value` 对象。
`time` 是系统时间，用于跟踪数据流的进度，`diff` 通常表示行的插入或删除（+1 或 -1）。
因此，`tuple` 表示给定系统时间的 `row` 的插入/删除操作。
