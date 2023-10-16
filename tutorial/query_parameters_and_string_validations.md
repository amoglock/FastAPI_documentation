<h3>Параметры запроса и валидация строк</h3>

FastAPI позволяет объявлять дополнительную информацию и валидацию для параметров.

Возьмем, в качестве примера, такое приложение:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/")
async def read_items(q: str | None = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

Параметр запроса `q` это тип `Union[str, None]`(или `str | None` в Python 3.10), что означает, что его тип - `str`, но 
так же может быть `None`, а фактически, значение по умолчанию это `None`, так FastAPI будет знать, что параметр
необязателен.

> **Заметка**
> 
> FastApi будет знать что значение `q` необязательно, потому, что значение по умолчанию `None`.
> 
> `Union` в `Union[str, None]` предоставит редактору возможность оказывать лучшую поддержку и выявление ошибок.

<h3>Добавочная проверка</h3>

Мы собираемся обеспечить, что несмотря на то, что `q` опционален, но всегда когда он есть, его длина не превышает 50 символов.

<h4>Импорт `Query` и `Annotated`</h4>

Чтобы достичь этого, сперва импортируем:
* `Query` из `FastAPI`
* `Annotated` из `typing` (или из`typing_extensions` в Python ниже версии 3.9)

В Python 3.9 или выше, `Annotated` это часть стандартной библиотеки, поэтому вы можете импортировать его из `typing`.

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()
```

> **Для информации**
> 
> В FastAPI добавлена поддержка `Annotated` (и стала рекомендованной) в версии 0.95.0.
> 
> Если у вас более старая версия, можете получать ошибки, когда попытаетесь использовать `Annotated`.
> 
> Убедитесь, что вы обновили версию FastAPI по крайней мере на 0.95.1 перед использованием `Annotated`.

<h3>Использование `Annotated` в типе для параметра `q`</h3>

`Annotated` может быть использован, чтобы добавить метаданные вашим параметрам.

Сейчас самое время использовать это в FastAPI.

У нас есть такая аннотация типов:

```python
q: str | None = None
```

Что мы сделаем, так это обернем ее в `Annotated` и она станет такой:

```python
q: Annotated[str | None] = None
```

Обе эти версии означают одно и то же, `q` это параметр, который может быть `str` или `None`, и по умолчанию он `None`.

А теперь перейдем к самому интересному.

<h3>Добавление `Query` к `Annotated` в параметр `q`</h3>

Теперь, когда у нас есть `Annotated`, где мы можем разместить больше метаданных, добавьте к нему `Query`, и установите 
параметр `max_length` на 50:

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(max_length=50)] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

Заметьте, что значение по умолчанию все еще `None`, поэтому параметр остается необязательным.

Но теперь, когда есть `Query(max_lenght=50)` внутри `Annotated`, мы сообщаем FastAPI, что хотим извлечь значение из 
параметров запроса (в любом случае было бы по умолчанию) и нужно иметь дополнительную валидацию этого значения (вот 
зачем мы это делаем, чтобы получить дополнительную проверку).

FastAPI теперь будет:
* Проверять данные, убедившись, что максимальная длина 50 символов
* Показывать понятную ошибку для клиента когда данные неверные
* Документировать параметр в OpenAPI схеме операции пути (поэтому будет показываться в автоматической документации)

Альтернативный (старый) `Query` как значение по умолчанию

Предыдущие версии FastAPI(до 0.95.0) обязывали использовать `Query` как значение по умолчанию для вашего параметра,
вместо размещения его в `Annotated`, поэтому велики шансы, что вы увидите код который использует это. Поэтому разберемся.

> **Совет**
> 
> Для нового кода и везде где возможно, используйте `Annotated`, как показано выше. Там больше преимуществ (показано ниже)
> и нет недостатков.

Вот как вы могли бы использовать `Query()` как значение по умолчанию в параметре вашей функции, установив параметр 
`max_lenght` на 50:

```python
from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: str | None = Query(default=None, max_length=50)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

Так как в этом случае (не используя `Annotated`) мы должны заменить значение по умолчанию `None` в функции с `Query()`,
нам нужно установить значение с параметром `Query(default=None)`, он служит той же цели, чтобы определить значение
по умолчанию (по крайней мере в FastAPI).

Поэтому...

```python
q: Union[str, None] = Query(default=None)
```

...делает параметр необязательным, со значением None по умолчанию, так же как и:

```python
q: Union[str, None] = None
```

А в Python 3.10 и выше:

```python
q: str | None = Query(default=None)
```

...делает параметр необязательным, со значением None по умолчанию, так же как и:

```python
q: str | None = None
```

Но он явно объявляет это как параметр запроса.

> **Для информации**
> 
> Имейте в виду, что самая важная часть, чтобы сделать параметр необязательным это:
> > `= None`
> 
> или
> 
> >`Query(default=None)`
> 
> Так это будет использовать None, как значение по умолчанию, и этот способ делает параметр необязательным.
> 
> Часть `Union[str, None]` позволяет редактору предоставить лучшую поддержку, но это не то, что говорит FastAPI, что 
> этот параметр необязателен.

Затем мы можем передать больше параметров в `Query`. В нашем случае это параметр `max_length` который применяется к
строкам:

```python
q: Union[str, None] = Query(default=None, max_length=50)
```

Будут проверяться данные, показывать понятную ошибку когда данные неправильные и документировать параметр в схему 
операции пути OpenAPI.

<h4>`Query` по умолчанию или в `Annotated`</h4>

Имейте в виду, что когда используется `Query` внутри `Annotated` вы не можете использовать параметр `default` для `Query`.

Вместо этого используйте значение по умолчанию в параметре функции. Иначе могут быть противоречия.

Например, такое не допускается:

```python
q: Annotated[str, Query(default="rick")] = "morty"
```

...потому что неясно значение по умолчанию должно быть `"rick"` или `"morty"`.

Поэтому следовало бы использовать (предпочтительно):

```python
q: Annotated[str, Query()] = "rick"
```

...или в более старом коде вы найдете:

```python
q: str = Query(default="rick")
```

<h3>Преимущества `Annotated`</h3>

Использование `Annotated` рекомендовано вместо значения по умолчанию в параметрах функции, это лучше по многим причинам.

Значение по умолчанию параметра функции это фактическое значение, которое в целом более интуитивно понятно для Python.

Вы можете вызвать ту же функцию в других местах без FasAPI, и она работала бы как ожидалось. Если есть обязательный
параметр (без значения по умолчанию), ваш редактор сообщит об ошибке, Python также пожалуется если вы запустите ее не 
передав требуемый параметр.

Когда вы не используете `Annotated`, а вместо этого используете (старый) стиль значения по умолчанию, если вы вызываете
эту функцию без FastAPI в других местах, нужно не забыть передать аргументы в функцию чтобы она работала корректно, иначе
значения будут отличаться от ожидаемых (например, информация о запросе или что-то подобное вместо str). Редактор и Python 
не будут жаловаться запуская эту функцию, только когда операции внутри завершатся ошибкой.

Поскольку аннотированный может содержать более одной аннотации к метаданным, теперь вы можете использовать ту же функцию
даже с другими инструментами, такими как Typer. 🚀

<h3>Добавим больше проверок</h3>

Вы можете добавить параметр `min_length`:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Annotated[str | None, Query(min_length=3, max_length=50)] = None
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

<h3>Добавим регулярные выражения</h3>

Вы можете объявить `pattern` регулярного выражения, это значит параметр должен совпадать:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Annotated[
        str | None, Query(min_length=3, max_length=50, pattern="^fixedquery$")
    ] = None
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

Конкретно этот паттерн регулярного выражения проверяет, что значение переданного параметра:
* `^`: начинается с символов, которые идут после этого знака, не имеет символов до этого.
* `fixedquery`: имеет точное значение `fixedquery`.
* `$`: заканчивается здесь, не имеет символов после `fixedquery`.

Если exact вы чувствуете себя потерянным со всеми этими "регулярными выражениями", не волнуйтесь. Это сложная тема для
многих людей. Вы можете делать многое, без необходимости использовать регулярные выражения.

Но всякий раз когда они понадобятся и отправитесь их изучать, знайте что можно использовать их напрямую в FastAPI.

<h3>Pydantic V1 `regex` вместо `pattern`</h3>

До Pydantic версии 2 и FastAPI 0.100.0, параметр назывался `regex` вместо `pattern`, но теперь это устарело.

Вы можете все еще видеть код, использующий это:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Annotated[
        str | None, Query(min_length=3, max_length=50, regex="^fixedquery$")
    ] = None
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

Но знайте, что это устарело и следует быть обновлено, чтобы использовать новый параметр `pattern`. 🤓

<h3>Значения по умолчанию</h3

Конечно, можно использовать значения по умолчанию отличные от `None`.

Скажем, что вы хотите объявить параметр запроса `q`, использовать `min_length` равным `3`, и иметь значение по 
умолчанию `fixedquery`:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[str, Query(min_length=3)] = "fixedquery"):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

> **Заметка**
> 
> Наличие значения по умолчанию любого типа, включая `None`, делает параметр опциональным (необязательным).

<h3>Делаем его обязательным</h3>

Когда нам не нужно объявлять много проверок или метаданных, можно делать параметр запроса `q` обязательным просто не
указывая значение по умолчанию, например:

```python
q: str
```

вместо:

```python
q: Union[str, None] = None
```

А сейчас объявляем его с `Query`, например такое:

```python
q: Annotated[Union[str, None], Query(min_length=3)] = None
```

Итак, когда вам нужно объявить значение обязательным когда используем `Query`, вы можете просто не объявлять значение по
умолчанию:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[str, Query(min_length=3)]):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

<h3>Обязательный параметр с Ellipsis `(...)`</h3>

Существует альтернативный способ чтобы явно объявлять, что значение обязательно. Вы можете установить значение по
умолчанию с помощью значения `...`:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[str, Query(min_length=3)] = ...):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

> **Для информации**
> 
> Если вы не видели `...` ранее: это специальное значение, это элемент Python, называемый Ellipsis.
> 
> Он использовался Pydantic и FastAPI чтобы явно объявлять, что значение обязательно.


Это будет указывать FastAPI что этот параметр обязателен.

<h3>Обязательный параметр с `None`</h3>

Вы можете объявить, что параметр может принимать значение None, но все еще оставаться обязательным. Это могло бы
заставлять клиентов отправлять значение, даже если значение `None`.

Чтобы это сделать, вы можете объявить, что `None` это валидный тип данных, но использовать `...` как значение по умолчанию:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(min_length=3)] = ...):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

> **Совет**
> 
> Pydantic, обеспечивающий всю проверку данных и сериализацию в FastAPI, имеет особое поведение когда вы используете  
> `Optional` или `Union[Something, None]` без значения по умолчанию. Вы можете прочитать об этом больше в документации
> Pydantic в разделе <a href="https://pydantic-docs.helpmanual.io/usage/models/#required-optional-fields">Required Optional fields</a>


<h3>Использование Pydantic `Required` вместо Ellipsis `(...)`</h3>

Если вам некомфортно использовать `...`, вы можете импортировать и использовать `Required` из Pydantic:

```python
from typing import Annotated

from fastapi import FastAPI, Query
from pydantic import Required

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[str, Query(min_length=3)] = Required):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

> **Совет**
> 
> Помните, что в большинстве случаев, когда что-то обязательно, вы можете просто игнорировать значение по умолчанию,
> поэтому вам обычно не нужно использовать ни `...`, ни `Required`.

<h3>Список/несколько значений параметра запроса</h3>

Когда вы явно объявляете параметр запроса с `Query`, вы можете также объявить ему принимать список значений, или в другом
случае, принимать несколько значений.

Например, объявить параметр запроса `q`, который может появляться несколько раз в URL, вы можете написать:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[list[str] | None, Query()] = None):
    query_items = {"q": q}
    return query_items
```

Тогда, вот в таком URL:

`http://localhost:8000/items/?q=foo&q=bar`

вы можете принять несколько значений параметров запроса `q` (`foo` и `bar`) в Python `list` внутри вашей функции
операции пути, в параметре функции `q`.

Поэтому, ответ от этого URL, может быть таким:

```JSON
{
  "q": [
    "foo",
    "bar"
  ]
}
```

> **Совет**
> 
> Чтобы объявить параметр запроса с типом `list`, как в примере выше, нужно явно использовать `Query`,
> в противном случае это было бы истолковано как тело запроса.

Документация API будет обновлена соответственно, чтобы предоставить несколько значений:

<img src="https://fastapi.tiangolo.com/img/tutorial/query-params-str-validations/image02.png" width="430" height="450">

<h4>Список/несколько значений параметра запроса со значениями по умолчанию</h4>

И вы также можете объявить список по умолчанию, если значения не предоставлены:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[list[str], Query()] = ["foo", "bar"]):
    query_items = {"q": q}
    return query_items
```

Если вы перейдете по:

`http://localhost:8000/items/`

значения `q` по умолчанию будут: `["foo", "bar"]`, а ответ будет:

```JSON
{
  "q": [
    "foo",
    "bar"
  ]
}
```

<h4>Использование `list`</h4>

Вы можете еще использовать `list` напрямую, вместо `List[str]` (или `list[str]` в Python 3.9+)

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[list, Query()] = []):
    query_items = {"q": q}
    return query_items
```

> **Заметка**
> 
> Имейте в виду, что в данном случае, FastAPI не проверяет содержимое списка.
> 
> Например, `List[int]` мог бы проверять (и документировать), что содержимое списка это числа. Но просто `list` не будет.

<h3>Объявление еще больше метаданных</h3>

Вы можете добавить больше информации о параметре.

Эта информация будет включена в сгенерированное OpenAPI и использоваться документацией интерфейса и внешними инструментами.

> **Заметка**
> 
> Имейте в виду, что различные инструменты имеют разные уровни поддержи OpenAPI.
> 
> Некоторые из них могут не показывать всю дополнительную информацию, хотя в большинстве случаев, пропущенная функция уже
> запланирована для разработки.


Вы можете добавить `title`

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Annotated[str | None, Query(title="Query string", min_length=3)] = None
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

И `description`:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Annotated[
        str | None,
        Query(
            title="Query string",
            description="Query string for the items to search in the database that have a good match",
            min_length=3,
        ),
    ] = None
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

<h3>Псевдоним параметров</h3>

Представьте, что вы хотите, чтобы параметр был `item-query`.

Вот такой:

`http://127.0.0.1:8000/items/?item-query=foobaritems`

Но `item-query` это невалидное имя переменной Python.

Максимально близкое название могло быть `item_query`.

Но вам все равно нужно чтобы оно было именно `item-query`...

Тогда вы можете объявить `alias` и этот псевдоним будет использоваться для поиска значений параметра:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(alias="item-query")] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

<h3>Устаревающие параметры</h3>

Теперь давайте предположим, что вам больше не нравится параметр. 

Вы должны оставить его на какое-то время, потому что его используют пользователи. Но вы хотите понятно показать в
документации, что этот параметр устаревает (рекомендуется не использовать его).

Тогда передайте параметр `deprecated=True` в `Query`:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Annotated[
        str | None,
        Query(
            alias="item-query",
            title="Query string",
            description="Query string for the items to search in the database that have a good match",
            min_length=3,
            max_length=50,
            pattern="^fixedquery$",
            deprecated=True,
        ),
    ] = None
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

Документация покажет что-то вроде такого:

<img src="https://fastapi.tiangolo.com/img/tutorial/query-params-str-validations/image01.png" width="430" height="370">

<h3>Исключение из OpenAPI</h3>

Чтобы исключить параметр запроса из сгенерированной схемы OpenAPI (и таким образом, из систем автоматической документации),
установите параметр `include_in_schema` в `Query` на `False`:

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    hidden_query: Annotated[str | None, Query(include_in_schema=False)] = None
):
    if hidden_query:
        return {"hidden_query": hidden_query}
    else:
        return {"hidden_query": "Not found"}
```

<h3>Резюме</h3>

Вы можете объявить дополнительные проверки и метаданные для ваших параметров.

Общие проверки и метаданные:

* `alias`
* `title`
* `description`
* `deprecated`

Определенные проверки для строк:

* `min_length`
* `max_length`
* `pattern`

В этих примерах вы увидели как объявлять проверки для значений `str`.

Посмотрим следующие главы, чтобы увидеть как объявить проверки для других типов данных, например чисел.