<h3>Дополнительные модели</h3>

Продолжая с предыдущим примером, обычным делом будет иметь более одной модели.

Это особенно актуально для моделей пользователя, потому что:

    * Входная модель обязательно должна иметь пароль.
    * Выходная модель не должна иметь пароль.
    * Модель базы данных вероятно бы имела хешированный пароль.

> **Внимание!!!**
> 
> Никогда не храните незашифрованные пароли пользователей. Всегда храните "безопасный хеш", который затем можете проверить.
> 
> Если вы не знаете, вы научитесь, что такое "безопасный хеш" в разделах о безопасности.

<h3>Несколько моделей</h3>

Вот основная идея как могут выглядеть модели с их полями пароля и места где они используются:

```python
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

class UserInDB(BaseModel):
    username: str
    hashed_password: str
    email: EmailStr
    full_name: str | None = None

def fake_password_hasher(raw_password: str):
    return "supersecret" + raw_password

def fake_save_user(user_in: UserIn):
    hashed_password = fake_password_hasher(user_in.password)
    user_in_db = UserInDB(**user_in.dict(), hashed_password=hashed_password)
    print("User saved! ..not really")
    return user_in_db

@app.post("/user/", response_model=UserOut)
async def create_user(user_in: UserIn):
    user_saved = fake_save_user(user_in)
    return user_saved
```

<h4>О `**user_in.dict()`</h4>

<h5>Метод `.dict()` Pydantic</h5>

`user_in` это модель Pydantic класса `UserIn`.

Модели Pydantic имеют метод `.dict()` который возвращает `dict` с данными модели.

Поэтому, если мы создаем объект Pydantic, например, `user_in`:

```python
user_in = UserIn(username="john", password="secret", email="john.doe@example.com")
```

А затем вызываем:

```python
user_dict = user_in.dict()
```

У нас теперь есть `dict` с данными в переменной `user_dict` (это `dict` вместо модели Pydantic).

И если мы вызываем:

```python
print(user_dict)
```

Мы бы получили Python `dict` со значениями:

```python
{
  'username': 'john',
  'password': 'secret',
  'email': 'john.doe@example.com',
  'full_name': None
}
```

<h5>Распаковка `dict`</h5>

Если мы берем `dict`, например `user_dict` и передаем его в функцию (или класс) с `**user_dict`, Python будет 
"разворачивать" его. Он передаст ключи и значения от `user_dict` напрямую, как аргументы ключ-значение.

Отсюда, продолжая с `user_dict` выше, пишем:

```python
UserInDB(**user_dict)
```

Результат был бы эквивалентен чему-то вроде:

```python
UserInDB(
    username="john",
    password="secret",
    email="john.doe@example.com",
    full_name=None,
)
```

Или точнее, используя `user_dict` напрямую, с каким бы ни было содержанием которое он может иметь в будущем:

```python
UserInDB(
    username = user_dict["username"],
    password = user_dict["password"],
    email = user_dict["email"],
    full_name = user_dict["full_name"],
)
```

<h4>Pydantic модель из другого содержимого</h4>

Как в примере выше, мы получили `user_dict` из `user_in.dict()`, такой код:

```python
user_dict = user_in.dict()
UserInDB(**user_dict)
```

Был бы эквивалентен этому:

```python
UserInDB(**user_in.dict())
```

Потому, что `user_in.dict()` это `dict`, а затем мы делаем его "распаковку" Python, передав словарь в `UserInDB`,
добавив `**`.

Поэтому, мы получаем модель Pydantic из данных другой модели.

<h4>Распаковка `dict` и дополнительных ключевых слов</h4>

А затем, добавляя дополнительный аргумент ключевого слова `hashed_password=hashed_password`, вот так:

```python
UserInDB(**user_in.dict(), hashed_password=hashed_password)
```

...закончится все тем, что будет похоже на:

```python
UserInDB(
    username = user_dict["username"],
    password = user_dict["password"],
    email = user_dict["email"],
    full_name = user_dict["full_name"],
    hashed_password = hashed_password,
)
```

> **Внимание**
> 
> Поддержка дополнительных функций это просто для демонстрации возможного потока данных, но они, конечно, не предоставляют
> какую-то реальную безопасность.

<h3>Уменьшаем дублирование</h3>

Уменьшение дублирования кода это одна из основных идей в FastAPI.

Так как дублирование кода увеличивает шансы на ошибки, вопросы безопасности, проблемы десинхронизации кода (когда вы 
обновляете в одном месте, но не в других), и т.д.

И все эти модели делятся множеством данных и дублируют имена аттрибутов и типов.

Мы бы могли сделать лучше.

Мы можем объявлять модель `UserBase` которая служит базой для других наших моделей. А затем мы можем делать подклассы от
этой модели, которые наследуют ее аттрибуты (объявление типов, проверку, и т.д.).

Все преобразования данных, проверки, документация, и т.д. все равно будет работать как обычно.

Таким образом, мы можем объявлять просто отличия между моделями (незашифрованный `password`, `hashed_password` и без пароля):

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserBase(BaseModel):
    username: str   
    email: EmailStr
    full_name: str | None = None

class UserIn(UserBase):
    password: str

class UserOut(UserBase):
    pass

class UserInDB(UserBase):
    hashed_password: str

...
```

<h3>`Union` или `anyOf`</h3>

Вы можете объявлять чтобы ответ был `Union` из двух типов, это означает, что ответ может быть любым из этих двух.

Он был бы объявлен в FastAPI с помощью `anyOf`.

Чтобы это сделать, используйте стандартную подсказку типов Python 
<a href="https://docs.python.org/3/library/typing.html#typing.Union">`typing.Union`</a>

> **Заметка**
> 
> Когда объявляется <a href="https://pydantic-docs.helpmanual.io/usage/types/#unions">`Union`</a>, сначала включите наиболее
> конкретный тип, за которым следует менее конкретный тип. В примере ниже, более конкретный `PlaneItem` идет перед `CarItem`
> в `Union[PlaneItem, CarItem]`.

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class BaseItem(BaseModel):
    description: str
    type: str

class CarItem(BaseItem):
    type: str = "car"

class PlaneItem(BaseItem):
    type: str = "plane"
    size: int

items = {
    "item1": {"description": "All my friends drive a low rider", "type": "car"},
    "item2": {
        "description": "Music is my aeroplane, it's my aeroplane",
        "type": "plane",
        "size": 5,
    },
}

@app.get("/items/{item_id}", response_model=Union[PlaneItem, CarItem])
async def read_item(item_id: str):
    return items[item_id]
```

<h4>`Union` в Python 3.10</h4>

В этом примере мы передаем `Union[PlaneItem, CarItem]` как значение аргумента `response_model`.

Так как мы передаем его как значение в аргумент вместо размещения его в аннотации типа, нам нужно использовать `Union`
даже Python 3.10.

Еси бы это была аннотация типа, нам бы нужно было использовать вертикальную черту:

```python
some_variable: PlaneItem | CarItem
```

Но если мы размещаем ее в `response_model=PlaneItem | CarItem` мы бы получали ошибку, потому, что Python пытался бы 
совершить недопустимую операцию между `PlaneItem` и `CarItem`, вместо определения их как описание типов.

<h3>Список моделей</h3>

Таким же способом вы можете объявлять ответы списками объектов.

Для этого используйте стандартный `typing.List` Python (или просто `list` в Python 3.9 или выше):

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str
items = [
    {"name": "Foo", "description": "There comes my hero"},
    {"name": "Red", "description": "it`s my aeroplane"},
]

@app.get("/items/", response_model=list[Item])
async def read_items():
    return items
```

<h3>Ответ произвольным `dict`</h3>

Вы также можете объявлять ответ, используя простой произвольный `dict`, просто объявив тип ключей и значений, без 
использования модели Pydantic.

Это полезно если не знаете допустимое поле/имена аттрибута (которые могли бы понадобиться для модели Pydantic) заранее.

В таком случае, вы можете использовать `typing.Dict` (или просто `dict` в Python 3.9 или выше):

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("keyword-weights/", response_model=dict[str, float])
async def read_keyword_weights():
    return {"foo": 2.3, "bar": 3.4}
```

<h3>Резюме</h3>

Используйте несколько моделей Pydantic и легко наследуйтесь от них для каждого случая.

Вам не обязательно иметь одну модель данных для каждой сущности, если эта сущность должны иметь разные "состояния". Как 
в случае с "сущностью" пользователя с состоянием включающим `password`, `password_hash` и без пароля.