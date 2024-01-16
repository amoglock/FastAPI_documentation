# Фоновые задачи

Вы можете определять фоновые задачи, которые будут запущены ***после*** возвращения ответа.

Это полезно для операций, которым нужно происходить после запроса, но клиенту не нужно дожидаться их завершения перед
приемом ответа.

Это включает в себя, например:

* Отправка уведомлений по электронной почте после выполнения действия:
  * Так как соединение с сервером почты и отправка письма имеет тенденцию быть "медленной" (несколько секунд), вы можете
  вернуть ответ сразу и отправить уведомление в фоне.
* Обработка данных:
  * Например, скажем, вы получаете файл, который должен пройти через медленный процесс, вы можете вернуть ответ 
  "Accepted" (HTTP 202) и обработать его в фоне.

## Использование `BackgroundTasks`

Первое, импортируйте `BackgroundTasks` и определите параметр в вашей ***функции операции пути*** с типом объявления
`BackgroundTasks`:

```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()

@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    ...
```

FastAPI создаст объект типа `BackgroundTasks` за вас и передаст его как этот параметр.

## Создайте функцию задачи

Создайте функцию, чтобы запустить ее как фоновую задачу.

Это просто стандартная функция, которая может принимать параметры.

Это может `async def` или обычная `def` функция, FastAPI будет знать как корректно ее обработать.

В нашем случае, функция задачи пишет в файл (симуляция отправки письма).

И так-как эта операция не использует `async` и `await`, мы определяем функцию с обычным `def`:

```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()

def write_notification(email: str, message=""):
  with open("log.txt", mode="w") as email_file:
    content = f"notification for {email}: {message}"
    email_file.write(content)  

@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    ...
```

## Используйте фоновую задачу

Внутри вашей ***функции операции пути*** передайте вашу функцию задачи в объект ***фоновых задач*** с помощью метода 
`.add_task()`:

```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()

def write_notification(email: str, message=""):
  with open("log.txt", mode="w") as email_file:
    content = f"notification for {email}: {message}"
    email_file.write(content)  

@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_notification, email, message="some notification")
    return {"message": "Notification sent in the background"}
```

`.add_task()` принимает в качестве аргументов:
  * Функцию задачи, которую нужно запустить в фоне (`write_notification`).
  * Любую последовательность аргументов, которые должны быть переданы по порядку в функцию задачи (`email`).
  * Любые ключевые аргументы, которые должны быть переданы в функцию задачи (`message="some notification`).

## Внедрение зависимости

Использование `BackgroundTasks` также работает с системой внедрения зависимости, вы можете объявлять параметр типа 
`BackgroundTasks` на разных уровнях: в ***функции операции пути***, в зависимости (зависимом), в подзависимости, и т.д.

FastAPI знает, что делать в каждом случае и как переиспользовать один и тот же объект, потому что все фоновые задачи 
объединяются вместе и затем запускаются в фоне.

```python
from typing import Annotated

from fastapi import BackgroundTasks, Depends, FastAPI

app = FastAPI()

def write_log(message: str):
    with open("log.txt", mode="a") as log:
        log.write(message)

def get_query(background_tasks: BackgroundTasks, q: str | None = None):
    if q:
        message = f"found query: {q}\n"
        background_tasks.add_task(write_log, message)
    return q

@app.post("/send-notification/{email}")
async def send_notification(
        email: str, background_task: BackgroundTasks, q: Annotated[str, Depends(get_query)]
):
    message = f"message to {email}\n"
    background_task.add_task(write_log, message)
    return {"message": "message sent"}
```

В этом примере, сообщения будут записаны в файл `log.txt` после того как ответ оправлен.

Если в запросе был `q`, он будет записан в лог в фоновой задаче.

А затем, другая фоновая задача, сгенерированная в ***функции операции пути*** напишет сообщение, используя параметр
пути `email`.

## Технические детали

Класс `BackgroundTasks` идет напрямую из <a href="https://www.starlette.io/background/">`starlette.background`</a>.

Он импортирован/включен напрямую в FastAPI, поэтому вы можете импортировать его из `fastapi` и избежать случайного импорта
альтернативного варианта `BackgroundTask` (без `s` на конце) из `starlette.background`.

Но, только используя `BackgroundTasks` (а не `BackgroundTask`), возможно использовать его как параметр ***функции операции
пути***, а FastAPI может обработать остальное за вас, просто, словно напрямую используем объект `Request`.

Но в FastAPI все еще возможно использовать отдельно и `BackgroundTask`, но вам нужно создать объект в вашем коде и вернуть
Starlette `Response`, включающий его.

Вы можете посмотреть более подробно в <a href="https://www.starlette.io/background/">официальной документации Starlette
для фоновых задач</a>.

## Предупреждение

Если вам нужно обеспечить сложное фоновое вычисление и вам необязательно нужно, чтобы оно было запущено в том же процессе
(например, вам не нужно делиться памятью, переменными, и т.п.), вы может быть выгоднее использовать другие более 
масштабные инструменты, например <a href="https://docs.celeryq.dev/">Celery</a>.

Они, как правило, требуют более сложной конфигурации, менеджер очереди сообщений/работы, вроде RabbitMQ или Redis, но они
позволяют вам запускать фоновые задачи в разных процессах, и особенно, на разных серверах.

Чтобы увидеть примеры, посмотрите <a href="https://fastapi.tiangolo.com/project-generation/">Project Generators</a>, все 
они включают в себя настроенный Celery.

Но если вам нужен доступ к переменным и объектам из того же приложения FastAPI, или вам нужно обеспечить небольшие 
фоновые задачи (вроде отправки уведомления на почту), вы можете просто использовать `BackgroundTasks`.

## Резюме

Импортируйте и используйте `BackgroundTasks` с параметрами в ***функциях операции пути*** и зависимостями, чтобы добавить
фоновые задачи.