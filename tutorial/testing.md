# Тестирование

Благодаря [Starlette](https://www.starlette.io/testclient/), тестирование приложений FastAPI легкое и приятное.

Оно основано на [HTTPX](https://www.python-httpx.org/), который, в свою очередь, разработан на Requests, поэтому они 
очень похожи и интуитивны.

С ним вы можете использовать [pytest](https://docs.pytest.org/) непосредственно в FastAPI.

## Использование `TestClient`

> **Для информации**
> 
> Чтобы использовать `TestClient`, сначала установите [`httpx`](https://www.python-httpx.org/).
> 
> Например, `pip install httpx`.

Импортируйте `TestClient`.

Создайте `TestClient`, передав ему ваше приложение FastAPI.

Создайте функции с именами, которые начинаются на `test_` (это стандарт соглашения `pytest`).

Используйте объект `TestClient` так же как вы это делаете с `httpx`.

Напишите простые `assert` выражения со стандартными выражениями Python, которые вам нужно проверить (снова, стандартный 
`pytest`).

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}


client = TestClient(app)


def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```

> **Совет**
> 
> Обратите внимание, что тестируемые функции обычные `def`, не `async def`.
> 
> И вызовы в клиент это также обычные вызовы, не использующие `await`.
> 
> Это позволяет вам использовать `pytest` напрямую без сложностей.

> **Технические детали**
> 
> Еще вы можете использовать `from starlette.testclient import TestClient`.
> 
> FastAPI предоставляет тот же `starlette.testclient` как `fastapi.testclient`, просто это удобнее для вас, разработчика.
> Но на самом деле он идет напрямую из Starlette.

> **Совет**
> 
> Если вы хотите вызвать `async` функции в ваших тестах отдельно от отправки запросов к вашему приложению FastAPI (например,
> ассинхронные функции базы данных), посмотрите тему про ассинхронные тесты в продвинутом руководстве.

## Отделение тестов

В настоящем приложении вам, вероятно, понадобилось бы иметь ваши тесты в отдельном файле.

А ваше приложение FastAPI может состоять из нескольких файлов/модулей, и т.д.

### Файл приложения FastAPI

Допустим у вас есть структура файлов как была описана в 
[Больших приложениях](https://github.com/amoglock/FastAPI_documentation/blob/master/tutorial/bigger_applications.md):

```text
.
├── app
│   ├── __init__.py
│   └── main.py
```

В файле `main.py` у вас приложение FastAPI:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}
```

### Тестирование файла

Затем у вас может быть файл `test_main.py` с вашими тестами. Он может жить в том же пакете Python (та же директория с 
файлом `__init__.py`):

```text
.
├── app
│   ├── __init__.py
│   ├── main.py
│   └── test_main.py
```

Так как этот файл в том же пакете, вы можете использовать относительные импорты, чтобы импортировать объект `app` из 
модуля `main` (`main.py`):

```python
from fastapi.testclient import TestClient

from .main import app

client = TestClient(app)


def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```

А код для тестов тот же как и до этого.

## Тестирование: расширенный пример

Теперь, давайте расширим пример и добавим больше деталей, чтобы посмотреть как тестировать различные части.

### Расширенный файл приложения FastAPI

Давайте продолжим с той же файловой структурой какая была:

```text
.
├── app
│   ├── __init__.py
│   ├── main.py
│   └── test_main.py
```

Скажем, что теперь файл `main.py` с вашим приложением FastAPI имеет несколько других ***операций пути***.

У него есть операция `GET`, которая может возвращать ошибку.

У него есть операция `POST`, которая может возвращать несколько ошибок.

Обе ***операции пути*** требуют заголовок `X-Token`:

```text
from typing import Annotated

from fastapi import FastAPI, Header, HTTPException
from pydantic import BaseModel

fake_secret_token = "coneofsilence"

fake_db = {
    "foo": {"id": "foo", "title": "Foo", "description": "There goes my hero"},
    "bar": {"id": "bar", "title": "Bar", "description": "The bartenders"},
}

app = FastAPI()


class Item(BaseModel):
    id: str
    title: str
    description: str | None = None


@app.get("/items/{item_id}", response_model=Item)
async def read_main(item_id: str, x_token: Annotated[str, Header()]):
    if x_token != fake_secret_token:
        raise HTTPException(status_code=400, detail="Invalid X-Token header")
    if item_id not in fake_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return fake_db[item_id]


@app.post("/items/", response_model=Item)
async def create_item(item: Item, x_token: Annotated[str, Header()]):
    if x_token != fake_secret_token:
        raise HTTPException(status_code=400, detail="Invalid X-Token header")
    if item.id in fake_db:
        raise HTTPException(status_code=400, detail="Item already exists")
    fake_db[item.id] = item
    return item
```

