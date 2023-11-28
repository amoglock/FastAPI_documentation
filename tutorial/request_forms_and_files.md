<h3>Запрос форм и файлов</h3>

Вы можете объявлять файлы и поля форм в одно времяЮ используя `File` и `Form`.

> **Для информации**
> 
> Чтобы получать загруженные файлы и/или данные форм, сперва установите 
> <a href="https://andrew-d.github.io/python-multipart/">`python-multipart`</a>.
> 
> Например `pip install python-multipart`

<h4>Импорт `File` и `Form`</h4>

```python
from typing import Annotated

from fastapi import FastAPI, File, Form, UploadFile

app = FastAPI()
...
```

<h4>Объявление параметров `File` и `Form`</h4>

Создание параметров файла и формы такое же как для `Body` и `Query`:

```python
from typing import Annotated

from fastapi import FastAPI, File, Form, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(
        file: Annotated[bytes, File()],
        fileb: Annotated[UploadFile, File()],
        token: Annotated[str, Form()],
):
    return {
        "file_size": len(file),
        "token": token,
        "fileb_content_type": fileb.content_type,
    }
```

Файлы и поля формы будут загружены как данные формы и вы получите файлы и поля формы.

И вы можете объявлять несколько файлов как `bytes`, а также как `UploadFile`.

> **Предупреждение!!!**
> 
> Вы можете объявлять несколько параметров `File` и `Form` в операции пути, но вы не можете еще объявлять поля `Body`
> которые ожидают получить JSON, так как запрос будет иметь тело закодированное используя `multipat/form-data` вместо
> `application/json`.
> 
> Это не ограничение FastAPI, это часть протокола HTTP.

<h4>Резюме</h4>

Используйте `File` и `Form` вместе когда вам нужно получить данные и файлы в одном запросе.