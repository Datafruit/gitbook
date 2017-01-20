#Sugo PlyQL

plyql是类似SQL的语言，可以解析成sugo-plywood的表达并执行。

plyql只支持`SELECT`，`DESCRIBE`，和`SHOW TABLES`查询。

## 示例
这些例子我们查询一个Druid的broker节点：`192.168.60.100` 端口： `8082`.

首先，我们可以为数据源列表发出一个`SHOW TABLES`查询，我们将其传递到`--query` 或者 `-q`

MYSQL 查询Druid数据源列表
```sql
plyql -h 192.168.60.100:8082 -q 'SHOW TABLES'
```

HTTP接口请求
```http
http://192.168.60.100:8082/api/plyql/sql
post请求: application/json 参数
{
  "query": 
  "SHOW TABLES"
}
```

返回JSON:

```json
{
  "result": [
    {
      "Tables_in_database": "wikipedia"
    },
    ....
  ]
}
```

查询Druid数据源表数据结构
```sql
plyql -h 192.168.60.100:8082 -q 'DESCRIBE wikipedia'
```

返回数据源的列定义：
```json
[
  {
    "name": "__time",
    "type": "TIME"
  },
  {
    "name": "added",
    "type": "NUMBER",
    "unsplitable": true
  },
  {
    "name": "channel",
    "type": "STRING"
  },
  {
    "name": "cityName",
    "type": "STRING"
  },
  {
    "name": "comment",
    "type": "STRING"
  },
  {
    "name": "commentLength",
    "type": "STRING"
  },
  {
    "name": "count",
    "type": "NUMBER",
    "unsplitable": true
  },
  {
    "name": "countryIsoCode",
    "type": "STRING"
  },
  {
    "name": "countryName",
    "type": "STRING"
  },
  {
    "name": "deleted",
    "type": "NUMBER",
    "unsplitable": true
  },
  {
    "name": "delta",
    "type": "NUMBER",
    "unsplitable": true
  },
  {
    "name": "deltaByTen",
    "type": "NUMBER",
    "unsplitable": true
  },
  {
    "name": "delta_hist",
    "type": "NUMBER",
    "special": "histogram"
  },
  "... results omitted ..."
]
```

这里是一个简单的查询，获取最大的` __time `信息。此查询显示数据库中最新事件的时间。

```sql
plyql -h 192.168.60.100:8082 -q 'SELECT MAX(__time) AS maxTime FROM wikipedia'
```

返回:

```json
[
  {
    "maxTime": {
      "type": "TIME",
      "value": "2015-09-12T23:59:00.000Z"
    }
  }
]
```

现在你可能要检查，不同的标签趋势。

您可能会在这样的页面列上做分组：

```sql
plyql -h 192.168.60.100:8082 -q '
SELECT page as pg, 
COUNT() as cnt 
FROM wikipedia 
GROUP BY page 
ORDER BY cnt DESC 
LIMIT 5;
'
```

这将抛出一个错误，因为没有时间过滤指定plyql查询的范围。

这种行为可以禁用使用`--allow eternity`，但不推荐这样做，这样做时，当数据量过大时，它可以发出计算禁止查询。

再试一次，用一个时间过滤器：
  
```sql
plyql -h 192.168.60.100:8082 -q '
SELECT page as pg, 
COUNT() as cnt 
FROM wikipedia 
WHERE "2015-09-12T00:00:00" <= __time AND __time < "2015-09-13T00:00:00"
GROUP BY page 
ORDER BY cnt DESC 
LIMIT 5;
'
```

结果集:
  
```json
[
  {
    "cnt": 314,
    "pg": "Jeremy Corbyn"
  },
  {
    "cnt": 255,
    "pg": "User:Cyde/List of candidates for speedy deletion/Subpage"
  },
  {
    "cnt": 228,
    "pg": "Wikipedia:Administrators' noticeboard/Incidents"
  },
  {
    "cnt": 186,
    "pg": "Wikipedia:Vandalismusmeldung"
  },
  {
    "cnt": 160,
    "pg": "Total Drama Presents: The Ridonculous Race"
  }
]
```
  
Plyql有一个选项 `--interval` (`-i`) 自动过滤器加上`interval`间隔时间。
如果您不想键入时间筛选器，则这样设置非常有用。

```sql
plyql -h 192.168.60.100:8082 -i P1Y -q '
SELECT page as pg, 
COUNT() as cnt 
FROM wikipedia 
GROUP BY page 
ORDER BY cnt DESC 
LIMIT 5;
'
```

通过`TIME_BUCKET`函数可以对时间分解

```sql
plyql -h 192.168.60.100:8082 -i P1Y -q '
SELECT SUM(added) as TotalAdded 
FROM wikipedia 
GROUP BY TIME_BUCKET(__time, PT6H, "Etc/UTC");
'
```

返回:

```json
[
  {
    "TotalAdded": 15426936,
    "split0": {
      "start": "2015-09-12T00:00:00.000Z",
      "end": "2015-09-12T06:00:00.000Z",
      "type": "TIME_RANGE"
    }
  },
  {
    "TotalAdded": 25996165,
    "split0": {
      "start": "2015-09-12T06:00:00.000Z",
      "end": "2015-09-12T12:00:00.000Z",
      "type": "TIME_RANGE"
    }
  },
  "... 结果省略了 ..."
]
```

注意分组列如果没有选择但仍有返回, 列如`TIME_BUCKET(__time, PT1H, 'Etc/UTC') as 'split'`
是查询列中的一个。
时间分割也支持，这里是一个例子：
```sql
plyql -h 192.168.60.100:8082 -i P1Y -q '
SELECT TIME_PART(__time, HOUR_OF_DAY, "Etc/UTC") as HourOfDay, 
SUM(added) as TotalAdded 
FROM wikipedia 
GROUP BY 1 
ORDER BY TotalAdded DESC LIMIT 3;
'
```
注意，这个 `GROUP BY` 指的是选择中的第一列。
这将返回：
```json
[
  {
    "TotalAdded": 8077302,
    "HourOfDay": 10
  },
  {
    "TotalAdded": 5998730,
    "HourOfDay": 17
  },
  {
    "TotalAdded": 5210222,
    "HourOfDay": 18
  }
]
```

支持对直方图的分位数。
假设您想要使用直方图计算 0.95 分位数的三角洲筛选的城市是旧金山。
```sql
plyql -h 192.168.60.100:8082 -i P1Y -q '
SELECT 
QUANTILE(delta_hist WHERE cityName = "San Francisco", 0.95) as P95 
FROM wikipedia;
'
```

它也是可能做到多维度分组查询的
```sql
plyql -h 192.168.60.100:8082 -i P1Y -q '
SELECT TIME_BUCKET(__time, PT1H, "Etc/UTC") as Hour, 
page as PageName, 
SUM(added) as TotalAdded 
FROM wikipedia 
GROUP BY 1, 2 
ORDER BY TotalAdded DESC 
LIMIT 3;
'
```

返回:

```json
[
  {
    "TotalAdded": 242211,
    "PageName": "Wikipedia‐ノート:即時削除の方針/過去ログ16",
    "Hour": {
      "start": "2015-09-12T15:00:00.000Z",
      "end": "2015-09-12T16:00:00.000Z",
      "type": "TIME_RANGE"
    }
  },
  {
    "TotalAdded": 232941,
    "PageName": "Користувач:SuomynonA666/Заготовка",
    "Hour": {
      "start": "2015-09-12T14:00:00.000Z",
      "end": "2015-09-12T15:00:00.000Z",
      "type": "TIME_RANGE"
    }
  },
  {
    "TotalAdded": 214017,
    "PageName": "User talk:Estela.rs",
    "Hour": {
      "start": "2015-09-12T12:00:00.000Z",
      "end": "2015-09-12T13:00:00.000Z",
      "type": "TIME_RANGE"
    }
  }
]
```

这里是一个高级的示例，获取前 5 页编辑时间。
PlyQL 的一个显著特征是它对待 （aka 表） 的数据集作为可以嵌套在另一个表的只是另一种数据类型。
这使我们能够嵌套查询数据像这样︰
```sql
plyql -h 192.168.60.100:8082 -i P1Y -q '
SELECT page as Page, 
COUNT() as cnt, 
(
  SELECT 
  SUM(added) as TotalAdded 
  GROUP BY TIME_BUCKET(__time, PT1H, "Etc/UTC") 
  LIMIT 3 -- only get the first 3 hours to keep this example output small
) as "ByTime" 
FROM wikipedia 
GROUP BY page 
ORDER BY cnt DESC 
LIMIT 5;
'
```

返回:

```json
[
  {
    "cnt": 314,
    "Page": "Jeremy Corbyn",
    "ByTime": [
      {
        "TotalAdded": 1075,
        "split0": {
          "start": "2015-09-12T01:00:00.000Z",
          "end": "2015-09-12T02:00:00.000Z",
          "type": "TIME_RANGE"
        }
      },
      {
        "TotalAdded": 0,
        "split0": {
          "start": "2015-09-12T07:00:00.000Z",
          "end": "2015-09-12T08:00:00.000Z",
          "type": "TIME_RANGE"
        }
      },
      {
        "TotalAdded": 10553,
        "split0": {
          "start": "2015-09-12T08:00:00.000Z",
          "end": "2015-09-12T09:00:00.000Z",
          "type": "TIME_RANGE"
        }
      }
    ]
  },
  {
    "cnt": 255,
    "Page": "User:Cyde/List of candidates for speedy deletion/Subpage",
    "ByTime": [
      {
        "TotalAdded": 73,
        "split0": {
          "start": "2015-09-12T00:00:00.000Z",
          "end": "2015-09-12T01:00:00.000Z",
          "type": "TIME_RANGE"
        }
      },
      {
        "TotalAdded": 3363,
        "split0": {
          "start": "2015-09-12T01:00:00.000Z",
          "end": "2015-09-12T02:00:00.000Z",
          "type": "TIME_RANGE"
        }
      },
      {
        "TotalAdded": 336,
        "split0": {
          "start": "2015-09-12T02:00:00.000Z",
          "end": "2015-09-12T03:00:00.000Z",
          "type": "TIME_RANGE"
        }
      }
    ]
  },
  "... results omitted ..."
]
```

## 运算符

名称                    | 描述
------------------------|-------------------------------------
+                       | 加法运算符
-                       | 减号运算符 / 一元求反
*                       | 乘法运算符
/                       | 除法运算符
=, IS                   | 等于运算符
!=, <>, IS NOT          | 不等于运算符
>=                      | 大于或等于运算符
>                       | 大于运算符
<=                      | 小于或等于运算符
<                       | 小于运算符
BETWEEN ... AND ...     | 检查一个值是否在值范围内
NOT BETWEEN ... AND ... | 检查一个值是否不在值范围内
LIKE, CONTAINS          | 简单的模式(模糊)匹配
NOT LIKE, CONTAINS      | 否定的简单模式(模糊)匹配
REGEXP                  | 模式匹配使用正则表达式
NOT REGEXP              | 正则表达式的否定
NOT, !                  | 取否
AND                     | 逻辑并
OR                      | 逻辑或


## 函数

<a id="TIME_BUCKET" href="#TIME_BUCKET">#</a>
**TIME_BUCKET**(operand, duration, timezone)

将时间在给定的`timezone(时区)`中按给定的`duration(粒度)`查询

示例: `TIME_BUCKET(time, 'P1D', 'America/Los_Angeles')`

这将把`time`变量装入日期块，其中按`America/Los_Angeles`时区的天粒度定义。

<a id="TIME_PART" href="#TIME_PART">#</a>
**TIME_PART**(operand, part, timezone)

按指定的时区获取对应参数的时间参数值

例如: `TIME_PART(time, 'DAY_OF_YEAR', 'America/Los_Angeles')`
这将把'time`变量分成（整数）数字，表示一年中的哪一天。
可能的部分值是:

* `SECOND_OF_MINUTE`, `SECOND_OF_HOUR`, `SECOND_OF_DAY`, `SECOND_OF_WEEK`, `SECOND_OF_MONTH`, `SECOND_OF_YEAR`
* `MINUTE_OF_HOUR`, `MINUTE_OF_DAY`, `MINUTE_OF_WEEK`, `MINUTE_OF_MONTH`, `MINUTE_OF_YEAR`
* `HOUR_OF_DAY`, `HOUR_OF_WEEK`, `HOUR_OF_MONTH`, `HOUR_OF_YEAR`
* `DAY_OF_WEEK`, `DAY_OF_MONTH`, `DAY_OF_YEAR`
* `WEEK_OF_MONTH`, `WEEK_OF_YEAR`
* `MONTH_OF_YEAR`
* `YEAR`


<a id="TIME_FLOOR" href="#TIME_FLOOR">#</a>
**TIME_FLOOR**(operand, duration, timezone)

在给定的`timezone(时区)`中将时间置于最近的`duration(粒度)`。
示例: `TIME_FLOOR(time, 'P1D', 'America/Los_Angeles')`

这将把`time`变量置于一天的开始，其中天在`America/Los_Angeles`时区中定义。

<a id="TIME_SHIFT" href="#TIME_SHIFT">#</a>
**TIME_SHIFT**(operand, duration, step, timezone)

在给定的`timezone(时区)`中，通过`duration(粒度)` * `step`向前移动时间。
`step`可能是负数。

示例: `TIME_SHIFT(time, 'P1D', -2, 'America/Los_Angeles')`

这将在两天后移动`time`变量，其中天在`America/Los_Angeles`时区中定义。

<a id="TIME_RANGE" href="#TIME_RANGE">#</a>
**TIME_RANGE**(operand, duration, step, timezone)

在给定的`timezone(时区)`中创建一个范围形式`time`和一个`duration` *`step`远离`time`的点。
`step`可能是负数。

示例: `TIME_RANGE(time, 'P1D', -2, 'America/Los_Angeles')`

这将在两天后移动`time`变量，其中天在`America/Los_Angeles`时区中定义，并创建一个时间-2 * P1D - >时间范围。

<a id="SUBSTR" href="#SUBSTR">#</a>
**SUBSTR**(*str*, *pos*, *len*)

从字符串*str*返回一个子串*len*个字符，从位置*pos*开始

<a id="CONCAT" href="#CONCAT">#</a>
**CONCAT**(*str1*, *str2*, ...)

返回由连接参数产生的字符串。 可以有一个或多个参数

<a id="EXTRACT" href="#EXTRACT">#</a>
**EXTRACT**(*str*, *regexp*)

返回第一个匹配的组，结果窗体匹配 *regexp* 到 *str*

<a id="LOOKUP" href="#LOOKUP">#</a>
**LOOKUP**(*str*, *lookup-namespace*)

返回键的值 *str* 内 *查找-命名空间*

<a id="IFNULL" href="#IFNULL">#</a>
**IFNULL**(*expr1*, *expr2*)

返回 *expr* 如果不为空，否则返回 *expr2*

<a id="FALLBACK" href="#FALLBACK">#</a>
**FALLBACK**(*expr1*, *expr2*)

这个等同于 **IFNULL**(*expr1*, *expr2*)

<a id="OVERLAP" href="#OVERLAP">#</a>
**OVERLAP**(*expr1*, *expr2*)

检查 *expr1* 和 *expr2* 是否重叠

### Mathematical Functions

<a id="ABS" href="#ABS">#</a>
**ABS**(*expr*)

返回*expr*的绝对值。

<a id="POW" href="#POW">#</a>
**POW**(*expr1*, *expr2*)

返回 *expr1* 的幂 *expr2*。

<a id="POWER" href="#POWER">#</a>
**POWER**(*expr1*, *expr2*)  

这个等同于 **POW**(*expr1*, *expr2*)  

<a id="SQRT" href="#SQRT">#</a>
**SQRT**(*expr*)

返回*expr*的平方根

<a id="EXP" href="#EXP">#</a>
  **EXP**(*expr*)

返回 e（自然对数的基数） 的值*expr*的幂。

## Aggregations

<a id="COUNT" href="#COUNT">#</a>
**COUNT**(*expr?*)

如果在没有表达式的情况下使用（或者作为`COUNT(*)`）返回行数的计数。
当提供表达式时，返回*expr*不为null的行的计数。

<a id="COUNT" href="#COUNT">#</a>
**COUNT**(**DISTINCT** *expr*), **COUNT_DISTINCT**(*expr*) 

返回具有不同*expr*值的行数的计数。

<a id="SUM" href="#SUM">#</a>
**SUM**(*expr*) 

返回所有*expr*值的总和。

<a id="MIN" href="#MIN">#</a>
**MIN**(*expr*) 

返回所有*expr*值的最小值。

<a id="MAX" href="#MAX">#</a>
**MAX**(*expr*) 

返回所有*expr*值的最大值。

<a id="AVG" href="#AVG">#</a>
**AVG**(*expr*)

返回所有*expr*值的平均值。

<a id="QUANTILE" href="#QUANTILE">#</a>
**QUANTILE**(*expr*, *quantile*)

返回所有*expr*值的上层*分位数*。

<a id="CUSTOM" href="#CUSTOM">#</a>
**CUSTOM**(*custom_name*)

返回名为*custom_name*的用户定义聚合。
