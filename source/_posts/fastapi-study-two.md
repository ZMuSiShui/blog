---
title: FastAPI(二)
date: 2021-03-11 21:25:23
tags:
  - FastAPI
  - Python
  - 学习笔记
categories:
  - Python
  - FastAPI
---

# 路径参数

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

路径参数的值`item_id`将作为参数传递给函数`item_id`。

运行此示例并转到 [http://127.0.0.1:8000/items/zhangsan](http://127.0.0.1:8000/items/zhangsan) ，将看到以下响应：

```json
{"item_id":"zhangsan"}
```

## 路径参数可指定参数类型

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

`item_id`被声明为`int`

运行此示例并转到 [http://127.0.0.1:8000/items/zhangsan](http://127.0.0.1:8000/items/zhangsan) ，将看到以下响应：

```json
{
    "detail":[
        {
            "loc":[
                "path",
                "item_id"
            ],
            "msg":"value is not a valid integer",
            "type":"type_error.integer"
        }
    ]
}
```

因为路径参数`item_id`的值为`"zhangsan"`，而不是`int`。

如果提供`float`而不是，则会出现相同的错误`int`

**因此，FastAPI 也可以提供数据验证。**

当访问 [http://127.0.0.1:8000/items/3](http://127.0.0.1:8000/items/3) 时：

```json
{"item_id":3}
```

**注意**：`3`，作为Python `int`，函数收到（返回）的值不是string `"3"`。

使用该类型声明，**FastAPI** 将提供自动请求“解析”。

# 查询参数

当声明不属于路径参数的其他功能参数时，它们将自动解释为“查询”参数。

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

该查询是`?`URL中位于后面的键值对的集合，以`&`字符分隔。

例如，在URL中：

```
http://127.0.0.1:8000/items/?skip=0&limit=10
```

...查询参数为：

- `skip`：值为 `0`
- `limit`：值为 `10`

由于它们是URL的一部分，因此它们是“自然”的字符串。

但是，当使用Python类型声明它们时（在上面的示例中为`int`），它们将转换为该类型并针对该类型进行验证。

应用于路径参数的所有相同过程也适用于查询参数：

- 编辑器支持（显然）
- 数据“解析”
- 资料验证
- 自动文档

## 默认值

由于查询参数不是路径的固定部分，因此它们可以是可选的，并且可以具有默认值。

在上面的示例中，它们的默认值为`skip=0`和`limit=10`。

因此，转到URL：

```
http://127.0.0.1:8000/items/
```

将与执行以下操作相同：

```
http://127.0.0.1:8000/items/?skip=0&limit=10
```

但是，例如，如果您去：

```
http://127.0.0.1:8000/items/?skip=20
```

函数中的参数值为：

- `skip=20`：因为您在网址中进行了设置
- `limit=10`：因为这是默认值

## 可选参数

同样，可以通过将可选查询参数的默认值设置为来声明可选查询参数`None`：

```python
from typing import Optional

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Optional[str] = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

在这种情况下，函数参数`q`将是可选的，并且`None`默认情况下为默认值。

**注意**，路径参数`item_id`是一个路径参数，`q`而不是路径参数，因此它是一个查询参数。

## 查询参数类型转换

还可以声明`bool`类型，它们将被转换：

```python
from typing import Optional

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Optional[str] = None, short: bool = False):
    item = {"item_id": item_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```

在这种情况下，如果您要执行以下操作：

```
http://127.0.0.1:8000/items/foo?short=1
```

或者

```
http://127.0.0.1:8000/items/foo?short=True
```

或者

```
http://127.0.0.1:8000/items/foo?short=true
```

或者

```
http://127.0.0.1:8000/items/foo?short=on
```

或者

```
http://127.0.0.1:8000/items/foo?short=yes
```

或任何其他大小写变体（大写，大写的首字母等），函数将看到`short`带有`bool`值的参数`True`。否则为`False`。

## 多个路径和查询参数

可以同时声明多个路径参数和查询参数。

而且不必以任何特定的顺序声明它们。

它们将通过名称进行检测：

```python
from typing import Optional

from fastapi import FastAPI

app = FastAPI()


@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: Optional[str] = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```

## 必须的查询参数

当您为查询参数声明默认值时，则不需要此值。

如果不想添加特定值，而只是将其设为可选值，则将默认值设置为`None`。

但是，当需要一个查询参数时，就不能声明任何默认值：

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item
```

这里的查询参数`needy`是类型的必需查询参数`str`。

如果您在浏览器中打开一个URL，例如：

```
http://127.0.0.1:8000/items/foo-item
```

...没有添加必需的参数`needy`，您将看到类似以下的错误：

```json
{
    "detail": [
        {
            "loc": [
                "query",
                "needy"
            ],
            "msg": "field required",
            "type": "value_error.missing"
        }
    ]
}
```

作为`needy`必填参数，您需要在URL中进行设置：

```
http://127.0.0.1:8000/items/foo-item?needy=sooooneedy
```

...这将起作用：

```json
{
    "item_id": "foo-item",
    "needy": "sooooneedy"
}
```

当然，您可以根据需要定义一些参数，一些具有默认值，而某些则完全可选：

```python
from typing import Optional

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_user_item(
    item_id: str, needy: str, skip: int = 0, limit: Optional[int] = None
):
    item = {"item_id": item_id, "needy": needy, "skip": skip, "limit": limit}
    return item
```

在这种情况下，有3个查询参数：

- `needy`，是必需的`str`。
- `skip`，`int`默认值为`0`。
- `limit`，可选的`int`。

# Request Body

当需要将数据从客户端（例如，浏览器）发送到API时，可以将其作为**请求正文**发送。

要发送数据，应该使用以下一项`POST`（较常见）`PUT`，`DELETE`或`PATCH`。

## 导入 Pydantic 的 BaseModel

首先，需要导入`pydantic` 中的 `BaseModel`：

```python
from pydantic import BaseModel
```

## 创建数据模型

将数据模型声明为继承自的类`BaseModel`。

对所有属性使用标准Python类型：

```python
class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
```

与声明查询参数时相同，当模型属性具有默认值时，则不需要此属性。否则，它是必需的。使用`None`使其仅是可选的。

例如，上面的该模型声明一个JSON“ `object`”（或Python `dict`），例如：

```json
{
  "name": "iPhone4s",
  "description": "Phone",
  "price": 4999,
  "tax": 4999
}
```

...由于`description`和`tax`是可选的（默认值为`None`），此JSON“ `object`”也将有效：

```json
{
  "name": "iPhone4s",
  "price": 4999
}
```

## 声明为参数

```python
@app.post("/items/")
async def create_item(item: Item):
    return item
```

将 `item` 类型声明为创建的模型，`Item`。

## 结果

仅使用该Python类型声明，**FastAPI**将：

- 以JSON格式读取请求的正文。
- 转换相应的类型（如果需要）。
- 验证数据。
  - 如果数据无效，它将返回一个清晰的错误，指出错误数据的确切位置和内容。
- 在参数中为您提供接收到的数据 `item` 。
  - 当您在函数中将其声明为type时`Item`，您还将获得所有属性及其类型的所有编辑器支持（完成等）。
- 为您的模型生成[JSON模式](https://json-schema.org/)定义，如果对您的项目有意义的话，您也可以在其他任何地方使用它们。
- 这些架构将是生成的OpenAPI架构的一部分，并由自动文档UI使用。

**如果使用 PyCharm 作为编辑器，则可以使用 Pydantic PyCharm 插件**

它改进了对Pydantic模型的编辑器支持，其中包括：

- 自动完成
- 类型检查
- 重构
- 搜寻
- 检查

## 使用模型

在函数内部，可以直接访问模型对象的所有属性：

```python
@app.post("/items/")
async def create_item(item: Item):
    item_dict = item.dict()
    if item.tax:
        price_with_tax = item.price + item.tax 	# 直接访问
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```

# Request Body + 路径参数

可以同时声明路径参数和请求正文。

**FastAPI** 将识别出与路径参数匹配的功能参数应从**路径中获取**，声明为 Pydantic 模型的功能参数应从 **Request Body** 中获取。

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


app = FastAPI()


@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.dict()}
```

# Request Body + 路径参数 + 查询参数

还可以同时声明 **Request Body** 、**路径参数** 、**查询参数**。

**FastAPI**将识别它们中的每一个，并从正确的位置获取数据。

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


app = FastAPI()


@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item, q: Optional[str] = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result
```

功能参数将被识别如下：

- 如果在**路径**中也声明了该参数，它将用作路径参数。
- 如果参数是一个的**单一类型**（如`int`，`float`，`str`，`bool`，等等）将被解释为一个**查询**参数。
- 如果参数声明为 **Pydantic模型** 的类型，则它将被解释为 **Request Body** 。

# 查询参数和字符串验证

```python
from typing import Optional

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/")
async def read_items(q: Optional[str] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

查询参数`q`的类型为`Optional[str]`，这意味着它的类型为`str`但也可以为类型`None`，并且默认值为`None`。

## 最长字符验证

例如：`q`是可选的，但只要提供了它，它的长度就不会超过50个字符

### 导入 Qurey

```python
from fastapi import Query
```

### 使用`Query`作为默认值

将其用作参数的默认值，将参数设置`max_length`为50：

```python
from typing import Optional

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Optional[str] = Query(None, max_length=50)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

Query(None, max_length=50)

- `Query` 的第一个参数用来定义一个默认值
- `Query` 的第二个参数用来设置字符串长度

所以，当 Query 只有第一个参数时：

```python
q: Optional[str] = Query(None)
```

等同于：

```python
q: Optional[str] = None
```

## 最短字符验证

还可以添加一个参数`min_length`：

```python
from typing import Optional

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Optional[str] = Query(None, min_length=3, max_length=50)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

## 正则表达式验证

可以定义参数应匹配的正则表达式：

```python
from typing import Optional

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Optional[str] = Query(None, min_length=3, max_length=50, regex="^fixedquery$")
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

这个正则表达式可以检查接收到的参数值：

- `^`：匹配输入字符串的开始位置。
- `fixedquery`：需要匹配的字符。
- `$`：匹配输入字符串的结尾位置。

## 默认值

可以向 `Query` 的第一个参数传入 `None` 用作查询参数的默认值

假设想要声明查询参数 `q`，使其 `min_length` 为 `3`，并且默认值为 `fixedquery`：

```python
from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: str = Query("fixedquery", min_length=3)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

## 声明为必需参数

当在使用 `Query` 且需要声明一个值是必需的时，可以将 `...` 用作第一个参数值：

```python
from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: str = Query(..., min_length=3)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

## 同时定义多个查询参数

当你使用 `Query` 显式地定义查询参数时，可以声明它去接收一组值。

例如，要声明一个可在 URL 中出现多次的查询参数 `q`，可以这样写：

```python
from typing import List, Optional

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Optional[List[str]] = Query(None)):
    query_items = {"q": q}
    return query_items
```

然后，输入如下网址：

```
http://localhost:8000/items/?q=foo&q=bar
```

该 URL 的响应将会是：

```json
{
  "q": [
    "foo",
    "bar"
  ]
}
```

## 总结

通用的校验和元数据：

- `alias`
  - `alias` 参数声明一个别名，该别名将用于在 URL 中查找查询参数值
- `title`
  - 添加标题
- `description`
  - 添加描述
- `deprecated`
  - 弃用参数，当 `deprecated=True` 时代表此参数已弃用

特定于字符串的校验：

- `min_length`
  - 最短字符串长度
- `max_length`
  - 最长字符串长度
- `regex`
  - 正则表达式

# 路径参数和数值校验

与使用 `Query` 为查询参数声明的方式相同，也可以使用 `Path` 为路径参数声明相同类型的校验。

## 导入 Path

```python
from fastapi import Path
```

## 声明元数据

可以声明与 `Query` 相同的所有参数。

例如，要声明路径参数 `item_id`的 `title` 元数据值，你可以输入：

```python
from typing import Optional

from fastapi import FastAPI, Path, Query

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    item_id: int = Path(..., title="The ID of the item to get"),
    q: Optional[str] = Query(None, alias="item-query"),
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results

# 路径参数总是必需的，因为它必须是路径的一部分。
# 所以，应该在声明时使用 ... 将其标记为必需参数。
# 然而，即使使用 None 声明路径参数或设置一个其他默认值也不会有任何影响，它依然会是必需参数。
```

## 对参数排序

假设想要声明一个必需的 `str` 类型查询参数 `q`。

而且不需要为该参数声明任何其他内容，所以实际上并不需要使用 `Query`。

但是仍然需要使用 `Path` 来声明路径参数 `item_id`。

如果将带有「默认值」的参数放在没有「默认值」的参数之前，Python 将会报错。

因此，你可以将函数声明为：

```python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    q: str, item_id: int = Path(..., title="The ID of the item to get")
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

或者：

```python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    *, item_id: int = Path(..., title="The ID of the item to get"), q: str
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

## 数值校验

### 大于等于

使用  `Query`  和  `Path` 可以声明字符串约束，但也可以声明数值约束。

像下面这样，添加 `ge=1` 后，`item_id` 将必须是一个大于（greater than）或等于（equal）`1` 的整数。

```python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    *, item_id: int = Path(..., title="The ID of the item to get", ge=1), q: str
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

### 大于和小于等于

同样的规则适用于：

- `gt`：大于（greater than）
- `le`：小于等于（less than or equal）

```python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    *,
    item_id: int = Path(..., title="The ID of the item to get", gt=0, le=1000),
    q: str,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

### 浮点数(大于和小于)

数值校验同样适用于 `float` 值。

```python
from fastapi import FastAPI, Path, Query

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    *,
    item_id: int = Path(..., title="The ID of the item to get", ge=0, le=1000),
    q: str,
    size: float = Query(..., gt=0, lt=10.5)
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

## 总结

数值校验：

- `gt`：大于（`g`reater `t`han）
- `ge`：大于等于（`g`reater than or `e`qual）
- `lt`：小于（`l`ess `t`han）
- `le`：小于等于（`l`ess than or `e`qual）