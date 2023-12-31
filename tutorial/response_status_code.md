<h3>Статус-код ответа</h3>

Точно также как вы можете устанавливать модель ответа, можно еще определять HTTP статус-код, использующийся для ответа 
с параметром `status_code`, в любых операциях пути:

* `@app.get()`
* `@app.post()`
* `@app.put()`
* `@app.delete()`
* и т.д.

```python
from fastapi import FastAPI

app = FastAPI()

@app.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}
```

> **Заметка**
> 
> Обратите внимание, что `status_code` это параметр метода "декоратора" (`get`, `post`, и т.д.). А не вашей функции операции
> пути, как все параметры и тело.

Параметр `status_code` принимает число со статус-кодом HTTP.

> **Для информации**
> 
> `status_code` в качестве альтернативы может принимать `IntEnum`, такие, как <a href="https://docs.python.org/3/library/http.html#http.HTTPStatus">`http.HTTPStatus`</a>
> Python.

Он будет:

* Возвращать этот статус-код в ответе.
* Документировать его в схеме OpenAPI (и отсюда, в пользовательских интерфейсах):

<img src="https://fastapi.tiangolo.com/img/tutorial/response-status-code/image01.png" width="420" height="425">

> **Заметка**
> 
> Некоторые коды ответа (смотри следующий раздел) показывают, что ответ не имеет тела.
> 
> FastAPI знает это и будет создавать документацию OpenAPI, которая будет указывать, что тела ответа нет.

<h4>О статус-кодах HTTP</h4>

> **Заметка**
> 
> Если вы уже знаете что такое статус-код HTTP, переходите к следующей секции.

В HTTP вы отправляете числовой статус-код из трех цифр как часть ответа.

Эти статус-коды имеют названия, связанные так, чтобы их распознать, но важная часть это само число.

Вкратце:

* `100` и выше это для "информации". Вы редко используете их напрямую. Ответы с этими статус-кодами не могут иметь тело.
* `200` и выше это для "успешных" ответов. Это один наиболее частых кодов которые вы будете использовать.
  * `200` это статус-код по умолчанию, который означает, что все было "ОК".
  * Другим примером может быть `201`, "Created". Он наиболее часто используется после создания новой записи в базе данных.
  * Специальный случай это `204`, "No Content". Этот ответ используется когда нет содержимого, чтобы вернуть клиенту и
  поэтому ответ не может иметь тело.
* `300` и выше это для "Переадресации (Redirection)". Ответы с таким статус-кодом могут или не могут иметь тело, кроме
`304` "Not Modified", у которого не должно быть ничего.
* `400` и выше это ответы "Client error". Это второй тип который вы вероятно будете использовать наиболее часто.
  * Например `404`, для ответа "Not Found".
  * Для универсальных ошибок от клиента вы можете просто использовать `400`.
* `500` и выше это для ошибок сервера. Вы практически никогда не используете их напрямую. Когда что-то идет не так в 
какой-то части кода вашего приложения или сервера, автоматически будет возвращаться один из этих статус-кодов.

> **Подсказка**
> 
> Узнать больше о каждом статус-коде и какой код для чего можно здесь 
> <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Status">MDN documentation about HTTP status codes</a>.

<h4>Сокращения, чтобы запомнить названия</h4>

Давайте снова рассмотрим предыдущий пример:

```python
from fastapi import FastAPI

app = FastAPI()

@app.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}
```

`201` это статус-код для "Created".

Но вам не обязательно нужно помнить, что означает каждый из этих кодов.

Вы можете использовать удобные переменные из `fastapi.status`.

```python
from fastapi import FastAPI, status

app = FastAPI()

@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_items(name: str):
    return {"name": name}
```

Это просто удобно, они содержат то же самое число, но таким способом вы можете использовать автозавершение редактора
чтобы их найти:

<img src="https://fastapi.tiangolo.com/img/tutorial/response-status-code/image02.png" width="420" height="115">

> **Технические детали**
> 
> Вы можете еще использовать `from starlette import status`
> 
> FastAPI предоставляет тот же `starlette.status` как `fastapi.status` просто удобнее для вас, разработчика. Но это 
> он идет напрямую из Starlette.

<h4>Изменение значения по умолчанию</h4>

Позже, в продвинутом руководстве, вы увидите как возвращать статус-код отличающийся от установленного вами по умолчанию.