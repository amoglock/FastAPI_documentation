# Статические файлы

Вы можете обслуживать статические файлы автоматически из директории, использующей `StaticFiles`.

## Используйте `StaticFiles`

* Импортируйте `StaticFiles`.
* "Смонтируйте" экземпляр `StaticFiles()` в определенный путь.

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"), name="static")
```

> **Технические детали**
> 
> Вы можете также использовать `from starlette.staticfiles import StaticFiles`.
> 
> FastAPI предоставляет тот же `starlette.staticfiles` как `fastapi.staticfiles`, просто это удобнее для вас, разработчика.
> Но на самом деле он идет напрямую из Starlette.

### Что такое "Монтирование"

"Монтирование" означает добавление полноценного "независимого" приложения в определенный путь, который, затем, заботится
об обработке всех подпутей.

В этом разница с использованием `APIRouter`, так как смонтированное приложение полностью независимо. Документация и 
OpenAPI из вашего основного приложения не будет включать в себя что-либо их подключенного приложения, и т.д.

Вы можете прочитать больше об этом в Продвинутом руководстве.

## Детали

Первый `"/static"` относится к подпути этого "подприложения", которое будет "смонтировано". Поэтому, любой путь, который 
начинается на `"/static"`, будет им обработан.

`directory="static"` относится к имени директории, которая содержит ваши статические файлы.

`name="static"` дает название, которое будет использовано внутри с помощью FastAPI.

Все эти параметры могут отличаться от "`static`", настройте их с потребностями и определенными деталями для вашего 
приложения.

## Больше информации

Для получения более подробной информации и возможностей, проверьте 
[документацию Starlette про статические файлы](https://www.starlette.io/staticfiles/). 