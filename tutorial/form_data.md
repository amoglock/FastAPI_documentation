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

<h4>О "Полях форм"</h4>

Метод форм HTML (<form></form>) отправляет на сервер обычно используя "специальное" кодирование для данных, оно 
отличается от JSON. 

FastAPI проследит, чтобы прочитать эти данные из правильного места, вместо JSON.

> **Технические детали**
> 
> Данные из форм обычно закодированы с использованием "media type" `application/x-www-form-urlencoded`.
> 
> Но когда формы включают в себя файлы, они закодированы как `multipart/form-data`. Вы прочитаете об обработке файлов в
> следующей главе.
> 
> Если вы хотите прочитать больше об таком кодировании и полях формы, направляйтесь в 
> <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST>">MDN web docs for `POST`</a>.

> **Предупреждение!!!**
> 
> Вы можете объявлять несколько параметров `Form` в операции пути, но вы не можете также определить поля `Body` которую
> вы ожидаете получить как JSON, так как этот запрос будет закодирован используя `application/x-form-urlencoded` вместо
> `application-json`.
> 
> Это не ограничение FastAPI, это часть протокола HTTP.

<h4>Резюме</h4>

Используйте `Form` чтобы объявить форму данных во входных параметрах.