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

Но существуют несколько случаев в которых вы можете получить выгоду используя `UploadFile`.

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

