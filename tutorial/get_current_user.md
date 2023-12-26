<h3>Получение текущего пользователя</h3>

В предыдущей главе системы безопасности (которая основывалась на системе внедрения зависимости) функцией операции пути
принимался `token` как `str`:

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


@app.get("/items/")
async def read_items(token: Annotated[str, Depends(oauth2_scheme)]):
    return {"token": token}
```

Но пока это не то чтобы полезно.

Давайте заставим ее выдавать нам текущего пользователя.

<h4>Создать модель пользователя</h4>

Сперва давайте создадим Pydantic модель пользователя.

Точно таким же способом как мы используем Pydantic, чтобы объявить тела запроса, мы можем использовать ее где угодно еще:

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenurl="token")

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None
...
```

<h4>Создать зависимость `get_current_user`</h4>

Давайте создадим зависимость `get_current_user`.

Помните, что зависимости могут иметь подзависимости?

`get_current_user` будет иметь зависимость с тем же `oauth2_scheme` созданным нами до этого.

Точно так же как мы делали ранее непосредственно в операции пути, наша новая зависимость `get_current_user` будет
принимать `token` как `str` из подзависимости `oauth2_scheme`:

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenurl="token")

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None
    
async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    ...
```

<h4>Получить пользователя</h4>

`get_current_user` будет использовать (ненастоящую) созданную нами служебную функцию, которая принимает токен как `str`
и возвращает нашу модель `User`:

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenurl="token")

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None

def fake_decode_token(token):
    return User(
        username=token + "fakedecoded", email="john@example.com", full_name="John Doe"
    )
    
async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    user = fake_decode_token(token)
    return user
```

<h4>Вставить текущего пользователя</h4>

Итак, сейчас мы можем использовать туже `Depends` с нашим `get_current_user` в операции пути:

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenurl="token")

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None

def fake_decode_token(token):
    return User(
        username=token + "fakedecoded", email="john@example.com", full_name="John Doe"
    )
    
async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    user = fake_decode_token(token)
    return user

@app.get("/users/me")
async def read_users_me(current_user: Annotated[User, Depends(get_current_user)]):
    return current_user
```

Обратите внимание, что мы объявляем тип `current_user` как Pydantic модель `User`.

Это поможет нам в функции со всеми завершениями и проверками типов.

> **Совет**
> 
> Вы можете помнить, что тело запроса также объявлены с помощью моделей Pydantic.   
> 
> Здесь FastAPI не будет запутана, так как вы используете `Depends`.

> **Обратите внимание**
> 
> Разработанный способ этой системы зависимости позволяет нам иметь разные зависимости, но все возвращают модель `User`.
> 
> Мы не ограничены иметь только одну зависимость которая может возвращать этот тип данных.

<h4>Другие модели</h4>

Сейчас вы можете получать текущего пользователя напрямую в функции операции пути и выполнять механизмы безопасности на 
уровне **Внедрения Зависимости**, используя `Depends`.

И вы можете использовать любую модель или данные для требований безопасности (в нашем случае - Pydantic модель `User`).

Но вы не ограничены в использовании какой-то определенной модели данных, класса или типа.

Хотите иметь `id` и `email`, и не иметь любой `username` в вашей модели? Разумеется. Вы можете использовать те же 
инструменты. 

Хотите просто иметь просто `str`? Или просто `dict`? Или непосредственно экземпляр класса модели базы данных? Все это 
работает точно таким же способом.

У вас нет пользователей, но есть роботы, боты, или другие системы у которых есть токен доступа? Опять, все работает 
точно так же.

Просто используйте любую модель, любой класс, любую базу данных, которая нужна для вашего приложения. Fast API познакомит 
вас с системой внедрения зависимостей.

<h4>Размер кода</h4>

Этот пример может выглядеть слишком подробным. Имейте в виду, что мы смешиваем безопасность, модели данных, служебные
функции и операции пути в одном файле.

Но это и есть ключевая точка.

Безопасность и внедрение зависимости пишется один раз.

И вы можете сделать ее такой сложной, как захотите. И все равно, написанной только один раз, в одном месте. Со всеми ее
гибкими возможностями.

Но у вас могут быть тысячи ендпоинтов (операций пути), использующих одну систему безопасности. 

И все они (или любое их количество, которое вам нужно) может использовать преимущество переиспользования зависимостей или
любых других созданных вами зависимостей.

И все эти тысячи операций пути могут быть маленькими на 3 строки кода:

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None


def fake_decode_token(token):
    return User(
        username=token + "fakedecoded", email="john@example.com", full_name="John Doe"
    )


async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    user = fake_decode_token(token)
    return user


@app.get("/users/me")
async def read_users_me(current_user: Annotated[User, Depends(get_current_user)]):
    return current_user
```

<h4>Резюме</h4>

Теперь вы можете получать текущего пользователя напрямую в вашей функции операции пути.

Мы почти на пол-пути.

Нам просто нужно добавить операцию пути для пользователя/клиента, чтобы на самом деле отправлять `username` и `password`.

Это будет следующей темой.