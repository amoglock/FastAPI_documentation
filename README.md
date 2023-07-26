<h3>Учебник - руководство пользователя</h3>

Это руководство поэтапно показывает как использовать основные возможности FastAPI.<br>
<br>
Каждый раздел руководства основан на предыдущих, но все темы разделены, поэтому вы можете перейти к любой конкретной, 
чтобы найти ответ по этой теме.<br><br>

Учебник так же работает как справочный материал.<br><br>

<h3>Запуск кода</h3>
Все блоки кода в руководстве можно скопировать и использовать(это проверенные файлы Python).<br><br>
Чтобы запустить любой пример, скопируйте код в файл `main.py` и запустите `uvicorn`:

```commandline
uvicorn main:app --reload
```

Сообщения сервера после запуска:

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [28720]
INFO:     Started server process [28722]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

**Настоятельно рекомендуется** писать или копировать код, редактировать и запускать локально.<br><br>
Использование его в редакторе по-настоящему показывает преимущества FastAPI, малое количество кода, кодорый нужно 
написать; все проверки типов данных, автодополнение кода и т.д.

***

<h3>Устнановка FastAPI</h3>

Первый шаг - установка FastAPI.<br><br>
Для использования руководства можно установить FastAPI со всеми зависимостями и функциями:

```
pip install "fastapi[all]"
```

...что так же включает в себя `uvicorn`, который можно использовать как сервер для запуска кода.<br>

>***Обратите внимание.***<br> Возможна установка по частям. Такой установки, вероятно, будет достаточно чтобы развернуть
> приложение:
>```
>pip install fastapi
>```
>Установка `uvicorn` который работает в качестве сервера:
> ```
>pip install "uvicorn[standard]"
>```
> Так можно установить любую отдельную зависимость, которую вам нужно использовать.

<h3>Продвинутое руководство пользователя</h3>

Существует **Продвинутое руководство пользователя**, которое можно прочитать после этого руководства.<br>

**Продвинутое руководство пользователя** построено на этом руководстве, использует ту же концепцию и раскрывает 
дополнительные возможности FastAPI.<br>

Но сперва лучше прочитать это руководство.<br>
Оно разработатано так, что используя только этот учебник, вы можете создать полноценное приложение. А затем расширить 
его различными способами, в зависимости от ваших потребностей, используя продвинутое руководство.