.. _intro:

========================
Celery 简介
========================

.. contents::
    :local:
    :depth: 1

何为任务队列?
=====================

任务队列是一种在线程或机器间分发任务的机制。

消息队列的输入是工作的一个单元，称为任务，独立的职程（Worker）进程持续监视队列中是否有需要处理的新任务。

Celery 通常使用中间人(broker)进行信息通信在客户端和职称(workers)中间. 流程：开始一个任务, 客户端添加一个信息到队列, 中间人(broker) 传达到一个worker.

Celery系统可以包含多个workers 和 brokers, 以获取高可用性和横向扩展。

Celery由Python编写, 但协议可以用任何语言实现.  迄今，已有 Ruby 实现的 RCelery_ node.js 实现的
node-celery_ 和 `PHP client`_. 语言互通也可以通过 :ref:`using webhooks <guide-webhooks>`实现。

.. _RCelery: http://leapfrogdevelopment.github.com/rcelery/
.. _`PHP client`: https://github.com/gjedeer/celery-php
.. _node-celery: https://github.com/mher/node-celery

需要什么？
===============

.. sidebar:: 版本需求
    :subtitle: Celery version 3.0 runs on

    - Python ❨2.5, 2.6, 2.7, 3.2, 3.3❩
    - PyPy ❨1.8, 1.9❩
    - Jython ❨2.5, 2.7❩.

    这是最后一个支持 Python 2.5 的版本，也即从下个版本需要 Python 2.6 或更新版本的 Python。最后一个支持 Python 2.4 的版本为 Celery 2.2 系列。

*Celery* 需要一个发送和接受消息的传输者。RabbitMQ 和 Redis 中间人的消息传输支持所有特性，但也提供大量其他实验性方案的支持，包括用 SQLite 进行本地开发。

Celery 可以单机运行，也可以在多台机器上运行，甚至可以跨越数据中心运行。

开始
===========

如果这是你第一次尝试 Celery，或你从以前版本刚步入 Celery 3.0，那么你应该阅读一下我们的入门教程：

- :ref:`first-steps`
- :ref:`next-steps`

Celery 是…
==========

.. _`mailing-list`: http://groups.google.com/group/celery-users

.. topic:: \ 

    - **简单**

        Celery易于使用和维护，并且它*不需要配置文件*。

        Celery 有一个活跃、友好的社区来让你寻求帮助，包括一个`mailing-list`_ 和 :ref:`IRC channel <irc-channel>`.

        下面是一个你可以实现的最简应用：

        .. code-block:: python

            from celery import Celery

            app = Celery('hello', broker='amqp://guest@localhost//')

            @app.task
            def hello():
                return 'hello world'

    - **高可用性**

        倘若连接丢失或失败，职程和客户端会自动重试，并且一些中间人通过 **主/主** 或 **主/从** 方式复制来提高可用性。

    - **快速**

        单个 Celery 进程每分钟可处理数以百万计的任务，而保持往返延迟在亚毫秒级（使用 RabbitMQ、py-librabbitmq 和优化过的设置）。

    - **灵活**

        Celery 几乎所有部分都可以扩展或单独使用。可以自制连接池、 序列化、压缩模式、日志、调度器、消费者、生产者、自动扩展、 中间人传输或更多。

.. topic:: 它支持

    .. hlist::
        :columns: 2

        - **中间人**

            - :ref:`RabbitMQ <broker-rabbitmq>`, :ref:`Redis <broker-redis>`,
            - :ref:`MongoDB <broker-mongodb>` (exp), ZeroMQ (exp)
            - :ref:`CouchDB <broker-couchdb>` (exp), :ref:`SQLAlchemy <broker-sqlalchemy>` (exp)
            - :ref:`Django ORM <broker-django>` (exp), :ref:`Amazon SQS <broker-sqs>`, (exp)
            - and more…

        - **并发(Concurrency)**

            - prefork (multiprocessing),
            - Eventlet_, gevent_
            - threads/single threaded

        - **结果存储**

            - AMQP, Redis
            - memcached, MongoDB
            - SQLAlchemy, Django ORM
            - Apache Cassandra

        - **序列化**

            - *pickle*, *json*, *yaml*, *msgpack*.
            - *zlib*, *bzip2* 压缩.
            - 密码学消息签名.

特性
========

.. topic:: \ 

    .. hlist::
        :columns: 2

        - **监视**

            整条流水线的监视时间由职程发出，并用于内建或外部的工具告知你集群的工作状况——而且是实时的。

            :ref:`深入了解… <guide-monitoring>`.

        - **工作流**

            一系列功能强大的称为“Canvas”的原语（Primitive）用于构建或简单、或复杂的工作流。包括分组、连锁、分割等等。

            :ref:`深入了解… <guide-canvas>`.

        - **时间和速率限制**

            你可以控制每秒/分钟/小时执行的任务数，或任务的最长运行时间， 并且可以为特定职程或不同类型的任务设置为默认值。

            :ref:`深入了解… <worker-time-limits>`.

        - **计划任务**

            你可以指定任务在若干秒后或在
            :class:`~datetime.datetime`运行，或你可以基于单纯的时间间隔或支持分钟、小时、每周的第几天、每月的第几天以及每年的第几月的 crontab 表达式来使用周期任务来重现事件。

            :ref:`Read more… <guide-beat>`.

        - **自动重载入**

            在开发中，职程可以配置为在源码修改时自动重载入，包含对 Linux 上的 inotify(7) 支持。

            :ref:`Read more… <worker-autoreloading>`.

        - **自动扩展**

            根据负载自动重调职程池的大小或用户指定的测量值，用于限制共享主机/云环境的内存使用，或是保证给定的服务质量。

            :ref:`Read more… <worker-autoscaling>`.

        - **资源泄露保护**

            The :option:`--maxtasksperchild` 选项用于控制用户任务泄露的诸如内存或文件描述符这些易超出掌控的资源。

            :ref:`Read more… <worker-maxtasksperchild>`.

        - **用户组件**

            每个职程组件都可以自定义，并且额外组件可以由用户定义。职程是用 “bootsteps” 构建的——一个允许细粒度控制职程内构件的依赖图。

.. _`Eventlet`: http://eventlet.net/
.. _`gevent`: http://gevent.org/

框架集成
=====================

Celery is easy to integrate with web frameworks, some of which even have
integration packages:

    +--------------------+------------------------+
    | `Django`_          | `django-celery`_       |
    +--------------------+------------------------+
    | `Pyramid`_         | `pyramid_celery`_      |
    +--------------------+------------------------+
    | `Pylons`_          | `celery-pylons`_       |
    +--------------------+------------------------+
    | `Flask`_           | not needed             |
    +--------------------+------------------------+
    | `web2py`_          | `web2py-celery`_       |
    +--------------------+------------------------+
    | `Tornado`_         | `tornado-celery`_      |
    +--------------------+------------------------+

The integration packages are not strictly necessary, but they can make
development easier, and sometimes they add important hooks like closing
database connections at :manpage:`fork(2)`.

.. _`Django`: http://djangoproject.com/
.. _`Pylons`: http://pylonshq.com/
.. _`Flask`: http://flask.pocoo.org/
.. _`web2py`: http://web2py.com/
.. _`Bottle`: http://bottlepy.org/
.. _`Pyramid`: http://docs.pylonsproject.org/en/latest/docs/pyramid.html
.. _`pyramid_celery`: http://pypi.python.org/pypi/pyramid_celery/
.. _`django-celery`: http://pypi.python.org/pypi/django-celery
.. _`celery-pylons`: http://pypi.python.org/pypi/celery-pylons
.. _`web2py-celery`: http://code.google.com/p/web2py-celery/
.. _`Tornado`: http://www.tornadoweb.org/
.. _`tornado-celery`: http://github.com/mher/tornado-celery/

Quickjump
=========

.. topic:: I want to ⟶

    .. hlist::
        :columns: 2

        - :ref:`get the return value of a task <task-states>`
        - :ref:`use logging from my task <task-logging>`
        - :ref:`learn about best practices <task-best-practices>`
        - :ref:`create a custom task base class <task-custom-classes>`
        - :ref:`add a callback to a group of tasks <canvas-chord>`
        - :ref:`split a task into several chunks <canvas-chunks>`
        - :ref:`optimize the worker <guide-optimizing>`
        - :ref:`see a list of built-in task states <task-builtin-states>`
        - :ref:`create custom task states <custom-states>`
        - :ref:`set a custom task name <task-names>`
        - :ref:`track when a task starts <task-track-started>`
        - :ref:`retry a task when it fails <task-retry>`
        - :ref:`get the id of the current task <task-request-info>`
        - :ref:`know what queue a task was delivered to <task-request-info>`
        - :ref:`see a list of running workers <monitoring-control>`
        - :ref:`purge all messages <monitoring-control>`
        - :ref:`inspect what the workers are doing <monitoring-control>`
        - :ref:`see what tasks a worker has registered <monitoring-control>`
        - :ref:`migrate tasks to a new broker <monitoring-control>`
        - :ref:`see a list of event message types <event-reference>`
        - :ref:`contribute to Celery <contributing>`
        - :ref:`learn about available configuration settings <configuration>`
        - :ref:`receive email when a task fails <conf-error-mails>`
        - :ref:`get a list of people and companies using Celery <res-using-celery>`
        - :ref:`write my own remote control command <worker-custom-control-commands>`
        - :ref:`change worker queues at runtime <worker-queues>`

.. topic:: Jump to ⟶

    .. hlist::
        :columns: 4

        - :ref:`Brokers <brokers>`
        - :ref:`Applications <guide-app>`
        - :ref:`Tasks <guide-tasks>`
        - :ref:`Calling <guide-calling>`
        - :ref:`Workers <guide-workers>`
        - :ref:`Daemonizing <daemonizing>`
        - :ref:`Monitoring <guide-monitoring>`
        - :ref:`Optimizing <guide-optimizing>`
        - :ref:`Security <guide-security>`
        - :ref:`Routing <guide-routing>`
        - :ref:`Configuration <configuration>`
        - :ref:`Django <django>`
        - :ref:`Contributing <contributing>`
        - :ref:`Signals <signals>`
        - :ref:`FAQ <faq>`
        - :ref:`API Reference <apiref>`

.. include:: ../includes/installation.txt
