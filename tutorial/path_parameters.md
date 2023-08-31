<h3>Параметры пути</h3>

Вы можете объявить "параметры" или "переменные" пути таким же синтаксисом, который используется в Python
для строк:
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```
Значение параметра пути `item_id` будет передано в функцию как аргумент `item_id`.

Поэтому, если запустить пример и перейти по http://127.0.0.1:8000/items/foo, вы увидите следующее:
```python
{"item_id": "foo"}
```

<h4>Параметры пути с типами</h4>
Вы можете объявить тип параметра пути в функции, используя стандартную аннотацию типов Python:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

В нашем случае, `item_id` объявлена как `int`.

```
Такое объявление дает возможность поддержки редактора кода в функции, с проверкой ошибок,
автозавершением, и т.д.
```

<h4>Конвертация(сериализация, парсинг, выстраивание) данных</h4>
Если запустить предыдущий пример и открыть в браузере http://127.0.0.1:8000/items/3, вы увидите:

```python
{"item": 3}
```

```
Обратите внимание, что значение полученное (и возвращенное) это 3 с типом int, а не str.
Поэтому, объвление типов, дает FastAPI автоматический парсинг запроса (конвертирует строку из http 
запроса в тип данных Python).  
```

<h3>Валидация данных</h3>
Но если перейти в браузере по http://127.0.0.1:8000/items/foo, вы увидите ошибку HTTP:

```JSON
{
    "detail": [
        {
            "loc": [
                "path",
                "item_id"
            ],
            "msg": "value is not a valid integer",
            "type": "type_error.integer"
        }
    ]
}
``` 

Так как параметр `item_id` имеет значение `"foo"`, которые не является типом `int`.
Такая же ошибка возникнет если бы вы передали `float` вместо `int`:<br>
http://127.0.0.1:8000/items/4.2

```
Итак, при объявлении типов данных в Python, FastAPI выполняет их валидацию.
Обратите внимание, что в ошибке четко указывается точка, где не прошла валидация.
Это очень полезно во время разработки и отладки кода который взаимодействует с вашм API.
```

<h3>Документация</h3>
Когда вы откроете браузер на http://127.0.0.1:8000/docs, вы увидите автоматическую, интерактивную
документацию API, например такую:

<img src="https://fastapi.tiangolo.com/img/tutorial/path-params/image01.png" width="393" height="350">

```
Просто объявив тип данных Python, FastAPI дает автоматичекую документацию (на основе Swagger UI).
Обратите внимание, что объявленный параметр пути должен быть integer.
```

<h3>Стандартные преимущества, альтернативная документация</h3>
Так как схема генерируется по стандарту OpenAPI, существует множество совместимых инструментов.

Поэтому, FastAPI предоставляет альтернативную документацию API (используя ReDoc), которая доступна по 
адресу http://127.0.0.1:8000/redoc:

<img src="https://fastapi.tiangolo.com/img/tutorial/path-params/image02.png" width="393" height="350">

Таким способом доступно множество совместимых инструментов. Включая инструменты генерации кода для 
множества языков.

<h4>Pydantic</h4>
Вся валидация данных выполняется "под капотом" с помощью Pydantic, поэтому вы получаете все его
преимущества. 

Вы можете также использовать объявление типов с `str`, `float`, `bool` и другими сложными типами данных.

Некоторые из них рассматриваются в дальнейших главах учебника.

<h4>Очередность имеет значение</h4>
При создании *операций с путями*, вы можете столкнуться с ситуациями, когда у вас фиксированный путь.

Например `/users/me`, допустим он дает данные о текущем пользователе.

Но так же у вас может быть путь `/users/{user_id}`, чтобы получить данные определенного пользователя по
его ID.

Так как *операции с путями* выполняются по порядку, вам нужно быть уверенным, что путь `/users/me` объявлен
перед `/users/{user_id}`:
```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}


@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

Иначе, путь `/users/{user_id}` может работать и для `/users/me`, принимая в параметр `user_id` значение `me`.

Аналогично, вы не можете переопределить операцию пути:
```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users")
async def read_users():
    return ["Rick", "Morty"]


@app.get("/users")
async def read_users2():
    return ["Bean", "Elfo"]
```

Всегда будет использоваться первая т.к. это первое совпадение.

<h4>Предопределенные значения</h4>
Если у вас есть *операция пути* которая принимает *параметр пути*, но вам нужно чтобы возможные значения
были определены заранее, вы можете использовать стандартный `Enum` Python.

**Создайте класс Enum**

Импортируем `Enum` и создаем подкласс, который наследуется от `str` и `Enum`.

С помощью наследования от `str`, документация API понимает, что значения должны иметь тип `string` и сможет
корректно их отображать.

Затем создаем атрибуты класса с определенными значениями, которые будут доступны как допустимые значения:

```python
from enum import Enum

from fastapi import FastAPI


class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"


app = FastAPI()
```


>**Для информации**<br>
>Enum доступен в Python начиная с версии 3.4

>*Подсказка*<br>
> Если вам интересно, "AlexNet", "ResNet" и "LeNet", это модели *машинного обучения*

**Создайте параметр пути**

Затем создайте параметр пути с аннотацией типов используя созданный класс enum (`ModelName`):

```python
from enum import Enum

from fastapi import FastAPI


class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"


app = FastAPI()


@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
```

**Проверьте документацию**

Так как доступные значения параметра пути уже определены, документация может их показать:

<img src="https://fastapi.tiangolo.com/img/tutorial/path-params/image03.png" width="430" height="500">

**Работа с Python enumerations**

Значение параметра пути будет элементом перечисления.

*Сравнение элементов*

Вы можете сравнить параметры с элементами созданными в `ModelName`:
```python
from enum import Enum

from fastapi import FastAPI


class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"


app = FastAPI()


@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}
```

**Получение значения элемента перечисления**

Можно получить значение (`str` в нашем случае). используя `model_name.value`, или в общем случае 
`your_enum_member.value`:

```python
from enum import Enum

from fastapi import FastAPI


class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"


app = FastAPI()


@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}
```

>*Подсказка*
> Можно так же использовать значение "lenet" с `ModelName.lenet.value`.

**Возврат элемента перечисления**

Вы можете вернуть элемент перечисления из операции пути, вложенный в тело JSON(как `dict`).

Элементы будут конвертированы в соответствующие значения (строки в нашем случае) перед их возвратом:

```python
from enum import Enum

from fastapi import FastAPI


class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"


app = FastAPI()


@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}

    return {"model_name": model_name, "message": "Have some residuals"}
```

В клиенте вы получите ответ JSON вроде:

```JSON
{
  "model_name": "alexnet",
  "message": "Deep Learning FTW!"
}
```

<h4>Параметры пути, содержащие пути</h4>
Допустим у вас есть операция пути `/files/{file_path}`.

Но вам нужен `file_path` содержащий в себе путь, например `home/johndoe/myfile.txt`.

Поэтому, URL для этого файла может быть таким: `/files/home/johndoe/myfile.txt`.

**Поддержка OpenAPI**

OpenAPI не поддерживает способ объявления параметра пути, содержащий в себе еще один путь, так как это
может привести к сценариям которые сложны для тестирования и определения.

Тем не менее вы все равно можете сделать это в FastAPI, используя один из встроенных инструментов Starlette.

Документация будет также работать, но не добавит информацию, говорящую, что параметр должен содержать путь.

**Конвертор пути**

Используя опцию, напрямую из Starlette вы можете объявить параметр пути содержащий путь, используя такой URL:

```
/files/{file_path:path}
```

В этом случае, имя параметра это `file_path`, а указание `:path` в конце, говорит, что параметр должен 
совпадать с каким-либо другим путем.

Отсюда, вы можете использовать его так:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

>**Совет**
>Вам может понадобиться параметр `/home/johndoe/myfile.txt`, с первым знаком `/`.
> В таком случае, URL может быть таким: `/files//home/johndoe/myfile.txt`, с двойным `//` между `files`
> и `home`.

<h4>Повторим</h4>
В FastAPI,используя краткое, понятное и стандартное объявление типов Python, вы получаете:

* Поддержку редактора: проверку ошибок, автозаполнение и т.д.
* Парсинг данных
* Валидацию данных
* Описание API и автоматическую документацию

И объявить тип нужно только один раз.

Это, возможно, главное видимое преимущество FastAPI по сравнению с альтернативными фреймворками 
(отдельно от производительности).
