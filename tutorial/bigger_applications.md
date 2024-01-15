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

### Добавим кастомные `tags`, `responses` и `dependencies`

Мы не добавляем ни префикс `/items`, ни `tags=["items"]` к каждой ***операции пути***, потому что мы добавили их к `APIRouter`.

Но мы все равно можем добавлять еще `tags`, которые применим к определенным ***операциям пути*** и также какие-то
дополнительные `responses`, определенные для этой ***операции пути***:

```python
app/routers/items.py
____________________________

from fastapi import APIRouter, Depends

from ..dependencies import get_token_header


router = APIRouter(
    prefix="/items",
    tags=["items"],
    dependencies=[Depends(get_token_header)],
    responses={404: {"description": "Not found"}},
)

...

@router.put(
  "/item_id",
  tags=["custom"],
  responces={403: {"description": "Operation forbidden"}},
)
async def update_item(item_id: str):
    ...
```

> **Совет**
> 
> Эта операция пути будет иметь комбинацию тегов: `["items", "custom"]`.
> 
> И она также будет иметь оба ответа в документации, один для `404`, и один для `403`.

## Основной `FastAPI`

Теперь, давайте рассмотрим модуль `app/main.py`.

Здесь вы импортируете и используете класс `FastAPI`.

Это будет главным файлом вашего приложения, который связывает все вместе.

И поскольку основная часть вашей логики теперь будет жить в своих собственных модулях, главный файл будет довольно простым.

### Импорт FastAPI

Вы импортируете и создаете класс `FastAPI` как обычно.

И мы даже можем объявить 
<a href="https://github.com/amoglock/FastAPI_documentation/blob/master/tutorial/global_dependencies.md">глобальные зависимости</a>,
которые будут совмещены для каждого `APIRouter`:

```python
app/main.py
________________

from fastapi import FastAPI, Depends

from .dependencies import get_query_token, get_token_header


app = FastAPI(dependencies=[Depends(get_query_token)])

...
```

### Импорт `APIROuter`

Теперь мы импортируем другие подмодули, которые имеют `APIRouter`:

```python
app/main.py
________________

from fastapi import FastAPI, Depends

from .dependencies import get_query_token, get_token_header
from .internal import admin
from .routers import items, users

app = FastAPI(dependencies=[Depends(get_query_token)])

...
```

Так как файлы `app/routers/user.py` и `app/routers/items.py` это подмодули, которые являются частью того же пакета `app`,
мы можем использовать одну точку `.` чтобы импортировать их, используя "относительный импорт".

### Как работает импортирование

Секция:
```python
from .routers import items, users
```

Означает:

* Запускается в том же пакете, где живет этот модуль (файл `app/main.py`, директория `app/`)...
* Ищет подпакет `routers` (директория `app/routers/`)...
* И из него импортирует подмодуль `items` (файл `app/routers/items.py`) и `users` (файл `app/routers/users.py`)...

Модуль `items` будет иметь переменную `router` (`items.router`). Это та, которую мы создали в файле `app/routers/items.py`,
это объект `APIRouter`.

И затем мы делаем то же самое для модуля `users`.

Мы можем импортировать их и так:

```python
from app.routers import items, users
```

> **Для информации**
> 
> Первая версия это "относительный импорт":
> ```python
> from .routers import items, users
> ```
> Вторая версия это "абсолютный импорт":
> ```python
> from app.routers import items, users
> ```
> 
> Чтобы узнать больше про пакеты Python и модули, почитайте 
> <a href="https://docs.python.org/3/tutorial/modules.html">официальную документацию Python о модулях</a>.

### Избегайте коллизий имен

Мы импортируем подмодуль `items` напрямую, вместо импорта просто его переменной `router`.

Это потому, что у нас есть еще одна переменная с именем `router` в подмодуле `users`.

Если бы импортировали их один за другим:

```python
from .routers.items import router
from .routers.users import router
```

То `router` из `users` перезаписал бы `router` из `items` и мы не смогли бы использовать их одновременно.

Поэтому, чтобы использовать их обоих в одном файле, мы импортируем подмодули напрямую:

```python
from .routers import items, users
```

### Подключение `APIRouter` для `users` и `items`

Теперь давайте включим `router` из подмодулей `users` и `items`:

```python
app/main.py
________________

from fastapi import FastAPI, Depends

from .dependencies import get_query_token, get_token_header
from .internal import admin
from .routers import items, users

app = FastAPI(dependencies=[Depends(get_query_token)])

app.include_router(users.router)
app.include_router(items.router)
...
```

> **Для информации**
> 
> `users.router` содержит `APIRouter` внутри файла `app/routers/users.py`.
> 
> И `items.router` содержит `APIRouter` внутри файла `app/routers/items.py`.

С помощью `app.include_router()` мы можем добавлять каждый `APIRouter` к главному приложению `FastAPI`.

Он включит все `router` как часть приложения.

> **Технические детали**
> 
> Фактически это внутренне создает ***операцию пути*** для каждой ***операции пути**, которая объявлена в `APIRouter`.
> 
> Поэтому, под капотом, все будет работать так, как если бы все это было одним приложением.

> **Для галочки**
> 
> Вам не нужно беспокоиться о производительности, когда подключаете роутеры.
> 
> Они потребуют микросекунды и это произойдет только при старте.
> 
> Поэтому это не влияет на производительность. ⚡

### Включение `APIRouter` с кастомным `prefix`, `tags`, `responses` и `dependencies`

Теперь представим, что ваша организация дала вам файл `app/internal/admin.py`.

Он содержит `APIRouter` с какими-то ***операциями пути*** администратора, которые ваша организация использует в нескольких
проектах.

Для этого примера он будет супер простым. Но скажем, из-за того, что он используется с несколькими проектами организации,
мы не можем изменять его и добавлять `prefix`, `dependencies`, `tags` и т.п. напрямую в `APIRouter`:

```python
from fastapi import APIRouter

router = APIRouter()

@router.post("/")
async def update_admin():
    return {"message": "Admin getting schwifty"}
```

Но мы все равно хотим установить кастомный `prefix` когда включаем `APIRouter`, так, чтобы все его ***операции пути***
начинались с `/admin`, мы хотим обезопасить его с `dependencies`, которые у нас уже есть для этого проекта, и мы хотим
включить в него `tags` и `responses`.

Мы можем объявить все это без необходимости изменять оригинальный `APIRouter`, передав эти параметры в `app.include_router()`:

```python
app/main.py
_________________

from fastapi import Depends, FastAPI

from .dependencies import get_query_token, get_token_header
from .internal import admin
from .routers import items, users

app = FastAPI(dependencies=[Depends(get_query_token)])


app.include_router(users.router)
app.include_router(items.router)
app.include_router(
    admin.router,
    prefix="/admin",
    tags=["admin"],
    dependencies=[Depends(get_token_header)],
    responses={418: {"description": "I'm a teapot"}},
)

...
```

Таким образом, оригинальный `APIRouter` останется неизмененным, поэтому мы все еще использовать тот же `app/internal/admin.py`
файл с другими проектами организации.

В результате, в нашем приложении каждая ***операция пути*** из модуля `admin` будет иметь:

* Префикс `/admin`.
* Тег `admin`.
* Зависимость `get_token_header`.
* Ответ `418`. 🍵

Но это будет влиять только на `APIRouter` в нашем приложении, а не на какой-то еще код, использующий его.

Поэтому, например, другие проекты могут использовать тот же `APIRouter` с отличающимся методом аутентификации.

### Включение ***операции пути***

Мы можем также добавлять ***операции пути*** напрямую к приложению `FastAPI`.

Здесь мы это делаем так... чтобы просто показать, что мы можем это 🤷:

```python
app/main.py
_________________

from fastapi import Depends, FastAPI

from .dependencies import get_query_token, get_token_header
from .internal import admin
from .routers import items, users

app = FastAPI(dependencies=[Depends(get_query_token)])


app.include_router(users.router)
app.include_router(items.router)
app.include_router(
    admin.router,
    prefix="/admin",
    tags=["admin"],
    dependencies=[Depends(get_token_header)],
    responses={418: {"description": "I'm a teapot"}},
)

@app.get("/")
async def root():
    return {"message": "Hello Bigger Applications!"}
```

И это будет работать корректно, вместе со всеми другими ***операциями пути***, добавленными с `app.include_router()`.

> **Очень технические детали**
> 
> **Заметка**: это совсем техническая деталь, которую, вероятно, вы можете **просто пропустить**.
> ***
> `APIRouter`ы не "монтируются", они не изолированы от остального приложения.
> 
> Это потому, что мы хотим включить их ***операции пути*** в схему OpenAPI и пользовательский интерфейс.
>
> Так как мы не можем просто изолировать и "смонтировать" их отдельно от остального, ***операции пути*** "клонируются"
> (пересоздаются), а не включены напрямую.

## Проверка автоматической документации API

Теперь запустим `uvicorn`, используя модуль `app.main` и переменную `app`:

```commandline
uvicorn app:main --reload
```

И откроем документацию http://127.0.0.1:8000/docs.

Вы увидите автоматическую документацию API, включающую в себя пути из всех подмодулей, использующую корректные пути 
(и префиксы), и корректные теги:

<img src="https://fastapi.tiangolo.com/img/tutorial/bigger-applications/image01.png">

## Включение одного роутера несколько раз с разными `prefix`

Еще вы можете использовать `.include_router()` несколько раз с одним и тем же роутером, используя разные префиксы.

Это может быть полезным, например, чтобы показать один и тот же API под разными префиксами, например `/api/v1` и 
`/api/latest`.

Это продвинутое использование, которое вам может и не понадобиться, но оно есть в случае вы так сделаете.

## Включение `APIRouter` в другой

Точно так же как вы можете включать `APIRoter` в приложение `FastAPI`, вы можете включать `APIRouter` в другой `APIRouter`,
используя:

```python
router.include_router(other_router)
```

Убедитесь, что вы делаете это перед включением `router` в приложение `FastAPI`, для того, чтобы ***операции пути*** из
`other_router` так же были включены.
