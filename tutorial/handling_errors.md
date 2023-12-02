<h3>Обработка ошибок</h3>

Существует множество ситуаций где вам нужно сообщить об ошибке клиенту, который использует ваш API.

Этим клиентом мог бы быть браузер, код от кого-то еще, какой-то девайс и т.п.

Вам может понадобиться сообщить клиенту, что:
* Клиент не имеет достаточных прав для операции.
* Клиент не имеет доступа к ресурсу.
* Элемент, к которому клиент попытался получить доступ, не существует.
* и т.д.

В этих случаях, вы бы, обычно, возвращали HTTP статус-код в диапазоне 400 (от 400 до 499).

Они похожи на 200 статус-коды HTTP(от 200 до 299). Эти статус-коды "200" значат, что в запросе каким-то образом все
прошло успешно.

Статус-коды в 400 диапазоне означают, что в клиенте произошла ошибка.

Помните все эти "404 Not Found" ошибки (и шутки)?

<h4>Использование `HTTPException`</h4>

Чтобы вернуть HTTP ответы с ошибками в клиент, вы используете `HTTPExceptions`.

<h5>Импорт `HTTPException`</h5>

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()
...
```

<h5>Поднятие `HTTPException` в вашем коде</h5>

`HTTPException` это обычное исключение Python с дополнительными данными, относящихся к API.

Так как это исключение Python, вы не делаете его `return`, вы "поднимаете" его (`raise`).

Это также означает, что если вы внутри служебной функции, которую вы вызываете внутри вашей функции операции пути и 
поднимаете `HTTPException` изнутри этой служебной функции, она не будет запускать остаток кода в функции операции пути. 
Она немедленно прекратит запрос и отправит ошибку HTTP из `HTTPException` в клиент.

Преимущество поднятия исключения над возвратом (`return`) значения будет более очевидным в разделе о зависимостях и
безопасности.

В этом примере, когда клиент запрашивает элемент по ID который не существует, вызывается исключение со статус-кодом `404`:

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}

@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}
```

<h4>Результирующий ответ</h4>

Если клиент запрашивает `http://example.com/items/foo` (`item_id` `"foo"`), то клиент получит статус-код 200 и JSON ответ:

```JSON
{
    "item": "The Foo Wrestlers"
}
```

Но если клиент запрашивает `http://example.com/items/bar` (несуществующий `item_id` `"bar"`), то клиент получит статус-код
404 (ошибка "not-found") и JSON ответ:

```JSON
{
  "detail": "Item not found"
}
```

> **Совет**
> 
> Когда вызывается `HTTPException`, вы можете передать любое значение которое может быть конвертировано в JSON как параметр
> `detail`, а не только `str`.
> 
> Вы можете передать `dict`, `list` и т.д.
> 
> Они автоматически обрабатываются FastAPI и конвертируются в JSON.

<h4>Добавление собственных заголовков</h4>

Есть такие ситуации, в которых полезно иметь возможность добавить собственные заголовки в ошибку HTTP. Например, для
некоторых видов безопасности.

Вам, вероятно, не понадобится использовать их напрямую в своем коде.

Но в случае если вам понадобилось это для продвинутого сценария, вы можете добавлять собственные заголовки:

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}

@app.get("/items-header/{item_id}")
async def read_item_header(item_id: str):
    if item_id not in items:
        raise HTTPException(
            status_code=404,
            detail="Item not found",
            headers={"X-Error": "There goes my error"},
        )
    return {"item": items[item_id]}
```

<h4>Установка собственных обработчиков ошибок</h4>

Вы можете добавлять собственные обработчики ошибок с помощью 
<a href="https://www.starlette.io/exceptions/">тех же утилит исключения из Starlette</a>.

Скажем, у вас есть собственное исключение `UnicornException`, которую вы (или используемая вами библиотека) можете поднимать.

И вы хотите обработать это исключение глобально в FastAPI.

Вы можете добавить обработчик собственного исключения с помощью `@app.exception_handler()`:

```python
from fastapi import FastAPI, Request
from fasapi.responses import JSONResponse

class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name

app = FastAPI()

@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."}    
    )

@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}
```

Здесь, если вы запрашиваете `/unicorns/yolo`, операция пути вызовет исключение `UnicornException`.

А обработано оно будет с помощью `unicorn_exception_handler`.

Поэтому, вы получите понятную ошибку, со статус-кодом `418` и содержимым JSON:

```JSON
{"message": "Oops! yolo did something. There goes a rainbow..."}
```

> **Технические детали**
> 
> Вы можете, также, использовать `from starlette.requests import Request` и `from starlette.responses import JSONResponse`.
> 
> FastAPI предоставляет одни и те же `starlette.responses` и `fastapi.responses` просто удобные для вас, разработчика.
> Но большинство доступных ответов приходят напрямую из Starlette. То же самое и для `Request`.

<h4>Переопределение дефолтных обработчиков ошибок</h4>

FastAPI имеет несколько дефолтных обработчиков ошибок.

Эти обработчики отвечают за возврат дефолтных ответов JSON когда вы вызываете `HTTPException`, и когда запрос имеет
неподходящие данные.

Вы можете переопределять эти обработчики по своему усмотрению.

<h5>Переопределение исключений проверки запроса</h5>

Когда запрос содержит неподходящие данные, FastAPI внутри себя вызывает `RequestValidationError`.

А он, также, включает в себя обработчик исключения по умолчанию.

Чтобы переопределить его, импортируйте `RequestValidationError` и используйте его вместе с 
`@app.exception_handler(RequestValidationError)` чтобы декорировать обработчик.

Этот обработчик будет получать `Request` и исключение.

```python
from fastapi import FastAPI, HTTPException
from fastapi.exceptions import RequestValidationError
from fastapi.responses import PlainTextResponse
from starlette.exceptions import HTTPExcepton as StarletteHTTPException

app = FastAPI()

@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don`t like 3.")
    return {"item_id": item_id}
```

Теперь, если вы перейдете по `/items/foo`, вместо получения JSON по умолчанию с содержанием:

```JSON 
{
  "detail": [
    {
      "loc": [
        "path",
        "item_id"
      ],
      "msg": "value is not a valid integer",
      "type": "type_error.integer"
    }
  ]
}
```

Вы получите текстовую версию с содержанием:

```text
1 validation error
path -> item_id
    value is not a valid integer
```

`RequestValidationError` против `ValidationError`

> **Внимание!!!**
> 
> Это технические детали которые вы можете пропустить, если они для вас не важны.

`RequestValidationError` это подкласс 
<a href="https://pydantic-docs.helpmanual.io/usage/models/#error-handling">`ValidationError`</a> Pydantic.

FastAPI использует его чтобы, если вы используете модель Pydantic в `response_model`, и в ваших данных есть ошибка, вы 
увидели ошибку в вашем логе.

Но клиент/пользователь не увидит ее. Вместо этого, клиент увидит "Internal Server Error" со статус-кодом `500`.

Это должно быть так, потому, что если в вашем ответе или где-то еще в коде(а не в запросе клиента), у вас есть 
`ValidationError` Pydantic, это будет ошибка самого кода.

И пока вы не исправите ее, ваши клиенты/пользователи не должны иметь доступ к внутренней информации об ошибке, так как
это может выявить уязвимость в безопасности.

<h4>переназначение `HTTPException` обработчика</h4>

Точно так же, вы можете переопределить обработчик `HTTPException`.

Например, вы можете захотеть вернуть обычный текстовый ответ вместо JSON для таких ошибок:

```python
from fastapi import FastAPI, HTTPException
from fastapi.exceptions import RequestValidationError
from fastapi.responses import PlainTextResponse
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()

@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don`t like 3.")
    return {"item_id": item_id}
```

> **Технические детали**
> 
> Вы можете использовать `from starlette.responses import PlainTextResponse`.
> 
> FastAPI предоставляет те же`starlette.responses` как `fastapi.responses` просто удобнее для вас, разработчика. Но 
> большинство доступных ответов приходят напрямую из Starlette.
