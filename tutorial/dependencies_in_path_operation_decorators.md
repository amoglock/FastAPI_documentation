<h3>Зависимости в декораторах операции пути</h3>

В некоторых случаях вам не нужно, на самом деле, возвратное значение зависимости внутри функции операции пути.

Или зависимость не возвращает значение.

Но вам оно все же нужно, чтобы она была выполненной.

Для этих случаев, вместо объявления параметра функции операции пути с `Depends`, вы можете добавлять `list` из 
`dependencies` в декоратор операции пути.

<h4>Добавление `dependencies` к декоратору операции пути</h4>

Это должен быть `list` из `Depends()`:

```python
from typing import Annotated

from fastapi import Depends, FastAPI, Header, HTTPException

app = FastAPI()

...

@app.get("/items/", dependencies=[Depends(verify_token), Depends(verify_key)])
async def read_items():
    return [{"item": "Foo"}, {"item": "Bar"}]
```

Эти зависимости будут выполнены/решены точно так же как обычные зависимости. Но их значение (если они что-то возвращают)
не будут переданы в вашу функцию операции пути.

> **Совет**
> 
> Некоторые редакторы проверяют на неиспользуемые параметры и показывают их как ошибки.
> 
> Используя эти `dependencies` в декораторе операции пути, вы можете быть уверены, что они выполнены, избегая ошибки 
> редактора или инструментов.
> 
> Это также может помочь избегать замешательства для новых разработчиков, которые видя неиспользуемые параметры в вашем
> коде, могут подумать, что они (зависимости) бесполезные.

> **Для информации**
> 
> В примере мы будем использовать выдуманные, самодельные заголовки `X-Key` и `X-Token`.
> 
> Но в реальных ситуациях, когда реализуется безопасность, вы бы получили больше выгоды от использования встроенных
> <a href="https://github.com/amoglock/FastAPI_documentation/blob/master/tutorial/security.md">Утилит безопасности (следующая глава)</a>.

<h4>Ошибки зависимостей и возврат значений</h4>

Вы можете использовать те же функции зависимостей, которые используете обычно.

<h5>Требования зависимостей</h5>

Они могут объявлять требования запроса (например, заголовки) или другие подзависимости:

```python
from typing import Annotated

from fastapi import Depends, FastAPI, Header, HTTPException

app = FastAPI()

async def verify_token(x_token: Annotated[str, Header()]):
    ...


async def verify_key(x_key: Annotated[str, Header()]):
    ...

@app.get("/items/", dependencies=[Depends(verify_token), Depends(verify_key)])
async def read_items():
    return [{"item": "Foo"}, {"item": "Bar"}]
```

<h5>Поднятие исключений</h5>

Эти зависимости могут вызывать исключения, так же как обычные зависимости:

```python
from typing import Annotated

from fastapi import Depends, FastAPI, Header, HTTPException

app = FastAPI()


async def verify_token(x_token: Annotated[str, Header()]):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")
    ...

async def verify_key(x_key: Annotated[str, Header()]):
    if x_key != "fake-super-secret-key":
        raise HTTPException(status_code=400, detail="X-Key header invalid")
    ...


@app.get("/items/", dependencies=[Depends(verify_token), Depends(verify_key)])
async def read_items():
    return [{"item": "Foo"}, {"item": "Bar"}]
```

<h5>Возврат значений</h5>

И они могут возвращать или не возвращать значения, значения могут не быть использованными.

Поэтому, вы можете переиспользовать обычную зависимость (возвращает значение) которую вы уже используете где-то в другом
месте. Даже несмотря на то, что значение не будет использовано, зависимость выполнится:

```python
from typing import Annotated

from fastapi import Depends, FastAPI, Header, HTTPException

app = FastAPI()


async def verify_token(x_token: Annotated[str, Header()]):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")


async def verify_key(x_key: Annotated[str, Header()]):
    if x_key != "fake-super-secret-key":
        raise HTTPException(status_code=400, detail="X-Key header invalid")
    return x_key


@app.get("/items/", dependencies=[Depends(verify_token), Depends(verify_key)])
async def read_items():
    return [{"item": "Foo"}, {"item": "Bar"}]
```

<h4>Зависимость для группы операций пути</h4>

Позже, при чтении о том как структурировать большие приложения, вероятно с множеством файлов, вы узнаете как объявлять
один параметр `dependencies` для группы операций пути.

<h4>Глобальные зависимости</h4>

Далее мы увидим как добавить зависимости во всё приложение `FastAPI`, так, чтобы они применялись к каждой операции пути.