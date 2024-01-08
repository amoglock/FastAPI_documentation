# Oauth2 с паролем (и хешированием), Bearer с JWT токенами

Теперь, когда у нас есть весь поток безопасности, давайте сделаем приложение безопасным на самом деле, используя JWT токены
и безопасное хеширование пароля.

Этот код - что-то такое, что вы можете на самом деле использовать в вашем приложении, хранить хеши паролей в базе данных, 
и т.д.

Мы собираемся начать оттуда, где мы остановились в предыдущей главе и нарастить его.

## Про JWT

JWT означает "JSON Web Tokens".

Это стандарт, чтобы кодировать объект JSON в длинную, непонятную строку без пробелов. Она может выглядеть так:

> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxw
> RJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Она не зашифрована, поэтому, любой мог бы восстановить информацию из содержимого.

Но она подписана. Поэтому, когда вы получаете токен который вы отправили, можно проверить, что его отправили именно вы.

Таким образом, вы можете создать токен со сроком действия, скажем, неделя. А затем когда пользователь возвращается на 
следующий день с этим токеном, вы знаете, что пользователь все еще залогинен в вашей системе.

По прошествии недели, токен будет просрочен и пользователь не будет авторизован и будет нужно войти снова, чтобы получить
новы токен. И если пользователь (или третье лицо) пытался поменять токен, чтобы изменить срок истечения, вы бы смогли
это заметить, так как подписи уже не совпадали.

Если хотите "поиграть" с JWT токенами и посмотреть как они работают, посмотрите https://jwt.io.

## Установка `python-jose`

Нам нужно установить `python-jose`, чтобы создавать и проверять JWT токены в Python:

```commandline
pip install "python-jose[cryptography]"
```

<a href="https://github.com/mpdavis/python-jose">Python-jose</a> требует криптографический бекенд как дополнительный.

Здесь мы используем рекомендованный: <a href="https://cryptography.io/">pyca/cryptography</a>.

> **Подсказка**
> 
> Эта инструкция ранее использовала <a href="https://pyjwt.readthedocs.io/">PyJWT</a>.
> 
> Но он был изменен для использования Python-jose, так как он предоставляет преимущества из PyJWT плюс некоторые 
> дополнения, которые могут вам понадобиться позже при постройке интеграций с другими инструментами.

## Хеширование пароля

"Хеширование" означает конвертацию какого-то содержимого (пароль в нашем случае) в последовательность байтов (просто 
строка), которая выглядит как бессмыслица.

Каждый раз когда вы передаете одно и то же содержимое (один и тот же пароль) вы получаете одну и ту же "бессмыслицу".

Но вы не можете конвертировать "бессмыслицу" обратно в пароль.

### Зачем использовать хеширование пароля

Если ваша база данных украдена, вор не сможет получить незашифрованные пароли ваших пользователей, только хеши.

Поэтому, вор не сможет использовать эти пароли в других системах (так как многие пользователи используют одинаковые
пароли везде, это может быть опасно).

## Установка `passlib`

Passlib отличный пакет Python, чтобы обрабатывать хеши паролей.

Он поддерживает много безопасных алгоритмов хеширования и утилит для работы с ними.

Рекомендованный алгоритм это "Bcrypt".

Поэтому, установим PassLib с Bcrypt:

```commandline
pip install "passlib[bcrypt]"
```

> **Совет**
> 
> С `passlib` вы можете даже настроить его, чтобы читать пароли, созданные с помощью **Django**, **Flask** или многих других.
> 
> Поэтому, вы могли бы, например, обмениваться одними данными из приложения Django в базу данных приложения FastAPI. Или
> постепенно перенести приложение Django, используя ту же базу данных.
> 
> А ваши пользователи могли бы войти из вашего приложения Django или приложения FastAPI одновременно.

## Хеш и проверка паролей

Импортируйте нужные нам инструменты из `passlib`.

Создайте PassLib "контекст". Это то, что будет использовано для хеширования и проверки паролей.

> **Совет**
> 
> Контекст PassLib также имеет функциональность, чтобы использовать различные алгоритмы хеширования, включая устаревшие, 
> только, чтобы предоставить их проверку, и т.д.
> 
> Например, вы можете использовать его, чтобы прочитать и проверить пароли, сгенерированные другими системами (например
> Django), а хешировать любые новые пароли другим алгоритмом, например, Bcrypt.
> 
> И быть совместимым со всем этим одновременно.

Создайте служебную функцию чтобы хешировать входящий от пользователя пароль.

И еще одну, чтобы проверять, что полученный пароль совпадает с хранящимся хешем.

И еще одну, чтобы аутентифицировать и вернуть пользователя.

```python
from passlib.context import CryptContext
from pydantic import BaseModel


class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None
    
 
class UserInDB(User):
    hashed_password: str
    

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)


def get_password_hash(password):
    pwd_context.hash(password)
    
    
def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)

    
def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user
```

> **Заметка**
> 
> Если вы проверяете новую (ненастоящую) базу данных `fake_users_db`, вы увидите как выглядит хешированый пароль:
> `"$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW"`.

## Обработка JWT токенов

Импортируйте установленные модули.

Создайте рандомный секретный ключ, который будет использован для подписи JWT токенов.

Чтобы сгенерировать секретный ключ, используйте команду:

```commandline
openssl rand -hex 32

09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7
```

И скопируйте вывод в переменную `SECRET_KEY` (не используйте ключ из примера).

Создайте переменную `ALGORITM` с алгоритмом, используемом для подписи JWT токена и установите его на `HS256`.

Создайте переменную для срока действия токена.

Объявите модель Pydantic, которая будет использована в ендпоинте токена для ответа.

Создайте служебную функцию, чтобы генерировать новый токен доступа.

```python
from datetime import timedelta, datetime

from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30


class Token(BaseModel):
    access_token: str
    token_type: str


class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None
    
 
class UserInDB(User):
    hashed_password: str
    

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)


def get_password_hash(password):
    pwd_context.hash(password)
    
    
def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)

    
def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user


def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt
```

## Обновление зависимостей

Обновим `get_current_user`, чтобы получить такой же токен как до этого, но на этот раз, используя JWT токены.

Расшифруем полученный токен, проверим его и вернем текущего пользователя.

Если токен неправильный, сразу вернем ошибку HTTP.

```python
from datetime import timedelta, datetime
from typing import Annotated

from fastapi import Depends, HTTPException, status
from fasrapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30


fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW",
        "disabled": False,
    }
}


class Token(BaseModel):
    access_token: str
    token_type: str
    
    
class TokenData(BaseModel):
    username: str | None = None


class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None
    
 
class UserInDB(User):
    hashed_password: str
    

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)


def get_password_hash(password):
    pwd_context.hash(password)
    
    
def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)

    
def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user


def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt


async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except JWTError:
        raise credentials_exception
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user
```

## Обновление операции пути `/token`

Создадим `timedelta` с временем истечения токена.

Создадим настоящий JWT токен доступа и вернем его

```python
from datetime import timedelta, datetime
from typing import Annotated

from fastapi import Depends, HTTPException, status, FastAPI
from fasrapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30


fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW",
        "disabled": False,
    }
}


class Token(BaseModel):
    access_token: str
    token_type: str
    
    
class TokenData(BaseModel):
    username: str | None = None


class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None
    
 
class UserInDB(User):
    hashed_password: str
    

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

app = FastAPI()

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)


def get_password_hash(password):
    pwd_context.hash(password)
    
    
def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)

    
def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user


def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt


async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except JWTError:
        raise credentials_exception
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user


async def get_current_active_user(
    current_user: Annotated[User, Depends(get_current_user)]
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user


@app.post("/token", response_model=Token)
async def login_for_access_token(
        form_data: Annotated[OAuth2PasswordRequestForm, Depends()]
):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}


@app.get("/users/me/", response_model=User)
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_active_user)]
):
    return current_user


@app.get("/users/me/items/")
async def read_own_items(
    current_user: Annotated[User, Depends(get_current_active_user)]
):
    return [{"item_id": "Foo", "owner": current_user.username}]
```

### Технические детали про JWT "субъект" `sub`

Спецификация JWT говорит, что существует ключ `sub`, с субъектом токена.

Его необязательно использовать, но именно туда вы бы поместили идентификатор пользователя, поэтому здесь мы используем его.

JWT может быть использован для других вещей отдельно от идентификации пользователя, позволяя выполнять операции напрямую
в вашем API.

К примеру, вы можете определить "автомобиль" или "запись в блоге".

Затем вы можете добавлять разрешения, касающиеся этого объекта, например "ехать" (для автомобиля) или "редактировать"
(для блога).

После этого, вы можете передать этот JWT токен пользователю (или боту) и они могут использовать его, чтобы выполнять эти
действия (ехать на автомобиле или редактировать запись блога) даже без необходимости иметь аккаунт, просто с помощью JWT
токена, сгенерированного для это вашим API.

Используя эти идеи, JWT можно использовать для гораздо более сложных сценариев.

В этих случаях, несколько таких сущностей могут иметь одинаковые ID, скажем `foo`(пользователь `foo`, автомобиль `foo`,
запись в блоге `foo`).

Поэтому, чтобы избежать коллизий ID, когда создается JWT токен для пользователя, вы можете добавить префикс значения 
ключа `sub`, например с `username:`. Отсюда, в этом примере, значение `sub` может быть: `username:johndoe`.

Очень важно иметь в виду, что ключ `sub` должен иметь уникальный идентификатор для всего приложения и это должна быть
строка.

## Проверим все это

Запустите сервер и перейдите в документацию: http://127.0.0.1:8000/docs.

Вы увидите интерфейс вроде:

<img src="https://fastapi.tiangolo.com/img/tutorial/security/image07.png">

Авторизируйтесь в приложении так же как и раньше.

Используя данные:

Имя пользователя: 'johndoe'. Пароль: 'secret'

> **Обратите внимание**
> 
> Обратите внимание, что нигде в коде нет незашифрованного пароля "`secret`", у нас только хеш версия.

<img src="https://fastapi.tiangolo.com/img/tutorial/security/image08.png">

Вызовите точку `/users/me/`, вы получите такой ответ:

```python
{
  "username": "johndoe",
  "email": "johndoe@example.com",
  "full_name": "John Doe",
  "disabled": false
}
```

<img src="https://fastapi.tiangolo.com/img/tutorial/security/image09.png">

Если вы откроете панель разработчика, вы сможете увидеть, что отправленные данные, включают в себя только токен, пароль 
отправился только в первом запросе, чтобы аутентифицировать пользователя и получить токен доступа, но не после.

<img src="https://fastapi.tiangolo.com/img/tutorial/security/image10.png">

> **Заметка**
> 
> Обратите внимание, что значение заголовка `Authorization` начинается с `Bearer`.

## Продвинутое использование со `scopes`

OAuth2 имеет понятие "scopes".

Вы можете использовать их, чтобы добавлять определенный набор ограничений в JWT токен.

Затем вы можете передавать этот токен напрямую пользователю или третьей стороне, чтобы взаимодействовать вашим API с 
набором ограничений.

Вы можете узнать как их использовать и как они интегрированы в FastAPI позже в **продвинутом руководстве**.

## Резюме

С тем, что вы видели до сих пор, вы можете устанавливать безопасность приложения FastAPI, используя стандарты OAuth2 и JWT.

Практически в любом фреймворке обработка безопасности довольно быстро становится довольно сложным предметом.

Многим пакетам, которые упрощают это, приходится идти на многочисленные компромиссы с моделью данных, базой данных и
доступными функциями. А некоторые из этих пакетов, которые очень сильно упрощают работу, на самом деле имеют недостатки в
безопасности.

****

FastAPI не идет на какие-либо компромиссы с базой данных, моделью данных или инструментом. 

Он дает вам гибкость выбора всего, что лучше подходит вашему проекту.

И вы можете использовать напрямую множество поддерживаемых и широко используемых пакетов, таких как `passlib` и `python-jose`,
так как FastAPI не требует какие-либо сложные механизмы для интеграции внешних пакетов.

Но он предоставляет вам инструменты, чтобы упростить процесс так сильно, как возможно, не ставя под угрозу гибкость, 
надежность или безопасность.

И вы можете использовать и внедрять стандартные протоколы безопасности, такие как OAuth2, относительно простым способом.

Вы можете узнать больше в **Продвинутом руководстве** о том как использовать OAuth2 "scopes", для более детализированной
системы разрешений, следующей этим же стандартам. OAuth2 с scopes это механизм использующийся большинством крупных
поставщиков аутентификации, таких как Facebook, Google, GitHub, Microsoft, Twitter и т.д, чтобы разрешать сторонним
приложениям взаимодействовать с их API-интерфейсами от имени своих пользователей.