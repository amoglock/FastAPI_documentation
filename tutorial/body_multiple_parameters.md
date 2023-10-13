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

<h3>Единичные значения в теле запроса</h3>

Точно так же, как есть `Query` и `Path` чтобы определять дополнительные данные для запроса и параметров пути, FastAPI 
предоставляет эквивалент `Body`.

Например, расширяя предыдущую модель, вы можете решить, что вам нужен новый ключ `importance` в том же теле, помимо
`item` и `user`.

Если вы объявите его как есть, так как это единичное значение, FastAPI предположит, что это параметр запроса.

Но вы можете сообщить FastAPI относиться к нему как к новому ключу тела запроса, используя `Body`.

```python
from typing import Annotated

from fastapi import Body, FastAPI
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
async def update_item(
        item_id: int, item: Item, user: User, importance: Annotated[int, Body()]
):
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    return results
```

В этом случае, FastAPI будет ожидать такое тело запроса:

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
    },
    "importance": 5
}
```

<h3>Множественные параметры тела и запроса</h3>

Конечно, вы можете также объявить дополнительные параметры запроса каждый раз когда вам это нужно, дополнительно к любым
параметрам тела.

Так как, по умолчанию, единичные значения определяются как параметры запроса, вам не нужно явно добавлять `Query`, вы
можете просто сделать:

```python
q: Union[str, None] = None
```

Или в Python 3.10 или выше:

```python
q: str | None = None
```

Например:

```python
from typing import Annotated

from fastapi import FastAPI, Body
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
async def update_item(
        *,
        item_id: int,
        item: Item,
        user: User,
        importance: Annotated[int, Body(gt=0)],
        q: str | None = None,
):
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    if q:
        results.update({"q": q})
    return results
```

> **Для информации**
> 
> `Body` так же имеет все те же дополнительные проверки и параметры метаданных как `Query`, `Path` и остальное, что вы 
> увидите позже.


<h3>Вставка единичного параметра тела</h3>

Скажем, что у вас есть один параметр тела `item` из модели Pydantic `Item`.

По умолчанию, FastAPI будет ожидать это тело напрямую.

Но если вы хотите, ожидать JSON с ключом `item`, а внутри содержимое модели, как это происходит, когда вы объявляете 
дополнительные параметры тела, вы можете использовать специальный `Body` параметр `embed`:

```python
item: Item = Body(embed=True)
```

Как здесь:

```python
from typing import Annotated

from fastapi import FastAPI, Body
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str   
    description: str | None = None
    price: float
    tax: float | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    results = {"item_id": item_id, "item": item}
    return results
``` 

В этом случае FastAPI будет ожидать такое тело:

```JSON
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    }
}
```

Вместо:

```JSON
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2
}
```

<h3>Резюме</h3>

Вы можете добавить несколько параметров тела в вашу функцию операции пути, даже несмотря на то, что запрос имеет только
одно тело.

Но FastAPI будет обрабатывать его, давать вам корректные данные в функцию и проверять и документировать правильную схему
в операции пути.

Вы можете так же объявлять единичные значения, чтобы они были получены как часть тела.

И вы можете сообщить FastAPI встраивать тело в ключ даже когда объявлен только один параметр. 