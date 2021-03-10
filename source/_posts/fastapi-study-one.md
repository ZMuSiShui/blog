---
title: FastAPI(一) - 快速入门
date: 2021-03-10 23:17:14
tags:
  - FastAPI
  - Python
  - 学习笔记
categories:
  - Python
  - FastAPI
---

# 简介

- **FastAPI 是一个高性能 Web 框架，用于构建 API。**

- **主要特性：**

- - 快速：非常高的性能，与 NodeJS 和 Go 相当
  - 快速编码：将功能开发速度提高约 200％ 至 300％
  - 更少的错误：减少约 40％ 的人为错误
  - 直观：强大的编辑器支持，自动补全无处不在，调试时间更少
  - 简易：旨在易于使用和学习，减少阅读文档的时间。
  - 简短：减少代码重复。
  - 稳健：获取可用于生产环境的代码，具有自动交互式文档
  - 基于标准：基于并完全兼容 API 的开放标准 OpenAPI 和 JSON Schema

- **官方链接：**https://fastapi.tiangolo.com/

# 安装 FastAPI

**注意：FastAPI 仅支持 Python 3.6 以上的版本**

- 快速安装

  ```shell
  pip install fastapi[all]
  ```

  以上安装还包括了 `uvicorn`，用作运行代码的服务器。

- 拆分安装

  - 安装 FastAPI

    ```shell
    pip install fastapi
    ```

  - 安装 uvicorn

    ```shell
    pip install uvicorn[standard]
    ```

  - 然后对想使用的每个可选依赖项也执行相同的操作即可

# Hello World

## 官网实例 

### main.py

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}

# 可以在此运行 uvicorn 服务
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000)
```

### 运行

```shell
$ uvicorn main:app --reload

INFO: Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO: Started reloader process [28720]
INFO: Started server process [28722]
INFO: Waiting for application startup.
INFO: Application startup complete.
```

`uvicorn main:app` 命令含义如下:

- `main`：`main.py` 文件（一个 Python「模块」）。
- `app`：在 `main.py` 文件中通过 `app = FastAPI()` 创建的对象。
- `--reload`：让服务器在更新代码后重新启动。仅在开发时使用该选项。

### 查看

打开浏览器访问 [http://127.0.0.1:8000](http://127.0.0.1:8000/)。

你将看到如下的 JSON 响应：

```json
{"message": "Hello World"}
```

### 总结

- 导入 `FastAPI`。
- 创建一个 `app` 实例。
- 编写一个**路径操作装饰器**（如 `@app.get("/")`）。
- 编写一个**路径操作函数**（如上面的 `async def root(): ...`）。
- 运行开发服务器（如 `uvicorn main:app --reload`）。

