<h3>Объявление примера запроса данных</h3>

Вы можете объявлять примеры данных которые принимает ваше приложение.

Есть несколько способов сделать это.

<h3>Дополнительная схема данных JSON в модели Pydantic</h3>

Вы можете объявлять `examples` для модели Pydantic которые будут добавлены в сгенерированную схему JSON.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

    model_config = {
        "json_schema_extra": {
            "examples": [
                {
                    "name": "Foo",
                    "description": "A very nice Item",
                    "price": 35.4,
                    "tax": 3.2,
                }
            ]
        }
    }

...
```

Эта дополнительная информация будет добавлена как есть в вывод схемы JSON для этой модели и будет использована в
документации API.

В Pydantic версии 2, вы бы использовали аттрибут `model_config`, который принимает `dict` как описано в
<a href="https://docs.pydantic.dev/latest/usage/model_config/">Pydantic`s docs: Model Config</a>.

Вы можете устанавливать `json_schema_extra` с `dict`, содержащим любые дополнительные данные которые вы бы хотели показать
в сгенерированной схеме JSON, включая `examples`.

> **Совет**
> 
> Вы могли бы использовать такую же технику, чтобы расширить схему JSON и добавить вашу собственную дополнительную информацию.
> Например, вы могли бы использовать ее, чтобы добавить метаданные для пользовательского интерфейса и т.д.

> **Для информации**
> 
> OpenAPI 3.1.0 (используется с FastAPI 0.99.0) добавило поддержку для `examples`, которые являются частью стандарта 
> схемы JSON.
> 
> До этого, оно только поддерживало ключевое слово `example` с единичным примером. Так все еще поддерживается OpenAPI 3.1.0
>, но является устаревшим и не частью стандарта схемы JSON. По этому вам рекомендуется перенести `example` в `examples`. 🤓 
> 
> Вы можете прочитать об этом больше в конце этой страницы.

<h3>Дополнительные аргументы `Field`</h3>

При использовании `Field()` с моделями Pydantic, вы можете также объявить добавочные `examples`:

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str = Field(examples=["Foo"])
    description: str | None = Field(default=None, examples=["A very nice Item"])
    price: float | None = Field(examples=[35.4])
    tax: float | None = Field(default=None, examples=[3.2])

...
```


