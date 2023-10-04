<h3>Параметры пути и валидация чисел</h3>

Таким же способом, что вы объявляли больше проверок и метаданных для параметров запроса с помощью `Query`, вы можете
объявить тип проверок и метаданных для параметров пути с помощью `Path`.

<h3>Импорт Path</h3>

Сперва импортируем `Path` из `fastapi` и импортируем `Annotated`:

```python
from typing import Annotated

from fastapi import FastAPI, Path, Query

app = FastAPI()
```

> **Для информации**
> 
> FastAPI добавил поддержку `Annotated` (и начал рекомендовать его) в версии 0.95.0.
> 
> Если у вас более старая версия, вы можете получить ошибки когда попытаетесь использовать `Annotated`.
> 
> Убедитесь, что вы обновили версию FastAPI по крайней мере до 0.95.1 перед использованием `Annotated`.

<h3>Объявление метаданных</h3>

Вы можете объявить такие же параметры как и для `Query`.

Например, чтобы объявить значение метаданных `title` для параметра пути `item_id` вы можете напечатать:

```python
from typing import Annotated

from fastapi import FastAPI, Path, Query

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get")],
    q: Annotated[str | None, Query(alias="item-query")] = None,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

> **Заметка**
> 
> Параметр пути всегда обязателен, так как он является частью пути.
> 
> Поэтому, вы можете объявить его с помощью `...`, чтобы отметить его как обязательный.
> 
> Тем не менее даже если вы объявили его с помощью `None` или набора значений по умолчанию, это ни на что бы не повлияло,
> он все равно всегда бы был обязательным.

<h3>Упорядочивайте параметры как вы хотите</h3>

> **Совет** 
> 
> Это, вероятно, так неважно или необходимо, если вы используете `Annotated`.

Скажем, что вы хотите объявить параметр запроса `q` как обязательный `str`.

И вам не нужно объявлять что-то еще для этого параметра, поэтому нет необходимости использовать `Query`.

Но вам все еще нужно использовать `Path` для параметра пути `item_id`. И вы не хотите использовать `Annotated` по какой-то
причине.

Python будет жаловаться если вы поместите значение с "умолчанием" перед значением которое не имеет такого.

Но вы можете переопределить их, и иметь значение без умолчания (параметр запроса `q`) первым.

Это не важно для FastAPI. Он будет определять параметры по их именам, типам и начальным объявлениям (`Query`, `Path` и т.д.),
порядок его не волнует.

Поэтому, вы можете объявить функцию как:

> **Совет**
> 
> Предпочтительно использовать `Annotated`, если это возможно.

```python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(q: str, item_id: int = Path(title="The ID of the item to get")):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

Но имейте в виду, что если вы используете `Annotated`, вы можете не иметь проблем с этим, это не будет важным если вы не 
используете значения по умолчанию функции для `Query()` или `Path()`

```python
from typing import Annotated

from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    q: str, item_id: Annotated[int, Path(title="The ID of the item to get")]
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

<h3>Упорядочивайте параметры так, как вам нужно, хитрости</h3>

> **Совет** 
> 
> Это, вероятно, так неважно или необходимо, если вы используете `Annotated`.

Здесь есть маленький трюк, который может быть полезным, но он не понадобится вам часто.

Если вы хотите:

* объявить параметр запроса `q` без `Query` или любого значения по умолчанию
* объявить параметр пути `item_id` используя `Path`
* расположить их в другом порядке
* не использовать `Annotated`

...Python имеет небольшой специальный синтаксис для этого.

Передайте `*`, как первый параметр функции.

Python не будет ничего делать с этой `*`, но он будет знать, что все последующие параметры должны быть обозначены как
ключевые аргументы (пары ключ-значение), также известные как `kwargs`. Даже если они не имеют значение по умолчанию.

```python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(*, item_id: int = Path(title="The ID of the item to get"), q: str):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

<h3>Лучше с  `Annotated`</h3>

Имейте в виду, что если вы используете `Annotated`, так как вы не используете значения параметров функции по умолчанию,
у вас не будет проблем с этим и вам, вероятно, не понадобится использовать `*`.

```python
from typing import Annotated

from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get")], q: str
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

