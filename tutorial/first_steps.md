<h3>Первые шаги</h3>

Простейший файл FastAPI может выглядеть так:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```
Скопируйте этот код в свой `main.py`.

Запустите сервер:

```commandline
uvicorn main:app --reload
```

Ответ успешно запущенного сервера:

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [28720]
INFO:     Started server process [28722]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

>**Обратите внимание**<br>
> Команда `uvicorn main:app` содержит:
> * `main`: файл `main.py` ("модуль" Python)
> * `app`: объект `app = FastAPI()`, созданный в `main.py`
> * `--reload`: сервер автоматически перезагружается после изменений в коде. Используется только при разработке.

В ответе сервера есть, примерно, такая строка:

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

Эта строка показывает URL-адрес, где запустилось приложение на вашей локальной машине.

<h3>Проверка</h3>
Откройте свой браузер по адресу http://127.0.0.1:8000.\
Вы увидите такой JSON-ответ:

```
{"message": "Hello World"}
```

<h3>Интерактивная документация API</h3>
Теперь перейдите по адресу http://127.0.0.1:8000/docs. <br>
Вы увидите автоматическую документацию вашего API (предоставленную [Swagger](https://github.com/swagger-api/swagger-ui/))
<img src="https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png">

<h3>Альтернативная документация API</h3>
А теперь, перейдем на http://127.0.0.1:8000/redoc. <br>
Вы увидите альтернативную автоматическую документацию (предоставленную [ReDoc](https://github.com/Rebilly/ReDoc/))
<img src="https://fastapi.tiangolo.com/img/index/index-02-redoc-simple.png">

[ss](https://google.com)