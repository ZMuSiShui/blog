---
title: FastAPI(三) - Request Body
date: 2021-03-12 22:40:01
tags:
  - FastAPI
  - Python
  - 学习笔记
categories:
  - Python
  - FastAPI
---

# Request Body

## 多个参数

我们已经知道了如何使用 `Path` 和 `Query`，下面来了解一下 Request Body 声明的更高级用法。

### 混合使用 Path 、Query 和 Request Body 参数

以通过将默认值设置为 `None` 来将请求体参数声明为可选参数：

```python
from typing import Optional

from fastapi import FastAPI, Path
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


@app.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int = Path(..., title="The ID of the item to get", ge=0, le=1000),
    q: Optional[str] = None,
    item: Optional[Item] = None,
):
    # 在这种情况下，将从请求体获取的 item 是可选的。因为它的默认值为 None
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    if item:
        results.update({"item": item})
    return results
```

### 多个 Request Body 参数

在上面的示例中，函数将期望一个具有 `Item` 的属性的 JSON 请求体，就像：

```json
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2
}
```

但是也可以声明多个请求体参数，例如 `item` 和 `user`：

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


class User(BaseModel):
    username: str
    full_name: Optional[str] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```

在这种情况下，**FastAPI** 将期望一个具有 Item 和 User 的请求体集合。它将使用参数名称作为请求体中的键（字段名称），类似于以下内容的请求体：

```python
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    },
    "user": {
        "username": "dave",
        "full_name": "Dave Grohl"
    }
}
```

它将执行对复合数据的校验，并且像现在这样为 OpenAPI 模式和自动化文档对其进行记录。

### 请求体中的单一值

```python
from typing import Optional

from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


class User(BaseModel):
    username: str
    full_name: Optional[str] = None


@app.put("/items/{item_id}")
async def update_item(
    item_id: int, item: Item, user: User, importance: int = Body(...)
):
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    return results
```

在这种情况下，**FastAPI** 将期望像这样的请求体：

```python
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    },
    "user": {
        "username": "dave",
        "full_name": "Dave Grohl"
    },
    "importance": 5
}
```

### 嵌入单个请求体参数

如果期望一个在值中包含 `item` 键内容的 JSON，可以使用一个特殊的 `Body` 参数 `embed`

```python
item: Item = Body(..., embed=True)
```

比如：

```python
from typing import Optional

from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item = Body(..., embed=True)):
    results = {"item_id": item_id, "item": item}
    return results
```

在这种情况下，**FastAPI** 将期望像这样的请求体：

```json
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    }
}
```

而不是：

```json
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2
}
```

## 字段

使用 Pydantic 的 `Field` 在 Pydantic 模型内部声明校验和元数据

### 导入 Field

```
from pydantic import Field
```

### 声明模型属性

然后，对模型属性使用 `Field`：

```python
from typing import Optional

from fastapi import Body, FastAPI
from pydantic import BaseModel, Field

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = Field(
        None, title="The description of the item", max_length=300
    )
    price: float = Field(..., gt=0, description="The price must be greater than zero")
    tax: Optional[float] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item = Body(..., embed=True)):
    results = {"item_id": item_id, "item": item}
    return results
```

`Field` 的工作方式和 `Query`、`Path` 和 `Body` 相同，包括它们的参数等等也完全相同。

实际上，`Query`、`Path` 和类似的类，创建的是由一个共同的 `Params` 类派生的子类的对象，该共同类本身又是 Pydantic 的 `FieldInfo` 类的子类。

Pydantic 的 `Field` 也会返回一个 `FieldInfo` 的实例。

`Body` 也直接返回 `FieldInfo` 的一个子类的对象。

当从 `fastapi` 导入 `Query`、`Path` 等对象时，他们实际上是返回特殊类的函数。

### 添加额外信息

可以在 `Field`、`Query`、`Body` 中声明额外的信息。这些信息将包含在生成的 JSON Schema 中。

## 嵌套模型

### List

可以将一个属性定义为拥有子元素的 `list` 类型：

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: list = []


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

这将使 `tags` 成为一个由元素组成的列表，不过它没有声明每个元素的类型。

### 具有子类型的 List

#### 从 typing 导入 List

首先，从 Python 的标准库 `typing` 模块中导入 `List`：

```python
from typing import List
```

### 声明具有子类型的 List

要声明具有子类型的类型，例如 `list`、`dict`、`tuple`：

- 从 `typing` 模块导入它们
- 使用方括号 `[` 和 `]` 将子类型作为「类型参数」传入

```python
from typing import List

my_list: List[str]
```

因此，在示例中，可以将 `tags` 明确地指定为一个「字符串列表」：

```python
from typing import List, Optional

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    # 声明 tags 字符串列表
    tags: List[str] = []


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

### Set

Python 具有一种特殊的数据类型来保存一组唯一的元素，即 `set`

然后可以导入 `Set` 并将 `tag` 声明为一个由 `str` 组成的 `set`：

```python
from typing import Optional, Set

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: Set[str] = set()


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

### 嵌套模型

Pydantic 模型的每个属性都具有类型

但是这个类型本身可以是另一个 Pydantic 模型

因此，可以声明拥有特定属性名称、类型和校验的深度嵌套的 JSON 对象

### 定义子模型

例如，可以定义一个 `Image` 模型：

```python
class Image(BaseModel):
    url: str
    name: str
```

### 将子模型用作类型

然后将其用于一个属性的类型：

```python
from typing import Optional, Set

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Image(BaseModel):
    url: str
    name: str


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: Set[str] = []
    # 声明 image 的类型为 Image
    image: Optional[Image] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

这意味着 **FastAPI** 将期望类似于以下内容的请求体：

```python
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2,
    "tags": ["rock", "metal", "bar"],
    "image": {
        "url": "http://example.com/baz.jpg",
        "name": "The Foo live"
    }
}
```

仅仅进行这样的声明，将通过 **FastAPI** 获得：

- 对被嵌入的模型也适用的编辑器支持（自动补全等）
- 数据转换
- 数据校验
- 自动生成文档

### 特殊的类型和校验

除了普通的单一值类型（如 `str`、`int`、`float` 等）外，还可以使用从 `str` 继承的更复杂的单一值类型。

例如，在 `Image` 模型中我们有一个 `url` 字段，可以把它声明为 Pydantic 的 `HttpUrl`，而不是 `str`：

```python
from typing import Optional, Set

from fastapi import FastAPI
# 导入 HttpUrl
from pydantic import BaseModel, HttpUrl

app = FastAPI()


class Image(BaseModel):
	# 声明类型为 HttpUrl
    url: HttpUrl
    name: str


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: Set[str] = set()
    image: Optional[Image] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

该字符串将被检查是否为有效的 URL，并在 JSON Schema / OpenAPI 文档中进行记录。

### 带有一组子模型的属性

还可以将 Pydantic 模型用作 `list`、`set` 等的子类型：

```python
from typing import List, Optional, Set

from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()


class Image(BaseModel):
    url: HttpUrl
    name: str


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: Set[str] = set()
    # 将 images 声明为 元素类型为 Image 的列表
    images: Optional[List[Image]] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

这将期望（转换，校验，记录文档等）下面这样的 JSON 请求体：

```python
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2,
    "tags": [
        "rock",
        "metal",
        "bar"
    ],
    "images": [
        {
            "url": "http://example.com/baz.jpg",
            "name": "The Foo live"
        },
        {
            "url": "http://example.com/dave.jpg",
            "name": "The Baz"
        }
    ]
}
```

### 深度嵌套模型

可以定义任意深度的嵌套模型：

```python
from typing import List, Optional, Set

from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()


class Image(BaseModel):
    url: HttpUrl
    name: str


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: Set[str] = set()
    # 将 images 声明为 元素类型为 Image 的列表
    images: Optional[List[Image]] = None


class Offer(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    # 将 items 声明为 元素类型为 Item 的列表
    items: List[Item]


@app.post("/offers/")
async def create_offer(offer: Offer):
    return offer
```

### 纯列表请求体

如果期望的 JSON 请求体的最外层是一个 JSON `array`，则可以在路径操作函数的参数中声明此类型，就像声明 Pydantic 模型一样：

```python
images: List[Image]
```

例如：

```python
from typing import List

from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()


class Image(BaseModel):
    url: HttpUrl
    name: str


@app.post("/images/multiple/")
async def create_multiple_images(images: List[Image]):
    return images
```

### 任意 dict 构成的请求体

可以将请求体声明为使用某类型的键和其他类型值的 `dict`

在下面的例子中，将接受任意键为 `int` 类型并且值为 `float` 类型的 `dict`：

```python
from typing import Dict

from fastapi import FastAPI

app = FastAPI()


@app.post("/index-weights/")
async def create_index_weights(weights: Dict[int, float]):
    return weights
```

**Tip**

- JSON 仅支持将 `str` 作为键
- Pydantic 具有自动转换数据的功能

### 总结

Pydantic 模型可以提供的极高灵活性，同时保持代码的简单、简短和优雅

- 编辑器支持（处处皆可自动补全！）
- 数据转换
- 数据校验
- 模式文档
- 自动生成文档