<h3>JSON-совместимый кодировщик</h3>

Есть несколько ситуаций где вам может понабиться конвертировать тип данных (например модель Pydantic) во что-то 
совместимое с JSON (например `dict`, `list` и т.д.).

Например, если вам нужно хранить его в базе данных.

Для этого FastAPI предоставляет функцию `jsonable_encoder()`.

<h4>Использование `jsonable_encoder`</h4>

Давайте представим, что у вас есть база данных `fake_db` которая принимает только данные совместимые с JSON.

Например, она не принимает объекты `datetime`, так как они не совместимы с JSON.

Поэтому объекты `datetime` должны быть конвертированы в `str`, содержащий данные в 
<a href="https://en.wikipedia.org/wiki/ISO_8601">ISO формате</a>.

Точно так же, эта база данных не принимала бы модель Pydantic (объект с аттрибутами), только `dict`.

Для этого вы и можете использовать `jsonable_encoder`.

Она принимает объект, например модель Pydantic, и возвращает версию совместимую с JSON:

```python
from datetime import datetime

from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

fake_db = {}

class Item(BaseModel):
    title: str
    timestamp: datetime
    description: str | None = None

app = FastAPI()

@app.put("/items/{id}")
def update_item(id: str, item: Item):
    json_compatible_item_data = jsonable_encoder(item)
    fake_db[id] = json_compatible_item_data
```

В этом примере, она конвертировала модель Pydantic в `dict`, а `datetime` в `str`.

Результат вызова это что-то, что может быть закодировано со стандартным
<a href="https://docs.python.org/3/library/json.html#json.dumps">`json.dumps()`</a> Python.

Она не возвращает большой `str`, содержащий данные в формате JSON (как строка). Она возвращает стандартную структуру данных
Python (например, `dict`) со значениями и подзначениями, которые полностью совместимы с JSON.

> **Обратите внимание**
> 
> `jsonable_encoder` на самом деле используется внутри FastAPI для конвертации данных. Но она полезна во множестве других
> сценариев.