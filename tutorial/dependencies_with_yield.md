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

<h4>Зависимости с `yield` и `HTTPException`</h4>

Вы видели, что вы можете использовать зависимости с `yield` и делать блоки `try`, чтобы ловить исключения.

Может возникнуть желание вызвать `HttpException` или что-то подобное в коде выхода после `yield`. Но это не сработает.

Код выхода в зависимостях с `yield` выполняется после того как запрос отправлен, поэтому 
<a href="https://fastapi.tiangolo.com/tutorial/handling-errors/#install-custom-exception-handlers">Обработчики ошибок</a>
будут уже запущены.

В коде выхода нет ничего, что перехватывало бы исключения, создаваемые вашими зависимостями (после `yield`).

Поэтому, если вы вызываете `HTTPException` после `yield`, дефолтный (или любой самодельный) обработчик ошибок, который
ловит `HTTPException` и возвращает ответ HTTP 400, больше не будет отлавливать это исключение.

Это то, что позволяет любому установленному в зависимости (например сессии БД), например, быть использованными в 
фоновых задачах.

Фоновые задачи запускаются после того как ответ отправлен. Поэтому нет способа поймать `HTTPException`, потому что нет
способа изменить уже отправленный ответ.

Но если фоновая задача создает ошибку базы данных, как минимум, вы можете откатить или "чисто закрыть" сессию в зависимости
с `yield` и, может быть, записать ошибку или ее отчет в систему слежения.

Если у вас есть какой-то код, который, вы знаете, что может вызывать исключение, делайте самые обычные / "Питонические" 
вещи и добавляйте блок `try` в эту секцию кода.

Если у вас кастомные исключения, которые вы бы хотели обрабатывать перед возвратом ответа и, вероятно, изменять ответ,
может быть даже, вызывая `HTTPException`, создавайте 
<a href="https://fastapi.tiangolo.com/tutorial/handling-errors/#install-custom-exception-handlers">Custom Exception Handler</a>.

> **Совет**
> 
> Вы все еще можете вызывать исключения, включающие `HTTPException`, перед `yield`. Но не после.

Последовательность выполнения более или менее похожа на эту диаграмму. Выполнение по времени идет сверху вниз. И каждый
столбец это одна из частей взаимодействия или выполнения кода.

Вы можете посмотреть эту диаграмму 
<a href="https://fastapi.tiangolo.com/tutorial/dependencies/dependencies-with-yield/#sub-dependencies-with-yield">здесь</a>.

> **Для информации**
> 
> Только один ответ будет отправлен в клиент. Он может быть одним из ответов ошибки или быть ответом от операции пути.
> 
> После того как один из таких ответов отправлен, другой ответ не может быть отправлен.

> **Совет**
> 
> Эта диаграмма показывает `HTTPException`, но вы можете вызывать любое другое исключение, для которого вы создаете 
> <a href="https://fastapi.tiangolo.com/tutorial/handling-errors/#install-custom-exception-handlers">Custom Exception Handler</a>.
> 
> Если вы вызываете любое исключение, оно будет передано зависимости с yield, включая `HTTPException`, а затем снова
> обработчику исключений. Если для такого исключения нет обработчика, оно будет обработано с помощью встроенного
> `ServerErrorMiddleware`, возвращающего статус код 500 HTTP, чтобы дать возможность клиенту знать, что на сервере 
> произошла ошибка.

<h4>Контекстные менеджеры</h4>

<h5>Что такое "менеджер контекста"</h5>

"Менеджеры контекста" это любой из тех объектов Python, которые можно использовать с выражением `with`.

Например, <a href="https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files">вы можете использовать
`with`, чтобы читать файл</a>:

```python
with open("./somefile.txt") as f:
    contents = f.read()
    print(contents)
```

Под `open("./somefile.txt")` подразумевается создание объекта, который называется "Контекстным менеджером".

Когда блок `with` завершается, он сам позаботится о том, чтобы закрыть файл, даже если там были исключения.

Когда вы создаете зависимость с `yield`, FastAPI будет внутренне конвертировать ее в контекстный менеджер, и совмещать
его с другими инструментами.

<h5>использование менеджеров контекста в зависимостях с `yield`</h5>

> **Внимание!!!**
> 
> Это, более-менее, "продвинутое" понятие.
> 
> Поэтому если вы только начинаете с FastAPI, можете пока его пропустить.

В Python вы можете создавать Контекстные Менеджеры путем 
<a href="https://docs.python.org/3/reference/datamodel.html#context-managers">создания класса с двумя методами:
`__enter__` и `__exit__`</a>.

Вы можете использовать их внутри зависимостей FastAPI с `yield`, используя выражения `with` или `async with` внутри
зависимой функции:

```python
class MySuperContexManager:
    def __init__(self):
        self.db = DBSession()

    def __enter__(self):
        return self.db

    def __exit__(self, exc_type, exc_value, traceback):
        self.db.close()


async def get_db():
    with MySuperContexManager() as db:
        yield db
```

> **Совет**
> 
> Другие способы создать контекстный менеджер это:
> * <a href="https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager">`@contextlib.contextmanager`</a> или
> * <a href="https://docs.python.org/3/library/contextlib.html#contextlib.asynccontextmanager">`@contextlib.asynccontextmanager`</a>
> 
> Используйте их, чтобы декорировать функцию с одиночным `yield`.
> 
> Это то, что FastAPI использует внутренне для зависимостей с `yield`.
> 
> Но вам не обязательно использовать декораторы для зависимостей FastAPI (и вы не должны этого делать).
> 
> FastAPI сделает это за вас.