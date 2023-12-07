<h3>Тело - обновления</h3>

<h4>Обновление заменой с `PUT`</h4>

Чтобы обновить элемент вы можете использовать операцию 
<a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT">HTTP `PUT`</a>.

Вы можете использовать `jsonable_encoder` чтобы конвертировать входные данные, в данные, которые могут храниться как JSON   
(например в NoSQL базе данных). Например, конвертируя `datetime` в `str`.

```python
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str | None = None
    description: str | None = None
    price: float | None = None
    tax: float = 10.5
    tags: list[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]

@app.put("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    update_item_encoded = jsonable_encoder(item)
    items[item_id] = update_item_encoded
    return update_item_encoded
```

`PUT` используется чтобы получить данные, которые должны заменить существующие.

<h5>Предупреждение по поводу замены</h5>

Это означает, что если вы хотите обновить элемент `bar`, используя `PUT`, с телом содержащим:

```JSON
{
  "name": "Barz",
  "price": 3,
  "description": None,
}
```

Так как оно не включает уже хранящийся аттрибут `"tax": 20.2`, входная модель возьмет значение по умолчанию `"tax": 10.5`.

А данные будут сохранены с этим "новым" значением `tax`.

<h4>Частичные обновления с `PATCH`</h4>

Еще вы можете использовать операцию <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PATCH">HTTP `PATCH`</a>
чтобы частично изменять данные.

Это значит, что вы можете отправить только данные которые вы хотите изменить, оставив остальное без изменений.

> **Заметка**
> 
> `PATCH` используется реже и менее известен чем `PUT`.
> 
> А многие команды используют только `PUT` даже для частичных изменений.
> 
> Вы свободны использовать их как вы хотите, FastAPI не накладывает какие-либо ограничения.
> 
> Но эта инструкция показывает вам, более или менее, как их предназначено использовать.

<h4>Использование параметра Pydantic `exclude_unset`</h4>

Если вы хотите принять частичные изменения, очень полезно использовать параметр `exclude_unset` в `.dict()` Pydantic модели.

Например, `item.dict(exclude_unset=True)`.

Что бы генерировало `dict` только с данными, которые были установлены во время модели `item`, исключая значения по умолчанию.

Затем вы можете использовать его для генерации `dict` только с данными которые были установлены (отправлены в запросе),
пропуская значения по умолчанию.

```python
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str | None = None
    description: str | None = None
    price: float | None = None
    tax: float = 10.5
    tags: list[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]

@app.patch("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    stored_item_data = items[item_id]
    stored_item_model = Item(**stored_item_data)
    update_data = item.dict(exclude_unset=True)
    updated_item = stored_item_model.copy(update=update_data)
    items[item_id] = jsonable_encoder(updated_item)
    return updated_item
```

<h4>Использование параметра `update` Pydantic</h4>

Теперь, вы можете создавать копию существующей модели, используя `.copy()`, и передавать параметр `update` с `dict`, 
содержащим данные для обновления.

Например, `stored_item_model.copy(update=update_data)`:

```python
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str | None = None
    description: str | None = None
    price: float | None = None
    tax: float = 10.5
    tags: list[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]

@app.patch("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    stored_item_data = items[item_id]
    stored_item_model = Item(**stored_item_data)
    update_data = item.dict(exclude_unset=True)
    updated_item = stored_item_model.copy(update=update_data)
    items[item_id] = jsonable_encoder(updated_item)
    return updated_item
```

<h4>Резюме для частичных изменений</h4>

В итоге, чтобы применить частичные изменения, вы можете:

* (Опционально) использовать `PATCH` вместо `PUT`.
* Извлекать сохраненные данные.
* Размещать данные в модели Pydantic.
* Формировать `dict` без значений по умолчанию из входящей модели (используя `exclude_unset`).
  * Таким способом вы можете обновлять только значения, фактически установленные пользователем, вместо переопределения
  значений уже хранящихся в вашей модели.
* Создавать копию хранящейся модели, изменяя ее аттрибуты с полученными частичными обновлениями (используя параметр `update`).
* Конвертировать скопированную модель во что-то, что может быть размещено в вашей базе данных (например, используя 
`jsonable_encoder`).
  * Это сравнимо с применением метода модели `.dict()` снова, но это гарантирует (и конвертирует) значения в типы данных,
  которые могут быть конвертированы в JSON, например, `datetime` в `str`.
* Сохранять данные в вашу базу данных.
* Возвращать измененную модель.

```python
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str | None = None
    description: str | None = None
    price: float | None = None
    tax: float = 10.5
    tags: list[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]


@app.patch("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    stored_item_data = items[item_id]
    stored_item_model = Item(**stored_item_data)
    update_data = item.dict(exclude_unset=True)
    updated_item = stored_item_model.copy(update=update_data)
    items[item_id] = jsonable_encoder(updated_item)
    return updated_item
```

> **Совет**
> 
> Фактически вы можете использовать ту же технику с HTTP операцией `PUT`.
> 
> Но наш пример использует `PATCH`, потому что он был создан для использования этих вариантов.

> **Заметка**
> 
> Обратите внимание, что входящая модель все еще проверяется.
> 
> Поэтому, если вы хотите принять частичные изменения в которых могут быть пропущены все аттрибуты. Вам нужно иметь 
> модель в которой все аттрибуты отмечены как необязательные (со значением по умолчанию `None`).
> 
> Чтобы разделить модели со всеми необязательными значениями для изменений от моделей с обязательными значениями для
> создания, вы можете использовать идеи, описанные в
> <a href="https://github.com/amoglock/FastAPI_documentation/blob/master/tutorial/extra_models.md">Дополнительные модели</a><br>.