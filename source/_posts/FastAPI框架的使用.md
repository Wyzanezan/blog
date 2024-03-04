---
title: FastAPI框架的使用
date: 2024-03-04 23:03:02
tags:
categories: Python
---

FastAPI 是一个现代、快速（高性能）的 Web 框架，基于标准的 Python 类型提示并使用 Python 3.8+ 构建 API。

<!--more-->

它有如下特点：
快速：非常高的性能，与 NodeJS 和 Go 相当
快速编码：将开发功能的速度提高约 200% 到 300%。
更少的 bug：减少约 40% 的人为（开发人员）引起的 bug。
直观：强大的编辑器支持，随时随地都可以完成。调试时间更少。
简单：旨在易于使用和学习。阅读文档的时间更少。
简洁：最大限度地减少代码重复。每个参数声明具有多个功能。更少的错误。
健壮：获取可用于生产的代码。具有自动交互式文档。
基于标准：基于（并完全兼容）API 开放标准：OpenAPI（以前称为 Swagger）和 JSON Schema。

FastAPI 基于 Python 3.8+ 的类型注释和异步编程特性，使得代码更具表达能力。它使用声明式的语法，支持基于函数的 API 定义和异步请求处理，这可提高性能特别是在 IO 密集型应用程序中。它拥有内置强大的依赖注入系统，你可以使用它来提供共享逻辑、数据提供、安全依赖（如用户认证）等，在处理请求时它能够自动解析和注入。
此外，FastAPI 还集成了 Swagger，它可以根据你的代码自动创建一份 OpenAPI 文档。

# FastAPI的使用

使用前我们需要安装相关的包，包括 fastapi 包和 uvicorn（一个 ASGI Server，异步网关接口服务器）包：

```python
pip install fastapi
pip install "uvicorn[standard]"
```

## 创建一个简单的服务

安装完成后，看一个简单的例子 main.py ：

```python
from typing import Optional
from enum import Enum

import uvicorn
from fastapi import FastAPI

app = FastAPI()


# 定义一个枚举类型
class UserGender(str, Enum):
    male = 'male'
    female = 'female'


@app.get("/")
async def index():
    return {"index": "hello fastapi"}


# 请求 http://127.0.0.1:8090/userinfo/1
@app.get("/userinfo/{user_id}")
async def userinfo(user_id: int):
    """
    param user_id: 指定了user_id是int类型，也可以不指定参数类型
    """
    return {"user_id": user_id, "userinfo": {}}


# 请求 http://127.0.0.1:8090/userlist/male
@app.get("/userlist/{gender}")
async def user_list(gender: UserGender):
    """
    :param gender: 定义参数为枚举类型
    """
    return {"gender": gender}


# 请求 http://127.0.0.1:8090/users?page_index=1&page_size=10，page_size是一个可选参数，有一个默认值
@app.get("/users")
async def users(page_index: int, page_size:  Optional[int] = 10):
    return {"page_index": page_index, "page_size": page_size}


# 请求 http://127.0.0.1:8090/users/1/friends?page_index=1
@app.get("/users/{user_id}/friends")
async def friends(user_id: int, page_index: int, page_size: Optional[int] = 10):
    return {"user_id": user_id, "page_index": page_index, "page_size": page_size}


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)

```

启动服务时，执行：python3 main.py 即可，还可以执行 uvicorn main:app --reload 启动服务。



## 请求参数的处理

请求参数包括 请求 url 中的路径参数、请求 url 中的查询字符串参数以及请求体中的参数。

那么如何处理请求 url 中的路径参数和查询字符串参数呢？
我们还可以给接口中的参数添加验证的逻辑，需要使用到 FastAPI 下的 Path 和 Query 两个库。
给请求 url 中的路径参数添加验证，需要使用 Path 库：

```python
import uvicorn
from fastapi import FastAPI, Path


app = FastAPI()


@app.get("/userinfo/{age}")
async def get(age: int = Path(..., title="user age", ge=18, lt=60)):
    return {"user_age": age}


@app.get("/customers/{username}")
async def get(username: str = Path(..., title="user name", min_length=6, max_length=20)):
    return {"username": username}


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=80

```

Path 中的第一个参数使用 ... 表示这个参数是必选的。Path 中还可以使用 regex 参数加上正则表达式的验证方式，如果传入的参数值不满足验证条件，会返回类似下面的错误信息：

```json
{
    "detail": [
        {
            "type": "string_too_short",
            "loc": [
                "path",
                "username"
            ],
            "msg": "String should have at least 6 characters",
            "input": "ndfs",
            "ctx": {
                "min_length": 6
            },
            "url": "https://errors.pydantic.dev/2.5/v/string_too_short"
        }
    ]
}
```



给查询字符串参数添加验证，需要使用 Query 库，其中 Query(None) 表示参数是可选的，Query(...) 表示参数是必选的：

```python
import uvicorn
from fastapi import FastAPI, Query


app = FastAPI()


@app.get("/userinfo")
async def get(age: int = Query(18, title="user age", ge=18, lt=60)):
    return {"user_age": age}


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)

```

上面的 Query 中的第一个参数表示该查询参数是可选的，并且他的默认值是 18。
还可以给查询字符串参数添加别名：

```python
import uvicorn
from fastapi import FastAPI, Query


app = FastAPI()


@app.get("/userinfo")
async def get(user_interest: str = Query(None, title="user interest", alias="user-interest")):
    return {"user_interest": user_interest}


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)

```

指定别名以后，我们的请求 url 就必须是 alias 指定的参数名：http://127.0.0.1:8090/userinfo?user-interest=篮球



如何处理请求体中的参数呢？
我们可以通过定义模型，让 FastAPI 框架为我们从请求体中提取对应的数据。

```python
from typing import Optional
from enum import Enum

import uvicorn
from fastapi import FastAPI
from pydantic import BaseModel


class Gender(str, Enum):
    male = 'male'
    female = 'female'


# 定义用户模型
class UserModel(BaseModel):
    username: str
    gender: Gender   # 定义一个枚举类型的数据
    desc: Optional[str] = ''


app = FastAPI()


@app.post("/users")
async def create_user(user_model: UserModel):
    # 使用 model_dump() 方法将 model 中的数据转换成一个 dict
    user_dict = user_model.model_dump()
    return user_dict


@app.put("/users/{user_id}")
async def modify_user(user_id: int, user_model: UserModel):
    user_dict = user_model.model_dump()
    user_dict.update({"user_id": user_id})
    return user_dict


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)

```

上面 post 接口的请求 url 是：[http://127.0.0.1:8090/users](http://127.0.0.1:8090/users，请求参数)，请求参数是 json 格式：

```json
{
    "username": "小美",
    "gender": "male"
}
```

put 接口的请求 url 是：[http://127.0.0.1:8090/users](http://127.0.0.1:8090/users，请求参数)/1，请求参数是 json 格式：

```json
{
    "username": "小美",
    "gender": "male"
}
```



下面的例子介绍在一个接口中如何使用多个模型：

```python
import uvicorn
from fastapi import FastAPI
from pydantic import BaseModel


app = FastAPI()


class User(BaseModel):
    username: str
    desc: str


class Cart(BaseModel):
    goods_name: str
    price: float


@app.post("/users")
async def user_cart(cart_id: int, user: User, cart: Cart):
    return {
        "cart_id": cart_id,
        "user": user.model_dump(),
        "cart": cart.model_dump()
    }


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)

```

上面的接口中，请求体是：

```json
{
    "user": {
        "username": "小美",
        "desc": ""
    },
    "cart": {
        "goods_name": "篮球",
        "price": 100
    }
}
```



我们还可以使用 fastapi.Body 来处理请求体中的参数，它支持我们在请求体中传入除了 user，cart 的其它参数，比如我们的请求体是下面这样：

```json
{
    "user": {
        "username": "小美",
        "desc": ""
    },
    "cart": {
        "goods_name": "篮球",
        "price": 100
    },
    "date": "2024-03-03"
}
```

上面的请求中增加了一个 date 数据，我们使用 Body 来处理：

```python
@app.post("/users")
async def user_cart(cart_id: int, user: User, cart: Cart, date: str = Body(...)):
    return {
        "cart_id": cart_id,
        "user": user.model_dump(),
        "cart": cart.model_dump(),
        "date": date
    }
```

当然。Body 中也支持添加验证逻辑。

如果想要对模型中的数据添加验证逻辑，需要这样使用：

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    username: str = Field('test', min_length=6, max_length=20)
    desc: str = Field(None, max_length=255)
```

如果我们需要在一个模型中嵌套其它的模型，可以这样使用：

```python
from pydantic import BaseModel, Field

class Address(BaseModel):
    address: str
    postcode: str


class User(BaseModel):
    username: str = Field('test', min_length=6, max_length=20)
    desc: str = Field(None, max_length=255)
    address: Address
```



如果我们想在模型中使用 list、set 等数据类型，可以这样使用：

```python
from typing import List

from pydantic import BaseModel

class Cart(BaseModel):
    goods: list   # 或者写成 List[str]
    price: float
```



## Cookie与Header参数

FastAPI中提供了 fastapi.Cookie 和 fastapi.Header 库供我们获取 cookie 参数和 header 参数。

```python
from typing import Union, Optional

import uvicorn
from fastapi import FastAPI, Cookie, Header


app = FastAPI()


@app.post("/users")
async def user_cart(api_token:  Optional[str] = Header(..., alias='api-token'),
                    user_type: Union[str, None] = Cookie(..., alias="user-type")):
    return {
        "api_token": api_token,
        "user_type": user_type
    }


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)
```

上面的代码中，Optional[str] 与 Union[str, None] 的功能一样，都是表示参数是可选参数。

我们也可以通过 response 对象设置 cookie：

```python
from typing import Union, Optional

import uvicorn
from fastapi import FastAPI, Cookie, Header, Response


app = FastAPI()


@app.post("/users")
async def user_cart(response: Response, api_token:  Optional[str] = Header(..., alias='api-token'),
                    user_type: Union[str, None] = Cookie(..., alias="user-type")):

    data = {
        "api_token": api_token,
        "user_type": user_type
    }
    # 设置 cookie
    response.set_cookie(key="user-type", value='2222', expires=60)
    return data
```



## 响应数据的处理

响应数据的格式也可以通过模型进行指定：

```python
import uvicorn
from fastapi import FastAPI, Cookie, Header, Response
from pydantic import BaseModel


app = FastAPI()


GOODS_LIST = {
    "1": {"goods_id": "1", "goods_name": "g1", "price": 12, "desc": "dd"},
    "2": {"goods_id": "2", "goods_name": "g2", "price": 13, "desc": "dd"},
    "3": {"goods_id": "3", "goods_name": "g3", "price": 14, "desc": "dd"},
}


class GoodsResponseModel(BaseModel):
    goods_id: str
    goods_name: str
    price: float


@app.post("/goods/{goods_id}", response_model=GoodsResponseModel, response_model_include={"goods_name"})
async def goods(goods_id: str):
    return GOODS_LIST.get(goods_id)


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)
```

通过 response_model 参数指定响应数据的格式，通过 response_model_include 参数指定在 response_model 的基础上只输出哪些数据。



## 状态码与异常处理

状态码与异常处理的例子如下：

```python
from typing import Optional

import uvicorn
from fastapi import FastAPI, Path, HTTPException, status
from pydantic import BaseModel


app = FastAPI()


class UserBase(BaseModel):
    id: Optional[int] = None
    username: str
    user_age: int
    desc: Optional[str] = None


class UserRequestModel(UserBase):
    password: str


class UserResponseModel(UserBase):
    ...


user_list = {
    "ttt": {"id": 1, "username": "ttt", "user_age": 22},
    "zzz": {"id": 2, "username": "zzz", "user_age": 24},
}


# 添加 status_code 参数指定成功响应的状态码
@app.post("/user/create", status_code=201, response_model=UserResponseModel)
async def create_user(user: UserRequestModel):
    userinfo = user.model_dump()
    userinfo.update({"id": 1})
    return userinfo


@app.get("/user/{username}", status_code=200, response_model=UserResponseModel)
async def get_user(username: str = Path(..., min_length=3)):
    user_detail = user_list.get(username)
    if user_detail:
        return user_detail

    # 不存在则抛出异常
    raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail=f"{username} not found")


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)
```

我们还可以自定义返回的异常类和异常处理函数，代码如下：

```python
from typing import Optional

import uvicorn
from fastapi import FastAPI, Path, Request
from fastapi.responses import JSONResponse
from pydantic import BaseModel


app = FastAPI()


class UserBase(BaseModel):
    id: Optional[int] = None
    username: str
    user_age: int
    desc: Optional[str] = None


class UserRequestModel(UserBase):
    password: str


class UserResponseModel(UserBase):
    ...


class UserNotFoundException(Exception):

    def __init__(self, username: str):
        self.username = username


user_list = {
    "ttt": {"id": 1, "username": "ttt", "user_age": 22},
    "zzz": {"id": 2, "username": "zzz", "user_age": 24},
}


# 添加 status_code 参数指定成功响应的状态码
@app.post("/user/create", status_code=201, response_model=UserResponseModel)
async def create_user(user: UserRequestModel):
    userinfo = user.model_dump()
    userinfo.update({"id": 1})
    return userinfo


@app.get("/user/{username}", status_code=200, response_model=UserResponseModel)
async def get_user(username: str = Path(..., min_length=3)):
    user_detail = user_list.get(username)
    if user_detail:
        return user_detail

    # 不存在则抛出异常
    # raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail=f"{username} not found")

    # 抛出自定义异常
    raise UserNotFoundException(username)


# 当抛出 UserNotFoundException 异常时, FastAPI 会调用这个方法进行处理
@app.exception_handler(UserNotFoundException)
async def user_not_found_exception_handler(request: Request, exc: UserNotFoundException):
    return JSONResponse(status_code=404, content={
        "error_code": 404,
        "message": f"{exc.username} not found",
        "data": {}
    })


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)
```

上面的代码中，自定义了一个异常类 UserNotFoundException 和一个异常处理函数 user_not_found_exception_handler()，当 fastapi 中捕获到 UserNotFoundException 异常时，会调用 user_not_found_exception_handler() 函数去处理异常。
或者我们可以定义一个通用的异常模型，通过这个异常模型返回错误信息：

```python
from typing import Optional

import uvicorn
from fastapi import FastAPI
from fastapi.responses import JSONResponse
from pydantic import BaseModel


app = FastAPI()


class UserBase(BaseModel):
    id: Optional[int] = None
    username: str
    user_age: int
    desc: Optional[str] = None


class UserRequestModel(UserBase):
    password: str


class UserResponseModel(UserBase):
    ...


# 自定义错误模型
class ErrorMessage(BaseModel):
    err_code: int
    message: str


user_list = {
    "ttt": {"id": 1, "username": "ttt", "user_age": 22},
    "zzz": {"id": 2, "username": "zzz", "user_age": 24},
}


# 添加 status_code 参数指定成功响应的状态码
@app.post("/user/create", status_code=201, response_model=UserResponseModel,
          responses={400: {"model": ErrorMessage}})
async def create_user(user: UserRequestModel):
    if user_list.get(user.username, None):
        error_message = ErrorMessage(err_code=400, message=f"{user.username} already exists")
        return JSONResponse(status_code=400, content=error_message.model_dump())

    userinfo = user.model_dump()
    userinfo.update({"id": 1})
    return userinfo


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)
```



## 依赖注入的使用

我们使用 fastapi 提供的 Depends 实现依赖注入：

```python
from typing import Optional

import uvicorn
from fastapi import FastAPI, Depends


app = FastAPI()


async def page_params(page_index: Optional[int] = 1, page_num: Optional[int] = 10):
    return {"page_index": page_index, "page_num": page_num}


@app.get("/users")
async def get_users(page_info: dict = Depends(page_params)):
    return page_info


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)
```

上面的例子中，把函数进行了注入，下面介绍把类进行注入的方式：

```python
from typing import Optional

import uvicorn
from fastapi import FastAPI, Depends


app = FastAPI()


class PageParams():

    def __init__(self, page_index: Optional[int] = 1, page_num: Optional[int] = 10):
        self.page_index = page_index
        self.page_num = page_num


@app.get("/users")
async def get_users(page_info: PageParams = Depends(PageParams)):
    return {"page_index": page_info.page_index, "page_num": page_info.page_num}


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)
```

也可以使用 dependencies 来添加依赖注入：

```python
from typing import Optional

import uvicorn
from fastapi import FastAPI, Depends, Header, HTTPException


app = FastAPI()


async def verify_auth(api_token: Optional[str] = Header(..., alias='api-token')):
    if not api_token:
        raise HTTPException(status_code=401, detail="Unauthorized")


class PageParams():

    def __init__(self, page_index: Optional[int] = 1, page_num: Optional[int] = 10):
        self.page_index = page_index
        self.page_num = page_num


# 使用 dependencies 添加依赖
@app.get("/users", dependencies=[Depends(verify_auth)])
async def get_users(page_info: PageParams = Depends(PageParams)):
    return {"page_index": page_info.page_index, "page_num": page_info.page_num}


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)
```

也可以在创建 app 时（即全局）添加依赖注入：

```python
async def verify_auth(api_token: Optional[str] = Header(..., alias='api-token')):
    if not api_token:
        raise HTTPException(status_code=401, detail="Unauthorized")


app = FastAPI(dependencies=[Depends(verify_auth)])
```



## API的身份认证

下面使用 JWT Token 进行身份验证，例子如下：

```python
from datetime import timedelta, timezone, datetime

import jwt
import uvicorn
from fastapi import FastAPI, Depends, Header, HTTPException
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel


TOKEN_KEY = 'wz745jd8j237#ma9ds%nd'
ALGORITHMS = 'HS256'
# 指定获取 token 的接口
token_url = OAuth2PasswordBearer(tokenUrl="/login")


class Token(BaseModel):

    access_token: str
    token_type: str


app = FastAPI()


def validate_user(username: str, password: str):
    if username == 'wz' and password == '1':
        return username
    return None


# token_url 会自动从 Header 中获取 Token
def get_current_username(token: str = Depends(token_url)):
    username = None
    token_failed_exception = HTTPException(status_code=401, detail="鉴权失败")

    try:
        # 解析 jwt token
        token_data = jwt.decode(token, TOKEN_KEY, ALGORITHMS)
        if token_data:
            username = token_data.get("username")
    except Exception as e:
        raise token_failed_exception

    if not username:
        raise token_failed_exception

    return username


# 使用 dependencies 添加依赖
@app.post("/login")
async def login(login_form: OAuth2PasswordRequestForm = Depends()):
    username = validate_user(login_form.username, login_form.password)
    if not username:
        raise HTTPException(status_code=401, detail="用户名或者密码不正确")

    # 生成 jwt token
    token_expires = datetime.now(timezone.utc) + timedelta(minutes=10)
    token_data = {
        "username": username,
        "exp": token_expires
    }
    token = jwt.encode(token_data, TOKEN_KEY, ALGORITHMS)

    return Token(access_token=token, token_type="tmp")


@app.post("/users")
async def users(username: str = Depends(get_current_username)):
    return {"username": username}


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)
```



## 连接数据库

连接数据库，我们使用 SQLAlchemy 这个库，还需要 mysqlclient 或者 PyMySQL。
其中，mysqlclient 对应的数据库连接串是：mysql+mysqldb://username:password@ip:port/db
PyMySQL 对应的数据库连接串是：mysql+pymysql://username:password@ip:port/db
安装相关库：

```python
pip install SQLAlchemy
pip install mysqlclient
pip install PyMySQL
```



下面是在 FastAPI 中操作数据库的例子，包括数据库的增删改查操作：

```python
from typing import List

import uvicorn
from fastapi import FastAPI, Depends, HTTPException, Path
from pydantic import BaseModel

from sqlalchemy import create_engine, Integer, String, select, asc
from sqlalchemy.orm import DeclarativeBase, sessionmaker, Mapped, mapped_column


class Base(DeclarativeBase):
    pass


# 定义数据库模型
class UserEntity(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    name: Mapped[str] = mapped_column(String(50), unique=True, nullable=False)
    gender: Mapped[str] = mapped_column(String(10), nullable=False)


engine = create_engine("mysql+pymysql://root:admin123456@127.0.0.1:3306/testdb", echo=True)

Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)


# 定义接口的模型
class UsersBaseModel(BaseModel):
    name: str
    gender: str


class UserCreateModel(UsersBaseModel):
    ...


class UserUpdateModel(UsersBaseModel):
    ...


class UserResponseModel(UsersBaseModel):
    id: int


# 创建数据库连接
def get_db_session():
    db = Session()
    try:
        yield db
    finally:
        db.close()


def set_attrs(obj, data: dict):
    if data:
        for key, val in data.items():
            setattr(obj, key, val)


app = FastAPI()


# 查询用户接口
@app.get("/user/list", response_model=List[UserResponseModel])
async def get_users(db: Session = Depends(get_db_session)):
    query = select(UserEntity).order_by(asc(UserEntity.id))
    results = db.execute(query).scalars().all()
    return results


# 创建用户接口
@app.post("/user/create", response_model=UserResponseModel)
async def create_user(user: UserCreateModel, db: Session = Depends(get_db_session)):
    query = select(UserEntity).where(UserEntity.name == user.name)
    records = db.execute(query).scalars().all()
    if records:
        raise HTTPException(status_code=400, detail="用户已存在")

    user = UserEntity(name=user.name, gender=user.gender)
    db.add(user)
    db.commit()
    return user


# 更新用户接口
@app.put("/user/{user_id}", response_model=UserResponseModel)
async def update_user(*, user_id: int = Path(...), user: UserUpdateModel, db: Session = Depends(get_db_session)):
    query = select(UserEntity).where(UserEntity.id == user_id)
    record = db.execute(query).scalars().one()
    if not record:
        raise HTTPException(status_code=404, detail="用户不存在")

    # 更新数据 方式一
    # record.name = user.name
    # record.gender = user.gender
    # db.commit()

    # 更新数据 方式二
    set_attrs(record, user.model_dump())
    db.commit()

    return record


# 删除用户接口
@app.delete("/user/{user_id}")
async def delete_user(user_id: int = Path(...), db: Session = Depends(get_db_session)):

    query = select(UserEntity).where(UserEntity.id == user_id)
    record = db.execute(query).scalars().one()
    if not record:
        raise HTTPException(status_code=404, detail="用户不存在")

    db.delete(record)
    db.commit()

    return "ok"


if __name__ == '__main__':
    uvicorn.run("main:app", reload=True, port=8090)
```



上面介绍了 FastAPI 中的一些基本用法，有兴趣的同学可以看看以下相关的 Github 仓库：

https://github.com/tiangolo/fastapi

https://github.com/mjhea0/awesome-fastapi

以及 FastAPI 的官方文档：https://fastapi.tiangolo.com/
