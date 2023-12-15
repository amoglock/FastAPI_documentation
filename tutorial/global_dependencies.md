<h3>Глобальные зависимости</h3>

Для некоторых видов приложений вам может быть необходимо добавить зависимости ко всему приложению.

Аналогично тому как вы можете 
<a href="https://github.com/amoglock/FastAPI_documentation/blob/master/tutorial/dependencies_in_path_operation_decorators.md">
добавлять зависимости в декораторы операции пути</a>, вы можете добавлять их к приложению `FastAPI`.

В этом случае, они будут применены ко всем операциям пути в приложении:

```python
from fastapi import Depends, FastAPI, Header, HTTPException
from typing_extensions import Annotated

...

app = FastAPI(dependencies=[Depends(verify_token), Depends(verify_key)])

...
```

А вся концепция в секции касательно 
<a href="https://github.com/amoglock/FastAPI_documentation/blob/master/tutorial/dependencies_in_path_operation_decorators.md">
добавления зависимости в декораторы операции пути</a>, все еще применяется, но в этом случае ко всем операциям пути в
приложении.

<h4>Зависимости для групп операций пути</h4>

Позже, при чтении о том как структурировать большие приложения, вероятно с множеством файлов, вы узнаете как объявлять 
один параметр dependencies для группы операций пути.
