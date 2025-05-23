---
keywords: [JSON函数, 数据转换, SQL 查询, 数据库 JSON, JSON 操作]
description: 列出了 GreptimeDB 中的所有 JSON 函数，包括函数的定义、使用方法和相关的 SQL 查询示例。
---

# JSON 函数（试验功能）

:::warning
JSON 类型目前仍处于实验阶段，在未来的版本中可能会有所调整。
:::

本页面列出了 GreptimeDB 中所有 JSON 类型相关的函数。

## 转换

JSON 类型与其他类型的值之间的转换。

* `parse_json(string)` 尝试将字符串解析为 JSON 类型。非法的 JSON 字符串将返回错误。
* `json_to_string(json)` 将 JSON 类型的值转换为字符串。

```sql
SELECT json_to_string(parse_json('{"a": 1, "b": 2}'));

+----------------------------------------------------------+
| json_to_string(parse_json(Utf8("{\"a\": 1, \"b\": 2}"))) |
+----------------------------------------------------------+
| {"a":1,"b":2}                                            |
+----------------------------------------------------------+
```

## 提取

通过给定的路径和给定的数据类型，从 JSON 中提取值。

* `json_get_bool(json, path)` 按照路径 `path` 从 JSON 中获取布尔值。
* `json_get_int(json, path)` 按照路径 `path` 从 JSON 中获取整数值。布尔值将被转换为整数。
* `json_get_float(json, path)` 按照路径 `path` 从 JSON 中获取浮点数值。布尔值、整数值将被转换为浮点数。
* `json_get_string(json, path)` 按照路径 `path` 从 JSON 中获取字符串。所有类型的 JSON 值都将被转换为字符串，包括数组、对象和 null。

`path` 是一个用于从 JSON 值中选择和提取元素的字符串。`path` 中支持的操作符有：

| 操作符                   | 描述                                      | 示例               |
| ------------------------ | ----------------------------------------- | ------------------ |
| `$`                      | 根元素                                    | `$`                |
| `@`                      | 过滤表达式中的当前元素                    | `$.event?(@ == 1)` |
| `.*`                     | 选择对象中的所有元素                      | `$.*`              |
| `.<name>`                | 选择对象中匹配名称的元素                  | `$.event`          |
| `:<name>`                | `.<name>` 的别名                          | `$:event`          |
| `["<name>"]`             | `.<name>` 的别名                          | `$["event"]`       |
| `[*]`                    | 选择数组中的所有元素                      | `$[*]`             |
| `[<pos>, ..]`            | 选择数组中基于 0 的第 `n` 个元素          | `$[1, 2]`          |
| `[last - <pos>, ..]`     | 选择数组中最后一个元素之前的第 `n` 个元素 | `$[0, last - 1]`   |
| `[<pos1> to <pos2>, ..]` | 选择数组中某个范围内的所有元素            | `$[1 to last - 2]` |
| `?(<expr>)`              | 选择所有匹配过滤表达式的元素              | `$?(@.price < 10)` |

如果 `path` 是无效的，函数将返回 `NULL`。

```sql
SELECT json_get_int(parse_json('{"a": {"c": 3}, "b": 2}'), 'a.c');

+-----------------------------------------------------------------------+
| json_get_int(parse_json(Utf8("{"a": {"c": 3}, "b": 2}")),Utf8("a.c")) |
+-----------------------------------------------------------------------+
|                                                                     3 |
+-----------------------------------------------------------------------+
```

## 验证

检查 JSON 值的类型。

* `json_is_null(json)` 检查 JSON 中的值是否为 `NULL`。
* `json_is_bool(json)` 检查 JSON 中的值是否为布尔值。
* `json_is_int(json)` 检查 JSON 中的值是否为整数。
* `json_is_float(json)` 检查 JSON 中的值是否为浮点数。
* `json_is_string(json)` 检查 JSON 中的值是否为字符串。
* `json_is_array(json)` 检查 JSON 中的值是否为数组。
* `json_is_object(json)` 检查 JSON 中的值是否为对象。

```sql
SELECT json_is_array(parse_json('[1, 2, 3]'));

+----------------------------------------------+
| json_is_array(parse_json(Utf8("[1, 2, 3]"))) |
+----------------------------------------------+
|                                            1 |
+----------------------------------------------+

SELECT json_is_object(parse_json('1'));

+---------------------------------------+
| json_is_object(parse_json(Utf8("1"))) |
+---------------------------------------+
|                                     0 |
+---------------------------------------+
```

* `json_path_exists(json, path)` 检查 JSON 中是否存在指定的路径。

如果 `path` 是无效的，函数将返回错误。

如果 `path` 或 `json` 是 `NULL`，函数将返回 `NULL`。

```sql
SELECT json_path_exists(parse_json('{"a": 1, "b": 2}'), 'a');

+------------------------------------------------------------------+
| json_path_exists(parse_json(Utf8("{"a": 1, "b": 2}")),Utf8("a")) |
+------------------------------------------------------------------+
|                                                                1 |
+------------------------------------------------------------------+

SELECT json_path_exists(parse_json('{"a": 1, "b": 2}'), 'c.d');

+--------------------------------------------------------------------+
| json_path_exists(parse_json(Utf8("{"a": 1, "b": 2}")),Utf8("c.d")) |
+--------------------------------------------------------------------+
|                                                                  0 |
+--------------------------------------------------------------------+

SELECT json_path_exists(parse_json('{"a": 1, "b": 2}'), NULL);

+-------------------------------------------------------------+
| json_path_exists(parse_json(Utf8("{"a": 1, "b": 2}")),NULL) |
+-------------------------------------------------------------+
|                                                        NULL |
+-------------------------------------------------------------+
```
