<h3>Подзависимости</h3>

Вы можете создавать зависимости, которые имеют подзависимости.

Они могут быть настолько глубокими, насколько вам нужно.

FastAPI позаботится о том, чтобы их решить.

<h4>Первая "зависимая" зависимость</h4>

Вы можете создать первую зависимость ("зависимое"), например:

```python
from typing import Annotated

from fastapi import Cookie, Depends, FastAPI

app = FastAPI()

def query_extractor(q: str | None = None):
    return q

...
```

Она объявляет необязательный параметр запроса `q` как `str`, а затем просто возвращает его.

Это очень просто (не очень полезно), но поможет нам сосредоточиться на том как работают подзависимости.

<h4>Вторая зависимость, "зависимая" и "зависящая"</h4>

Затем вы можете создать другую функцию зависимости ("зависимое"), которая в то же время объявляет зависимость из себя (
поэтому она также "зависящая"):

```python
from typing import Annotated

from fastapi import Cookie, Depends, FastAPI

app = FastAPI()


def query_extractor(q: str | None = None):
    return q


def query_or_cookie_extractor(
    q: Annotated[str, Depends(query_extractor)],
    last_query: Annotated[str | None, Cookie()] = None,
):
    if not q:
        return last_query
    return q
...
```

Давайте сфокусируемся на объявленных параметрах:

* Даже несмотря на то, что эта функция сама по себе является зависимой ("надежной"), она также объявляет другую 
зависимость (она "зависит" от чего-то еще).
  * Она зависит от `query_exctractor` и присваивает значение, возвращенное им в параметр `q`.
* Она также объявляет необязательный  `last_query` куки, как `str`:
  * Если пользователь не предоставил любой запрос `q`, мы используем последний использованный запрос, который мы сохранили
    в куки до этого.

<h4>Используй зависимость</h4>

Затем мы можем использовать зависимость:

```python
from typing import Annotated

from fastapi import Cookie, Depends, FastAPI

app = FastAPI()


def query_extractor(q: str | None = None):
    return q


def query_or_cookie_extractor(
    q: Annotated[str, Depends(query_extractor)],
    last_query: Annotated[str | None, Cookie()] = None,
):
    if not q:
        return last_query
    return q


@app.get("/items/")
async def read_query(
    query_or_default: Annotated[str, Depends(query_or_cookie_extractor)]
):
    return {"q_or_cookie": query_or_default}
```

> **Для информации**
> 
> Обратите внимание, что у нас только одна объявленная зависимость в функции операции пути, `query_or_cookie_extractor`.
> 
> Но FastAPI будет знать, что она должна сперва решить `query_extractor`, чтобы передать его результаты в `query_or_cookie_extractor`.