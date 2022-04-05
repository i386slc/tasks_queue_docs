# Первые шаги с Celery

**Celery** — это очередь задач с включенными батареями. Он прост в использовании, так что вы можете начать работу, не изучая все сложности проблемы, которую он решает. Он разработан на основе лучших практик, чтобы ваш продукт можно было масштабировать и интегрировать с другими языками, и он поставляется с инструментами и поддержкой, необходимыми для запуска такой системы в производственной среде.

В этом уроке вы изучите абсолютные основы использования **Celery**.

Узнаем о:

* Выбор и установка транспорта сообщений (брокера).
* Установка Celery и создание первой задачи.
* Запуск воркера и вызов задач.
* Отслеживание задач по мере их перехода через разные состояния и проверка возвращаемых значений.

Поначалу **Celery** может показаться сложным, но не волнуйтесь — этот урок поможет вам начать работу в кратчайшие сроки. Он намеренно сделан простым, чтобы не запутать вас дополнительными функциями. После того, как вы закончите это руководство, рекомендуется просмотреть остальную часть документации. Например, руководство «[Следующие шаги](sleduyushie-shagi.md)» продемонстрирует возможности Celery.



_Здесь должно быть оглавление..._

## Выбор брокера

**Celery** требует решения для отправки и получения сообщений; обычно это происходит в виде отдельной службы, называемой _брокером сообщений_.

Доступно несколько вариантов, в том числе:

### RabbitMQ

[RabbitMQ](http://www.rabbitmq.com) — полнофункциональный, стабильный, надежный и простой в установке. Это отличный выбор для производственной среды. Подробная информация об использовании RabbitMQ с Celery:

[Использование RabbitMQ](bekendy-i-brokery/ispolzovanie-rabbitmq.md)

Если вы используете Ubuntu или Debian, установите RabbitMQ, выполнив эту команду:

```bash
$ sudo apt-get install rabbitmq-server
```

Или, если вы хотите запустить его в **Docker**, выполните следующее:

```bash
$ docker run -d -p 5672:5672 rabbitmq
```

Когда команда завершится, брокер уже будет работать в фоновом режиме, готовый перемещать сообщения для вас: `Starting rabbitmq-server: SUCCESS`.

Не беспокойтесь, если вы не используете Ubuntu или Debian, вы можете перейти на этот веб-сайт, чтобы найти такие же простые инструкции по установке для других платформ, включая Microsoft Windows: [http://www.rabbitmq.com/download.html](http://www.rabbitmq.com/download.html)

### Redis

Redis также полнофункционален, но более подвержен потере данных в случае внезапного завершения или сбоя питания. Подробная информация об использовании **Redis**:

[Использование Redis](bekendy-i-brokery/ispolzovanie-redis.md)

Если вы хотите запустить его в **Docker**, выполните следующее:

```bash
$ docker run -d -p 6379:6379 redis
```

### Другие брокеры

В дополнение к вышеперечисленному существуют другие экспериментальные реализации транспорта, в том числе [Amazon SQS](bekendy-i-brokery/ispolzovanie-amazon-sqs.md).

Полный список см. в разделе [Обзор брокера](bekendy-i-brokery/#obzor-brokerov).

## Установка Celery

Celery находится в индексе пакетов Python (PyPI), поэтому его можно установить с помощью стандартных инструментов Python, таких как **pip** или **easy\_install**:

```bash
$ pip install celery
```

## Приложение

Первое, что вам нужно, это экземпляр **Celery**. Мы называем это приложением _Celery application_ или просто приложением _**app**_ для краткости. Поскольку этот экземпляр используется в качестве точки входа для всего, что вы хотите делать в Celery, например, для создания задач и управления воркерами, другие модули должны иметь возможность импортировать его.

В этом уроке мы храним все, что содержится в одном модуле, но для более крупных проектов вы хотите создать [специальный модуль](sleduyushie-shagi.md).

Создадим файл `tasks.py`:

```python
from celery import Celery

app = Celery('tasks', broker='pyamqp://guest@localhost//')

@app.task
def add(x, y):
    return x + y
```

_Первым аргументом_ **Celery** является имя текущего модуля. Это нужно только для того, чтобы имена могли генерироваться автоматически, когда задачи определены в модуле **\_\_main\_\_**.

_Второй аргумент_ — это аргумент ключевого слова **broker**, указывающий URL-адрес брокера сообщений, который вы хотите использовать. Здесь мы используем RabbitMQ (тоже вариант по умолчанию).

Дополнительные варианты см. в разделе «[Выбор брокера](pervye-shagi-s-celery.md#vybor-brokera)» выше — для RabbitMQ вы можете использовать `amqp://localhost`, а для Redis — `redis://localhost`.

Вы определили единственную задачу, называемую **add**, возвращающую сумму двух чисел.

## Запуск рабочего сервера Celery

Теперь вы можете запустить воркера, выполнив нашу программу с аргументом **worker**:

```bash
$ celery -A tasks worker --loglevel=INFO
```

{% hint style="info" %}
См. раздел «Устранение неполадок» ([Troubleshooting](https://docs.celeryq.dev/en/stable/getting-started/first-steps-with-celery.html#celerytut-troubleshooting)), если рабочий процесс не запускается.
{% endhint %}

В продакшне вы захотите запустить воркер в фоновом режиме в качестве демона. Для этого вам нужно использовать инструменты, предоставляемые вашей платформой, или что-то вроде супервизора ([supervisord](http://supervisord.org)) (дополнительную информацию см. в разделе Демонизация ([Daemonization](https://docs.celeryq.dev/en/stable/userguide/daemonizing.html#daemonizing))).

Для получения полного списка доступных параметров командной строки выполните:

```bash
$  celery worker --help
```

Также доступно несколько других команд, а также доступна помощь:

```python
$ celery --help
```

## Вызов задачи

Для вызова нашей задачи вы можете использовать метод [delay()](https://docs.celeryq.dev/en/stable/reference/celery.app.task.html#celery.app.task.Task.delay).

Это удобный ярлык для метода [apply\_async()](https://docs.celeryq.dev/en/stable/reference/celery.app.task.html#celery.app.task.Task.apply\_async), который дает больший контроль над выполнением задачи (см. Вызов задач ([Calling Tasks](https://docs.celeryq.dev/en/stable/userguide/calling.html#guide-calling))):

```python
>>> from tasks import add
>>> add.delay(4, 4)
```

Задача теперь обработана воркером, который вы запустили ранее. Вы можете убедиться в этом, посмотрев на вывод рабочей консоли.

Вызов задачи возвращает экземпляр [AsyncResult](https://docs.celeryq.dev/en/stable/reference/celery.result.html#celery.result.AsyncResult). Это можно использовать для проверки состояния задачи, ожидания завершения задачи или получения возвращаемого значения (или, если задача не удалась, для получения исключения и обратной трассировки).

Результаты не включены по умолчанию. Чтобы выполнять удаленные вызовы процедур или отслеживать результаты задач в базе данных, вам необходимо настроить Celery для использования серверной части результатов. Это описано в следующем разделе.

## Сохранение результатов

Если вы хотите отслеживать состояния задач, Celery должен их где-то хранить или отправлять. На выбор есть несколько встроенных бэкендов результатов: [SQLAlchemy](http://www.sqlalchemy.org) / [Django](http://djangoproject.com) ORM, [MongoDB](http://www.mongodb.org), [Memcached](http://memcached.org), [Redis](https://redis.io), [RPC](https://docs.celeryq.dev/en/stable/userguide/configuration.html#conf-rpc-result-backend) ([RabbitMQ](http://www.rabbitmq.com)/AMQP) и — или вы можете определить свой собственный.

В этом примере мы используем серверную часть результатов **rpc**, которая отправляет состояния обратно в виде временных сообщений. Серверная часть указывается через аргумент **backend** для [Celery](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery) (или с помощью параметра [**result\_backend**](https://docs.celeryq.dev/en/stable/userguide/configuration.html#std-setting-result\_backend), если вы решите использовать модуль конфигурации). Итак, вы можете изменить эту строку в файле `tasks.py`, чтобы включить бэкенд `rpc://`:

```python
app = Celery('tasks', backend='rpc://', broker='pyamqp://')
```

Или, если вы хотите использовать Redis в качестве серверной части результатов, но по-прежнему использовать RabbitMQ в качестве брокера сообщений (популярная комбинация):

```python
app = Celery('tasks', backend='redis://localhost', broker='pyamqp://')
```

Чтобы узнать больше о бэкэндах результатов, см. бэкенды результатов ([Result Backends](https://docs.celeryq.dev/en/stable/userguide/tasks.html#task-result-backends)).

Теперь, когда настроен бэкенд результатов, закройте текущий сеанс Python и снова импортируйте модуль задач, чтобы изменения вступили в силу. На этот раз вы сохраните экземпляр [AsyncResult](https://docs.celeryq.dev/en/stable/reference/celery.result.html#celery.result.AsyncResult), возвращаемый при вызове задачи:

```python
>>> from tasks import add    # закройте и снова откройте, чтобы обновить 'app'
>>> result = add.delay(4, 4)
```

Метод [ready()](https://docs.celeryq.dev/en/stable/reference/celery.result.html#celery.result.AsyncResult.ready) возвращает ответ, завершила ли задача обработку или нет:

```python
>>> result.ready()
False
```

Можно дождаться завершения результата, но это редко используется, поскольку превращает асинхронный вызов в синхронный:

```python
>>> result.get(timeout=1)
8
```

В случае, если задача вызвала исключение, [get()](https://docs.celeryq.dev/en/stable/reference/celery.result.html#celery.result.AsyncResult.get) повторно вызовет исключение, но вы можете переопределить это, указав аргумент **propagate**:

```python
>>> result.get(propagate=False)
```

Если задача вызвала исключение, вы также можете получить доступ к исходной трассировке:

```python
>>> result.traceback
```

{% hint style="danger" %}
Бэкенды используют ресурсы для хранения и передачи результатов. Чтобы убедиться, что ресурсы высвобождаются, вы должны в конечном итоге вызвать [get()](https://docs.celeryq.dev/en/stable/reference/celery.result.html#celery.result.AsyncResult.get) или [forget()](https://docs.celeryq.dev/en/stable/reference/celery.result.html#celery.result.AsyncResult.forget) для **КАЖДОГО** экземпляра [AsyncResult](https://docs.celeryq.dev/en/stable/reference/celery.result.html#celery.result.AsyncResult), возвращаемого после вызова задачи.
{% endhint %}

Полную ссылку на объект результата см. в [celery.result](https://docs.celeryq.dev/en/stable/reference/celery.result.html#module-celery.result).

## Конфигурация

**Celery**, как и бытовой прибор, не требует особых настроек для работы. У него есть вход и выход. Вход должен быть подключен к брокеру, а выход может быть дополнительно подключен к серверной части результатов. Однако, если вы внимательно посмотрите на заднюю часть, там есть крышка с множеством ползунков, циферблатов и кнопок: это конфигурация.

Конфигурация по умолчанию должна быть достаточно хорошей для большинства случаев использования, но есть много параметров, которые можно настроить, чтобы заставить Celery работать именно так, как нужно. Прочитать о доступных параметрах — хорошая идея, чтобы ознакомиться с тем, что можно настроить. Вы можете прочитать об опциях в справочнике по конфигурации и значениям по умолчанию ([Configuration and defaults](https://docs.celeryq.dev/en/stable/userguide/configuration.html#configuration)).

Конфигурация может быть установлена непосредственно в приложении или с помощью специального модуля конфигурации. Например, вы можете настроить сериализатор по умолчанию, используемый для сериализации полезной нагрузки задач, изменив параметр [**task\_serializer**](https://docs.celeryq.dev/en/stable/userguide/configuration.html#std-setting-task\_serializer):

```python
app.conf.task_serializer = 'json'
```

Если вы настраиваете сразу много параметров, вы можете использовать **update**:

```python
app.conf.update(
    task_serializer='json',
    accept_content=['json'],  # Игнорировать другой контент
    result_serializer='json',
    timezone='Europe/Oslo',
    enable_utc=True,
)
```

Для более крупных проектов рекомендуется использовать специальный модуль конфигурации. Не рекомендуется жестко задавать периодические интервалы задач и параметры маршрутизации задач. Гораздо лучше держать их в централизованном месте. Это особенно верно для библиотек, поскольку позволяет пользователям контролировать поведение своих задач. Централизованная конфигурация также позволит системному администратору вносить простые изменения в случае системных проблем.

Вы можете указать своему экземпляру Celery использовать модуль конфигурации, вызвав метод [app.config\_from\_object()](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery.config\_from\_object):

```python
app.config_from_object('celeryconfig')
```

Этот модуль часто называют «**celeryconfig**», но вы можете использовать любое имя модуля.

В приведенном выше случае модуль с именем `celeryconfig.py` должен быть доступен для загрузки из текущего каталога или по пути Python. Это может выглядеть примерно так:

`celeryconfig.py`:

```python
broker_url = 'pyamqp://'
result_backend = 'rpc://'

task_serializer = 'json'
result_serializer = 'json'
accept_content = ['json']
timezone = 'Europe/Oslo'
enable_utc = True
```

Чтобы убедиться, что ваш файл конфигурации работает правильно и не содержит синтаксических ошибок, вы можете попробовать его импортировать:

```bash
$ python -m celeryconfig
```

Полный справочник параметров конфигурации см. в разделе Конфигурация и значения по умолчанию ([Configuration and defaults](https://docs.celeryq.dev/en/stable/userguide/configuration.html#configuration)).

Чтобы продемонстрировать силу файлов конфигурации, вот как вы направляете некорректно работающую задачу в выделенную очередь:

`celeryconfig.py`:

```python
task_routes = {
    'tasks.add': 'low-priority',
}
```

Или вместо маршрутизации вы можете ограничить задачу по скорости, чтобы только 10 задач этого типа могли быть обработаны в минуту (10/мин):

`celeryconfig.py`:

```python
task_annotations = {
    'tasks.add': {'rate_limit': '10/m'}
}
```

Если вы используете RabbitMQ или Redis в качестве брокера, вы также можете указать воркерам установить новое ограничение скорости для задачи во время выполнения:

```bash
$ celery -A tasks control rate_limit tasks.add 10/m
worker@example.com: OK
    new rate limit set successfully
```

См. Маршрутизация задач ([Routing Tasks](https://docs.celeryq.dev/en/stable/userguide/routing.html#guide-routing)), чтобы узнать больше о маршрутизации задач, и параметр [task\_annotations](https://docs.celeryq.dev/en/stable/userguide/configuration.html#std-setting-task\_annotations), чтобы узнать больше об аннотациях, или Руководство по мониторингу и управлению ([Monitoring and Management Guide](https://docs.celeryq.dev/en/stable/userguide/monitoring.html#guide-monitoring)), чтобы узнать больше о командах удаленного управления и о том, как контролировать действия ваших воркеров.

## Куда идти дальше

Если вы хотите узнать больше, вы должны перейти к учебнику «[Следующие шаги](sleduyushie-shagi.md)», а после этого вы можете прочитать [руководство пользователя](../rukovodstvo-polzovatelya-celery.md).

## Поиск проблем

В разделе часто задаваемых вопросов также есть раздел устранения неполадок ([Frequently Asked Questions](https://docs.celeryq.dev/en/stable/faq.html#faq)).

### Воркер не запускается: Permission Error

* Если вы используете Debian, Ubuntu или другие дистрибутивы на основе Debian: Debian недавно переименовал специальный файл `/dev/shm` в `/run/shm`. Простой обходной путь — создать символическую ссылку:

```bash
# ln -s /run/shm /dev/shm
```

* Другие: Если вы предоставляете какие-либо аргументы [--pidfile](https://docs.celeryq.dev/en/stable/reference/cli.html#cmdoption-celery-worker-pidfile), [--logfile](https://docs.celeryq.dev/en/stable/reference/cli.html#cmdoption-celery-worker-f) или [--statedb](https://docs.celeryq.dev/en/stable/reference/cli.html#cmdoption-celery-worker-S), то вы должны убедиться, что они указывают на файл или каталог, доступный для записи и чтения пользователю, запускающему **worker**.

### Серверная часть результатов не работает или задачи всегда находятся в состоянии PENDING

Все задачи по умолчанию находятся в состоянии [PENDING](https://docs.celeryq.dev/en/stable/userguide/tasks.html#std-state-PENDING), поэтому состояние лучше было бы назвать «**unknown**». Celery не обновляет состояние при отправке задачи, и любая задача без истории считается отложенной (в конце концов, вы знаете идентификатор задачи).

1. Убедитесь, что у задачи не включен **ignore\_result**.
   * Включение этой опции заставит воркер пропустить обновление состояний.
2. Убедитесь, что параметр [task\_ignore\_result](https://docs.celeryq.dev/en/stable/userguide/configuration.html#std-setting-task\_ignore\_result) не включен.
3. Убедитесь, что у вас нет старых воркеров, которые все еще работают.
   * Легко запустить несколько рабочих процессов случайно, поэтому убедитесь, что предыдущий рабочий процесс правильно закрыт, прежде чем запускать новый.
   * Возможно, запущен старый рабочий процесс, для которого не настроен ожидаемый результат, и он перехватывает задачи.
   * Аргумент [--pidfile](https://docs.celeryq.dev/en/stable/reference/cli.html#cmdoption-celery-worker-pidfile) может быть установлен на абсолютный путь, чтобы этого не произошло.
4. Убедитесь, что клиент настроен с правильным бэкэндом.
   * Если по какой-то причине клиент настроен на использование другого бэкэнда, чем воркер, вы не сможете получить результат. Убедитесь, что серверная часть настроена правильно:

```python
>>> result = task.delay()
>>> print(result.backend)
```
