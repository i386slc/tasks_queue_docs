# Введение в Celery

В этом документе описывается текущая стабильная версия **Celery** (5.2). Для документации по разработке [перейдите сюда](http://docs.celeryproject.org/en/master/index.html).

## Введение в Celery

* Что такое очередь задач?
* Что вам нужно?
* Начинаем
* Celery это...
* Особенности
* Интеграция с фреймворком
* Быстрый переход
* Установка

## Что такое очередь задач ?

Очереди задач используются как механизм для распределения работы между потоками или машинами.

Вход очереди задач — это единица работы, называемая задачей. Выделенные рабочие процессы постоянно отслеживают очереди задач для выполнения новой работы.

**Celery** общается с помощью сообщений, обычно используя брокера в качестве посредника между клиентами и работниками. Чтобы инициировать задачу, клиент добавляет сообщение в очередь, затем брокер доставляет это сообщение исполнителю.

Система **Celery** может состоять из нескольких рабочих процессов и брокеров, уступая место высокой доступности и горизонтальному масштабированию.

**Celery** написан на Python, но протокол можно реализовать на любом языке. В дополнение к Python есть [node-celery](https://github.com/mher/node-celery) и [node-celery-ts](https://github.com/IBM/node-celery-ts) для **Node.js** и [PHP-клиент](https://github.com/gjedeer/celery-php).

Языковая совместимость также может быть достигнута путем раскрытия конечной точки HTTP и наличия задачи, которая ее запрашивает (веб-перехватчики).

## Что вам нужно ?

**Celery** требует транспорта сообщений для отправки и получения сообщений. Брокерские транспорты **RabbitMQ** и **Redis** имеют полную функциональность, но также есть поддержка множества других экспериментальных решений, включая использование **SQLite** для локальной разработки.

**Celery** может работать на одной машине, на нескольких машинах или даже в центрах обработки данных.

## Начинаем

Если вы впервые пытаетесь использовать **Celery** или если вы не следите за развитием в версии 3.1 и переходите с предыдущих версий, вам следует прочитать наши руководства по началу работы:

* [Первые шаги с Celery](pervye-shagi-s-celery.md)
* [Следующие шаги](sleduyushie-shagi.md)

{% hint style="info" %}
**Требования к версии**

**Celery** версии 5.2 работает на

* Python ❨3.7, 3.8, 3.9, 3.10❩
* PyPy3.7, 3.8 ❨7.3.7❩

Celery 4.x была последней версией, поддерживающей Python 2.7, для Celery 5.x требуется Python 3.6 или новее. Для Celery 5.1.x также требуется Python 3.6 или новее. Для Celery 5.2.x требуется Python 3.7 или новее.

Если вы используете более старую версию Python, вам необходимо использовать более старую версию **Celery**:

* Python 2.7 или Python 3.5: серия Celery 4.4 или более ранняя версия.
* Python 2.6: серия Celery 3.1 или более ранняя версия.
* Python 2.5: серия Celery 3.0 или более ранняя версия.
* Python 2.4 работал с серией Celery 2.2 или более ранней.

**Celery** — проект с минимальным финансированием, поэтому мы не поддерживаем Microsoft Windows. Пожалуйста, не открывайте никаких вопросов, связанных с этой платформой.
{% endhint %}

## Celery это...

* **Простота** - Celery прост в использовании и обслуживании, и ему _**не нужны файлы конфигурации**_. У него есть активное, дружелюбное сообщество, с которым вы можете обратиться за поддержкой, включая список рассылки и канал IRC. Вот одно из самых простых приложений, которые вы можете сделать:

```python
from celery import Celery

app = Celery('hello', broker='amqp://guest@localhost//')

@app.task
def hello():
    return 'hello world'
```

* **Высокая доступность** - Рабочие процессы и клиенты будут автоматически повторять попытку в случае потери или сбоя подключения, а некоторые брокеры поддерживают высокую доступность в виде репликации _Primary/Primary_ или _Primary/Replica_.
* **Быстрота** - Один процесс Celery может обрабатывать миллионы задач в минуту с двусторонней задержкой менее миллисекунды (с использованием **RabbitMQ**, **librabbitmq** и оптимизированных настроек).
* **Гибкость** - Почти каждая часть Celery может быть расширена или использована сама по себе, реализация пользовательских пулов, сериализаторы, схемы сжатия, ведение журнала, планировщики, потребители, производители, брокерские транспорты и многое другое.

### Поддерживает

* Брокеры
  * [RabbitMQ](https://docs.celeryq.dev/en/stable/getting-started/backends-and-brokers/rabbitmq.html#broker-rabbitmq)
  * [Redis](https://docs.celeryq.dev/en/stable/getting-started/backends-and-brokers/redis.html#broker-redis)
  * [Amazon SQS](https://docs.celeryq.dev/en/stable/getting-started/backends-and-brokers/sqs.html#broker-sqs)
  * и другие
* Параллелизм
  * prefork (многопроцессность)
  * [Eventlet](http://eventlet.net), [gevent](http://gevent.org)
  * thread (многопоточность)
  * solo (один поток)
* Хранилища результатов
  * AMQP, Redis
  * Memcached
  * SQLAlchemy, Django ORM
  * Apache Cassandra, ElasticSearch, Riak
  * MongoDB, CouchDB, Couchbase, ArangoDB
  * Amazon DynamoDB, Amazon S3
  * Microsoft Azure Block Blob, Microsoft Azure Cosmos DB
  * File system
* Сериализация
  * pickle, json, yaml, msgpack
  * сжатие zlib, bzip2
  * Подписание криптографических сообщений

## Особенности

* **Мониторинг** - Поток событий мониторинга создается рабочими процессами и используется встроенными и внешними инструментами, чтобы сообщить вам, что делает ваш кластер, в режиме реального времени. [Подробнее...](https://docs.celeryq.dev/en/stable/userguide/monitoring.html#guide-monitoring)
* **Рабочие процессы** - Простые и сложные рабочие процессы могут быть составлены с использованием набора мощных примитивов, которые мы называем **Canvas**, включая группировку, цепочку, фрагментацию и многое другое. [Подробнее...](https://docs.celeryq.dev/en/stable/userguide/canvas.html#guide-canvas)
* **Ограничения по времени и скорости** - Вы можете контролировать, сколько задач может выполняться в секунду/минуту/час или как долго задача может выполняться, и это может быть установлено по умолчанию для конкретного _worker_ или индивидуально для каждого типа задач. [Подробнее...](https://docs.celeryq.dev/en/stable/userguide/workers.html#worker-time-limits)
* **Планирование** - Вы можете указать время для запуска задачи в секундах или [datetime](https://docs.python.org/dev/library/datetime.html#datetime.datetime), или вы можете использовать периодические задачи для повторяющихся событий на основе простого интервала или выражений **Crontab**, поддерживающих минуты, часы, день недели, день месяца и месяц года. [Подробнее...](https://docs.celeryq.dev/en/stable/userguide/periodic-tasks.html#guide-beat)
* **Защита от утечки ресурсов** - Параметр [--max-tasks-per-child](https://docs.celeryq.dev/en/stable/reference/cli.html#cmdoption-celery-worker-max-tasks-per-child) используется для пользовательских задач с утечкой ресурсов, таких как память или файловые дескрипторы, которые просто находятся вне вашего контроля. [Подробнее...](https://docs.celeryq.dev/en/stable/userguide/workers.html#worker-max-tasks-per-child)
* **Пользовательские компоненты** - Каждый рабочий компонент может быть настроен пользователем, а дополнительные компоненты могут быть определены пользователем. Рабочий процесс создается с использованием «boot-steps» — графа зависимостей, позволяющего детально контролировать внутренние компоненты рабочего процесса.

## Интеграция с фреймворком

Celery легко интегрируется с веб-фреймворками, некоторые из них даже имеют пакеты интеграции:

| Фреймворк                                                            | Пакет интеграции                                                 |
| -------------------------------------------------------------------- | ---------------------------------------------------------------- |
| [Pyramid](http://docs.pylonsproject.org/en/latest/docs/pyramid.html) | [pyramid\_celery](https://pypi.python.org/pypi/pyramid\_celery/) |
| [Pylons](http://pylonshq.com)                                        | [celery-pylons](https://pypi.python.org/pypi/celery-pylons/)     |
| [Flask](http://flask.pocoo.org)                                      | не требуется                                                     |
| [web2py](http://web2py.com)                                          | [web2py-celery](https://pypi.python.org/pypi/web2py-celery/)     |
| [Tornado](http://www.tornadoweb.org)                                 | [tornado-celery](https://pypi.python.org/pypi/tornado-celery/)   |
| [Tryton](http://www.tryton.org)                                      | [celery\_tryton](https://pypi.python.org/pypi/celery\_tryton/)   |

Для [Django](https://djangoproject.com) см. [Первые шаги с Django](https://docs.celeryq.dev/en/stable/django/first-steps-with-django.html#django-first-steps).

Пакеты интеграции не являются строго необходимыми, но они могут упростить разработку, а иногда они добавляют важные хуки, такие как закрытие соединений с базой данных в _fork(2)_.

## Быстрый переход

### я бы хотел

* [get the return value of a task](https://docs.celeryq.dev/en/stable/userguide/tasks.html#task-states)
* [use logging from my task](https://docs.celeryq.dev/en/stable/userguide/tasks.html#task-logging)
* [learn about best practices](https://docs.celeryq.dev/en/stable/userguide/tasks.html#task-best-practices)
* [create a custom task base class](https://docs.celeryq.dev/en/stable/userguide/tasks.html#task-custom-classes)
* [add a callback to a group of tasks](https://docs.celeryq.dev/en/stable/userguide/canvas.html#canvas-chord)
* [split a task into several chunks](https://docs.celeryq.dev/en/stable/userguide/canvas.html#canvas-chunks)
* [optimize the worker](https://docs.celeryq.dev/en/stable/userguide/optimizing.html#guide-optimizing)
* [see a list of built-in task states](https://docs.celeryq.dev/en/stable/userguide/tasks.html#task-builtin-states)
* [create custom task states](https://docs.celeryq.dev/en/stable/userguide/tasks.html#custom-states)
* [set a custom task name](https://docs.celeryq.dev/en/stable/userguide/tasks.html#task-names)
* [track when a task starts](https://docs.celeryq.dev/en/stable/userguide/tasks.html#task-track-started)
* [retry a task when it fails](https://docs.celeryq.dev/en/stable/userguide/tasks.html#task-retry)
* [get the id of the current task](https://docs.celeryq.dev/en/stable/userguide/tasks.html#task-request-info)
* [know what queue a task was delivered to](https://docs.celeryq.dev/en/stable/userguide/tasks.html#task-request-info)
* [see a list of running workers](https://docs.celeryq.dev/en/stable/userguide/monitoring.html#monitoring-control)
* [purge all messages](https://docs.celeryq.dev/en/stable/userguide/monitoring.html#monitoring-control)
* [inspect what the workers are doing](https://docs.celeryq.dev/en/stable/userguide/monitoring.html#monitoring-control)
* [see what tasks a worker has registered](https://docs.celeryq.dev/en/stable/userguide/monitoring.html#monitoring-control)
* [migrate tasks to a new broker](https://docs.celeryq.dev/en/stable/userguide/monitoring.html#monitoring-control)
* [see a list of event message types](https://docs.celeryq.dev/en/stable/userguide/monitoring.html#event-reference)
* [contribute to Celery](https://docs.celeryq.dev/en/stable/contributing.html#contributing)
* [learn about available configuration settings](https://docs.celeryq.dev/en/stable/userguide/configuration.html#configuration)
* [get a list of people and companies using Celery](https://docs.celeryq.dev/en/stable/community.html#res-using-celery)
* [write my own remote control command](https://docs.celeryq.dev/en/stable/userguide/workers.html#worker-custom-control-commands)
* [change worker queues at runtime](https://docs.celeryq.dev/en/stable/userguide/workers.html#worker-queues)

### перейти к

* [Brokers](https://docs.celeryq.dev/en/stable/getting-started/backends-and-brokers/index.html#brokers)
* [Applications](https://docs.celeryq.dev/en/stable/userguide/application.html#guide-app)
* [Tasks](https://docs.celeryq.dev/en/stable/userguide/tasks.html#guide-tasks)
* [Calling](https://docs.celeryq.dev/en/stable/userguide/calling.html#guide-calling)
* [Workers](https://docs.celeryq.dev/en/stable/userguide/workers.html#guide-workers)
* [Daemonizing](https://docs.celeryq.dev/en/stable/userguide/daemonizing.html#daemonizing)
* [Monitoring](https://docs.celeryq.dev/en/stable/userguide/monitoring.html#guide-monitoring)
* [Optimizing](https://docs.celeryq.dev/en/stable/userguide/optimizing.html#guide-optimizing)
* [Security](https://docs.celeryq.dev/en/stable/userguide/security.html#guide-security)
* [Routing](https://docs.celeryq.dev/en/stable/userguide/routing.html#guide-routing)
* [Configuration](https://docs.celeryq.dev/en/stable/userguide/configuration.html#configuration)
* [Django](https://docs.celeryq.dev/en/stable/django/index.html#django)
* [Contributing](https://docs.celeryq.dev/en/stable/contributing.html#contributing)
* [Signals](https://docs.celeryq.dev/en/stable/userguide/signals.html#signals)
* [FAQ](https://docs.celeryq.dev/en/stable/faq.html#faq)
* [API Reference](https://docs.celeryq.dev/en/stable/reference/index.html#apiref)

## Установка

Вы можете установить Celery либо через индекс пакетов Python (PyPI), либо из исходного кода.

Для установки с помощью **pip**:

```python
$ pip install -U Celery
```

### Пакеты

Celery также определяет группу пакетов, которые можно использовать для установки Celery и зависимостей для данной функции.

Вы можете указать их в своих требованиях или в командной строке **pip**, используя скобки. Можно указать несколько пакетов, разделив их запятыми.

```python
$ pip install "celery[librabbitmq]"

$ pip install "celery[librabbitmq,redis,auth,msgpack]"
```

Доступны следующие пакеты:

### Сериализаторы

* **celery\[auth]** - для использования сериализатора безопасности **auth**.
* **celery\[msgpack]** - для использования сериализатора **msgpack**.
* **celery\[yaml]** - для использования сериализатора **yaml**.

### Параллелизм

* **celery\[eventlet]** - для использования пула событий [eventlet](https://pypi.python.org/pypi/eventlet/).
* **celery\[gevent]** - для использования пула [gevent](https://pypi.python.org/pypi/gevent/).

### Транспорты и бэкенды

* **celery\[librabbitmq]** - для использования библиотеки librabbitmq C.
* **celery\[redis]** - для использования Redis в качестве транспорта сообщений или в качестве серверной части.
* **celery\[sqs]** - для использования Amazon SQS в качестве транспорта сообщений (экспериментально).
* **celery\[tblib]** - для использования возможности [task\_remote\_tracebacks](https://docs.celeryq.dev/en/stable/userguide/configuration.html#std-setting-task\_remote\_tracebacks).
* **celery\[memcache]** - для использования Memcached в качестве бэкэнда (используя [pylibmc](https://pypi.python.org/pypi/pylibmc/))
* **celery\[pymemcache]** - для использования Memcached в качестве бэкэнда (реализация на чистом Python).
* **celery\[cassandra]** - для использования Apache Cassandra в качестве бэкэнда с драйвером DataStax.
* **celery\[couchbase]** - за использование Couchbase в качестве бэкэнда.
* **celery\[arangodb]** - за использование ArangoDB в качестве бэкэнда.
* **celery\[elasticsearch]** - за использование Elasticsearch в качестве серверной части результатов.
* **celery\[riak]** - за использование Riak в качестве бэкэнда.
* **celery\[dynamodb]** - для использования AWS DynamoDB в качестве серверной части.
* **celery\[zookeeper]** - для использования Zookeeper в качестве транспорта для сообщений.
* **celery\[sqlalchemy]** - для использования SQLAlchemy в качестве бэкэнда (поддерживается).
* **celery\[pyro]** - для использования транспорта сообщений Pyro4 (экспериментальный).
* **celery\[slmq]** - для использования транспорта SoftLayer Message Queue (экспериментальный).
* **celery\[consul]** - для использования хранилища ключей/значений Consul.io в качестве транспорта сообщений или бэкенда результатов (экспериментально).
* **celery\[django]** - указывает самую низкую версию, возможную для поддержки Django. Вероятно, вам не следует использовать это в своих требованиях, оно здесь только для информационных целей.

### Загрузка и установка из исходников

Загрузите последнюю версию Celery из PyPI: [https://pypi.org/project/celery/](https://pypi.org/project/celery/)

Вы можете установить его, выполнив следующие действия:

```python
$ tar xvfz celery-0.0.0.tar.gz
$ cd celery-0.0.0
$ python setup.py build
# python setup.py install
```

Последняя команда должна быть выполнена как привилегированный пользователь, если вы в настоящее время не используете virtualenv.

### Использование версии для разработки

#### С pip

Для разработки версии Celery также требуются версии для разработки [kombu](https://pypi.python.org/pypi/kombu/), [amqp](https://pypi.python.org/pypi/amqp/), [billiard](https://pypi.python.org/pypi/billiard/) и [vine](https://pypi.python.org/pypi/vine/).

Вы можете установить их последний снимок, используя следующие команды **pip**:

```python
$ pip install https://github.com/celery/celery/zipball/master#egg=celery
$ pip install https://github.com/celery/billiard/zipball/master#egg=billiard
$ pip install https://github.com/celery/py-amqp/zipball/master#egg=amqp
$ pip install https://github.com/celery/kombu/zipball/master#egg=kombu
$ pip install https://github.com/celery/vine/zipball/master#egg=vine
```

#### С git

Пожалуйста, смотрите раздел [Contributing](https://docs.celeryq.dev/en/stable/contributing.html#contributing).
