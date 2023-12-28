# Простой OAuth2 с паролем и Bearer

Теперь давайте отталкиваться от прошлой главы и добавим пропущенные части, чтобы иметь завершенный поток безопасности.

## Получение `username` и `password`

Мы собираемся использовать утилиты безопасности FastAPI, чтобы получить `username` и `password`.

OAuth2 определяет, что когда используя "поток пароля" (что мы и делаем), клиент/пользователь должен отправлять поля
`username` и `password` как данные формы.

А спецификация говорит, что эти поля должны быть названы именно так. Поэтому `user-name` или `email` не будут работать.

Но не беспокойтесь, вы можете показать их так, как желаете, для конечных пользователей во фронтенде.

А ваши модели базы данных могут использовать любые имена, которые вы захотите. 

Но для операции пути **входа в систему**, нам нужно использовать эти названия, чтобы они были совместимыми со 
спецификацией (и были способны, например, использовать встроенную систему документации API).

Также, спецификация устанавливает, что `username` и `password` должны быть отправлены как данные формы (поэтому здесь не
JSON).

### `scope`

Также, спецификация говорит, что клиент может отправить еще одно поле формы "`scope`".

Название поля формы - `scope` (в единственном числе), но на самом деле это длинная строка с "scopes", разделенными 
пробелами.

Каждый "scope" это просто строка (без пробелов).

Обычно они используются, чтобы объявлять определенные требования безопасности, например:
* `users:read` или `users:write` частые примеры.
* `instagram_basic` используется Facebook/Instagram.
* `https://www.googleapis.com/auth/drive` используется Google.

> **Для информации**
> 
> В OAuth2, "scope" это просто строка которая объявляет определенные требования допуска. 
> 
> Не важно есть ли в нем какие-то другие символы вроде `:` или если это URL.
> 
> Эти детали зависят от конкретной реализации.
> 
> Для OAuth2 все они просто строки.

## Код, чтобы получить `username` и `password`

Теперь давайте использовать утилиты предоставленные FastAPI, чтобы все это обработать.

### `OAuth2PasswordRequestForm`

Сперва импортируем `OAuth2PasswordRequestForm` и используем ее как зависимость с `Depends` в операции пути для `/token`:

```python
from typing import Annotated

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
...

app = FastAPI()
...

@app.post("/token")
async def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    ...
```

`OAuth2PasswordRequestForm` это класс зависимости который объявляет форму тела запроса с:

* `username`.
* `password`.
* Необязательным полем `scope` как большая строка, составленной из строк, разделенных пробелами.
* Необязательным `grant_type`.

> **Совет**
> 
> На самом деле спецификация OAuth2 требует поле `grant_type` с установленным значением `password`, но 
> `OAuth2PasswordRequestForm` не принуждает к этому.
> 
> Если оно вам обязательно нужно, то используйте `OAuth2PasswordRequestFormStrict` вместо `OAuth2PasswordRequestForm`.

* Необязательным `client_id` (для нашего примера он нам не нужен).
* Необязательным `client_secret` (для нашего примера он нам не нужен).

> **Для информации**
> 
> `OAuth2PasswordRequestForm` это не специальный класс для FastAPI как `OAuth2PasswordBearer`.
> 
> `OAuth2PasswordBearer` делает для FastAPI известным, что он это схема безопасности. Поэтому он добавил этот способ в OpenAPI.
> 
> Но `OAuth2PasswordRequestForm` это просто класс зависимости, который вы можете написать сами, или можете объявить 
> напрямую параметр `Form`.
> 
> Но так как это часто используемый случай, он предоставлен FastAPI напрямую, просто, чтобы упростить его.

### Используйте данные формы

> **Совет**
> 
> Экземпляр класса зависимости `OAuth2PasswordRequestForm` не будет иметь аттрибут `scope` с длинной строкой разделенной 
> пробелами, вместо этого, у него будет аттрибут `scopes` с актуальным списком строк для каждого отправленного scope.
> 
> Мы не используем `scopes` в этом примере, но если вам это нужно, такая функциональность есть.

Теперь, получим данные пользователя из (придуманной) базы данных, используя поле `username` из формы.

Если такого пользователя нет, мы возвращаем ошибку, говорящую "incorrect username or password".

Для ошибки, мы используем исключение `HTTPException`:

```python
from typing import Annotated

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "fakehashedsecret",
        "disabled": False,
    },
    "alice": {
        "username": "alice",
        "full_name": "Alice Wonderson",
        "email": "alice@example.com",
        "hashed_password": "fakehashedsecret2",
        "disabled": True,
    },
}

app = FastAPI()
...

@app.post("/token")
async def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detaiil="incorrect username or password")
    ...
```

### Проверьте пароль

На этом этапе у нас есть данные пользователя из нашей базы данных, но мы не проверили пароль.

Давайте сперва поместим данные в модель Pydantic `UserInDB`.

Вам никогда не следует хранить незашифрованные пароли, поэтому, мы будем использовать (ненастоящую) систему хеширования
пароля.

Если пароль не совпадает, мы возвращаем ту же ошибку как ранее.


#### Хеширование пароля

"Хеширование" означает: конвертация какого-то содержимого (в нашем случае пароль) в последовательность байтов (просто 
строка), которая не несет смысла.

Каждый раз когда вы передаете одно и то же содержимое (один и тот же пароль) вы получаете одну и ту же "бессмыслицу".

Но вы не можете конвертировать "бессмыслицу" обратно в пароль.

###### Зачем использовать хеширование пароля

Если ваша база данных украдена, у вора не будет незашифрованных паролей пользователей, только хеши.

Поэтому, вор не сможет использовать эти пароли в другой системе (так как многие пользователи используют одинаковые пароли
везде, это было бы опасно).

```python
from typing import Annotated

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel

fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "fakehashedsecret",
        "disabled": False,
    },
    "alice": {
        "username": "alice",
        "full_name": "Alice Wonderson",
        "email": "alice@example.com",
        "hashed_password": "fakehashedsecret2",
        "disabled": True,
    },
}

app = FastAPI()

def fake_hash_password(password: str):
    return "fakehashed" + password

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None


class UserInDB(User):
    hashed_password: str

@app.post("/token")
async def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detaiil="incorrect username or password")
    user = UserInDB(**user_dict)
    hashed_password = fake_hash_password(form_data.password)
    if not hashed_password == user.hashed_password:
        raise HTTPException(status_code=400, detaiil="incorrect username or password")
    ...
```

#### Про `**user_dict`

`UserInDB(**user_dict)` означает:

Передать ключи и значения `user_dict` как аргументы ключ-значение, эквивалентное этому:

```python
UserInDB(
    username = user_dict["username"],
    email = user_dict["email"],
    full_name = user_dict["full_name"],
    disabled = user_dict["disabled"],
    hashed_password = user_dict["hashed_password"],
)
```

> **Для информации**
> 
> Для более полного объяснения о `**user_dict` вернитесь в документацию про
> <a href="https://github.com/amoglock/FastAPI_documentation/blob/master/tutorial/extra_models.md">Дополнительные модели</a>.

## Возвращение токена

Ответ ендпоинта `token` должен быть объектом JSON.

Ему нужно иметь `token_type`. В нашем случае, так как мы используем "Bearer" токены, тип токена должен быть "`bearer`".

И ему нужно иметь `access_token`, со строкой, содержащей наш токен доступа.

Для этого простого примера, мы собираемся просто возвращать тот же `usrname` в качестве токена.

> **Совет**
> 
> В следующей главе, вы увидите настоящую реализацию безопасности, с хешированием пароля и JWT токенами.
> 
> Но сейчас, давайте сосредоточимся на нужных нам деталях.

```python
from typing import Annotated

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel

fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "fakehashedsecret",
        "disabled": False,
    },
    "alice": {
        "username": "alice",
        "full_name": "Alice Wonderson",
        "email": "alice@example.com",
        "hashed_password": "fakehashedsecret2",
        "disabled": True,
    },
}

app = FastAPI()

def fake_hash_password(password: str):
    return "fakehashed" + password

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None


class UserInDB(User):
    hashed_password: str

@app.post("/token")
async def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detaiil="incorrect username or password")
    user = UserInDB(**user_dict)
    hashed_password = fake_hash_password(form_data.password)
    if not hashed_password == user.hashed_password:
        raise HTTPException(status_code=400, detaiil="incorrect username or password")
    
    return {"access_token": user.username, "token_type": "bearer"}
```

> **Совет**
> 
> По спецификации, вам следует возвращать JSON с `access_token` и `token_type`, так же как и в примере.
> 
> Это то, что вам нужно делать самостоятельно в вашем коде, и быть уверенными, что вы используете эти ключи JSON.
> 
> Это, практически, единственная вещь которую вам нужно запомнить, правильно сделать самому, чтобы соответствовать
> спецификации.
> 
> Остальное FastAPI сделает за вас.

## Обновление зависимостей

Теперь мы собираемся обновить наши зависимости.

Мы хотим получать `current_user` только если пользователь активен.

Поэтому, мы создаем дополнительную зависимость `get_current_active_user`, которая, в свою очередь, использует 
`get_current_user` как зависимость.

Обе этих зависимости будут просто возвращать HTTP ошибку, если пользователь не существует или неактивен.

Поэтому, в нашем ендпоинте, мы получим пользователя только если он существует, правильно аутентифицировался и активен:

```python
from typing import Annotated

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel

fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "fakehashedsecret",
        "disabled": False,
    },
    "alice": {
        "username": "alice",
        "full_name": "Alice Wonderson",
        "email": "alice@example.com",
        "hashed_password": "fakehashedsecret2",
        "disabled": True,
    },
}

app = FastAPI()

def fake_hash_password(password: str):
    return "fakehashed" + password

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None


class UserInDB(User):
    hashed_password: str

    
def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)
    
    
def fake_decode_token(token):
    # This doesn't provide any security at all
    # Check the next version
    user = get_user(fake_users_db, token)
    return user    
    

async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    user = fake_decode_token(token)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},            
        )
    return user


async def get_current_active_user(
        current_user: Annotated[User, Depends(get_current_user)]
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user
    
    
@app.post("/token")
async def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detaiil="incorrect username or password")
    user = UserInDB(**user_dict)
    hashed_password = fake_hash_password(form_data.password)
    if not hashed_password == user.hashed_password:
        raise HTTPException(status_code=400, detaiil="incorrect username or password")
    
    return {"access_token": user.username, "token_type": "bearer"}

@app.get("/users/me")
async def read_users_me(
        current_user: Annotated[User, Depends(get_current_active_user)]
):
    return current_user
```

> **Для информации**
> 
> Дополнительный заголовок `WWW-Authenticate` со значением `bearer` мы возвращаем так как он тоже часть спецификации.
> 
> Любой HTTP(ошибка) статус-код 401 "UNAUTHORIZED" предполагает так же вернуть заголовок  `WWW-Authenticate`.
> 
> В случае токенов bearer (наш случай), значение этого заголовка должно быть `Bearer`.
> 
> На самом деле вы можете пропускать этот дополнительный заголовок и он все равно будет работать.
> 
> Но здесь он предоставлен, чтобы соответствовать спецификации.
> 
> Также, могут существовать инструменты (сейчас или в будущем), которые ожидают и используют его, и это может быть 
> полезно для вас или ваших пользователей, сейчас или в будущем.
> 
> В этом преимущество стандартов...

## Смотрим все это в действии

Откройте интерактивную документацию http://127.0.0.1:8000/docs.

### Аутентификация

Кликните на кнопку "Authorize".

Используйте учетные данные:

Пользователь: `johndoe`

Пароль: `secret`

<img src="https://fastapi.tiangolo.com/img/tutorial/security/image04.png">

После аутентификации в системе, вы увидите что-то вроде этого:

<img src="https://fastapi.tiangolo.com/img/tutorial/security/image05.png">

### Получите ваши данные пользователя

Теперь используем операцию `GET` по адресу `/users/me`.

Вы получите ваши данные пользователя, например:

```python
{
  "username": "johndoe",
  "email": "johndoe@example.com",
  "full_name": "John Doe",
  "disabled": false,
  "hashed_password": "fakehashedsecret"
}
```

<img src="https://fastapi.tiangolo.com/img/tutorial/security/image06.png">

Если кликнуть на иконку замка и выйти, а затем попытаться снова выполнить ту же операцию, вы получите ошибку HTTP 401:

```python
{
  "detail": "Not authenticated"
}
```

### Неактивный пользователь

Теперь попробуем пройти аутентификацию с неактивным пользователем:

Пользователь: `alice`

Пароль: `secret2`

И попробуем использовать операцию `GET` по адресу `/users/me`.

Вы получите "inactive user":

```python
{
  "detail": "Inactive user"
}
```

## Резюме

Теперь у вас инструменты, чтобы реализовать сложную систему безопасности, основанную на `username` и `password` для 
вашего API.

Используя эти инструменты, вы можете делать систему безопасности совместимой с любой базой данных и любой моделью 
пользователя или данных.

Единственная пропущенная деталь это то, что на самом деле все это пока не "безопасно".

В следующей главе вы увидите как использовать библиотеку безопасного хеширования пароля и JWT токены.