Таги Dependency Injection
=========================

Список доступных тагов:

* ``data_collector``
* ``form.type``
* ``form.type_extension``
* ``form.type_guesser``
* ``kernel.cache_warmer``
* ``kernel.event_listener``
* ``kernel.event_subscriber``
* ``monolog.logger``
* ``monolog.processor``
* ``templating.helper``
* ``routing.loader``
* ``translation.loader``
* ``twig.extension``
* ``validator.initializer``

Подключаем пользовательские хелперы в PHP шаблоны
-------------------------------------------------

Для подключения пользовательского хелпера, добавьте его в виде службы
в один из ваших конфигурационных файлов, пометьте его тагом ``templating.helper``
и определите атрибут ``alias`` (в шаблонах хелпер будет доступен через
этот алиас):

.. configuration-block::

    .. code-block:: yaml

        services:
            templating.helper.your_helper_name:
                class: Fully\Qualified\Helper\Class\Name
                tags:
                    - { name: templating.helper, alias: alias_name }

    .. code-block:: xml

        <service id="templating.helper.your_helper_name" class="Fully\Qualified\Helper\Class\Name">
            <tag name="templating.helper" alias="alias_name" />
        </service>

    .. code-block:: php

        <?php
        $container
            ->register('templating.helper.your_helper_name', 'Fully\Qualified\Helper\Class\Name')
            ->addTag('templating.helper', array('alias' => 'alias_name'))
        ;

.. _reference-dic-tags-twig-extension:

Подключение пользовательских расширений для Twig
------------------------------------------------

Для активации расширения Twig добавьте его в виде службы в один
из ваших конфигурационных файлов и пометьте эту службу тагом
``twig.extension``:

.. configuration-block::

    .. code-block:: yaml

        services:
            twig.extension.your_extension_name:
                class: Fully\Qualified\Extension\Class\Name
                tags:
                    - { name: twig.extension }

    .. code-block:: xml

        <service id="twig.extension.your_extension_name" class="Fully\Qualified\Extension\Class\Name">
            <tag name="twig.extension" />
        </service>

    .. code-block:: php

        <?php
        $container
            ->register('twig.extension.your_extension_name', 'Fully\Qualified\Extension\Class\Name')
            ->addTag('twig.extension')
        ;

О том, как создавать расширения для Twig, читайте в `документации Twig`_.

.. _dic-tags-kernel-event-listener:

Подключение пользовательских слушателей
---------------------------------------

Для того чтобы подключить слушателя (listener), добавьте его в виде
службы в один из ваших конфигурационных файлов и пометьте её тагом
``kernel.event_listener``. Вы можете указать наименование события,
которое будет слушать ваша служба, а также метод, который будет
вызван:

.. configuration-block::

    .. code-block:: yaml

        services:
            kernel.listener.your_listener_name:
                class: Fully\Qualified\Listener\Class\Name
                tags:
                    - { name: kernel.event_listener, event: xxx, method: onXxx }

    .. code-block:: xml

        <service id="kernel.listener.your_listener_name" class="Fully\Qualified\Listener\Class\Name">
            <tag name="kernel.event_listener" event="xxx" method="onXxx" />
        </service>

    .. code-block:: php

        <?php
        $container
            ->register('kernel.listener.your_listener_name', 'Fully\Qualified\Listener\Class\Name')
            ->addTag('kernel.event_listener', array('event' => 'xxx', 'method' => 'onXxx'))
        ;

.. note::

    Вы также можете указать приоритет (положительное или отрицательное целое число)
    в виде атрибута тага kernel.event_listener (также как метод или событие). Это
    позволяет вам управлять очерёдностью вызова слушателей одного и того же события.

.. _dic-tags-kernel-event-subscriber:

Подключение пользовательских Подписчиков
----------------------------------------

.. versionadded:: 2.1

   Возможность добавления подписчиков событий доступна начиная с версии 2.1.

Для активации подписчика, добавьте его в виде службы в один из конфигурационных
файлов и пометьте его тагом ``kernel.event_subscriber``:

.. configuration-block::

    .. code-block:: yaml

        services:
            kernel.subscriber.your_subscriber_name:
                class: Fully\Qualified\Subscriber\Class\Name
                tags:
                    - { name: kernel.event_subscriber }

    .. code-block:: xml

        <service id="kernel.subscriber.your_subscriber_name" class="Fully\Qualified\Subscriber\Class\Name">
            <tag name="kernel.event_subscriber" />
        </service>

    .. code-block:: php

        <?php
        $container
            ->register('kernel.subscriber.your_subscriber_name', 'Fully\Qualified\Subscriber\Class\Name')
            ->addTag('kernel.event_subscriber')
        ;

.. note::

    Служба должна реализовывать интерфейс
    :class:`Symfony\Component\EventDispatcher\EventSubscriberInterface`.

.. note::

    Если ваша служба создаётся фабрикой, вы **ДОЛЖНЫ** корректно указать параметр
    ``class`` для этого тага, иначе работать не будет.

Подключение пользовательских шаблонизаторов
-------------------------------------------

Для того чтобы активировать ещё один шаблонизатор, добавьте его
в виде службы в один из конфигурационных файлов и пометьте её тагом
``templating.engine``:

.. configuration-block::

    .. code-block:: yaml

        services:
            templating.engine.your_engine_name:
                class: Fully\Qualified\Engine\Class\Name
                tags:
                    - { name: templating.engine }

    .. code-block:: xml

        <service id="templating.engine.your_engine_name" class="Fully\Qualified\Engine\Class\Name">
            <tag name="templating.engine" />
        </service>

    .. code-block:: php

        <?php
        $container
            ->register('templating.engine.your_engine_name', 'Fully\Qualified\Engine\Class\Name')
            ->addTag('templating.engine')
        ;

Подключение пользовательских загрузчиков для Маршрутов
------------------------------------------------------

Для того чтобы подключить загрузчик маршрутов, добавьте его
в виде службы в один из конфигурационных файлов и пометьте её тагом ``routing.loader``:

.. configuration-block::

    .. code-block:: yaml

        services:
            routing.loader.your_loader_name:
                class: Fully\Qualified\Loader\Class\Name
                tags:
                    - { name: routing.loader }

    .. code-block:: xml

        <service id="routing.loader.your_loader_name" class="Fully\Qualified\Loader\Class\Name">
            <tag name="routing.loader" />
        </service>

    <?php
    .. code-block:: php

        $container
            ->register('routing.loader.your_loader_name', 'Fully\Qualified\Loader\Class\Name')
            ->addTag('routing.loader')
        ;

.. _dic_tags-monolog:

Использование отдельного канала для логгирования в Monolog
----------------------------------------------------------

Монолог позволяет вам делить обработчики между несколькими каналами логгирования.
Служба логгера использует канал ``app``, но вы можете изменять канал при
инъекции логгера в службу.

.. configuration-block::

    .. code-block:: yaml

        services:
            my_service:
                class: Fully\Qualified\Loader\Class\Name
                arguments: [@logger]
                tags:
                    - { name: monolog.logger, channel: acme }

    .. code-block:: xml

        <service id="my_service" class="Fully\Qualified\Loader\Class\Name">
            <argument type="service" id="logger" />
            <tag name="monolog.logger" channel="acme" />
        </service>

    .. code-block:: php

        <?php
        $definition = new Definition('Fully\Qualified\Loader\Class\Name', array(new Reference('logger'));
        $definition->addTag('monolog.logger', array('channel' => 'acme'));
        $container->register('my_service', $definition);;

.. note::

    Это работает, только когда служба логгера инжектится через аргумент конструктора,
    при использовании сеттера работать не будет.

.. _dic_tags-monolog-processor:

Добавление процессоров для Monolog
----------------------------------

Monolog позволяет вам добавлять процессоры в логгер или в хэндлеры для
добавления дополнительных данных в записи. Процессор получает запись в
качестве аргумента и должен вернуть её после добавления дополнительных
данных в атрибут записи ``extra``.

Давайте посмотрим, как вы можете использовать встроенный ``IntrospectionProcessor``
для добавления файла, строки, класса и метода, где был вызван логгер.

Вы можете добавить процессор глобально:

.. configuration-block::

    .. code-block:: yaml

        services:
            my_service:
                class: Monolog\Processor\IntrospectionProcessor
                tags:
                    - { name: monolog.processor }

    .. code-block:: xml

        <service id="my_service" class="Monolog\Processor\IntrospectionProcessor">
            <tag name="monolog.processor" />
        </service>

    .. code-block:: php

        <?php
        $definition = new Definition('Monolog\Processor\IntrospectionProcessor');
        $definition->addTag('monolog.processor');
        $container->register('my_service', $definition);

.. tip::

    Если служба не может быть вызвана (при помощи ``__invoke``) вы можете
    добавить атрибут ``method`` в таг, чтобы использовать указанный метод.

Вы также можете добавить процессор для некоторого хэндлера при помощи атрибута
``handler``:

.. configuration-block::

    .. code-block:: yaml

        services:
            my_service:
                class: Monolog\Processor\IntrospectionProcessor
                tags:
                    - { name: monolog.processor, handler: firephp }

    .. code-block:: xml

        <service id="my_service" class="Monolog\Processor\IntrospectionProcessor">
            <tag name="monolog.processor" handler="firephp" />
        </service>

    .. code-block:: php

        <?php
        $definition = new Definition('Monolog\Processor\IntrospectionProcessor');
        $definition->addTag('monolog.processor', array('handler' => 'firephp');
        $container->register('my_service', $definition);

Вы также можете добавить процессор для некоторого канала логгинга при помощи
атрибута ``channel``. Этот атрибут зарегистрирует процессор только для канала
``security``, используемого в компоненте Security:

.. configuration-block::

    .. code-block:: yaml

        services:
            my_service:
                class: Monolog\Processor\IntrospectionProcessor
                tags:
                    - { name: monolog.processor, channel: security }

    .. code-block:: xml

        <service id="my_service" class="Monolog\Processor\IntrospectionProcessor">
            <tag name="monolog.processor" channel="security" />
        </service>

    .. code-block:: php

        <?php
        $definition = new Definition('Monolog\Processor\IntrospectionProcessor');
        $definition->addTag('monolog.processor', array('channel' => 'security');
        $container->register('my_service', $definition);

.. note::

    Вы не можете использовать оба атрибута ``handler`` и ``channel`` для одного
    и того же тага так как хэндлеры доступны для всех каналов.

..  _`документации Twig`: http://twig.sensiolabs.org/doc/extensions.html

.. toctree::
    :hidden:

    Translation source: 2011-11-28 b4bbe36
    Corrected from:
