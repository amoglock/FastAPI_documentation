<h3>Тело - вложенные модели</h3>

С помощью FastAPI вы можете объявлять, проверять, документировать и использовать сколь угодно глубоко вложенные модели (
благодаря Pydantic).

<h3>Поля списком</h3>

Вы можете объявлять аттрибут чтобы он был подтипом. Например, Python `list`:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str  
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list = []

...
```

Это сделает `tags` списком, хотя не объявляет тип элементов списка.

<h3>Поля списком с параметром типа</h3>

Но Python имеет определенный способ объявить списки с внутренними типами, или "параметрами типа".

<h4>Импорт `List` из typing</h4>

В Python 3.9 и выше вы можете использовать стандартный `list` чтобы объявить эти аннотации типов, как мы увидим ниже. 💡

Но в Python версии до 3.9 (3.6 и выше), сперва вам нужно импортировать `List` из стандартного модуля `typing`:

```python
from typing import List, Union
...
```

<h4>Объявление `list` с параметром типа</h4>

Чтобы объявить типы, которые имеют типы параметров (внутренние типы), вроде `list`, `dict`, `tuple`:
* Если у вас Python версии ниже 3.9, импортируйте их эквивалентные версии из модуля`typing`
* Передайте внутренний тип (типы) как "параметры типа", используя квадратные скобки:`[` и`]`

В Python 3.9 это могло бы быть так:

```python
my_list: list[str]
```

В версиях Python до 3.9, так:

```python
from typing import List

my_list: List[str]
```

Все это стандартный синтаксис Python для объявления типов.

Используйте такой же стандартный синтаксис для аттрибутов модели с встроенными типами.

Отсюда, в нашем примере, мы можем делать `tags` конкретно "списком строк":

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

...
```

<h3>Набор типов</h3>

Но затем мы думаем об этом и понимаем, что теги не должны повторяться, они бы, вероятно, были уникальными строками.

И Python имеет специальный тип данных для наборов уникальных элементов, `set`.

Затем мы можем объявить `tags` как набор строк:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()

...
```

Вот так, даже если вы получаете запрос с дублирующимися данными, они будут конвертированы в набор уникальных элементов.

И всякий раз когда вы выводите эти данные, даже если источник имел дубликаты, они будут выведены как набор уникальных
значений.

И это будет тоже соответственно прокомментировано / документировано.

<h3>Вложенные модели</h3>

Каждый аттрибут модели Pydantic имеет тип.

Но этот тип может сам быть другой моделью Pydantic.

Поэтому, вы можете объявлять глубоко вложенные JSON "объекты" с определенными именами аттрибутов, типов и проверок.

Все это сколь угодно глубоко вложено.

<h4>Определение подмодели</h4>

Например, мы можем объявить модель `Image`:

```python
from fastapi import FastAPI
from pydantic import BaseModel

class Image(BaseModel):
    url: str
    name: str
    
...
```

<h4>Использование подмодели как тип</h4>

И затем мы можем использовать ее как тип аттрибута:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Image(BaseModel):
    url: str
    name: str
    
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()
    image: Image | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

Это бы означало, что FastAPI ожидала бы тело подобное такому:

```JSON
{
  "name": "Foo",
  "description": "The pretender",
  "price": 42.0,
  "tax": 3.2,
  "tags": ["rock", "metal", "bar"],
  "image": {
        "url": "http://example.com/baz.jpg",
        "name": "The Foo live" 
  }
}
```

Еще раз, просто делая такое объявление с FastAPI вы получаете:

* Поддержку редактора (автозавершение и т.п.), даже для вложенных моделей
* Преобразование данных
* Проверку данных
* Автоматическую документацию

<h3>Специальные типы и проверка</h3>

Кроме обычных единичных типов как `str`, `int`, `float`, и т.д. Вы можете использовать более сложные единичные типы, 
которые наследуются от `str`.

Вы можете увидеть все варианты, проверив документацию для <a href="https://pydantic-docs.helpmanual.io/usage/types/">Pydantic`s exotic types</a>.
Некоторые примеры вы увидите в следующей главе.

Например, поскольку в модели `Image` у нас есть поле `url`, мы можем объявить чтобы оно было `HttpUrl` Pydantic, вместо `str`:

```python
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str
...
```

Строка будет проверяться на соответствие URL, и документирована, как таковая, в JSON Schema / OpenAPI.

<h3>Аттрибуты со списками подмоделей</h3>

Вы также можете использовать модели Pydantic как подтипы `list`, `set`, и т.д.

```python
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str

class Item(BaseModel):
    name: str   
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()
    images: list[Image] | None = None

...
```

Это будет ожидать (конвертация, проверка, документирование и т.д.) тело JSON, например:

```JSON
{
  "name": "Foo",
  "description": "The pretender",
  "price": 42.0,
  "tax": 3.2,
  "tags": [
    "rock",
    "metal",
    "bar"
  ],
  "images": [
    {
      "url": "http://example.com/baz.jpg", 
      "name": "The Foo live"
    },
    {
      "url": "http://example.com/dave.jpg", 
      "name": "The Baz"
    }
  ]
}
```

> **Для информации**
> 
> Обратите внимание, как ключ `images` теперь имеет список объектов Image.

<h3>Глубоко вложенные модели</h3>

Вы можете объявлять сколь угодно глубоко вложенные модели:

```python
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str

class Item(BaseModel):
    name: str   
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()
    images: list[Image] | None = None

class Offer(BaseModel):
    name: str
    description: str | None = None
    price: float
    items: list[Item]

@app.post("/offers/")
async def create_offer(offer: Offer):
    return offer
```

> **Для информации**
> 
> Обратите внимание, что `Offer` имеет список `Item`, который, в свою очередь, имеет дополнительный список `Image`

<h3>Тела чистых списков</h3>

Если значение ожидаемого вами верхнего уровня тела JSON это `array` JSON (Python `list`), вы можете объявлять тип в
параметре функции, так же как модели Pydantic:

```python
images: List[Image]
```

или для Python 3.9 и выше:

```python
images: list[Image]
```

как здесь:

```python
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str

@app.post("/images/multiple/")
async def create_multiple_images(images: list[Image]):
    return images
```

<h3>Поддержка редактора везде</h3>

И вы везде получаете поддержку редактора.

Даже для элементов внутри списков:

<img src="https://fastapi.tiangolo.com/img/tutorial/body-nested-models/image01.png" width="460" height="215">

Вы не смогли бы получить такой вид поддержки редактора если бы работали напрямую с `dict` вместо моделей Pydantic.

Но вам тоже не нужно беспокоиться о них, входящие словари конвертированы автоматически и ваш вывод так же конвертирован
автоматически в JSON.

<h3>Тела из `dict`</h3>

Вы так же можете объявлять тело как `dict` с ключами определенного типа и значениями другого типа.

Без необходимости знать заранее допустимые имена поля/аттрибута (как это было бы в случае с моделями Pydantic).

Это может быть полезным если вы хотите получить ключи которые вы еще не знаете.

***

Другой полезный случай это когда вам нужно иметь ключи другого типа, например `int`.

Вот что мы собираемся здесь увидеть.

В этом случает, вы бы принимали любой `dict` если в нем есть ключи `int` со значениями `float`:

```python
from fastapi import FastAPI

app = FastAPI()

@app.post("/index-weights/")
async def create_index_weights(weights: dict[int, float]):
    return weights
```

> **Совет**
> 
> Имейте в виду, что JSON поддерживает ключи только как `str`.
> 
> Но Pydantic автоматически конвертирует данные.
> 
> Это значит, что даже если клиенты вашего API могут отправлять только строки в качестве ключей, до тех пор пока эти
> строки содержат чистые числа, Pydantic будет конвертировать и проверять их.
> 
> И `dict` который вы получаете как `weights`, будет иметь ключи `int` и значения `float`.

<h3>Резюме</h3>

С помощью FastAPI у вас есть максимально гибкие возможности, предоставленные моделями Pydantic, сохраняя ваш код простым,
коротким и элегантным.

Но со всеми преимуществами:

* Поддержка редактора (везде!)
* Конверсия данных (парсинг/сериализация)
* Проверка данных
* Документирование схемы
* Автоматическая документация