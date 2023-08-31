<h3>Параметра запроса</h3>
Когда вы объявляете параметры функции, которые не являются параметрами пути, они автоматически определяются как
параметры запроса.

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

Запрос, это набор пар ключ-значение которые идут после знака `?` в URL, разделенные символом `&`.

Например, в URL:

```
http://127.0.0.1:8000/items/?skip=0&limit=10
```

...параметры запроса это:

* `skip`: со значением `0`
* `limit`: со значением `10`

Поскольку параметры являются частью URL, они являются обычными строками.

Но, когда вы объявляете их с помощью типизации Python (в примере выше как `int`), они конвертируются в этот тип и
проверяются на соответствие ему.

Все процессы доступные для параметров пути так же доступны для параметров запроса
* Поддержка редактора
* Парсинг данных
* Валидация данных
* Автоматическая документация

<h3>Значения по умолчанию</h3>
Так как параметры запроса это не часть пути, они могут быть опциональными и иметь значения по умолчанию.

В примере выше они имеют значения по умолчанию `skip=0` и `limit=10`.

Поэтому, переход по URL:

```
http://127.0.0.1:8000/items/
```

будет таким же как и по:

```
http://127.0.0.1:8000/items/?skip=0&limit=10
```

Но, если вы перейдете, например, по:

```
http://127.0.0.1:8000/items/?skip=20
```

Значение параметров функции будет:
* `skip=20`: так как указано в URL
* `limit=10`: так как было указано значение по умолчанию

<h3>Опциональные параметры</h3>

Таким же способом вы можете объявить опциональные параметры запроса, указав значение по умолчанию как None:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: str, q: str | None = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

В этом случае, параметр функции `q` будет опциональным и иметь значение None по умолчанию.

>Обратите внимание, что FastAPI самостоятельно может определить, что `item_id` это параметр пути, а `q` нет,
> поэтому это параметр запроса.

<h3>Преобразование типа параметра запроса</h3>

Вы можете объявить тип `bool` и он будет преобразован:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: str, q: str | None = None, short: bool = False):
    item = {"item_id": item_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```
В таком случае если перейти по:

```
http://127.0.0.1:8000/items/foo?short=1
```

или

```
http://127.0.0.1:8000/items/foo?short=True
```

или

```
http://127.0.0.1:8000/items/foo?short=true
```

или

```
http://127.0.0.1:8000/items/foo?short=on
```

или

```
http://127.0.0.1:8000/items/foo?short=yes
```

или любой другой случай, ваша функция будет видеть параметр `short` типа `bool` со значением `True`. Иначе `False`.

<h3>Несколько параметров пути и запроса</h3>

Вы можете объявить несколько параметров пути и запроса за раз, FastAPI знает что есть что.

И вам не нужно определять их каким-то определенным образом.

Параметры будут определены по их имени:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: str | None = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```

<h3>Требования параметров запроса</h3>

Когда вы объявляете значение по умолчанию для параметров (не параметров пути. На данный момент мы видели только
параметры запроса), они являются не обязательными.

Если вы не хотите добавлять конкретное значение, а просто делаете его опциональным, установите значение None.

Но если вам нужно сделать параметр обязательным, вы можете просто не объявлять какое-либо значение по умолчанию:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item
```

Здесь параметр `needy` это обязательный параметр типа `str`.

Если вы откроете браузер по URL, например: 

```
http://127.0.0.1:8000/items/foo-item
```

...без добавления обязательного параметра `needy`, вы увидите такую ошибку:

```JSON
{
    "detail": [
        {
            "loc": [
                "query",
                "needy"
            ],
            "msg": "field required",
            "type": "value_error.missing"
        }
    ]
}
```

Так как `needy` это обязательный параметр, его можно указать в URL:

```
http://127.0.0.1:8000/items/foo-item?needy=sooooneedy
```

...тогда все будет работать:

```JSON
{
    "item_id": "foo-item",
    "needy": "sooooneedy"
}
```

Ну и, конечно, можно объявить некоторые параметры обязательными, некоторые со значениями по умолчанию, а некоторые
опциональными:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_user_item(
    item_id: str, needy: str, skip: int = 0, limit: int | None = None
):
    item = {"item_id": item_id, "needy": needy, "skip": skip, "limit": limit}
    return item
```

В нашем случае у нас три параметра запроса:
* `needy` требуется `str`.
* `skip` это `int` со значением по умолчанию `0`.
* `limit` опциональный `int`.

>**Подсказка**
> 
> Можно использовать `Enum` точно так же как и с параметрами пути.
