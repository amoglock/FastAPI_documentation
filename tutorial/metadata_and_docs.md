# Метаданные и URLs документации

Вы можете настраивать отдельные конфигурации метаданных в вашем приложении FastAPI.

## Метаданные для API

Вы можете устанавливать следующие поля, которые используются в спецификации OpenAPI и интерфейсе автоматической 
документации API:

| Параметр           | Тип      | Описание                                                                                                                                                                                                                                                                                                                                                                                                                        |
|--------------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `title`            | `str`    | Название API                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `summary`          | `str`    | Краткое сводка об API                                                                                                                                                                                                                                                                                                                                                                                                           |
| `description`      | `str`    | Краткое описание API. Можно использовать разметку.                                                                                                                                                                                                                                                                                                                                                                              |
| `version`          | `string` | Версия API. Это версия приложения, не OpenAPI. Например `2.5.0`.                                                                                                                                                                                                                                                                                                                                                                |
| `terms_of_servise` | `str`    | URL с условиями услуг для API. Если указан, это должен быть URL.                                                                                                                                                                                                                                                                                                                                                                |
| `contact`          | `dict`   | Контактная информация для открытого API. Может содержать несколько полей:<br/>`name`: `str` - Имя контактного лица/организации.<br/>`url`: `str` - URL, указывающий на контактную информацию. ДОЛЖЕН быть в формате URL.<br/>`email`: `str` - Электронный адрес лица/организации. ДОЛЖЕН быть в формате электронного адреса.                                                                                                    |
| `license_info`     | `dict`   | Информация о лицензии для открытого API. Может содержать несколько полей:<br/> `name`: `str` - ОБЯЗАТЕЛЬНО (если установлен `license_info`). Название лицензии, используемой API.<br/> `identifier`: `str` - <a href="https://spdx.org/licenses/">SPDX</a> выражение лицензии API. Поле `identifier` взаимоисключающее по отношению к полю `url`.<br/>`url`: `str` - URL лицензии, используемой API. ДОЛЖНА быть в формате URL. |

Вы можете установить это следующим образом:

```python
from fastapi import FastAPI

description = """
ChimichangApp API helps you do awesome stuff. 🚀

## Items

You can **read items**.

## Users

You will be able to:

* **Create users** (_not implemented_).
* **Read users** (_not implemented_).
"""

app = FastAPI(
    title="ChimichangApp",
    description=description,
    summary="Deadpool's favorite app. Nuff said.",
    version="0.0.1",
    terms_of_service="http://example.com/terms/",
    contact={
        "name": "Deadpoolio the Amazing",
        "url": "http://x-force.example.com/contact/",
        "email": "dp@x-force.example.com",
    },
    license_info={
        "name": "Apache 2.0",
        "url": "https://www.apache.org/licenses/LICENSE-2.0.html",
    },
)


@app.get("/items/")
async def read_items():
    return [{"name": "Katana"}]
```

> **Совет**
> 
> Вы можете использовать язык разметки в поле `description` и он будет отображен на выходе.

С такой конфигурацией, автоматическая документация API выглядела бы так:

![](https://fastapi.tiangolo.com/img/tutorial/metadata/image01.png)

## Идентификатор лицензии

Начиная с OpenAPI 3.1.0 и FastAPI 0.99.0, вы можете установить `license_info` с `identifier` вместо `url`.

Например:

```python
from fastapi import FastAPI

description = """
ChimichangApp API helps you do awesome stuff. 🚀

## Items

You can **read items**.

## Users

You will be able to:

* **Create users** (_not implemented_).
* **Read users** (_not implemented_).
"""

app = FastAPI(
    title="ChimichangApp",
    description=description,
    summary="Deadpool's favorite app. Nuff said.",
    version="0.0.1",
    terms_of_service="http://example.com/terms/",
    contact={
        "name": "Deadpoolio the Amazing",
        "url": "http://x-force.example.com/contact/",
        "email": "dp@x-force.example.com",
    },
    license_info={
        "name": "Apache 2.0",
        "identifier": "MIT",
    },
)


@app.get("/items/")
async def read_items():
    return [{"name": "Katana"}]
```

## Метаданные для тегов

Вы можете добавлять дополнительные метаданные для различных тегов, используемые для группировки ваших операций пути, с
помощью параметра `openapi_tags`.

Он принимает список, содержащий один словарь для каждого тега.

Каждый словарь может содержать:

* `name` (обязательно): `str` с таким же названием тега, который вы используете в параметре `tags` в ***операции пути**
и `APIRouter`.
* `description`: `str` с кратким описанием для тега. Можно использовать разметку, она отобразится в документации.
* `externalDocs`: `dict` описывающий внешнюю документацию с:
  * `description`: `str` с кратким описанием внешней документации.
  * `url`(обязательно): `str` с URL для внешней документации.

### Создание метаданных для тегов

Давайте попробуем это на примере с тегами для `users` и `items`.

Создайте метаданные для ваших тегов и передайте их в параметр `openapi_tags`:

```python
from fastapi import FastAPI

tags_metadata = [
    {
        "name": "users",
        "description": "Operations with users. The **login** logic is also here.",
    },
    {
        "name": "items",
        "description": "Manage items. So _fancy_ they have their own docs.",
        "externalDocs": {
            "description": "Items external docs",
            "url": "https://fastapi.tiangolo.com/",
        },
    },
]

app = FastAPI(openapi_tags=tags_metadata)
```

Обратите внимание, что вы можете использовать разметку внутри описаний, например "login" будет выделен жирным (**login**),
а "fancy" курсивом (*fancy*).

> **Совет**
> 
> Вам не обязательно добавлять метаданные для всех используемых тегов.

### Используйте ваши теги

Используйте параметр `tags` с ***операциями пути*** (и `APIRouter`), чтобы присвоить им различные теги:

```python
from fastapi import FastAPI

tags_metadata = [
    {
        "name": "users",
        "description": "Operations with users. The **login** logic is also here.",
    },
    {
        "name": "items",
        "description": "Manage items. So _fancy_ they have their own docs.",
        "externalDocs": {
            "description": "Items external docs",
            "url": "https://fastapi.tiangolo.com/",
        },
    },
]

app = FastAPI(openapi_tags=tags_metadata)


@app.get("/users/", tags=["users"])
async def get_users():
    return [{"name": "Harry"}, {"name": "Ron"}]


@app.get("/items/", tags=["items"])
async def get_items():
    return [{"name": "wand"}, {"name": "flying broom"}]
```

> **Для информации**
> 
> Прочитать больше о тегах можно в 
> [Конфигурации операции пути](https://github.com/amoglock/FastAPI_documentation/blob/master/tutorial/path_operation_configuration.md).

### Проверьте документацию

Теперь, если проверить документацию, она покажет все дополнительные метаданные:
![](https://fastapi.tiangolo.com/img/tutorial/metadata/image02.png)

### Порядок тегов

Последовательность каждого словаря метаданных тега, так же определяет порядок отображения их в интерфейсе документации.

Например, несмотря на то, что в алфавитном порядке `users` шли бы после `items`, они будут выше, потому, что мы добавили
их метаданные как первый словарь в списке.

## OpenAPI URL

По умолчанию, схема OpenAPI обслуживается на `/openapi.json`.

Но вы можете настроить это с помощью параметра `openapi_url`.

Например, установить чтобы она обслуживалась на `/api/v1/openapi.json`.

```python
from fastapi import FastAPI

app = FastAPI(openapi_url="/api/v1/openapi.json")


@app.get("/items/")
async def read_items():
    return [{"name": "Foo"}]
```

Если вы хотите полностью отключить схему OpenAPI, вы можете установить `openapi_url=None`, что также отключит интерфейс 
документации пользователя, который ее использует.

## URL документации

Вы можете настроить пользовательские интерфейсы для двух встроенных документаций.

* **Swagger UI**: обслуживается на `/docs`.
  * Вы можете установить его URL с помощью параметра `docs_url`.
  * Вы можете отключить его, установив `docs_url=None`.
* **ReDoc**: обслуживается на `/redoc`.
  * Вы можете установить его URL с помощью параметра `redoc_url`.
  * Вы можете отключить его, установив `redoc_url=None`.

Например, установить чтобы Swagger UI обслуживался на `/documentation` и отключить ReDoc:

```python
from fastapi import FastAPI

app = FastAPI(docs_url="/documentation", redoc_url=None)


@app.get("/items/")
async def read_items():
    return [{"name": "Foo"}]
```