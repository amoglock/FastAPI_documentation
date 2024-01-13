# Большие приложения - несколько файлов

Если строите приложение или web API, в редких случаях вы можете разместить все в единственном файле.

FastAPI предоставляет удобный инструмент, чтобы структурировать ваше приложение, сохраняя при этом всю гибкость.

> **Для информации**
> 
> Если вы пришли из Flask, это было бы эквивалентом Flask Blueprints.

## Пример файловой структуры

Скажем, у вас есть такая файловая структура:

```text
.
├── app
│   ├── __init__.py
│   ├── main.py
│   ├── dependencies.py
│   └── routers
│   │   ├── __init__.py
│   │   ├── items.py
│   │   └── users.py
│   └── internal
│       ├── __init__.py
│       └── admin.py
```

> **Совет**
> 
> Здесь у нас несколько `__init__.py` файлов: по одному в каждой директории или поддиректории.
> 
> Это то, что позволяет импортировать код из одного файла в другой.
> 
> Например, в `app/main.py` у вас могла быть такая строка:
> ```python
> from app.routers import items
>```

* Все файлы находятся в директории `app`. И она имеет пустой файл `app/__init__.py`, поэтому это "Пакет Python" (коллекция
"Python модулей"): `app`.
* Он содержит файл `app/main.py`. Так как он внутри Python пакета (папки с файлом `__init__.py`), это "модуль" этого
пакета: `app.main`.
* Также есть файл `app/dependencies.py`, как и `app/main.py`, это "модуль": `app.dependencies`.
* Есть поддиректория `app/routers` с другим файлом `__init__.py`, поэтому это "подпакет Python": `app.routers`.
* Файл `app/routers/items.py` находится внутри пакета `app/routers`, поэтому это подмодуль: `app.routers.items`.
* То же самое с `app/routers/users.py`, это другой подмодуль: `app.routers.users`.
* Есть другая поддиректория `app/internal`, с другим файлом `__init__.py`, поэтому это другой "подпакет Python": `app.internal`.
* А файл `app/internal/admin.py` это другой подмодуль: `app.internal.admin`.

<img src="https://fastapi.tiangolo.com/img/tutorial/bigger-applications/package.svg">

Та же файловая структура с комментариями:

```text
.
├── app                  # "app" is a Python package
│   ├── __init__.py      # this file makes "app" a "Python package"
│   ├── main.py          # "main" module, e.g. import app.main
│   ├── dependencies.py  # "dependencies" module, e.g. import app.dependencies
│   └── routers          # "routers" is a "Python subpackage"
│   │   ├── __init__.py  # makes "routers" a "Python subpackage"
│   │   ├── items.py     # "items" submodule, e.g. import app.routers.items
│   │   └── users.py     # "users" submodule, e.g. import app.routers.users
│   └── internal         # "internal" is a "Python subpackage"
│       ├── __init__.py  # makes "internal" a "Python subpackage"
│       └── admin.py     # "admin" submodule, e.g. import app.internal.admin
```

## `APIRouter`

Допустим файл выделенный чтобы просто обрабатывать пользователей, находится в подмодуле `/app/routers/users.py`.

Вы хотите иметь ***операцию пути***, относящуюся к вашим пользователям, отдельно от остального кода, чтобы поддерживать
порядок.

Но это все еще часть одного приложения FastAPI/web API (это часть одного "пакета Python").

Вы можете создавать ***операции пути*** для таких модулей, используя `APIRouter`.

### Импорт `APIRouter`

Вы импортируете его и создаете "экземпляр" точно так же как с классом `FastAPI`:

```python
app/routers/users.py
________________________________

from fastapi import APIRouter

router = APIRouter()
```

### ***Операции пути*** с `APIRouter`

А затем вы используете его, чтобы объявить ваши ***операции пути***.

Используйте его так же, как вы бы использовали класс `FastAPI`:

```python
app/routers/users.py
________________________________

from fastapi import APIRouter

router = APIRouter()

@router.get("/users/", tags=["users"])
...

@router.get("/users/me", tags=["users"])
...

@router.get("/users/{username}", tags=["users"])
...
```

Вы можете думать о `APIRouter` как о "мини `FastAPI" классе.

Поддерживаются все те же опции.

Все те же `parameters`, `responses`, `dependencies`, `tags`, и т.д.

> **Совет**
> 
> В этом примере переменная называется `router`, но вы можете называть как хотите.

Мы собираемся включить этот `APIRouter` в основное приложение `FastAPI`, но сперва, давайте посмотрим на зависимости и
другой `APIRouter`.

## Зависимости

Мы видим, что нам понадобится использовать несколько зависимостей в разных местах приложения.

Поэтому мы размещаем их в своем модуле `dependencies` (`app/dependencies.py`).

Теперь мы будем использовать простую зависимость, чтобы прочитать кастомный заголовок `X-Token`.

```python
app/dependencies.py
___________________________

from typing import Annotated

from fastapi import Header, HTTPException

async def get_token_header(x_token: Annotated[str, Header()]):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")
```

> **Совет**
> 
> Сейчас мы используем придуманный заголовок для упрощения примера.
> 
> Но в реальных ситуациях вы получите лучшие результаты, используя встроенные 
> <a href="https://github.com/amoglock/FastAPI_documentation/blob/master/tutorial/security.md">инструменты безопасности</a>.

## Другой модуль с `APIRouter`

Допустим у вас сть еще ендпоинты, предназначенные для обработки "items" из вашего приложения, в модуле `app/router/items.py`.

У вас есть ***операции пути*** для:

* `/items/`
* `/items/{item_id}`

Все они в той же структуре как `app/routers/users.py`.

Но мы хотим чтобы код был немного элегантнее и проще.

Мы знаем, что все ***операции пути** в этом модуле имеют одинаковые:

* `Префикс` пути: `/items`.
* `tags`: (один тег: `items`).
* Дополнительные `responses`.
* `dependencies`: всем им нужно, чтобы мы создали зависимость `X-Token`.

Поэтому, вместо добавления всего этого к каждой ***операции пути***, мы можем добавить их к `APIRouter`:

```python
from fastapi import APIRouter, Depends, HTTPException

from ..dependencies import get_token_header

router = APIRouter(
    prefix="/items",
    tags=["items"],
    dependencies=[Depends(get_token_header)],
    responses={404: {"description": "Not found"}},
)

fake_items_db = {"plumbus": {"name": "Plumbus"}, "gun": {"name": "Portal Gun"}}


@router.get("/")
async def read_items():
    return fake_items_db


@router.get("/{item_id}")
async def read_item(item_id: str):
    if item_id not in fake_items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"name": fake_items_db[item_id]["name"], "item_id": item_id}


@router.put(
    "/{item_id}",
    tags=["custom"],
    responses={403: {"description": "Operation forbidden"}},
)
async def update_item(item_id: str):
    if item_id != "plumbus":
        raise HTTPException(
            status_code=403, detail="You can only update the item: plumbus"
        )
    return {"item_id": item_id, "name": "The great Plumbus"}
```

Так как путь каждой ***операции пути*** начинается с `/`, как в:

```python
@router.get("/{item_id}")
async def read_item(item_id: str):
    ...
```

Префикс не должен включать в себя окончание `/`.

Поэтому префикс в этом случае это `/items`.

Мы можем добавлять список `тегов` и дополнительных `responses`, которые будут применены ко всем ***операциям пути***,
включенным в этот роутер.

И мы можем добавлять список `зависимостей`, которые будут добавлены ко всем ***операциям пути*** в роутере и будут 
выполнены/решены для каждого запроса, сделанного к ним.

> **Совет**
> 
> Обратите внимание, что это сильно похоже на 
> <a href="https://github.com/amoglock/FastAPI_documentation/blob/master/tutorial/dependencies_in_path_operation_decorators.md">Зависимости в декораторах операции пути</a>,
> нет значения, которое будет передано в вашу ***функцию операции пути***.

В конечном итоге, элементы пути теперь:

* `/items/`
* `/items/{item_id}

Как мы и планировали.

* Они будут помечены списком тегов, который содержит простую строку `"items"`.
  * Эти "теги" особенно полезны для систем автоматической интерактивной документации (использующих OpenAPI).
* Все они будут включать предопределенный `responses`.
* Все ***операции пути*** будут иметь список `зависимостей` вычисленные/выполненные перед ними.
  * Если вы еще объявляете зависимости для определенной ***операции пути***, они тоже будут выполнены.
  * первыми выполняются зависимости роутера, затем  
  <a href="https://github.com/amoglock/FastAPI_documentation/blob/master/tutorial/dependencies_in_path_operation_decorators.md">`зависимости` в декораторе</a>,
  а затем нормальные зависимости параметров.
  * Еще вы можете добавить зависимости `безопасности` со `scopes`.

> **Совет**
> 
> Наличие `зависимостей` в `APIRouter` может быть использовано, например, чтобы требовать аутентификацию для целой группы
> ***операций пути***. Даже если зависимости не добавлены индивидуально для каждой из них.

> **Для галочки**
> 
> Параметры `prefix`, `tags`, `responses` и `dependencies` (во многих других случаях) просто возможность из FastAPI, 
чтобы помочь вам избавиться от дублирования кода.

### Импорт зависимостей

Этот код живет в модуле `app.routers.items`, файл `app/routers/items.py`.

А нам нужно получать функцию зависимости из модуля `app.dependencies`, файл `app/dependencies.py`.

Поэтому мы используем импорт с `..` для зависимостей:

```python
from fastapi import APIRouter, Depends, HTTPException

from ..dependencies import get_token_header

...
```

#### Как работает относительный импорт

> **Совет**
> 
> Если вы хорошо знаете как работают импорты, переходите к следующей секции ниже.

Одна точка `.`, как в:

```python
from .dependencies import get_token_header
```

Означала бы:

* Запускается в том же пакете, где находится этот модуль (файл `app/routers/items.py`) (директория `app/routers/`)...
* Находит модуль `dependencies` (воображаемый файл в `app/routers/dependencies.py)...
* И из него импортирует функцию `get_token_header`.

Но такого файла не существует, наши зависимости в файле `app/dependencies.py`.

Вспомните как выглядит наше приложение/файловая структура:

<img src="https://fastapi.tiangolo.com/img/tutorial/bigger-applications/package.svg">

*****

Две точки `..` как в:

```python
from ..dependencies import get_token_header
```

Означают:

* Запускается в том же пакете, где находится этот модуль (файл `app/routers/items.py`) (директория `app/routers/`)...
* Переходит в родительский пакет (директория `app/`)...
* И здесь находит модуль `dependencies` (файл `app/dependencies.py`)...
* И из него импортировать функцию `get_token_header`.

Все работает корректно! 🎉

*****

Точно так же, если бы использовали три точки `...`, как в:

```python
from ...dependencies import get_token_header
```

Что означало бы:

* Запускается в том же пакете, где находится этот модуль (файл `app/routers/items.py`) (директория `app/routers/`)...
* Переходит в родительский пакет (директория `app/`)...
* Затем переходит к родителю этого пакета (родительского пакета нет, `app` это верхний уровень 😱)...
* И там, находит модуль `dependencies` (файл `app/dependencies.py`)...
* И из него импортировать функцию `get_token_header`.

Это относилось бы к какому-то пакету выше `app/`, с его собственным файлом `__init__.py`, и т.д. Но у нас его нет. Поэтому,
это бы привело к ошибке в нашем примере.🚨

Но теперь мы знаем как это работает, поэтому вы можете использовать относительные импорты в ваших приложениях, не важно
насколько они сложные. 🤓

