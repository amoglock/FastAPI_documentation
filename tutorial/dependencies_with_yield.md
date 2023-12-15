<h3>Зависимости с yield</h3>

FastAPI поддерживает зависимости, которые выполняют некоторые дополнительные шаги после завершения.

Чтобы сделать такое, используйте `yield` вместо `return` и затем напишите дополнительные действия.

> **Совет**
> 
> Убедитесь, что используете `yield` один-единственный раз.

> **Технические детали**
> 
> Любая функция, допустимая к использованию с:
> * <a href="https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager">`@contextlib.contextmanager`</a> или
> * <a href="https://docs.python.org/3/library/contextlib.html#contextlib.asynccontextmanager">`@contextlib.asynccontextmanager`</a>
> 
> Была бы допустимой, чтобы использовать ее как зависимость FastAPI.
> 
> На самом деле, FastAPI внутри использует эти два декоратора.

<h4>Зависимость базы данных с `yield`</h4>

Например, вы можете использовать это, чтобы создать сессию базы данных и закрыть ее после завершения.

Перед отправкой ответа выполнится только код, предшествующий и включающий выражение `yield`:

```python
async def get_db():
    db = DBSession()
    try:
        yield db
    ...
```

Код следующий после выражения `yield` выполнится после того как ответ был доставлен:

```python
async def get_db():
    db = DBSession()
    try:
        yield db
    finally:
        db.close()
```

> **Совет**
> 
> Вы можете использовать `async` или обычные функции.
> 
> FastAPI сделает все правильно с каждой из них, так же как и с обычными зависимостями.

<h4>Зависимости с `yield` и `try`</h4>

Если вы используете блок `try` в зависимости вместе с `yield`, вы примете любое исключение которое было вызвано во время
использования зависимости.

Например, если какой-то код в какой-то момент посередине, в другой зависимости или операции пути, совершает "откат"
транзакции базы данных или создает любую другую ошибку, вы получите исключение в вашей зависимости.

Поэтому, вы можете искать это определённое исключение внутри зависимости с `except SomeException`.

Таким же образом, вы можете использовать `finally` чтобы быть уверенным, что шаги для выхода выполнены, не важно было 
исключение или нет.

```python
async def get_db():
    db = DBSession()
    try:
        yield db
    finally:
        db.close()
```

<h4>Подзависимости с `yield`</h4>

У вас могут быть подзависимости и "деревья" подзависимостей любого размера и формы, и любая из них может использовать `yield`.

FastAPI позаботится, чтобы "код выхода" в каждой зависимости с `yield` был запущен в правильном порядке.

Например, `dependency_c` может иметь зависимость на `dependency_b`, а `dependency_b` на `dependency_a`:

```python
from typing import Annotated

from fastapi import Depends


async def dependency_a():
    dep_a = generate_dep_a()
    try:
        yield dep_a


async def dependency_b(dep_a: Annotated[DepA, Depends(dependency_a)]):
    dep_b = generate_dep_b()
    try:
        yield dep_b


async def dependency_c(dep_b: Annotated[DepB, Depends(dependency_b)]):
    dep_c = generate_dep_c()
    try:
        yield dep_c

```

И все они могут иметь `yield`.

В этом случае `dependency_c`, чтобы выполнить свой код выхода, нужно значение от `dependency_b` (названое здесь `dep_b`).

А в свою очередь, `dependency_b` необходимо значение от `dependency_a` (названое здесь `dep_a`), чтобы быть доступным для
ее кода выхода.

```python
from typing import Annotated

from fastapi import Depends


async def dependency_a():
    dep_a = generate_dep_a()
    try:
        yield dep_a
    finally:
        dep_a.close()


async def dependency_b(dep_a: Annotated[DepA, Depends(dependency_a)]):
    dep_b = generate_dep_b()
    try:
        yield dep_b
    finally:
        dep_b.close(dep_a)


async def dependency_c(dep_b: Annotated[DepB, Depends(dependency_b)]):
    dep_c = generate_dep_c()
    try:
        yield dep_c
    finally:
        dep_c.close(dep_b)
```

Точно так же, вы можете зависимости, смешанные с `yield` и `return`.

И вы можете иметь одну зависимость, которая требует несколько других зависимостей с `yield` и т.д.

Вы можете иметь любое количество комбинаций зависимостей как вы хотите.

FastAPI сделает все, чтобы запустить правильный порядок.

> **Технические детали**
> 
> Это работает благодаря <a href="https://docs.python.org/3/library/contextlib.html">Context Managers</a> Python.
> 
> FastAPI использует их внутри.
