<h3>Form данные</h3>

Когда вам нужно отправить поля формы вместо JSON, вы можете использовать `Form`.

> **Для информации**
> 
> Чтобы использовать формы, сперва установите <a href="https://andrew-d.github.io/python-multipart/">python-multipart</a>.
> 
> Например, так `pip install python-multipart`.

<h4>Импорт `Form`</h4>

Импортируем `Form` из `fastapi`:

```python
from typing import Annotated

from fastapi import FastAPI, Form

app = FastAPI()
...
```

<h4>Объявление параметров`Form`</h4>

Создайте параметры формы таким же способом как для `Body` или `Query`:

```python
from typing import Annotated

from fastapi import FastAPI, Form

app = FastAPI()

@app.post("/login")
async def login(username: Annotated[str, Form()], password: Annotated[str, Form()]):
    return {"username": username}
```

Например, в одном из способов спецификации OAuth2 (называемом "password flow") требуется отправить `username` и 
`password` как поля формы.

Эта спецификация требует чтобы поля были явно названы `username` и `password`, и были отправлены как поля формы, а не JSON.

С помощью `Form` вы можете объявлять такие же конфигурации как в `Body`(`Query`, `Path`, `Cookie`), включающие валидацию,
примеры, псевдонимы (например `user-name` вместо `username`), и т.д.

> **Для информации**
> 
> `Form` это класс который наследуется напрямую от `Body`.

> **Совет**
> 
> Чтобы объявить форму тел, вам нужно явно использовать `Form`, потому что без нее, параметры будут интерпретированы как 
> параметры запроса или параметры тела (JSON).

