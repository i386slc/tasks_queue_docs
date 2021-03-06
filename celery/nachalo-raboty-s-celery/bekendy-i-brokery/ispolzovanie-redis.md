# Использование Redis

## Установка

Для поддержки Redis вам необходимо установить дополнительные зависимости. Вы можете установить как Celery, так и эти зависимости за один раз, используя [пакет](../vvedenie-v-celery.md#pakety) **celery\[redis]**:

```bash
$ pip install -U "celery[redis]"
```

## Конфигурирование

Настройка проста, просто настройте расположение вашей базы данных Redis:

```python
app.conf.broker_url = 'redis://localhost:6379/0'
```

Где URL-адрес имеет формат:

```bash
redis://:password@hostname:port/db_number
```

все поля после схемы являются необязательными и по умолчанию будут иметь значение **localhost** на порту **6379**, используя базу данных **0**.

Если необходимо использовать соединение сокета Unix, URL-адрес должен быть в формате:

```bash
redis+socket:///path/to/redis.sock
```

Указание другого номера базы данных при использовании сокета Unix возможно путем добавления параметра **virtual\_host** к URL-адресу:

```bash
redis+socket:///path/to/redis.sock?virtual_host=db_number
```

Также легко подключиться напрямую к списку Redis **Sentinel**:

```python
app.conf.broker_url = (
    'sentinel://localhost:26379;sentinel://localhost:26380;sentinel://localhost:26381'
)
app.conf.broker_transport_options = { 'master_name': "cluster1" }
```

Дополнительные параметры можно передать клиенту Sentinel с помощью **sentinel\_kwargs**:

```python
app.conf.broker_transport_options = {
    'sentinel_kwargs':
        { 'password': "password" }
}
```

## Время ожидания видимости

Тайм-аут видимости определяет количество секунд, в течение которых рабочий процесс подтвердит выполнение задачи, прежде чем сообщение будет повторно доставлено другому рабочему процессу. Обязательно ознакомьтесь с предостережениями ниже.

Этот параметр задается параметром [**broker\_transport\_options**](https://docs.celeryq.dev/en/stable/userguide/configuration.html#std-setting-broker\_transport\_options):

```python
app.conf.broker_transport_options = {'visibility_timeout': 3600}  # 1 час.
```

Время ожидания видимости по умолчанию для Redis составляет 1 час.

## Результаты

Если вы также хотите хранить состояние и возвращаемые значения задач в Redis, вам следует настроить следующие параметры:

```python
app.conf.result_backend = 'redis://localhost:6379/0'
```

Полный список параметров, поддерживаемых серверной частью результатов Redis, см. в разделе [Параметры серверной части Redis](https://docs.celeryq.dev/en/stable/userguide/configuration.html#conf-redis-result-backend).

Если вы используете **Sentinel**, вы должны указать **master\_name**, используя настройку [result\_backend\_transport\_options](https://docs.celeryq.dev/en/stable/userguide/configuration.html#std-setting-result\_backend\_transport\_options):

```python
app.conf.result_backend_transport_options = {'master_name': "mymaster"}
```

### Тайм-ауты подключения

Чтобы настроить время ожидания подключения для серверной части результатов Redis, используйте ключ **retry\_policy** в разделе [result\_backend\_transport\_options](https://docs.celeryq.dev/en/stable/userguide/configuration.html#std-setting-result\_backend\_transport\_options):

```python
app.conf.result_backend_transport_options = {
    'retry_policy': {
       'timeout': 5.0
    }
}
```

См. [retry\_over\_time()](https://docs.celeryq.dev/projects/kombu/en/master/reference/kombu.utils.functional.html#kombu.utils.functional.retry\_over\_time) для возможных вариантов политики повтора.

## Предостережения

### Время ожидания видимости

Если задача не подтверждена в течение [времени ожидания видимости](ispolzovanie-redis.md#vremya-ozhidaniya-vidimosti), задача будет повторно доставлена другому воркеру и выполнена.

Это вызывает проблемы с задачами ETA/обратного отсчета/повторных попыток, когда время выполнения превышает время ожидания видимости; на самом деле, если это произойдет, он будет выполняться снова и снова в цикле.

Таким образом, вы должны увеличить время ожидания видимости, чтобы оно соответствовало времени самого длинного ETA, которое вы планируете использовать.

Обратите внимание, что Celery будет повторно доставлять сообщения при завершении рабочего процесса, поэтому длительный тайм-аут видимости задержит повторную доставку «потерянных» задач только в случае сбоя питания или принудительного завершения рабочих процессов.

Тайм-аут видимости не повлияет на периодические задачи, так как это концепция, не связанная с ETA/обратным отсчетом.

Вы можете увеличить это время ожидания, настроив параметр транспорта с тем же именем:

```python
app.conf.broker_transport_options = {'visibility_timeout': 43200}
```

Значение должно быть целым числом, описывающим количество секунд.

### Выселение ключей

В некоторых случаях Redis может удалять ключи из базы данных.

Если вы столкнулись с ошибкой, например:

```bash
InconsistencyError: Probably the key ('_kombu.binding.celery') has been
removed from the Redis database.
```

тогда вы можете настроить сервер redis так, чтобы он не удалял ключи, установив в файле конфигурации redis:

* параметр **maxmemory**
* параметр **maxmemory-policy** для **noeviction** или **allkeys-lru**

Дополнительные сведения см. в документации сервера Redis о политиках выселения: [https://redis.io/topics/lru-cache](https://redis.io/topics/lru-cache)

### Групповой порядок результатов

Версии Celery до 4.4.6 включительно использовали несортированный список для хранения объектов результатов для групп в серверной части Redis. Это может привести к тому, что эти результаты будут возвращены в другом порядке, чем связанные с ними задачи в исходном экземпляре группы. В Celery 4.4.7 появилось поведение отказа, которое устраняет эту проблему и гарантирует, что групповые результаты возвращаются в том же порядке, в котором были определены задачи, что соответствует поведению других бэкэндов. В Celery 5.0 это поведение было изменено на отказ. Поведение управляется опцией конфигурации **result\_chord\_ordered**, которая может быть установлена следующим образом:

```python
# Указание этого для рабочих процессов,
# работающих под управлением Celery 4.4.6 или более ранней версии,
# не имеет никакого эффекта.
app.conf.result_backend_transport_options = {
    'result_chord_ordered': True    # или False
}
```

Это несовместимое изменение поведения рабочих процессов во время выполнения, использующих один и тот же бэкенд Redis для хранения результатов, поэтому все рабочие процессы должны следовать либо новому, либо старому поведению, чтобы избежать поломки. Для кластеров с некоторыми рабочими процессами, работающими под управлением Celery 4.4.6 или более ранней версии, это означает, что рабочие процессы, работающие под управлением 4.4.7, не нуждаются в специальной настройке, а рабочие процессы, работающие под управлением версии 5.0 или более поздней, должны иметь для параметра **result\_chord\_ordered** значение `False`. Для кластеров, в которых нет рабочих процессов с версией 4.4.6 или более ранней, но есть несколько рабочих процессов с версией 4.4.7, рекомендуется установить для параметра **result\_chord\_ordered** значение `True` для всех рабочих процессов, чтобы упростить миграцию в будущем. Миграция между поведениями приведет к нарушению результатов, которые в настоящее время хранятся в серверной части Redis, и приведет к поломке, если последующие задачи будут выполняться перенесенными workers — планируйте соответствующим образом.
