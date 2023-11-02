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

