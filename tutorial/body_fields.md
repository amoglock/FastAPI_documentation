<h3>Тело - Fields</h3>

Точно так же, как вы можете объявить дополнительную проверку и метаданные в параметрах функции операции пути с помощью
`Query`, `Path` и `Body`, вы можете объявить проверку и метаданные внутри моделей Pydantic, используя Pydantic `Field`.

<h3>Импорт `Field`</h3>

Сперва нужно импортировать его:

```python
from typing import Annotated

from fastapi import FastAPI, Body
from pydantic import BaseModel, Field

app = FastAPI()
```

> **!!! Предупреждение**
> 
> Заметьте, что `Field` импортирован напрямую из `pydantic`, а не из `fastapi` как все остальное (`Query`, `Path`, `Body`)
> и т.д.

<h3>Объявление аттрибутов модели</h3>

Затем вы можете использовать `Field` с аттрибутами модели:

```python
from typing import Annotated

from fastapi import FastAPI, Body
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = Field(
        default=None, title="The description of the item", max_length=300
    )
    price: float = Field(gt=0, description="The price must be greater than zero")
    tax: float | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    results = {"item_id": item_id, "item": item}
    return results
```

`Field` работает так же как `Query`, `Path` и `Body`, имеет те же самые параметры, и т.д.

> **Технические детали**
> 
> На самом деле, `Query`, `Path` и другое, что вы увидите дальше, создают объекты подклассов общего класса `Param`,
> который сам по себе подкласс класса `FieldInfo` Pydantic.
> 
> А Pydantic `Field` возвращает сам экземпляр `FieldInfo`.
> 
> `Body` так же возвращает объекты подкласса `FieldInfo` напрямую. А есть другие, которые вы увидите позже, они
> являются подклассами класса `Body`.
> 
> Помните, что когда вы импортируете `Query`, `Path` и другие из `fastapi`, на самом деле это функции которые возвращают
> специальные классы.

> **Заметка**
> 
> Обратите внимание как каждый аттрибут модели с типом, значением по умолчанию и `Field` имеет такую же структуру как 
> параметр функции операции пути, с `Field` вместо `Path`, `Query`, `Body`.

<h3>Добавление дополнительной информации</h3>

Вы можете объявлять дополнительную информацию в `Field`, `Query`, `Body`, и т.д. И она будет включена в сгенерированную
схему JSON.

Вы научитесь больше как добавлять дополнительную информацию позже в документации, когда будете учиться объявлять примеры.

> **!!! Предупреждение**
> 
> Дополнительные ключи, переданные в `Field` будут также присутствовать в результирующей схеме OpenAPI для вашего 
> приложения. Так как эти ключи не обязательно могут быть частью спецификации OpenAPI, некоторые инструменты OpenAPI,
> например <a href="https://validator.swagger.io/">the OpenAPI validator</a>, могут не работать с вашей сгенерированной
> схемой.

<h3>Резюме</3>

Вы можете использовать Pydantic `Field` чтобы объявить дополнительные проверки и метаданные для аттрибутов модели.

Вы также можете использовать дополнительные ключевые аргументы, чтобы передать дополнительные метаданные схемы JSON.