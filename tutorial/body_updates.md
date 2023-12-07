<h3>Тело - обновления</h3>

<h4>Обновление заменой с `PUT`</h4>

Чтобы обновить элемент вы можете использовать операцию 
<a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT">HTTP `PUT`</a>.

Вы можете использовать `jsonable_encoder` чтобы конвертировать входные данные, в данные, которые могут храниться как JSON   
(например в NoSQL базе данных). Например, конвертируя `datetime` в `str`.

```python
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str | None = None
    description: str | None = None
    price: float | None = None
    tax: float = 10.5
    tags: list[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]

@app.put("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    update_item_encoded = jsonable_encoder(item)
    items[item_id] = update_item_encoded
    return update_item_encoded
```

`PUT` используется чтобы получить данные, которые должны заменить существующие.

<h5>Предупреждение по поводу замены</h5>

Это означает, что если вы хотите обновить элемент `bar`, используя `PUT`, с телом содержащим:

```JSON
{
  "name": "Barz",
  "price": 3,
  "description": None,
}
```

Так как оно не включает уже хранящийся аттрибут `"tax": 20.2`, входная модель возьмет значение по умолчанию `"tax": 10.5`.

А данные будут сохранены с этим "новым" значением `tax`.

