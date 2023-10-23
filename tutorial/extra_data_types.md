<h3>Дополнительные типы данных</h3>

До сих пор, вы использовали простые типы данных, как:

* `int`
* `foat`
* `str`
* `bool`

Но вы можете также использовать более сложные типы данных.

И вы все еще будете иметь такие же возможности какие видели до этого:

* Отличная поддержка редактора.
* Конвертация данных из входящих запросов.
* Конвертация данных для исходящих запросов.
* Проверка данных.
* Автоматическая аннотация и документация.

<h3>Другие типы данных</h3>

Существует несколько дополнительных типов данных которые вы можете использовать:

* `UUID`:
  * Стандарт "Универсальный Уникальный Идентификатор", часто просто ID в большинстве баз данных и системах.
  * В запросах и ответах будет представлен как `str`.
* `datetime.datetime`:
  * Python `datetime.datetime`.
  * В запросах и ответах будет представлен как `str` в формате ISO 8601, например: `2008-09-15T15:53:00+05:00`.
* `datetime.date`:
  *  Python `datetime.date`.
  * В запросах и ответах будет представлен как `str` в формате ISO 8601, например: `2008-09-15`.
* `datetime.time`:
  *  Python `datetime.time`.
  * В запросах и ответах будет представлен как `str` в формате ISO 8601, например: `14:23:55.003`.
* `datetime.timedelta`:
  *  Python `datetime.timedelta`.
  * В запросах и ответах будет представлен как `float` общих секунд.
  * Pydantic также предоставляет его представление как "ISO 8601 кодирование с разницей во времени", 
<a href="https://pydantic-docs.helpmanual.io/usage/exporting_models/#json_encoders">смотри документацию для информации</a>.
* `frozenset`:
  * В запросах и ответах рассматривается так же как `set`:
    * В запросах список будет прочитан, убраны дубликаты и конвертирован в `set`.
    * В ответах `set` будет конвертирован в `list`.
    * Сгенерированная схема будет указывать, что значения `set` уникальные (используя `uniqueItems` схемы JSON).
* `bytes`:
  * Стандартный Python `bytes`.
  * В запросах и ответах будет представлен как `str`.
  * Сгенерированная схема будет указывать, что это `str` в `binary` "формате".
* `Decimal`:
  * Стандартный Python `Decimal`.
  * В запросах и ответах будет обработан так же как `float`.
* Вы можете поверять все подходящие типы данных здесь: 
<a href="https://docs.pydantic.dev/latest/usage/types/types/">Pydantic data types</a>.

<h3>Пример</h3>

Вот пример операции пути с параметрами использующих несколько из типов выше:

```python
from datetime import  datetime, time, timedelta
from typing import Annotated
from uuid import UUID

from fastapi import FastAPI, Body

app = FastAPI()

@app.put("items/{item_id}/")
async def read_items(
        item_id: UUID,
        start_datetime: Annotated[datetime | None, Body()] = None,
        end_datetime: Annotated[datetime | None, Body()] = None,
        repeat_at: Annotated[time | None, Body()] = None,
        process_after: Annotated[timedelta | None, Body()] = None,
):
    start_process = start_datetime + process_after
    duration = end_datetime - start_process
    return {
      "item_id": item_id,
      "start_datetime": start_datetime,
      "end_datetime": end_datetime,
      "repeat_at": repeat_at,
      "process_after": process_after,
      "start_process": start_process,
      "duration": duration,
    }
```

Обратите внимание, что параметры внутри функции имеют их естественные типы данных и вы можете, например, проводить
обычные манипуляции с датой, например:

```python
    start_process = start_datetime + process_after
    duration = end_datetime - start_process
```

