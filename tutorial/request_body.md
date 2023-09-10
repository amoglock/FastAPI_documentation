<h3>Тело запроса</h3>

Когда вам нужно отправить данные из клиента (допустим, это браузер) в ваш API, вы отправляете его как тело запроса.

Тело запроса это данные, отправленные клиентом в API. Тело ответа это данные, которые API отправляет клиенту.

Ваш API почти всегда должен отправлять тело ответа. Но клиентам не обязательно всегда нужно отправлять тело запроса.

Чтобы объявить тело запроса, используйте модели Pydantic со всеми их возможностями и преимуществами.

> *Для информации*
> 
> Чтобы отправить данные, вам нужно использовать один из запросов: `POST` (самый используемый), `PUT`, 'DELETE' или `PATCH`.
> 
> Отправка тела `GET` запросом, имеет неопределенное поведение в спецификации, однако, поддерживается FastAPI, только для 
> очень сложных/"экстремальных" случаев применения.
> 
> Как бы не было печально, интерактивная документация Swagger UI не покажет документацию для тела в `GET` запросе, а также
> прокси могут не поддерживать его.

<h3>Импорт `BaseModel` из Pydantic</h3>

Во-первых, нужно импортировать `BaseModel` из `pydantic`:

```python
from fastapi import FastAPI
from pydantic import BaseModel
```

<h3>Создание модели данных</h3>

Затем, объявите модель данных как класс, унаследованный от `BaseModel`.

Используйте стандартные типы данных Python для всех атрибутов:

```python
from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
```

Точно так же как и объявляются параметры запроса, когда аттрибут модели имеет значение по умолчанию, он не обязателен.
В обратном случае - он требуется. Используйте `None` чтобы сделать его опциональным.

Например, модель выше объявляет "`объект`" JSON (или Python `dict`) типа такого:

```JSON
{
    "name": "Foo",
    "description": "An optional description",
    "price": 45.2,
    "tax": 3.5
}
```

...так как `description` и `tax` опциональные (со значением по умолчанию `None`), этот "`объект`" JSON может быть и таким:

```JSON
{
    "name": "Foo",
    "price": 45.2
}
```

<h3>Объявление модели как параметр</h3>

Чтобы добавить модель как параметр операции, объявите ее так же как объявляли параметры пути и запроса:

```python
from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    return item
```

...и объявите его тип данных как созданную модель `Item`.

<h3>Результаты</h3>

Просто объявив типы данных Python, FastAPI будет:
* Читать тело запроса как JSON.
* Конвертировать соответствующие типы данных (если необходимо).
* Валидировать данные.
    * Если данные неверные, вернет понятное сообщение ошибки, прямо указывающее, где были некорректные данные.
* Предоставляет полученные данные в параметре `item`:
    * Так как его объявляли в функции типом `Item`, у вас будет поддержка редактора (автозаполнение, и т.д) для всех
    аттрибутов и их типов.
* Генерировать схему JSON, объявленную для вашей модели, вы можете использовать ее везде в проекте если это имеет смысл.
* Эти схемы будут частью сгенерированной схемы OpenAPI и использованы автоматической документацией.

<h3>Автоматическая документация</h3>

JSON схемы ваших моделей будут частью сгенерированной схемы OpenAPI и будут показаны в интерактивной документации API:

<img src="https://fastapi.tiangolo.com/img/tutorial/body/image01.png" width="530" height="390">

И будут также использованы в документации API внутри каждой операции пути которые требуются:

<img src="https://fastapi.tiangolo.com/img/tutorial/body/image02.png" width="530" height="460">

<h3>Поддержка редактора</h3>

В редакторе, внутри функции вы получите подсказки типов и автозаполнение, везде (такого бы не было, если объявили тип 
`dict` вместо модели Pydantic):

<img src="https://fastapi.tiangolo.com/img/tutorial/body/image03.png" width="530" height="240">

Вы так же получаете сообщения ошибок для неправильных операций с типами:

<img src="https://fastapi.tiangolo.com/img/tutorial/body/image04.png" width="530" height="250">

Это все не просто так, весь фреймворк был построен вокруг этого дизайна.

И он был тщательно протестирован на этапе проектирования, чтобы убедиться, что все будет работать во всех редакторах.

Были даже внесены некоторые изменения в Pydantic, чтобы это поддерживать.

Предыдущие скриншоты были взяты из Visual Studio Code.

Но вы можете получить такую же поддержку редактора Pycharm и большинства других редакторов Python:

<img src="https://fastapi.tiangolo.com/img/tutorial/body/image05.png" width="530" height="250">

> **Подсказка**
> Если вы используете PyCharm, то можно использовать Pydantic Pycharm Plugin.
> 
> Он предоставляет поддержку редактора для моделей Pydantic с:
> * Автозаполнением
> * Проверкой типов
> * Рефакторинг
> * Поиском
> * Проверками

<h3>Использование модели</h3>

Внутри функции, вы можете получить все аттрибуты объекта модели напрямую:

```python
from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    item_dict = item.dict()
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```

<h3>Тело запроса + параметры пути</h3>

Вы можете объявить параметры пути и параметры запроса одновременно.

FastAPI будет понимать, что параметры функции, которые совпадают с параметрами пути, следует брать из пути. А
параметры функции, которые объявлены как модель Pydantic, следует брать из тела запроса.

```python
from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


app = FastAPI()


@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.dict()}
```

<h3>Тело запроса + путь + параметры запроса</h3>

Вы также можете объявить *тело*, *путь* и *параметры запроса* одновременно.

FastAPI будет понимать каждый из них и брать данные из правильного места.

```python
from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


app = FastAPI()


@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item, q: str | None = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result
```

Параметры этой функции будут определены следующим образом:

* Если параметр объявлен в пути, он будет использован как параметр пути.
* Если параметр единичный тип данных (как `int`, `float`, `str`. `bool` и т.д.), он будет определен как параметр запроса.
* Если параметр объявлен как модель Pydantic, он будет определен как тело запроса.

> **Заметка**
> 
> FastAPI будет знать, что значение `q` необязательное, потому, что значение по умолчанию установлено как `=None`.
> 
> `Union` в `Union[str, None]` не использовано в FastAPI, но позволит вашему редактору кода предоставить лучшую поддержку
> и выявление ошибок.

<h3>Без Pydantic</h3>

Если вы не хотите использовать Pydantic модели, вы можете использовать параметры `Body`. Эту документацию можно посмотреть
далее.

