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

<h4>Описание из docstring</h4>

Так как описания имеют тенденцию быть длинными и занимать несколько строк, вы можете объявить описание операции пути
в docstring функции и FastAPI прочитает ее оттуда.

Вы можете использовать <a href="https://en.wikipedia.org/wiki/Markdown">Markdown</a> в docstring, он будет корректно
реализован и показан (принимая во внимание отступ docstring).

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

@app.post("/items/", response_model=Item, summary="Create an item")
async def create_item(item: Item):
    """
    Create an item with all the information:

    - **name**: each item must have a name
    - **description**: a long description
    - **price**: required
    - **tax**: if the item doesn't have tax, you can omit this
    - **tags**: a set of unique tag strings for this item
    """
    return item
```

Он будет использован в интерактивной документации:

<img src="https://fastapi.tiangolo.com/img/tutorial/path-operation-configuration/image02.png">

<h4>Описание ответа</h4>

Вы можете указывать описание ответа с помощью параметра `response_description`:

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
    response_description="The created item",
)
async def create_item(item: Item):
    """
    Create an item with all the information:

    - **name**: each item must have a name
    - **description**: a long description
    - **price**: required
    - **tax**: if the item doesn't have tax, you can omit this
    - **tags**: a set of unique tag strings for this item
    """
    return item
```

> **Для информации**
> 
> Обратите внимание, что `response_description` относится конкретно к ответу, `description` в основном относится к 
> операции пути.

> **Для проверки**
> 
> OpenAPI предусматривает, что для каждой операции пути требуется описание ответа.
> 
> Поэтому, если вы его не предоставите, FastAPI автоматически сгенерирует один "Успешный ответ".

<img src="https://fastapi.tiangolo.com/img/tutorial/path-operation-configuration/image03.png">

<h4>Отказ операции пути</h4>

Если вам нужно пометить операцию пути как не рекомендованную к использованию, но без ее удаления, передайте параметр
`deprecated`:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/", tags=["items"])
async def read_items():
    return [{"name": "Foo", "price": 42}]

@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]

@app.get("/elements/", tags=["items"], deprecated=True)
async def read_elements():
    return [{"item_id": "Foo"}]
```

Она будет понятно помечена как не рекомендованная в документации:

<img src="https://fastapi.tiangolo.com/img/tutorial/path-operation-configuration/image04.png">

Посмотрите как выглядят нежелательные и обычные операции пути:

<img src="https://fastapi.tiangolo.com/img/tutorial/path-operation-configuration/image05.png">

<h4>Резюме</h4>

Вы можете легко настраивать и добавлять метаданные для вашей операции пути, передав параметры в декораторы операции пути.