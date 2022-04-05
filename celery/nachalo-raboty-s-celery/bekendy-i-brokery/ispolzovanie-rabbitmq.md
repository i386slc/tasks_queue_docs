# Использование RabbitMQ

## Установка и настройка

**RabbitMQ** — это брокер по умолчанию, поэтому он не требует никаких дополнительных зависимостей или начальной настройки, кроме расположения URL-адреса экземпляра брокера, который вы хотите использовать:

```python
broker_url = 'amqp://myuser:mypassword@localhost:5672/myvhost'
```

Описание URL-адресов брокера и полный список различных параметров конфигурации брокера, доступных для Celery, см. в разделе «[Настройки брокера](https://docs.celeryq.dev/en/stable/userguide/configuration.html#conf-broker-settings)» и см. ниже сведения о настройке имени пользователя, пароля и виртуального хоста.

## Установка сервера RabbitMQ

См. [Установка RabbitMQ](https://www.rabbitmq.com/download.html) на веб-сайте RabbitMQ. Для macOS см. [Установка RabbitMQ в macOS](https://docs.celeryq.dev/en/stable/getting-started/backends-and-brokers/rabbitmq.html#installing-rabbitmq-on-macos).

{% hint style="info" %}
Если вы получаете ошибки **nodedown** после установки и использования **rabbitmqctl**, эта запись в блоге может помочь вам определить источник проблемы:

[http://www.somic.org/2009/02/19/on-rabbitmqctl-and-badrpcnodedown/](http://www.somic.org/2009/02/19/on-rabbitmqctl-and-badrpcnodedown/)
{% endhint %}

## Настройка RabbitMQ

Чтобы использовать Celery, нам нужно создать пользователя RabbitMQ, виртуальный хост и разрешить этому пользователю доступ к этому виртуальному хосту:

```bash
$ sudo rabbitmqctl add_user myuser mypassword

$ sudo rabbitmqctl add_vhost myvhost

$ sudo rabbitmqctl set_user_tags myuser mytag

$ sudo rabbitmqctl set_permissions -p myvhost myuser ".*" ".*" ".*"
```

Замените соответствующие значения для **myuser**, **mypassword** и **myvhost** выше.

Дополнительную информацию об управлении доступом см. в [Руководстве администратора RabbitMQ](http://www.rabbitmq.com/admin-guide.html).

## Установка RabbitMQ на macOS

Самый простой способ установить RabbitMQ на macOS — использовать [Homebrew](https://github.com/mxcl/homebrew/), новую и блестящую систему управления пакетами для macOS.

Сначала установите Homebrew с помощью однострочной команды, предоставленной [документацией Homebrew](https://github.com/Homebrew/homebrew/wiki/Installation):

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Наконец, мы можем установить RabbitMQ с помощью **brew**:

```bash
$ brew install rabbitmq
```

После того, как вы установили RabbitMQ с **brew**, вам нужно добавить следующее в свой путь, чтобы иметь возможность запускать и останавливать брокера: добавьте его в файл запуска для вашей оболочки (например, `.bash_profile` или `.profile`).

```bash
PATH=$PATH:/usr/local/sbin
```

## Настройка имени хоста системы

Если вы используете DHCP-сервер, который дает вам случайное имя хоста, вам необходимо постоянно настраивать имя хоста. Это связано с тем, что RabbitMQ использует имя хоста для связи с узлами.

Используйте команду **scutil**, чтобы навсегда установить имя хоста:

```bash
$ sudo scutil --set HostName myhost.local
```

Затем добавьте это имя хоста в `/etc/hosts`, чтобы можно было преобразовать его обратно в IP-адрес:

```bash
127.0.0.1       localhost myhost myhost.local
```

Если вы запустите **rabbitmq-server**, ваш узел Rabbit теперь должен быть _rabbit@myhost_, как проверено **rabbitmqctl**:

```bash
$ sudo rabbitmqctl status
Status of node rabbit@myhost ...
[{running_applications,[{rabbit,"RabbitMQ","1.7.1"},
                    {mnesia,"MNESIA  CXC 138 12","4.4.12"},
                    {os_mon,"CPO  CXC 138 46","2.2.4"},
                    {sasl,"SASL  CXC 138 11","2.1.8"},
                    {stdlib,"ERTS  CXC 138 10","1.16.4"},
                    {kernel,"ERTS  CXC 138 10","2.13.4"}]},
{nodes,[rabbit@myhost]},
{running_nodes,[rabbit@myhost]}]
...done.
```

Это особенно важно, если ваш DHCP-сервер дает вам имя хоста, начинающееся с IP-адреса (например, `23.10.112.31.comcast.net`). В этом случае RabbitMQ попытается использовать `rabbit@23`: недопустимое имя хоста.

## Запуск/остановка сервера RabbitMQ

Чтобы запустить сервер:

```bash
$ sudo rabbitmq-server
```

вы также можете запустить его в фоновом режиме, добавив параметр `-detached` (примечание: только один дефис):

```bash
$ sudo rabbitmq-server -detached
```

Никогда не используйте команду **kill** (_kill(1)_) для остановки сервера RabbitMQ, а используйте команду **rabbitmqctl**:

```bash
$ sudo rabbitmqctl stop
```

Когда сервер запущен, вы можете продолжить чтение [Настройка RabbitMQ](ispolzovanie-rabbitmq.md#nastroika-rabbitmq).
