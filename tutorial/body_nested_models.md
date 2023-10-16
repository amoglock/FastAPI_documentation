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

