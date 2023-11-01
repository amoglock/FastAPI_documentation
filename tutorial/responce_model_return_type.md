<h3>Модель ответа - возвращаемый тип</h3>

Вы можете объявлять какой тип использовать для ответа, с помощью аннотации возвращаемого типа функции операции пути.

Вы можете использовать аннотации типа таким же способом как для входных данных в параметрах функции, вы можете использовать
модели Pydantic, списки, словари, скалярные значения вроде чисел, логические типы и т.д.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list[str] = []

@app.post("/items/")
async def create_item(item: Item) -> Item:
    return item

@app.get("/items/")
async def read_items() -> list[Item]:
    return [
        Item(name="Portal Gun", price=42.0),
        Item(name="Plumbus", price=32.0),
    ]
```

FastAPI будет использовать этот тип возвращаемых данных для:

* Проверки возвращаемых данных.
  * Если данные недопустимы (например вы пропустили поле), это значит, что работа вашего приложения нарушена, не возвращает,
    то что следовало бы, и оно вернет ошибку сервера вместо возвращения некорректных данных. Таким способом вы и ваши клиенты
    могут быть уверены, что они будут получать данные и в ожидаемой форме.
* Добавления схемы JSON для ответа в операции пути OpenAPI.
  * Это будет использоваться для автоматической документации.
  * Она также будет использоваться инструментами автоматической генерации клиентского кода.

Но самое важное:
* Будет ограничивать и фильтровать выходные данные к тому, что установлено в возвращаемом типе.
  * Это особенно важно для безопасности, мы увидим больше об этом ниже.   

<h3>Параметр `response_model`</h3>

Существуют некоторые случаи где вам нужно, или вы хотите возвращать некие данные, которые не совсем то, что заявлено в типе.

Например, вы могли бы захотеть вернуть словарь или объект базы данных, но объявляете его как модель Pydantic. Таким способом
модель Pydantic делала бы всю документацию данных, проверку, и т.д. для объекта который вы вернули (например, словарь
или объект базы данных).

Если вы добавили аннотацию возвращаемого типа, инструменты и редакторы жаловались бы (правильной) ошибкой, говоря вам, что 
ваша функция возвращает тип (например словарь) которые отличается от того, что вы объявили (например модель Pydantic).

И таких случаях, вы можете использовать параметр декоратора операции пути `response_model` вместо типа возврата.

Вы можете использовать параметр `response_model` в любых операциях пути:

* `@app.get()`
* `@app.post()`
* `@app.put()`
* `@app.delete()`
* и т.д.

```python
from typing import Any

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list[str] = []

@app.post("/items/", response_model=list[Item])
async def read_items() -> Any:
    return [
      {"name": "Portal Gun", "price": 42.0},
      {"name": "Plumbus", "price": 32.0},
    ]
```

> **Заметка**
> 
> Обратите внимание, что `response_model` это параметр метода "декоратора" (`get`, `post` и т.д.). А не вашей функции
> операции пути, как все параметры и тело.

`response_model` получает тот же тип который вы бы объявили для поля модели Pydantic, поэтому, он может быть моделью
Pydantic, но он может также быть, например, `list` или моделями Pydantic, как `List[Item]`.

FastAPI будет использовать эту `response_model` чтобы делать всю документацию, проверку, и т.д. и еще, чтобы конвертировать
и фильтровать выходные данные в объявленный тип.

> **Совет**
> 
> Если у вас есть строгие проверки типов в вашем редакторе, mypy, и т.д., вы можете объявлять тип возврата функции как `Any`.
> 
> Этим способом вы говорите редактору, что вы специально возвращаете "что угодно". Но FastAPI все равно будет делать
> документацию, проверку, фильтрацию и т.д. с помощью `response_model`.

<h3>Приоритет `response_model`</h3>

Если вы объявляете и тип возврата, и `response_model`, `response_model` будет иметь приоритет и использоваться FastAPI.

Таким способом вы можете добавлять правильные аннотации типов вашим функциям, даже когда вы возвращаете тип, отличающийся
от модели ответа, чтобы он был использован в редакторе и инструментах вроде mypy. А у вас FastAPI все еще делает проверку
данных, документацию, и т.д., используя `response_model`.

Вы также можете использовать `response_model=None` чтобы отключить создание модели ответа для операции пути, вам, возможно,
потребуется сделать это если вы добавляете аннотации типов для того, что не подходит полям Pydantic, вы увидите такой пример
в одной из секций ниже.

<h3>Возврат одних и тех же входных данных</h3>

Здесь мы объявляем модель `UserIn`, она будет содержать незашифрованный пароль:

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: str | None = None
...
```

> **Для информации**
> 
> Чтобы использовать `EmailStr`, сперва установите <a href="https://github.com/JoshData/python-email-validator">`email_validator`</a>.
> 
> Например `pip install email-validator` или `pip install pydantic[email]`.

И мы используем эту модель, чтобы объявить наши входящие данные и ту же модель для выходящих:

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: str | None = None

# Не делайте так в настоящей разработке!
@app.post("/user/")
async def create_user(user: UserIn) -> UserIn:
    return user
```

Теперь, всякий раз когда браузер создает пользователя с паролем, API будет возвращать пароль в ответе.

В этом случае это может не быть проблемой, так как пароль отправляет тот же пользователь.

Но если мы используем ту же модель для других операций пути, мы бы отправляли пароли наших пользователей каждому клиенту.

> **Опасность**
> 
> Никогда не храните незашифрованный пароль пользователя или не отправляйте его в ответе как здесь, если только вы 
> не знаете всех предупреждений и вы знаете, что делаете. 

<h3>Добавление модели вывода</h3>

Вместо этого, мы можем создать модель с незашифрованным паролем для входящих данных и модель для выходящих без него:

```python
from typing import Any

from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: str | None = None

class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: str | None = None
...
```

Здесь, даже если наша функция операции пути возвращает того же пользователя, содержащего пароль, что и на входе:

```python
from typing import Any

from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: str | None = None

class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: str | None = None

@app.post("/user/", response_model=UserOut)
async def create_user(user: UserIn) -> Any:
    return user
```

Мы объявили, чтобы `response_model` была нашей моделью `UserOut`, которая не включает в себя пароль:

```python
@app.post("/user/", response_model=UserOut)
```

Поэтому, FastAPI позаботится о фильтрации всех данных, которые не объявлены в модели вывода (используя Pydantic).

<h3>`response_model` или возвращаемый тип</h3>

В таком случае, так как две модели отличаются, если мы обозначили тип возврата функции как `UserOut`, редактор и
инструменты жаловались бы, что мы возвращаем неподходящий тип, поскольку это разные классы.

Вот почему в примере нам нужно объявлять его в параметре `response_model`.

...но продолжайте читать дальше, чтобы узнать как обойти это.

<h3>Тип возврата и фильтрация данных</h3>

Давайте продолжим с предыдущего примера. Мы хотели описать функцию с одним типом, а возвращать что-то, что включает
больше данных.

Мы хотим чтобы FastAPI продолжал фильтровать данные, используя модель ответа.

В предыдущем примере, так как классы отличались, нам нужно было использовать параметр `response_model`. Но, это так же
означает, что мы не получаем поддержку от редактора и инструментов проверки типа возвращаемого функцией.

Но в большинстве случаев, где нам нужно сделать что-то типа этого, мы хотим чтобы модель просто фильтровала/удаляла 
какие-то данные, как в примере.

И в таких случаях мы можем использовать классы и наследование, чтобы воспользоваться преимуществами аннотаций типов 
функции, чтобы получить лучшую поддержку в редакторе и инструментах, и все еще получать фильтрацию данных FastAPI.

```python
from fastapi import FastApi()
from pydantic import BaseModel, EmailStr

app = FastApi()

class BaseUser(BaseModel):
    username: str
    email: EmailStr
    full_name: str | None = None

class UserIn(BaseUser):
    password: str

@app.post("/user/")
async def create_user(user: UserIn) -> BaseUser:
    return user
```

Вот так мы получаем поддержку инструментов от редакторов и mypy, так как этот код правильный в условиях типов, но мы
также получаем фильтрацию данных от FastAPI.

Как это работает? Давайте это выясним.🤓

<h4>Аннотации типов и инструментарий</h4>

Сперва, давайте посмотрим как редакторы, mypy и другие инструменты видели бы это.

`BaseUser` имеет три базовых поля. Затем `UserIn` наследуется от `BaseUser` и добавляет поле `password`, поэтому, он
будет включать в себя все поля из обеих моделей.

Мы указываем тип возврата функции как `BaseUser`, но в действительности возвращаем экземпляр `UserIn`.

Редактор, mypy, и другие инструменты не жаловались бы на это, потому, что в терминах типизации, `UserIn` это подкласс
`BaseUser`, который означает, что это допустимый тип когда ожидается что-то, что является `BaseUser`.

<h4>Фильтрация данных FastAPI</h4>

Теперь FastAPI будет видеть возвращаемый тип и убедится, что ваш возврат включает только поля которые объявлены в типе.

FastAPI делает несколько вещей внутри себя и Pydantic, чтобы убедиться, что те же самые правила наследования класса не
использованы для фильтрации возвращаемых данных, иначе вы могли бы вернуть гораздо больше данных, чем рассчитывали.

Таким образом вы можете получать лучшее из двух миров: аннотации типов с поддержкой инструментария и фильтрацию данных.

<h3>Смотрите все в документации</h3>

Когда вы видите автоматическую документацию, вы можете проверять, что входящая и выходящая модели будут иметь свои
собственные схемы:

<img src="https://fastapi.tiangolo.com/img/tutorial/response-model/image01.png" width="393" height="370">

И обе модели будут использованы для интерактивной документации API:

<img src="https://fastapi.tiangolo.com/img/tutorial/response-model/image02.png" width="393" height="360">

<h3>Иная аннотация возвращаемого типа</h3>

Могут быть случаи когда вы возвращаете что-то, что не является подходящим полем Pydantic и вы описываете это в функции,
только чтобы получить поддержку, предоставленную инструментарием (редактор, mypy, и т.д.).

<h4>Возврат Ответа напрямую</h4>

Самый частый случай мог бы быть <a href="https://fastapi.tiangolo.com/advanced/response-directly/">
возвратом Ответа напрямую как описано далее в расширенной документации</a>.

```python
from fastapi import FastAPI, Response
from fastapi.responses import JSONResponse, RedirectResponse

app = FastAPI()

@app.get("/portal")
async def get_portal(teleport: bool = False) -> Response:
    if teleport:
        return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
    return JSONResponse(content={"message": "Here's your interdimensional portal."})
```

Этот простой пример обработан автоматически FastAPI, потому что описание возвращаемого типа это класс (или подкласс) `Response`.

И инструменты тоже будут корректны, так как и `RedirectResponse`, и `JSONResponse` это подклассы `Response`, поэтому
описание типа правильное.

<h4>Описание Подкласса Ответа</h4>

Вы также можете использовать подкласс `Response` в аннотации типа:

```python
from fastapi import FastAPI
from fastapi.responses import RedirectResponse

app = FastAPI()

@app.get("/teleport")
async def get_teleport() -> RedirectResponse:
    return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
```

Это будет тоже работать, потому что `RedirectResponse` это подкласс `Response` и FastAPI будет автоматически обрабатывать 
этот простой случай.

<h4>Неправильное описание возвращаемого типа</h4>

Но когда вы возвращаете какой-то произвольный объект, тип которого недопустим в Pydantic (например, объект базы данных) и
вы описываете его в функции, FastAPI будет пытаться создать модель ответа Pydantic из этой аннотации типа, и потерпит неудачу.

То же самое может случиться если у вас было что-то вроде объединения нескольких типов где один или несколько из них - 
недопустимые типы Pydantic, например это может быть провалом 💥:

```python
from fastapi import FastAPI, Response
from fastapi.responses import RedirectResponse

app = FastAPI()


@app.get("/portal")
async def get_portal(teleport: bool = False) -> Response | dict:
    if teleport:
        return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ") 
    return {"message": "Here`s your interdimensional portal."}
```

...такой код проваливается потому что аннотация типа это не тип Pydantic и не просто единичный класс `Response` или его
подкласс, это объединение (любой из двух) между `Response` и `dict`.

<h4>Отключение модели ответа</h4>

Продолжаем с примером выше, вам можете не хотеть иметь проверку данных по умолчанию, документацию, фильтрацию и т.д., это
реализовано в FastAPI.

Но может вы хотите все равно сохранять описание типа возврата в функции, чтобы получить поддержку инструментов вроде 
редакторов и проверки типов (например mypy).

В таком случае, вы можете отключать создание модели ответа установив `response_model=None`:

```python
from fastapi import FastAPI, Response
from fastapi.responses import RedirectResponse

app = FastAPI()

@app.get("/portal", response_model=None)
async def get_portal(teleport: bool = False) -> Response | dict:
    if teleport:
        return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
    return {"message": "Here's your interdimensional portal."}
```

Так FastAPI будет пропускать создание модели ответа и таким способом у вас могут быть любые описания типов возврата 
которые вам нужны, не влияющие на ваше приложение FastAPI. 🤓

<h3>Шифрование параметров Модели Ответа</h3>

Ваша модель ответа может иметь значения по умолчанию, например:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float = 10.5
    tags: list[str] = []
...
```

* `description: Union[str, None] = None` (или `str | None = None` в Python 3.10) имеет значение `None`.
* `tax: float = 10.5` имеет `10.5`.
* `tags: List[str] = []` имеет пустой список по умолчанию: `[]`.

Но вы можете захотеть исключить их из результата если они на самом деле не были сохранены.

Например, если у вас есть модели со многими необязательными аттрибутами в NoSQL базе, но вы не хотите отправлять очень
длинные JSON ответы состоящие из значений по умолчанию.

<h4>Использование параметра `response_model_exclude_unset`</h4>

Вы можете установить параметр декоратора операции пути `response_model_exclude_unset=True`:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float = 10.5
    tags: list[str] = []

items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}

@app.get("/items/{item_id}", response_model=Item, response_model_exclude_unset=True)
async def read_item(item_id: str):
    return items[item_id]
```

И значения по умолчанию не будут включены в ответ, только фактически установленные.

Поэтому, если вы отправляете запрос в эту операцию пути для элемента с ID `foo`, ответ (не включающий значения по умолчанию)
будет таким:

```JSON
{
    "name": "Foo",
    "price": 50.2
}
```

> **Для информации**
> 
> FastAPI использует `.dict()` модели Pydantic с его 
> <a href="https://pydantic-docs.helpmanual.io/usage/exporting_models/#modeldict">параметром `exclude_unset`</a>
> чтобы получить такой результат.

> **Для информации**
> 
> Вы также можете использовать:
> 
>   * `response_model_exclude_defaults=True`
>   * `response_model_exclude_none=True`
> 
> Как описано в <a href="https://pydantic-docs.helpmanual.io/usage/exporting_models/#modeldict">документации Pydantic</a> 
> для`exclude_defaults` и `exclude_none`.

<h4>Данные со значениями для полей по умолчанию</h4>

Но если выши данные имеют значения для полей модели у которых есть умолчания, например как элемент с ID `bar`:

```JSON
{
  "name": "Bar",
  "description": "The bartenders",
  "price": 62,
  "tax": 20.2
}
```

они будут включены в ответ.

<h4>Данные с такими же значениями как и умолчания</h4>

Если данные имеют те же значения, что и умолчания, как, например, элемент с ID `baz`:

```Python
{
    "name": "Baz",
    "description": None,
    "price": 50.2,
    "tax": 10.5,
    "tags": []
}
```

FastAPI достаточно умный (на самом деле это Pydantic) чтобы понять это, даже несмотря на то, что `description`, `tax` и 
`tags` имеют те же значения, как и умолчания, они будут установлены явно (вместо того чтобы брать умолчания).

Поэтому они будут включены в ответ JSON.

> **Совет**
> 
> Обратите внимание, что значения по умолчанию могут любыми, а не только `None`.
> 
> Они могут быть списком (`[]`), `float` от `10.5`, и т.д.

<h4>`response_model_include` и `response_model_exclude`</h4>

Вы также можете использовать параметры декоратора операции пути `response_model_include` и `response_model_exclude`.

Они принимают `set` из `str` с названием аттрибутов, чтобы включать (игнорируя остальное) или исключать (включая остальное).

Это может быть полезно как быстрый короткий путь если у вас есть только одна модель Pydantic и вы хотите удалить какие-то
данные из вывода.

> **Совет**
> 
> Но все же рекомендовано использовать идеи выше, используя несколько классов, вместо этих параметров.
> 
> Это потому, что JSON схема, сгенерированная в OpenAPI(и документации) вашего приложения, все еще будет единственной для
> полной модели, даже если вы используете `response_model_include` и `response_model_exclude` чтобы пропускать какие-то
> аттрибуты.
> 
> Это так же применяется к `response_model_by_alias` которая работает похоже.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float = 10.5


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The Bar fighters", "price": 62, "tax": 20.2},
    "baz": {
        "name": "Baz",
        "description": "There goes my baz",
        "price": 50.2,
        "tax": 10.5,
    },
}

@app.get(
    "/items/{item_id}/name",
    response_model=Item,
    response_model_include={"name", "description"},
)
async def read_item_name(item_id: str):
    return items[item_id]

@app.get("/items/{item_id}/public", response_model=Item, response_model_exclude={"tax"})
async def read_item_public_data(item_id: str):
    return items[item_id]
```

> **Подсказка**
> 
> Синтаксис {"name", "description"} создает `set` с этими двумя значениями.
> 
> Он эквивалентен `set(["name", "description"])`.

<h4>Использование `list` вместо `set`</h4>

Если вы забываете использовать `set` и вместо этого используете `list` или `tuple`, FastAPI все равно будет конвертировать
их в `set` и работать правильно:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float = 10.5


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The Bar fighters", "price": 62, "tax": 20.2},
    "baz": {
        "name": "Baz",
        "description": "There goes my baz",
        "price": 50.2,
        "tax": 10.5,
    },
}


@app.get(
    "/items/{item_id}/name",
    response_model=Item,
    response_model_include=["name", "description"],
)
async def read_item_name(item_id: str):
    return items[item_id]


@app.get("/items/{item_id}/public", response_model=Item, response_model_exclude=["tax"])
async def read_item_public_data(item_id: str):
    return items[item_id]
```

<h3>Резюме</h3>

Используйте параметр декоратора операции пути `response_model` чтобы объявить модели ответа и особенно, чтобы обеспечить
фильтрацию приватных данных.

Используйте `response_model_exclude_unset` чтобы возвращать только явно установленные значения.