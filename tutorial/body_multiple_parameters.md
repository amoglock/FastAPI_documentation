<h3>Тело запроса - несколько параметров</h3>

Сейчас, когда мы видели как использовать `Path` и `Query`, давайте рассмотрим более продвинутое использование объявления
тела запроса.

<h3>Смешение `Path`, `Query` и параметров тела запроса.</h3>

Во-первых, конечно, вы можете свободно сочетать `Path`, `Query` и параметры тела запроса, и FastAPI будет знать,
что ему делать.

А вы можете также объявить параметры тела запроса как опциональные, путем установки значения по умолчанию `None`:

```python
from typing import Annotated

from fastapi import FastAPI, Path
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


@app.put("/items/{item_id}")
async def update_item(
    item_id: Annotated[int, Path(title="The ID of the item to get", ge=0, le=1000)],
    q: str | None = None,
    item: Item | None = None,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    if item:
        results.update({"item": item})
    return results
```

> **Заметка**
> 
> Обратите внимание, в этом случае, `item` который будет взят из тела, является необязательным. Так как он имеет значение
> по умолчанию `None`.

<h4>Несколько параметров тела запроса</h4>

В предыдущем примере, операции пути ожидал бы тело JSON с аттрибутами из `Item`, например такими:

```JSON
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2
}
```

Но вы можете объявить еще несколько параметров, например `item` и `user`:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


class User(BaseModel):
    username: str
    full_name: str | None = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```

В этом случае, FastAPI заметит, что существует больше одного параметра тела запроса в функции (два параметра это модели
Pydantic).

Отсюда, он затем будет использовать параметр имен как ключи (имя полей) в теле, и ожидать тело, например такое:

```JSON
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

> **Заметка**
> 
> Обратите внимание, что хотя `item` был объявлен таким же способом как до этого, теперь ожидается, что он будет находиться
> внутри тела запроса с ключом `item`.

FastAPI будет делать автоматическую конвертацию из запроса, потому что параметр `item` ожидает определенное содержание
и то же самое относится для `user`.

Он будет выполнять проверку сложных данных, и документировать их как схему OpenAPI и автоматически включать в документацию.

<h3>Особые значения в теле запроса<h3>

Точно так же, как есть `Query` и `Path` чтобы определять дополнительные данные для запроса и параметров пути, FastAPI 
предоставляет эквивалент `Body`.