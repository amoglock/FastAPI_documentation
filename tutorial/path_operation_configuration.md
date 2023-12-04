<h3>Конфигурация операции пути</h3>

Существуют несколько параметров которые вы можете передавать в декораторе операции пути, чтобы настроить его.

> **Предупреждение!!!**
> 
> Обратите внимание, что эти параметры передаются напрямую в декораторе операции пути, а не в функцию операции пути.

<h4>Статус-код ответа</h4>

Вы можете определить (HTTP) `status_code`, чтобы он использовался в операции пути.

Вы можете передавать напрямую код в `int`, например `404`.

Но если вы не помните каждый номер, вы можете использовать сокращение которое содержится в `status`:

```python
from fastapi import FastAPI, status
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str   
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()

@app.post("/items/", response_model=Item, status_code=status.HTTP_201_CREATED)
async def create_item(item: Item):
    return Item
```

Этот статус-код будет использован в ответе и добавлен в схему OpenAPI.

> **Технические детали**
> 
> Вы так же можете использовать `from starlette import status`.
> 
> FastAPI предоставляет тот же `starlette.status` как `fastapi.status`, просто удобнее для вас, разработчика. Но он 
> идет напрямую из Starlette.

<h4>Теги</h4>

Вы можете добавлять теги в операцию пути, передайте параметр `tags` с `list` или `str`(чаще всего просто один `str`):

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str   
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()

@app.post("/items/", response_model=Item, tags=["items"])
async def create_item(item:Item):
    return Item

@app.get("/items/", tags=["items"])
async def read_items():
    return [{"name": "Foo", "price": 42}]

@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]
```

Они будут добавлены в схему OpenAPI и использованы в автоматическом интерфейсе документации:

<img src="https://fastapi.tiangolo.com/img/tutorial/path-operation-configuration/image01.png">

<h4>Теги с перечислением</h4>

Если у вас большое приложение, в конечном итоге у вас может накопиться несколько тегов и вы бы захотели быть уверенными,
что вы всегда используете один тег для аналогичных операций.

В таких случаях, имеет смысл хранить теги в `Enum`.

FastAPI поддерживает его точно так же как обычные строки:

```python
from enum import Enum

from fastapi import FastAPI

app = FastAPI()

class Tags(Enum):
    items = "items"
    users = "users"

@app.get("/items/", tags=[Tags.items])
async def get_items():
    return ["Portal gun", "Plumbus"]

@app.get("/users", tags=[Tags.users])
async def read_users():
    return ["Rick", "Morty"]
```

<h4>Сводка и описание</h4>

Вы можете добавлять `summary` и `description`:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()

@app.post(
    "/items/",
    response_model=Item,
    summary="Create an item",
    description="Create an item with all the information, name, description, price, tax and unique tags",
)
async def create_item(item: Item):
    return item
```