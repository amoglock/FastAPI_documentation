<h3>Запрос файлов</h3>

Вы можете устанавливать файлы которые должны быть загружены в клиент, используя `File`.

> **Для информации**
> 
> Чтобы получить загруженные файлы, сперва установите <a href="https://andrew-d.github.io/python-multipart/">`python-multipart</a>.
> 
> Например `pip install python-multipart`.
> 
> Это потому что загруженные файлы отправляются как "данные формы".

<h4>Импорт `File`</h4>

Импортируйте `File` и `UploadFile` из `fastapi`:

```python
from typing import Annotated

from fastapi import FastAPI, File, UploadFile

app = FastAPI()
...
```

<h4>Определение параметров`File`</h4>

Создайте параметры файла таким же способом как для `Body` или `Form`:

```python
from typing import Annotated

from fastapi import FastAPI, File, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(file: Annotated[bytes, File()]):
    return {"file_size": len(file)}

@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile):
    return {"filename": file.filename}
```

> **Для информации**
> 
> `File` это класс который наследуется напрямую от `Form`.
> 
> Но помните, что когда вы импортируете `Query`, `Path`, `File` и другие из `fastapi`, на самом деле это функции которые
> возвращают специальные классы.

> **Совет**
> 
> Чтобы объявить тела Файла, вам нужно использовать `File`, потому что иначе параметры будут приняты как параметры запроса
> или параметры тела (JSON).

Файлы будут загружены как "данные формы".

Если вы объявляете тип параметра вашей функции операции пути как `bytes`, FastAPI будет читать файл, а отправите содержимое
как `bytes`.

Имейте в виду, что это означает, что все содержимое файла будет храниться в памяти. Это будет работать хорошо для
небольших файлов.

Но существуют несколько случаев в которых вы можете получить выгоду, используя `UploadFile`.

<h4>Параметры файла с `UploadFile`</h4>

Объявление параметра файла с типом `UploadFile`:

```python
from typing import Annotated

from fastapi import FastAPI, File, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(file: Annotated[bytes, File()]):
    return {"file_size": len(file)}

@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile):
    return {"filename": file.filename}
```

Использование `UploadFile` имеет некоторые преимущества над `bytes`:

* Вам не нужно использовать `File()` в параметре по умолчанию.
* Он использует "буферизированный" файл:
  * Файл хранится в памяти до лимита максимального ее размера, а после прохождения этого лимита он будет расположен на диске.
* Это значит, что он будет хорошо работать для больших файлов как картинки, видео, большие двоичные файлы, и т.д. без 
потребления всей памяти.
* Вы можете получить метаданные из загруженного файла.
* Он имеет `async` интерфейс <a href="https://docs.python.org/3/glossary.html#term-file-like-object">file-like</a>.
* Он выявляет текущий Python <a href="https://docs.python.org/3/library/tempfile.html#tempfile.SpooledTemporaryFile">`SpooledTemporaryFile`</a>
объект который вы можете передавать напрямую в другие библиотеки которые ожидают file-like объект.

<h4>`UploadFile`</h4>

`UploadFile` имеет следующие аттрибуты:
* `filename`: `str` с начальным именем файла который был загружен (например `myimage.jpg`).
* `content_type`: `str` с типом содержимого (MIME type / media type)(например `image/jpeg`).
* `file`: `SpooledTemporaryFile` (file-like объект). Это текущий Python файл, который вы можете передать напрямую в другие
функции или библиотеки, ожидающие "file-like" объект.

`UploadFile` имеет следующие `async` методы. Все они вызывают соответствующие методы файла изнутри (используя внутренний
`SpooledTemporaryFile`).
* `write(data)`: Записывает данные (`str` или `bytes`) в файл.
* `read(size)`: Читает размер (`int`) байтов/символов в файле.
* `seek(offset)`: Переходит на позицию байта `offset` (`int`) в файле.
  * Например, `await myfile.seek(0)` перешло бы к началу файла.
  * Это особенно полезно если вы запускаете `await myfile.read()` однажды, а затем нужно прочитать содержимое снова.
* `close()`: Закрывает файл.

Так как все эти методы `async`, вам нужен "await" к ним.

Например, внутри `async` функции операции пути вы можете получать содержимое:

```python
contents = await myfile.read()
```

Если вы внутри обычной `def` функции операции пути, вы можете получить доступ к `UploadFile.file` напрямую, например:

```python
contents = myfile.file.read()
```

> **`async` технические детали**
> 
> Когда вы используете `async` методы, FastAPI запускает методы файла в пуле потоков и ожидает их.

> ** Технические детали Starlette**
> 
> FastAPI `UploadFile` наследуется напрямую из Starlette `UploadFile`, но добавляет несколько необходимых частей чтобы
> сделать его совместимым с Pydantic и другими частями FastAPI.

<h3>Что такое "Данные формы"</h3>

Способ HTML форм (<form></form>) отправляет данные на сервер обычно используя "специальную" кодировку для этих данных, она
отличается от JSON.

FastAPI будет позволять читать эти данные из правильного места, вместо JSON.

> **Технические детали**
> 
> Данные из форм обычно закодированы с использованием "media type" `application/x-www-form-urlencoded` когда в них не 
> включены файлы.
> 
> Но когда в форме есть файлы, она закодирована как `multipart/form-data`. Если вы используете `File`, FastAPI будет знать,
> что ему нужно получить файлы из корректного части тела.
> 
> Если вы хотите прочитать больше об этих кодировках и полях формы, пройдите по 
> <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST">MDN web docs for `POST`</a>.

> **Предупреждение!!!**
> 
> Вы можете объявлять несколько параметров `File` и `Form` в операции пути, но вы не можете также объявлять поля `Body`,
> что ожидает получить JSON, так у запроса будет тело, закодированное используя `multipart/form-data` вместо
> `application/json`.
> 
> Это не ограничение FastAPI, это часть протокола HTTP.

<h4>Необязательная загрузка файла</h4>

Вы можете делать файл необязательным, используя стандартную аннотацию типа и установив значение по умолчанию `None`:

```python
from typing import Annotated

from fastapi import FastAPI, File, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(file: Annotated[bytes | None, File()] = None):
    if not file:
        return {"message": "no file"}
    else:
        return {"file_size": len(file)}

@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile | None = None):
    if not file:
        return {"message": "No upload file"}
    else:
        return {"filename": file.filename}
```

<h4>`UploadFile` с дополнительными Metadata</h4>

Еще вы можете использовать `File()` с `UploadFile`, например, чтобы установить дополнительные метаданные:

```python
from typing import Annotated

from fastapi import FastAPI, File, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(file: Annotated[bytes, File(description="A file read as bytes")]):
    return {"file_size": len(file)}

@app.post("/uploadfile/")
async def create_upload_file(
        file: Annotated[UploadFile, File(description="A file read as UploadFile")],
):
    return {"filename": file.filename}
```

<h4>Несколько загрузок файла</h4>

Есть возможность загрузить несколько файлов за раз.

Они были бы связаны с одним и тем же "полем формы", отправленным с использованием "данных формы".

Чтобы это использовать, объявите список `bytes` или `UploadFile`:

```python
from typing import Annotated

from fastapi import FastAPI, File, UploadFile
from fastapi.responses import HTMLResponse

app = FastAPI()

@app.post("/files/")
async def create_files(files: Annotated[list[bytes], File()]):
    return {"file_sizes": [len(file) for file in files]}

@app.post("/uploadfiles/")
async def create_upload_files(files: list[UploadFile]):
    return {"filenames": [file.filename for file in files]}

@app.get("/")
async def main():
    content = """
    <body>
    <form action="/files/" enctype="multipart/form-data" method="post">
    <input name="files" type="file" multiple>
    <input type="submit">
    </form>
    <form action="/uploadfiles/" enctype="multipart/form-data" method="post">
    <input name="files" type="file" multiple>
    <input type="submit">
    </form>
    </body>
    """
    return HTMLResponse(content=content)
```

Вы получите, как заявлено, `list` из `bytes` или `UploadFile`.

> **Технические детали**
> 
> Вы можете также использовать `from starlette.responses import HTMLResponse`.
> 
> FastAPI предоставляет те же `starlette.responses` как `fastapi.responses`, просто удобнее для вас, разработчика. Но
> большинство доступных ответов приходят напрямую из Starlette.

<h4>Несколько загрузок файла с дополнительными Metadata</h4>

И точно так же как и ранее, вы можете использовать `File()` чтобы устанавливать дополнительные параметры, даже для
`UploadFile`:

```python
from typing import Annotated

from fastapi import FastAPI, File, UploadFile
from fastapi.responses import HTMLResponse

app = FastAPI()

@app.post("/files/")
async def create_files(
        files: Annotated[list[bytes], File(description="Multiple files as bytes")],
):
    return {"file_sizes": [len(file) for file in files]}

@app.post("/uploadfiles/")
async def create_upload_files(
        files: Annotated[
          list[UploadFile], File(description="Multiple files as UploadFile")
        ],
):
    return {"filenames": [file.filename for file in files]}

@app.get("/")
async def main():
    content = """
    <body>
    <form action="/files/" enctype="multipart/form-data" method="post">
    <input name="files" type="file" multiple>
    <input type="submit">
    </form>
    <form action="/uploadfiles/" enctype="multipart/form-data" method="post">
    <input name="files" type="file" multiple>
    <input type="submit">
    </form>
    </body>
    """
    return HTMLResponse(content=content)
```

<h4>Резюме</h4>

Используйте `File`, `bytes` и `UploadFile` чтобы объявить файлы, которые будут загружены в запрос, отправленные как
форма данных.