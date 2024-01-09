# Middleware

Вы можете добавлять middleware к вашим приложениям FastAPI. 

"Middleware" это функция, которая работает с каждым **запросом** до того как он обработан какой-либо определенной 
***операцией пути***. А также с каждым **ответом** перед его возвратом.

* Она принимает каждый **запрос**, который приходит в ваше приложение.
* Затем, она может что-то сделать с этим **запросом** или запустить любой ***нужный код***.
* Затем, она передает **запрос** на оставшуюся обработку приложением (в какую-то ***операцию пути***).
* Затем, она принимает **ответ**, созданный приложением (какой-то ***операцией пути***).
* Она может что-то сделать с этим **ответом** или запустить любой ***нужный код***.
* Затем она возвращает **ответ**.

> **Технические детали**
> 
> Если у вас есть зависимости с `yield`, код выхода запустится ***после*** middleware.
> 
> Если там были любые фоновые задачи (описанные позже), они запустятся ***после*** всех middleware.

## Создание middleware

Чтобы создать middleware, вы используете декоратор `@app.middleware("http")` сверху функции.

Функция middleware принимает:

* `request`
* Функцию `call_next`, которая примет `request` как параметр.
  * Эта функция передаст `request` в соответствующую ***операцию пути***.
  * Затем, она возвращает `response` созданный соответствующей ***операцией пути***.
* Затем, в дальнейшем вы можете изменять `response` перед его возвратом.

```python
import time

from fastapi import FastAPI, Request

app = FastAPI()


@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

> **Совет**
> 
> Имейте в виду, что самодельные заголовки могут быть добавлены, 
> <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers">используя 'X-' префикс</a>.
> 
> Но если у вас заголовки которые вы хотите, чтобы видел клиент в браузере, вам нужно добавлять их в ваши конфигурации
> CORS (<a href="https://fastapi.tiangolo.com/tutorial/cors/">CORS(Cross-Origin Resource Sharing)</a>), используя параметр
> `expose_headers`, описанный в <a href="https://www.starlette.io/middleware/#corsmiddleware">документации Starlette CORS</a>. 

> **Технические детали**
> 
> Вы можете так же использовать `from starlette.requests import Request`.
> 
> FastAPI предоставляет его удобнее для вас, разработчика. Но он приходит напрямую из Starlette.

## До и после `response`

Вы можете добавить код, чтобы запустить его с `request`, до любой ***операции пути*** принимающей его.

А так же после полученного `response`, перед его возвратом.

Например, вы можете добавлять кастомный заголовок `X-Process-Time`, содержащий время в секундах, которое потребовалось
на обработку запроса и создание ответа:

```python
import time

from fastapi import FastAPI, Request

app = FastAPI()


@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

## Другие middleware

Позже вы можете прочитать больше про другие middleware в продвинутом руководстве.

Вы прочтете о том как обрабатывать CORS с middleware в следующем разделе.