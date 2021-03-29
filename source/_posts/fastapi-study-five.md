---
title: FastAPI(五)
date: 2021-03-16 20:33:53
tags:
  - FastAPI
  - Python
  - 学习笔记
categories:
  - Python
  - FastAPI
---

## Cookie

### 导入 Cookie

首先，从 FastAPI 中导入 Cookie :

```python
from fastapi import Cookie
```

### 声明 Cookie 

```python
from typing import Optional

from fastapi import Cookie, FastAPI

app = FastAPI()


@app.get("/items/")
async def read_items(ads_id: Optional[str] = Cookie(None)):
    return {"ads_id": ads_id}
```

## Header

### 导入 Header

首先导入 Header : 

```python
from typing import Optional

from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(user_agent: Optional[str] = Header(None)):
    return {"User-Agent": user_agent}
```

### 声明 Header

```python
from typing import Optional

from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(user_agent: Optional[str] = Header(None)):
    return {"User-Agent": user_agent}
```

### 自动转换

大多数标准的headers用 "连字符" 分隔，也称为 "减号" (`-`)。

但是像 `user-agent` 这样的变量在Python中是无效的。

因此, 默认情况下, `Header` 将把参数名称的字符从下划线 (`_`) 转换为连字符 (`-`) 来提取并记录 headers.

同时，HTTP headers 是大小写不敏感的，因此，因此可以使用标准Python样式(也称为 "snake_case")声明它们。

如果需要禁用下划线到连字符的自动转换，设置`Header`的参数 `convert_underscores` 为 `False`:

```python
from typing import Optional

from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(
    strange_header: Optional[str] = Header(None, convert_underscores=False)
):
    return {"strange_header": strange_header}
```

## 响应模型

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
    tags: List[str] = []


@app.post("/items/", response_model=Item)
# 注意，response_model是「装饰器」方法（get，post 等）的一个参数。
async def create_item(item: Item):
    return item
```

FastAPI 将使用此 `response_model` 来：

- 将输出数据转换为其声明的类型。
- 校验数据。
- 在 OpenAPI 的*路径操作*中为响应添加一个 JSON Schema。
- 并在自动生成文档系统中使用。

- 会将输出数据限制在该模型定义内。

### 返回与输入相同的数据

现在声明一个 `UserIn` 模型，它将包含一个明文密码属性。并使用同一模型声明输出数据：

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: Optional[str] = None

        
@app.post("/user/", response_model=UserIn)
async def create_user(user: UserIn):
    return user
```

每当浏览器使用一个密码创建用户时，API 都会在响应中返回相同的密码。

### 添加输出模型

可以创建一个有明文密码的输入模型和一个没有明文密码的输出模型：

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: Optional[str] = None


class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: Optional[str] = None


@app.post("/user/", response_model=UserOut)
async def create_user(user: UserIn):
    return user
```

### 响应模型编码参数

响应模型可以具有默认值，例如：

```python
from typing import List, Optional

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: float = 10.5
    tags: List[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item, response_model_exclude_unset=True)
async def read_item(item_id: str):
    return items[item_id]
```

- `description: Optional[str] = None` 具有默认值 `None`。
- `tax: float = 10.5` 具有默认值 `10.5`.
- `tags: List[str] = []` 具有一个空列表作为默认值： `[]`.

### 使用 `response_model_exclude_unset` 参数

```python
from typing import List, Optional

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: float = 10.5
    tags: List[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item, response_model_exclude_unset=True)
async def read_item(item_id: str):
    return items[item_id]
```

作用是：响应中将不会包含那些默认值，而是仅有实际设置的值。

## 额外的模型

- **输入模型**需要拥有密码属性。
- **输出模型**不应该包含密码。
- **数据库模型**很可能需要保存密码的哈希值。

### 多个模型

下面是应该如何根据它们的密码字段以及使用位置去定义模型的大概思路：

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: Optional[str] = None


class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: Optional[str] = None


class UserInDB(BaseModel):
    username: str
    hashed_password: str
    email: EmailStr
    full_name: Optional[str] = None


def fake_password_hasher(raw_password: str):
    return "supersecret" + raw_password


def fake_save_user(user_in: UserIn):
    hashed_password = fake_password_hasher(user_in.password)
    user_in_db = UserInDB(**user_in.dict(), hashed_password=hashed_password)
    print("User saved! ..not really")
    return user_in_db


@app.post("/user/", response_model=UserOut)
async def create_user(user_in: UserIn):
    user_saved = fake_save_user(user_in)
    return user_saved
```

#### 关于 `**user_in.dict()`

##### Pydantic 的 `.dict()`

`user_in` 是一个 `UserIn` 类的 Pydantic 模型.

Pydantic 模型具有 `.dict（）` 方法，该方法返回一个拥有模型数据的 `dict`。

因此，如果像下面这样创建一个 Pydantic 对象 `user_in`：

```python
user_in = UserIn(username="john", password="secret", email="john.doe@example.com")
```

然后调用：

```python
user_dict = user_in.dict()
```

现在有了一个数据位于变量 `user_dict` 中的 `dict`（它是一个 `dict` 而不是 Pydantic 模型对象）。

如果调用：

```python
print(user_dict)
```

将获得一个这样的 Python `dict`：

```python
{
    'username': 'john',
    'password': 'secret',
    'email': 'john.doe@example.com',
    'full_name': None,
}
```

##### 解包 `dict`

如果将 `user_dict` 这样的 `dict` 以 `**user_dict` 形式传递给一个函数（或类），Python将对其进行「解包」。它会将 `user_dict` 的键和值作为关键字参数直接传递。

从上面的 `user_dict` 继续，编写：

```python
UserInDB(**user_dict)
```

会产生类似于以下的结果：

```python
UserInDB(
    username="john",
    password="secret",
    email="john.doe@example.com",
    full_name=None,
)
```

或者更确切地，直接使用 `user_dict` 来表示将来可能包含的任何内容：

```python
UserInDB(
    username = user_dict["username"],
    password = user_dict["password"],
    email = user_dict["email"],
    full_name = user_dict["full_name"],
)
```

所以：

```python
user_dict = user_in.dict()
UserInDB(**user_dict)
```

等同于：

```python
UserInDB(**user_in.dict())
```

##### 添加额外的关键字

可以在解包的同时添加额外的关键字

```python
UserInDB(**user_in.dict(), hashed_password=hashed_password)
```

最终的结果如下：

```python
UserInDB(
    username = user_dict["username"],
    password = user_dict["password"],
    email = user_dict["email"],
    full_name = user_dict["full_name"],
    hashed_password = hashed_password,
)
```

### 优化减少重复

上面的这些模型都共享了大量数据，并拥有重复的属性名称和类型。

可以声明一个 `UserBase` 模型作为其他模型的基类。然后可以创建继承该模型属性（类型声明，校验等）的子类。

所有的数据转换、校验、文档生成等仍将正常运行。

这样，可以仅声明模型之间的差异部分（具有明文的 `password`、具有 `hashed_password` 以及不包括密码）。

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()



class UserBase(BaseModel):

    username: str
    email: EmailStr
    full_name: Optional[str] = None



class UserIn(UserBase):

    password: str




class UserOut(UserBase):

    pass




class UserInDB(UserBase):

    hashed_password: str



def fake_password_hasher(raw_password: str):
    return "supersecret" + raw_password


def fake_save_user(user_in: UserIn):
    hashed_password = fake_password_hasher(user_in.password)
    user_in_db = UserInDB(**user_in.dict(), hashed_password=hashed_password)
    print("User saved! ..not really")
    return user_in_db


@app.post("/user/", response_model=UserOut)
async def create_user(user_in: UserIn):
    user_saved = fake_save_user(user_in)
    return user_saved

```

