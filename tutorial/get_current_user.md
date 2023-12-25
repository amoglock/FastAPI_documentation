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
