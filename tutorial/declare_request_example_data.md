<h3>Объявление примера запроса данных</h3>

Вы можете объявлять примеры данных которые принимает ваше приложение.

Есть несколько способов сделать это.

<h3>Дополнительная схема данных JSON в модели Pydantic</h3>

Вы можете объявлять `examples` для модели Pydantic которые будут добавлены в сгенерированную схему JSON.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

    model_config = {
        "json_schema_extra": {
            "examples": [
                {
                    "name": "Foo",
                    "description": "A very nice Item",
                    "price": 35.4,
                    "tax": 3.2,
                }
            ]
        }
    }

...
```

Эта дополнительная информация будет добавлена как есть в вывод схемы JSON для этой модели и будет использована в
документации API.

В Pydantic версии 2, вы бы использовали аттрибут `model_config`, который принимает `dict` как описано в
<a href="https://docs.pydantic.dev/latest/usage/model_config/">Pydantic`s docs: Model Config</a>.

Вы можете устанавливать `json_schema_extra` с `dict`, содержащим любые дополнительные данные которые вы бы хотели показать
в сгенерированной схеме JSON, включая `examples`.

> **Совет**
> 
> Вы могли бы использовать такую же технику, чтобы расширить схему JSON и добавить вашу собственную дополнительную информацию.
> Например, вы могли бы использовать ее, чтобы добавить метаданные для пользовательского интерфейса и т.д.

> **Для информации**
> 
> OpenAPI 3.1.0 (используется с FastAPI 0.99.0) добавило поддержку для `examples`, которые являются частью стандарта 
> схемы JSON.
> 
> До этого, оно только поддерживало ключевое слово `example` с единичным примером. Так все еще поддерживается OpenAPI 3.1.0
>, но является устаревшим и не частью стандарта схемы JSON. По этому вам рекомендуется перенести `example` в `examples`. 🤓 
> 
> Вы можете прочитать об этом больше в конце этой страницы.

<h3>Дополнительные аргументы `Field`</h3>

При использовании `Field()` с моделями Pydantic, вы можете также объявить добавочные `examples`:

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str = Field(examples=["Foo"])
    description: str | None = Field(default=None, examples=["A very nice Item"])
    price: float | None = Field(examples=[35.4])
    tax: float | None = Field(default=None, examples=[3.2])

...
```

<h3>`examples` в схеме JSON - OpenAPI</h3>

При использовании любого из:

* `Path()`
* `Query()`
* `Header()`
* `Cookie()`
* `Body()`
* `Form()`
* `File()`

Вы также можете объявлять группу `examples` с добавочной информацией которая в их схему JSON внутри OpenAPI.

<h4>`Body` с `examples`</h4>

Здесь мы передаем `examples` содержащие один пример ожидаемых данных в `Body()`:

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
async def update_item(
        item_id: int,
        item: Annotated[
            Item,
            Body(
                examples=[
                    {
                        "name": "Foo",
                        "description": "A very nice Item",
                        "price": 35.4,
                        "tax": 3.2,
                    }
                ],
            ),
        ],
):
    results = {"item_id": item_id, "item": Item}
    return results
```

<h4>Пример в документации UI</h4>

С любым из методов выше это может выглядеть вот так в `/docs`:

<img src="https://fastapi.tiangolo.com/img/tutorial/body-fields/image01.png" width="430" height="415">

<h4>`Body()` с несколькими `examples`</h4>

Конечно, вы можете передавать несколько `examples`:

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

@app.put("/items/{item_id}")
async def update_item(
        *,
        item_id: int,
        item: Annotated[
            Item,
            Body(
                examples=[
                    {
                        "name": "Foo",
                        "description": "A very nice Item",
                        "price": 35.4,
                        "tax": 3.2,
                    },
                    {
                        "name": "Bar",
                        "price": "35.4",
                    },
                    {
                        "name": "Baz",
                        "price": "thirty five point four",
                    },
                ],
            ),
        ],
):
    results = {"item_id": item_id, "item": item}
    return results
```

Когда вы делаете так, примеры будут частью внутренней схемы JSON для этого тела данных.

Тем не менее на сегодня (26.08.2023), Swagger UI, инструмент отвечающий за показ документации UI, не поддерживает
демонстрацию множественных примеров для данных в схеме JSON. Но читайте ниже для обходного пути.

<h4>Конкретные OpenAPI `examples`</h4>

С тех пор как схема JSON поддерживает `examples` OPenAPI имеет поддержку отдельного поля, так же называемого `examples`.

Этот конкретный OpenAPI `examples` входит в другую секцию спецификации OpenAPI. Она входит в детали для каждой операции
пути, а не внутрь каждой схемы JSON.

И Swagger UI имеет временную поддержку этих конкретных полей `examples`. Поэтому, вы можете использовать их, чтобы показать
различные примеры в документации UI.

Форма такого конкретного поля OpenAPI `examples` это `dict` с несколькими примерами (вместо `list`), каждый с
дополнительной информацией которая будет также добавлена в OpenAPI.

Она не войдет в каждую схему JSON, содержащуюся в OpenAPI, она будет снаружи, непосредственно в операции пути.

<h4>Использование параметра `openapi_examples`</h4>

Вы можете объявлять конкретный OpenAPI параметр `examples` в FastAPI с помощью параметра `openapi_examples` для:

* `Path()`
* `Query()`
* `Header()`
* `Cookie()`
* `Body()`
* `Form()`
* `File()`

Ключи для этого `dict` определяются для каждого примера, а каждое значение это еще один `dict`.

Каждый отдельный пример `dict` в `examples` может содержать:

* `summary`: Короткое описание примера.
* `description`: Подробное описание которое может содержать разметку Markdown.
* `value`: Это демонстрация актуального примера, например `dict`.
* `externalValue`: альтернативный `value`, URL указывающий на пример. Хотя это может поддерживаться меньшим количеством
инструментов чем `value`.

Вы можете использовать это например так:

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


@app.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int,
    item: Annotated[
        Item,
        Body(
            openapi_examples={
                "normal": {
                    "summary": "A normal example",
                    "description": "A **normal** item works correctly.",
                    "value": {
                        "name": "Foo",
                        "description": "A very nice Item",
                        "price": 35.4,
                        "tax": 3.2,
                    },
                },
                "converted": {
                    "summary": "An example with converted data",
                    "description": "FastAPI can convert price `strings` to actual `numbers` automatically",
                    "value": {
                        "name": "Bar",
                        "price": "35.4",
                    },
                },
                "invalid": {
                    "summary": "Invalid data is rejected with an error",
                    "value": {
                        "name": "Baz",
                        "price": "thirty five point four",
                    },
                },
            },
        ),
    ],
):
    results = {"item_id": item_id, "item": item}
    return results
```

