---
title: FastAPI(三) - 数据类型
date: 2021-03-12 22:40:20
tags:
  - FastAPI
  - Python
  - 学习笔记
categories:
  - Python
  - FastAPI
---

 # 常见数据类型

到目前为止，常见的数据类型有:

- `int`
- `float`
- `str`
- `bool`

这些数据类型会拥有相同的特性:

- 编辑器支持
- 传入请求的数据转换
- 响应数据转换
- 数据验证
- 自动补全文档

# 其他数据类型

- `UUID`:
  - 一种标准的 "通用唯一标识符" ，在许多数据库和系统中用作ID。
  - 在请求和响应中将以 `str` 表示。

- `datetime.datetime`:
  - 一个 Python `datetime.datetime`.
  - 在请求和响应中将表示为 ISO 8601 格式的 `str` ，比如: `2008-09-15T15:53:00+05:00`.
- `datetime.date`:
  - Python `datetime.date`.
  - 在请求和响应中将表示为 ISO 8601 格式的 `str` ，比如: `2008-09-15`.
- `datetime.time`:
  - 一个 Python `datetime.time`.
  - 在请求和响应中将表示为 ISO 8601 格式的 `str` ，比如: `14:23:55.003`.
- `datetime.timedelta`:
  - 一个 Python `datetime.timedelta`.
  - 在请求和响应中将表示为 `float` 代表总秒数。
  - Pydantic 也允许将其表示为 "ISO 8601 时间差异编码"
- `frozenset`:
  - 在请求和响应中，作为 `set` 对待：
    - 在请求中，列表将被读取，消除重复，并将其转换为一个 `set`。
    - 在响应中 `set` 将被转换为 `list` 。
    - 产生的模式将指定那些 `set` 的值是唯一的 
- `bytes`:
  - 标准的 Python `bytes`。
  - 在请求和相应中被当作 `str` 处理。
  - 生成的模式将指定这个 `str` 是 `binary` "格式"。
- `Decimal`:
  - 标准的 Python `Decimal`。
  - 在请求和相应中被当做 `float` 一样处理。

## 例子

下面是一个示例，其中的参数使用了上面的一些类型。

```python
from datetime import datetime, time, timedelta
from typing import Optional
from uuid import UUID

from fastapi import Body, FastAPI

app = FastAPI()


@app.put("/items/{item_id}")
async def read_items(
    item_id: UUID,
    start_datetime: Optional[datetime] = Body(None),
    end_datetime: Optional[datetime] = Body(None),
    repeat_at: Optional[time] = Body(None),
    process_after: Optional[timedelta] = Body(None),
):
    start_process = start_datetime + process_after
    duration = end_datetime - start_process
    return {
        "item_id": item_id,
        "start_datetime": start_datetime,
        "end_datetime": end_datetime,
        "repeat_at": repeat_at,
        "process_after": process_after,
        "start_process": start_process,
        "duration": duration,
    }
```

注意，函数内的参数有原生的数据类型，例如执行正常的日期操作，如:

```python
from datetime import datetime, time, timedelta
from typing import Optional
from uuid import UUID

from fastapi import Body, FastAPI

app = FastAPI()


@app.put("/items/{item_id}")
async def read_items(
    item_id: UUID,
    start_datetime: Optional[datetime] = Body(None),
    end_datetime: Optional[datetime] = Body(None),
    repeat_at: Optional[time] = Body(None),
    process_after: Optional[timedelta] = Body(None),
):
    start_process = start_datetime + process_after
    duration = end_datetime - start_process
    return {
        "item_id": item_id,
        "start_datetime": start_datetime,
        "end_datetime": end_datetime,
        "repeat_at": repeat_at,
        "process_after": process_after,
        "start_process": start_process,
        "duration": duration,
    }
```

