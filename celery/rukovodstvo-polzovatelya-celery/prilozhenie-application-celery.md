# Приложение Application Celery

* [Основное имя](prilozhenie-application-celery.md#osnovnoe-imya)
* Конфигурация
* Лень
* Разрыв цепи
* Абстрактные задачи

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
