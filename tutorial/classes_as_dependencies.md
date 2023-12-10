<h3>Классы как зависимости</h3>

Перед более глубоким погружением в системы внедрения зависимостей, давайте обновим предыдущий пример.

<h4>`dict` из предыдущего примера</h4>

В прошлом примере мы возвращаем `dict` из нашей зависимости ("зависимого"):

```python
from typing import Annotated

from fastapi import Depends, FastAPI

app = FastAPI()


async def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
async def read_items(commons: Annotated[dict, Depends(common_parameters)]):
    return commons


@app.get("/users/")
async def read_users(commons: Annotated[dict, Depends(common_parameters)]):
    return commons
```

Но затем мы получаем `dict` в параметре `commons` функции операции пути.

А мы знаем, что редакторы не могут предоставить много поддержки (например завершение) для словарей, потому, что они 
(редакторы) не знают типов ключей и значений.

Мы можем сделать лучше...

<h4>Что делает зависимость</h4>

До сих пор вы видели зависимости, объявленные как функции.

Но это не единственный способ объявить зависимости (хотя он был бы, вероятно, более частым).

Ключевой фактор это, то что зависимость должна быть "callable".

"Callable" в Python это что-то, что Python может "вызывать" как функцию.

Поэтому, если у вас есть объект `something` (который может не быть функцией) и вы можете "вызывать" его (выполнять):

```python
something()
```

или 

```python
something(some_argument, some_keyword_argument="foo")
```

Тогда это и есть "callable".

<h4>Классы как зависимости</h4>

Вы можете обратить внимание, чтобы создать экземпляр класса Python, вы используете одинаковый синтаксис.

Например:

```python
class Cat:
    def __init__(self, name: str):
        self.name = name

fluffy = Cat(name="Mr Fluffy")
```

В этом случае, `fluffy` это экземпляр класса `Cat`.

А чтобы создать `fluffy`, вы "вызываете" `Cat`.

Поэтому, класс Python - тоже callable.

Тогда, в FastAPI, вы можете использовать класс Python как зависимость.

Что на самом деле проверяет FastAPI, это то, что это "callable" (функция, класс или что-то еще) и объявленные параметры.

Если вы передаете "callable" как зависимость в FastAPI, он будет анализировать параметры для этого "callable", и обрабатывать
их точно так же как параметры для функции операции пути. Включая подзависимости.

Это также относится к вызываемым объектам вообще без параметров. То же самое, что было бы для функций операции пути 
без параметров.

Теперь мы можем изменить зависимость "зависимое" `common_parameters` выше, на класс `CommonQueryPrams`:

```python
from typing import Annotated

from fastapi import Depends, FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

class CommonQueryParams:
    def __init__(self, q: str | None = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/items/")
async def read_items(commons: Annotated[CommonQueryParams, Depends(CommonQueryParams)]):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

Обратите внимание на метод `__init__` используемый для создания экземпляра класса:

```python
from typing import Annotated

from fastapi import Depends, FastAPI

app = FastAPI()


fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


class CommonQueryParams:
    def __init__(self, q: str | None = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit
...
```

Он имеет те же параметры, как и наш предыдущий `common_parameters`:

```python
from typing import Annotated

from fastapi import Depends, FastAPI

app = FastAPI()


async def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}
...
```

Эти параметры и есть, то, что FastAPI будет использовать для "решения" зависимости.

В обоих случаях, они будут иметь:

* Необязательный параметр запроса `q` в `str`.
* Параметр запроса `skip` в `int` со значением по умолчанию `0`.
* Параметр запроса `limit` в `int` со значением по умолчанию `100`.

В обоих случаях данные будут конвертированы, проверены, документированы в схеме OpenAPI, и т.д.

<h4>Используйте это</h4>

Теперь вы можете объявлять вашу зависимость, используя этот класс:

```python
from typing import Annotated

from fastapi import Depends, FastAPI

app = FastAPI()


fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


class CommonQueryParams:
    def __init__(self, q: str | None = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit


@app.get("/items/")
async def read_items(commons: Annotated[CommonQueryParams, Depends(CommonQueryParams)]):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

FastAPI вызывает класс `CommonQueryParams`. Это создаёт "экземпляр" класса и экземпляр будет передан как параметр `commons`
в вашу функцию.

<h4>Аннотация типа против `Depends`</h4>

Заметьте как мы пишем `CommomQueryParams` дважды в коде выше:

```python
commons: Annotated[CommonQueryParams, Depends(CommonQueryParams)]
```

Последний `CommonQueryParams`, в:

```python
... Depends(CommonQueryParams)
```

Это то, что FastAPI будет на самом деле использовать, чтобы знать зависимость.

Из этого следует, что FastAPI извлечёт объявленные параметры и, что вызовет.
***

В нашем случае, первый `CommonQueryParams` в:

```python
commons: Annotated[CommonQueryParams, ...
```

Не имеет особого значения для FastAPI. FastAPI не будет использовать его для преобразования данных, проверки, и т.д.
(так как для этого он использует `Depends(CommonQueryParams)`).

На самом деле, вы можете просто написать:

```python
commons: Annotated[Any, Depends(CommonQueryParams)]
```

Как в примере:

```python
from typing import Any, Union

from fastapi import Depends, FastAPI
from typing_extensions import Annotated

app = FastAPI()


fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


class CommonQueryParams:
    def __init__(self, q: Union[str, None] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit


@app.get("/items/")
async def read_items(commons: Annotated[Any, Depends(CommonQueryParams)]):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

Но объявление типа поощряется, так-как таким образом ваш редактор будет знать, что будет предано как параметр `commons`,
а затем он может помогать вам с завершением кода, проверками типов, и т.д.:

<img src="https://fastapi.tiangolo.com/img/tutorial/dependencies/image02.png">

<h4>Упрощение</h4>

Но вы видите, чо у нас есть некоторое повторение кода, дважды написан `CommonQueryParams`:

```python
commons: Annotated[CommonQueryParams, Depends(CommonQueryParams)]
```

FastAPI предоставляет упрощение для таких случаев, в которых зависимость это определенный класс, его FastAPI будет
"вызывать" для создания самого экземпляра класса.

Для этих особенных случаев, вы можете делать следующее:

Вместо написания:

```python
commons: Annotated[CommonQueryParams, Depends(CommonQueryParams)]
```

Вы пишете:

```python
commons: Annotated[CommonQueryParams, Depends()]
```

Вы объявляете зависимость как тип параметра и используете `Depends()` без какого-либо параметра, вместо необходимости
писать еще раз весь класс внутри `Depends(CommonQueryParams)`.

Тот же пример, тогда бы выглядел так:

```python
from typing import Annotated

from fastapi import Depends, FastAPI

app = FastAPI()


fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


class CommonQueryParams:
    def __init__(self, q: str | None = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit


@app.get("/items/")
async def read_items(commons: Annotated[CommonQueryParams, Depends()]):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

И FastAPI будет знать что делать.

> **Подсказка**
> 
> Если это кажется скорее запутанным, чем полезным, не обращайте на это внимания, вам это не нужно.
> 
> Это просто упрощение. Так как FastAPI заботится о том, чтобы помочь вам минимизировать повторение вашего кода.