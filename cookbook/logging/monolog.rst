.. index::
   single: Журналирование

Как использовать Monolog для журналирования
===========================================
Библиотека Monolog_ предназначена для ведения журналов в PHP 5.3 
и используется Symfony2. Её прототипом послужила библиотека LogBook
в Python.

Использование
--------------
В Монологе, каждый элемент журналирования(логгер) определяет свой 
канал журналирования(logger). Каждый канал имеет стек обработчиков 
которые пишут журнал (лог) (причем обработчики могут быть общими).

.. tip::
    При установке логгера в сервис, можно использовать :ref:`свой канал<dic_tags-monolog>`
    и легко просматривать какая часть приложения оставила сообщение в журнале.

Простейшим обработчиком является ``StreamHandler``, который пишет журнал
в поток (по-умолчанию в ``app/logs/prod.log`` в production среде и
``app/logs/dev.log`` в среде разработки).

В состав Monolog также входит мощный обработчик, предназначенный для журналирования
в production среде: ``FingersCrossedHandler``. Он позволяет хранить сообщения в буфере
и записывать их в журнал только при условии того, что оно доходит до уровня контроллера 
(ERROR в конфигурации стандартной редакции)  перенаправляя их к другому обработчику.

Чтобы записать сообщение в журнал, просто получите доступ к сервису логгера
из контейнера в вашем контроллере::

    $logger = $this->get('logger');
    $logger->info('We just go the logger');
    $logger->err('An error occured');

.. tip::
    Использование методов интерфейса 
    :class:`Symfony\Component\HttpKernel\Log\LoggerInterface`
    позволит изменять реализацию логгера без внесения 
    изменений в ваш код.
    
Использование нескольких обработчиков
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Логер использует стек обработчиков которые вызываются последовательно.
Данная особенность позволяет легко записывать сообщения в журнал 
различными способами.

.. configuration-block::

    .. code-block:: yaml

        monolog:
            handlers:
                syslog:
                    type: stream
                    path: /var/log/symfony.log
                    level: error
                main:
                    type: fingerscrossed
                    action_level: warning
                    handler: file
                file:
                    type: stream
                    level: debug

    .. code-block:: xml

        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:monolog="http://symfony.com/schema/dic/monolog"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                                http://symfony.com/schema/dic/monolog http://symfony.com/schema/dic/monolog/monolog-1.0.xsd">

            <monolog:config>
                <monolog:handler
                    name="syslog"
                    type="stream"
                    path="/var/log/symfony.log"
                    level="error"
                />
                <monolog:handler
                    name="main"
                    type="fingerscrossed"
                    action-level="warning"
                    handler="file"
                />
                <monolog:handler
                    name="file"
                    type="stream"
                    level="debug"
                />
            </monolog:config>
        </container>

Конфигурация выше, определяет стек обработчиков которые будут вызваны в порядке
в котором они объявлены.

.. tip::
    Обработчик "file" не будет включен в стек, так как он сам используется в 
    качестве вложенного обработчика в production среде.

.. note::
    Если у вас появиться желание изменить настройки MonologBundle в другом
    файле настроек, то необходимо будет полностью переопределить весь стек.
    Он не может быть объединен с текущими настройками, т.к. в результате 
    объединения настроек невозможно управлять порядком вызова обработчиков.
    

Изменение форматирования
~~~~~~~~~~~~~~~~~~~~~~~~

Обработчик использует ``Formatter`` для форматирования записей, перед записью их в журнал.
Все обработчики Monolog по-умолчанию используют экземпляр ``Monolog\Formatter\LineFormatter``,
но его легко заменить своим собственным. Ваш собственный форматировщик должен использовать интерфейс
``Monolog\Formatter\LineFormatterInterface``.

.. configuration-block::

    .. code-block:: yaml

        services:
            my_formatter:
                class: Monolog\Formatter\JsonFormatter
        monolog:
            handlers:
                file:
                    type: stream
                    level: debug
                    formatter: my_formatter

    .. code-block:: xml

        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:monolog="http://symfony.com/schema/dic/monolog"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                                http://symfony.com/schema/dic/monolog http://symfony.com/schema/dic/monolog/monolog-1.0.xsd">

            <services>
                <service id="my_formatter" class="Monolog\Formatter\JsonFormatter" />
            </services>
            <monolog:config>
                <monolog:handler
                    name="file"
                    type="stream"
                    level="debug"
                    formatter="my_formatter"
                />
            </monolog:config>
        </container>


Дополнительная информация в сообщениях журнала
----------------------------------------------

Monolog позволяет добавлять дополнительные данные в сообщения 
перед их записью в журнал. Процессор может быть применен как ко всему стеку 
так и к какому-либо определенному обработчику из его состава.

Процессор - это сервис получающий запись в качестве первого аргумента и
логгер или обработчик в качестве второго, в зависимости от того на каком уровне
он вызывается.

.. configuration-block::

    .. code-block:: yaml

        services:
            my_processor:
                class: Monolog\Processor\WebProcessor
        monolog:
            handlers:
                file:
                    type: stream
                    level: debug
                    processors:
                        - Acme\MyBundle\MyProcessor::process
            processors:
                - @my_processor

    .. code-block:: xml

        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:monolog="http://symfony.com/schema/dic/monolog"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                                http://symfony.com/schema/dic/monolog http://symfony.com/schema/dic/monolog/monolog-1.0.xsd">

            <services>
                <service id="my_processor" class="Monolog\Processor\WebProcessor" />
            </services>
            <monolog:config>
                <monolog:handler
                    name="file"
                    type="stream"
                    level="debug"
                    formatter="my_formatter"
                >
                    <monolog:processor callback="Acme\MyBundle\MyProcessor::process" />
                </monolog:handler />
                <monolog:processor callback="@my_processor" />
            </monolog:config>
        </container>

.. tip::
    Если вашему процессору требуются зависимости, то можно объявить
    сервис и реализовать метод ``__invoke`` в классе, с тем чтобы сделать
    его вызываемым. После изменений процессор можно добавить в стек.

.. _Monolog: https://github.com/Seldaek/monolog
