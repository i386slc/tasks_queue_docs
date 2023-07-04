# Приложение Application Celery

* [Основное имя](prilozhenie-application-celery.md#osnovnoe-imya)
* [Конфигурация](prilozhenie-application-celery.md#konfiguraciya)
* [Лень](prilozhenie-application-celery.md#len)
* [Разрыв цепочки](prilozhenie-application-celery.md#razryv-cepochki)
* [Абстрактные задачи](prilozhenie-application-celery.md#abstraktnye-zadachi)

Библиотека Celery должна быть инициализирована перед использованием, этот экземпляр называется приложением (или **app** для краткости).

Приложение является потокобезопасным, так что несколько приложений Celery с разными конфигурациями, компонентами и задачами могут сосуществовать в одном и том же пространстве процессов.

Давайте создадим его сейчас:

```python
>>> from celery import Celery
>>> app = Celery()
>>> app
<Celery __main__:0x100469fd0>
```

Последняя строка показывает текстовое представление приложения: включая имя класса приложения (**Celery**), имя текущего основного модуля (**\_\_main\_\_**) и адрес объекта в памяти (`0x100469fd0`).

## Основное имя

Только один из них важен, и это имя основного модуля. Давайте посмотрим, почему это так.

Когда вы отправляете сообщение задачи в Celery, это сообщение не будет содержать никакого исходного кода, а только имя задачи, которую вы хотите выполнить. Это работает аналогично тому, как имена хостов работают в Интернете: каждый воркер поддерживает сопоставление имен задач с их фактическими функциями, называемое реестром задач.

Всякий раз, когда вы определяете задачу, эта задача также будет добавлена в локальный реестр:

```python
>>> @app.task
... def add(x, y):
...     return x + y

>>> add
<@task: __main__.add>

>>> add.name
__main__.add

>>> app.tasks['__main__.add']
<@task: __main__.add>
```

и там вы снова видите, что **\_\_main\_\_**; всякий раз, когда Celery не может определить, к какому модулю принадлежит функция, он использует имя основного модуля для создания начала имени задачи.

Это проблема только в ограниченном наборе вариантов использования:

1. Если модуль, в котором определена задача, запускается как программа.
2. Если приложение создано в оболочке Python (REPL).

Например, здесь, где модуль **tasks** также используется для запуска воркера с помощью **app.worker\_main()**:

`tasks.py`:

```python
from celery import Celery
app = Celery()

@app.task
def add(x, y): return x + y

if __name__ == '__main__':
    app.worker_main()
```

При выполнении этого модуля имена задач будут начинаться с `«__main__»`, но когда модуль импортируется другим процессом, скажем, для вызова задачи, имена задач будут начинаться с `«tasks»` (настоящее имя модуля). :

```python
>>> from tasks import add
>>> add.name
tasks.add
```

Вы можете указать другое имя для основного модуля:

```python
>>> app = Celery('tasks')
>>> app.main
'tasks'

>>> @app.task
... def add(x, y):
...     return x + y

>>> add.name
tasks.add
```

{% hint style="info" %}
Смотри также:

[Names](https://docs.celeryq.dev/en/stable/userguide/tasks.html#task-names)
{% endhint %}

## Конфигурация

Вы можете установить несколько параметров, которые изменят работу Celery. Эти параметры можно установить непосредственно в экземпляре приложения или использовать специальный модуль конфигурации.

Конфигурация доступна как [**app.conf**](https://docs.celeryq.dev/en/stable/reference/celery.app.utils.html#celery.app.utils.Settings):

```python
>>> app.conf.timezone
'Europe/London'
```

где вы также можете напрямую установить значения конфигурации:

```python
>>> app.conf.enable_utc = True
```

или обновить сразу несколько ключей, используя метод обновления **update**:

```python
>>> app.conf.update(
...     enable_utc=True,
...     timezone='Europe/London',
...)
```

Объект конфигурации состоит из нескольких словарей, с которыми обращаются по порядку:

1. Изменения, внесенные во время выполнения.
2. Модуль конфигурации (если есть)
3. Конфигурация по умолчанию ([celery.app.defaults](https://docs.celeryq.dev/en/stable/reference/celery.app.defaults.html#module-celery.app.defaults)).

Вы даже можете добавить новые источники по умолчанию, используя метод [app.add\_defaults()](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery.add\_defaults).

{% hint style="info" %}
Смотри также:

Перейдите к [справочнику по конфигурации](konfiguraciya-i-ustanovki-po-umolchaniyu.md), чтобы получить полный список всех доступных параметров и их значений по умолчанию.
{% endhint %}

### config\_from\_object

Метод [app.config\_from\_object()](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery.config\_from\_object) загружает конфигурацию из объекта конфигурации.

Это может быть модуль конфигурации или любой объект с атрибутами конфигурации.

Обратите внимание, что любая ранее установленная конфигурация будет сброшена при вызове [config\_from\_object()](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery.config\_from\_object). Если вы хотите установить дополнительную конфигурацию, вы должны сделать это позже.

### Пример 1: Использование имени модуля

Метод [app.config\_from\_object()](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery.config\_from\_object) может принимать полное имя модуля Python или даже имя атрибута Python, например: `«celeryconfig»`, `«myproj.config.celery»` или `«myproj.config:CeleryConfig»` :

```python
from celery import Celery

app = Celery()
app.config_from_object('celeryconfig')
```

Тогда модуль **celeryconfig** может выглядеть так:

`celeryconfig.py`:

```python
enable_utc = True
timezone = 'Europe/London'
```

и приложение **app** сможет использовать его до тех пор, пока возможен `import celeryconfig`.

### Пример 2: Передача фактического объекта модуля

Вы также можете передать уже импортированный объект модуля, но это не всегда рекомендуется.

{% hint style="info" %}
СОВЕТ.

Рекомендуется использовать имя модуля, поскольку это означает, что модуль не нужно сериализовать, когда используется пул **prefork**. Если у вас возникли проблемы с конфигурацией или ошибки **pickle**, попробуйте вместо этого использовать имя модуля.
{% endhint %}

```python
import celeryconfig

from celery import Celery

app = Celery()
app.config_from_object(celeryconfig)
```

### Пример 3: Использование класса/объекта конфигурации

```python
from celery import Celery

app = Celery()

class Config:
    enable_utc = True
    timezone = 'Europe/London'

app.config_from_object(Config)
# или используя полное имя объекта:
#   app.config_from_object('module:Config')
```

### config\_from\_envvar

[app.config\_from\_envvar()](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery.config\_from\_envvar) берет имя модуля конфигурации из переменной среды.

Например, чтобы загрузить конфигурацию из модуля, указанного в переменной среды с именем **CELERY\_CONFIG\_MODULE**:

```python
import os
from celery import Celery

#: Установить имя модуля конфигурации по умолчанию
os.environ.setdefault('CELERY_CONFIG_MODULE', 'celeryconfig')

app = Celery()
app.config_from_envvar('CELERY_CONFIG_MODULE')
```

Затем вы можете указать модуль конфигурации для использования через среду:

```bash
$ CELERY_CONFIG_MODULE="celeryconfig.prod" celery worker -l INFO
```

### Конфигурация с цензурой

Если вы когда-нибудь захотите распечатать конфигурацию в качестве отладочной информации или чего-то подобного, вы также можете отфильтровать конфиденциальную информацию, такую как пароли и ключи API.

Celery поставляется с несколькими утилитами, полезными для представления конфигурации, одна из них — [humanize()](https://docs.celeryq.dev/en/stable/reference/celery.app.utils.html#celery.app.utils.Settings.humanize):

```python
>>> app.conf.humanize(with_defaults=False, censored=True)
```

Этот метод возвращает конфигурацию в виде табличной строки. Это будет содержать только изменения конфигурации по умолчанию, но вы можете включить встроенные ключи и значения по умолчанию, включив аргумент **with\_defaults**.

Если вместо этого вы хотите работать с конфигурацией как со словарем, вы можете использовать метод [table()](https://docs.celeryq.dev/en/stable/reference/celery.app.utils.html#celery.app.utils.Settings.table):

```python
>>> app.conf.table(with_defaults=False, censored=True)
```

Обратите внимание, что Celery не сможет удалить всю конфиденциальную информацию, так как он просто использует регулярное выражение для поиска часто именуемых ключей. Если вы добавляете пользовательские настройки, содержащие конфиденциальную информацию, вы должны назвать ключи, используя имя, которое Celery идентифицирует как секрет.

Параметр конфигурации будет подвергаться цензуре, если имя содержит любую из следующих подстрок:

`API`, `TOKEN`, `KEY`, `SECRET`, `PASS`, `SIGNATURE`, `DATABASE`

## Лень

Экземпляр приложения является ленивым, то есть он не будет оцениваться до тех пор, пока он действительно не понадобится.

Создание экземпляра [Celery](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery) будет выполнять только следующие действия:

1. Создание экземпляр логических часов, используемый для событий.
2. Создание реестра задач.
3. Установка себя как текущего приложения (но не в том случае, если аргумент **set\_as\_current** был отключен)
4. Вызывает обратный вызов [app.on\_init()](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery.on\_init) (по умолчанию ничего не делает).

Декораторы [app.task()](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery.task) не создают задачи в тот момент, когда задача определена, вместо этого они откладывают создание задачи до того, как она будет использована, или после того, как приложение будет завершено.

В этом примере показано, как задача не создается до тех пор, пока вы не используете задачу или не получите доступ к атрибуту (в данном случае **repr()**):

```python
>>> @app.task
>>> def add(x, y):
...    return x + y

>>> type(add)
<class 'celery.local.PromiseProxy'>

>>> add.__evaluated__()
False

>>> add        # <-- вызывает repr(add)
<@task: __main__.add>

>>> add.__evaluated__()
True
```

_**Завершение**_ приложения происходит либо явно, вызывая [app.finalize()](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery.finalize), либо неявно, обращаясь к атрибуту **app.tasks**.

Завершение объекта будет:

1. Скопируйте задачи, которые должны быть разделены между приложениями. Задачи являются общими по умолчанию, но если общий аргумент для декоратора задач отключен, задача будет частной для приложения, к которому она привязана.
2. Оцените все ожидающие декораторы задач.
3. Убедитесь, что все задачи привязаны к текущему приложению. Задачи привязаны к приложению, чтобы они могли считывать значения по умолчанию из конфигурации.

{% hint style="info" %}
**default\_app**

У Celery не всегда были приложения, раньше было только модульное API. API совместимости был доступен в старых версиях до выпуска **Celery 5.0**, но потом был удален.

Celery всегда создает специальное приложение — «default\_app», и оно используется, если не было создано ни одного пользовательского приложения.

Модуль **celery.task** больше недоступен. Используйте методы экземпляра приложения, а не API на основе модуля:

```python
from celery.task import Task   # << СТАРЫЙ базовый класс Task.

from celery import Task        # << НОВЫЙ базовый класс.
```
{% endhint %}

## Разрыв цепочки

Хотя можно зависеть от текущего установленного приложения, лучше всего всегда передавать экземпляр приложения всему, что в нем нуждается.

Я называю это цепочкой приложений (_app chain_), поскольку она создает цепочку экземпляров в зависимости от передаваемого приложения.

Следующий пример считается плохой практикой:

```python
from celery import current_app

class Scheduler:

    def run(self):
        app = current_app
```

Вместо этого он должен принимать приложение **app** в качестве аргумента:

```python
class Scheduler:

    def __init__(self, app):
        self.app = app
```

Внутри Celery используется функция [celery.app.app\_or\_default()](https://docs.celeryq.dev/en/stable/reference/celery.app.html#celery.app.app\_or\_default), так что все также работает в API совместимости на основе модулей.

```python
from celery.app import app_or_default

class Scheduler:
    def __init__(self, app=None):
        self.app = app_or_default(app)
```

В процессе разработки вы можете установить переменную среды **CELERY\_TRACE\_APP** для создания исключения в случае разрыва цепочки приложений:

```bash
$ CELERY_TRACE_APP=1 celery worker -l INFO
```

{% hint style="info" %}
**Эволюция API**

Celery сильно изменился с 2009 года с момента его создания.

Например, вначале в качестве задачи можно было использовать любой callable:

```python
def hello(to):
    return 'hello {0}'.format(to)

>>> from celery.execute import apply_async

>>> apply_async(hello, ('world!',))
```

или вы также можете создать класс **Task** для установки определенных параметров или переопределения другого поведения.

```python
from celery import Task
from celery.registry import tasks

class Hello(Task):
    queue = 'hipri'

    def run(self, to):
        return 'hello {0}'.format(to)
tasks.register(Hello)

>>> Hello.delay('world!')
```

Позже было решено, что передача произвольных call-able’ов — это антишаблон, так как очень сложно использовать сериализаторы, отличные от **pickle**, и эта функция была удалена в 2.0, заменена декораторами задач:

```python
from celery import app

@app.task(queue='hipri')
def hello(to):
    return 'hello {0}'.format(to)
```
{% endhint %}

## Абстрактные задачи

Все задачи, созданные с помощью декоратора [app.task()](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery.task), будут наследоваться от базового класса [Task](https://docs.celeryq.dev/en/stable/reference/celery.app.task.html#celery.app.task.Task) приложения.

Вы можете указать другой базовый класс, используя базовый аргумент **base**:

```python
@app.task(base=OtherTask):
def add(x, y):
    return x + y
```

Чтобы создать собственный класс задач, вы должны наследовать его от нейтрального базового класса: **celery.Task**.

```python
from celery import Task

class DebugTask(Task):

    def __call__(self, *args, **kwargs):
        print('TASK STARTING: {0.name}[{0.request.id}]'.format(self))
        return self.run(*args, **kwargs)
```

{% hint style="info" %}
СОВЕТ:

Если вы переопределяете метод **\_\_call\_\_** задачи, очень важно, чтобы вы также вызывали **self.run** для выполнения тела задачи. Не вызывайте `super().__call__`. Метод **\_\_call\_\_** нейтрального базового класса **celery.Task** представлен только для справки. Для оптимизации это было развернуто в `celery.app.trace.build_tracer.trace_task`, вызовы которого выполняются непосредственно в пользовательском классе задач, если метод **\_\_call\_\_** не определен.
{% endhint %}

Нейтральный базовый класс особенный, потому что он еще не привязан ни к какому конкретному приложению. Как только задача будет привязана к приложению, она прочитает конфигурацию, чтобы установить значения по умолчанию и так далее.

Чтобы реализовать базовый класс, вам нужно создать задачу с помощью декоратора [app.task()](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery.task):

```python
@app.task(base=DebugTask)
def add(x, y):
    return x + y
```

Можно даже изменить базовый класс по умолчанию для приложения, изменив его атрибут [app.Task()](https://docs.celeryq.dev/en/stable/reference/celery.app.task.html#celery.app.task.Task):

```python
>>> from celery import Celery, Task

>>> app = Celery()

>>> class MyBaseTask(Task):
...    queue = 'hipri'

>>> app.Task = MyBaseTask
>>> app.Task
<unbound MyBaseTask>

>>> @app.task
... def add(x, y):
...     return x + y

>>> add
<@task: __main__.add>

>>> add.__class__.mro()
[<class add of <Celery __main__:0x1012b4410>>,
 <unbound MyBaseTask>,
 <unbound Task>,
 <type 'object'>]
```
