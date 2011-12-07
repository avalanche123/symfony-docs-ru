.. index::
   single: Internals

Составные части
===============

Похоже, что вы хотите понять, как работает Symfony2 и как его расширить.
Это радует! Этот раздел подробно объясняет внутренности Symfony2.

.. note::

    Чтение этого раздела необходимо, только если вы хотите понять, как работает
    Symfony2 за кулисами или если хотите расширять Symfony2.

Обзор
-----

Код Symfony2 сделан из нескольких независимых слоёв. Каждый следующий слой
надстраивается на предыдущем.

.. tip::

    Автозагрузка не управляется непосредственно фреймворком; она выполняется
    независимо с помощью класса
    :class:`Symfony\\Component\\ClassLoader\\UniversalClassLoader`
    и файла ``src/autoload.php``. За дополнительной информацией обращайтесь к
    :doc:`разделу </cookbook/tools/autoloader>`, посвящённому этой теме.

Компонент ``HttpFoundation``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

На самом глубоком уровне находится компонент :namespace:`Symfony\\Component\\HttpFoundation`.
HttpFoundation предоставляет основные объекты, необходимые для работы с HTTP.
Это объектно-ориентированная абстракция некоторых встроенных PHP функций и
переменных:

* Класс :class:`Symfony\\Component\\HttpFoundation\\Request` абстрагирует
  основные глобальные переменные в PHP, такие как ``$_GET``, ``$_POST``,
  ``$_COOKIE``, ``$_FILES`` и ``$_SERVER``;

* Класс :class:`Symfony\\Component\\HttpFoundation\\Response` абстрагирует
  некоторые PHP функции типа ``header()``, ``setcookie()`` и ``echo``;

* Класс :class:`Symfony\\Component\\HttpFoundation\\Session` и
  :class:`Symfony\\Component\\HttpFoundation\\SessionStorage\\SessionStorageInterface`
  абстрагируют функции ``session_*()`` для управления сессией.

Компонент ``HttpKernel``
~~~~~~~~~~~~~~~~~~~~~~~~

Поверх HttpFoundation располагается компонент :namespace:`Symfony\\Component\\HttpKernel`.
HttpKernel управляет динамической частью HTTP; это тонкая обёртка поверх классов
Request и Response, которая приводит способы обработки запросов к стандарту.
Компонент также предоставляет точки для расширений и инструменты, делающие
его идеальной стартовой площадкой для создания Web фреймворка без лишних проблем.

Также, дополнительно, он добавляет настраиваемость и расширяемость благодаря
компоненту Dependency Injection и мощной системе пакетов (Bundles).

.. seealso::

    Узнайте больше о компоненте :doc:`HttpKernel <kernel>`. Узнайте больше о
    :doc:`Dependency Injection </book/service_container>` и :doc:`Пакетах
    </cookbook/bundles/best_practices>`.

Пакет ``FrameworkBundle``
~~~~~~~~~~~~~~~~~~~~~~~~~

:namespace:`Symfony\\Bundle\\FrameworkBundle` это пакет, связывающий
основные компоненты и библиотеки вместе, что создаёт лёгкий и быстрый MVC
фреймворк. Он поставляется с правильной первоначальной конфигурацией и
соглашениями для облегчения изучения.

.. index::
   single: Internals; Kernel

Ядро (Kernel)
-------------

Класс :class:`Symfony\\Component\\HttpKernel\\HttpKernel` - это центральный
класс в Symfony2 и он в ответе за обработку клиентских запросов. Его главная
цель - "превратить" объект :class:`Symfony\\Component\\HttpFoundation\\Request`
в объект :class:`Symfony\\Component\\HttpFoundation\\Response`.

Каждый Symfony2 Kernel наследует
:class:`Symfony\\Component\\HttpKernel\\HttpKernelInterface`::

    function handle(Request $request, $type = self::MASTER_REQUEST, $catch = true)

.. index::
   single: Internals; Controller Resolver

Контроллеры (Controllers)
~~~~~~~~~~~~~~~~~~~~~~~~~

При преобразования запроса в ответ, Kernel полагается на "Controller". Контроллер
может быть любой валидной PHP-сущностью, которую можно вызвать тем или иным образом.

Ядро делегирует право выбора запустить тот или иной контроллер классу,
реализующему интерфейс
:class:`Symfony\\Component\\HttpKernel\\Controller\\ControllerResolverInterface`::

    public function getController(Request $request);

    public function getArguments(Request $request, $controller);

Метод
:method:`Symfony\\Component\\HttpKernel\\Controller\\ControllerResolverInterface::getController`
возвращает контроллер (PHP callable - функцию, метод, замыкание...), ассоциированный
с данным запросом. Каноническая реализация
(:class:`Symfony\\Component\\HttpKernel\\Controller\\ControllerResolver`)
ищет атрибут запроса ``_controller``, который хранит наименование контроллера
(строку "class::method", например ``Bundle\BlogBundle\PostController:indexAction``).

.. tip::

    Реализация по умолчанию использует
    :class:`Symfony\\Bundle\\FrameworkBundle\\EventListener\\RouterListener`
    для определения атрибута ``_controller`` из запроса (see :ref:`kernel-core-request`).

Метод
:method:`Symfony\\Component\\HttpKernel\\Controller\\ControllerResolverInterface::getArguments`
возвращает массив аргументов для передачи их в контроллер. Реализация по умолчанию
автоматически определяет аргументы, основываясь на атрибутах запроса.

.. sidebar:: Сопоставление аргументов метода контроллера по атрибутам запроса

    Для каждого аргумента метода Symfony2 пытается получить из запроса значение
    атрибута с таким же именем. Если он не определён, используется значение по
    умолчанию (если оно также определено)::

        // Symfony2 будет искать обязательный атрибут 'id'
        // и опциональный атрибут 'admin'
        public function showAction($id, $admin = true)
        {
            // ...
        }

.. index::
  single: Internals; Request Handling

Обработка запросов
~~~~~~~~~~~~~~~~~~

Метод ``handle()`` принимает ``Request`` и *всегда* возвращает ``Response``.
При конвертации объекта ``Request``, ``handle()`` полагается на Resolver и
упорядоченную цепь нотификаций о событиях (Event notifications, см. следующую
секцию для более подробной информации о каждом событии из этой цепи):

1. Перед тем как что-либо делать, срабатывает нотификация о событии ``kernel.request`` --
   если один из слушателей (listeners) возвращает объект ``Response``, процесс
   сразу переходит к шагу 8;

2. Вызывается Resolver для определения Контроллера, который необходимо выполнить;

3. Слушатели события ``kernel.controller`` теперь могут манипулировать
   методом Контроллера (изменить, обернуть...);

4. Kernel проверяет, что Контроллер представляет собой валидный PHP callable;

5. Для определения аргументов Контроллера вызывается Resolver;

6. Kernel выполняет Контроллер;

7. Если Контроллер не возвращает объект ``Response``, слушатели события
   ``kernel.view`` могут конвертировать данные, которые вернул Контроллер
   в объект ``Response``;

8. Слушатели события ``kernel.response`` могут манипулировать объектом ``Response`` (
   контент и заголовки);

9. Возвращается Ответ.

Если во время этого процесса возникает исключительная ситуация, срабатывает
событие ``kernel.exception`` и его слушатели получают возможность конвертировать
исключение (Exception) в Ответ. Если это удаётся, событие уведомляется, если нет,
исключение вызывается повторно.

Если вы не хотите, чтобы возникали исключения (для вложенных запросов, к примеру),
отключите событие ``kernel.exception`` передав ``false`` в качестве третьего аргумента
метода ``handle()``.

.. index::
  single: Internals; Internal Requests

Внутренние Запросы
~~~~~~~~~~~~~~~~~~

В любой момент во время обработки запроса (назовём его 'мастер'), может быть обработан
подзапрос. Вы можете передать тип запроса в метод ``handle()`` его вторым
параметром:

* ``HttpKernelInterface::MASTER_REQUEST``;
* ``HttpKernelInterface::SUB_REQUEST``.

Тип также передаётся во все события, и их слушатели могут действовать в
соответствии с переданным типом (некоторые действия могут соответствовать
только мастер-запросу).

.. index::
   pair: Kernel; Event

События
~~~~~~~

Каждое событие, создаваемое в Kernel, это дочерний класс
:class:`Symfony\\Component\\HttpKernel\\Event\\KernelEvent`. Это означает, что
каждое событие имеет доступ к одной и той же базовой информации:

* ``getRequestType()`` - возвращает *тип* запроса
  (``HttpKernelInterface::MASTER_REQUEST`` или ``HttpKernelInterface::SUB_REQUEST``);

* ``getKernel()`` - возвращает экземпляр Kernel, обрабатывающий этот запрос;

* ``getRequest()`` - возвращает объект ``Request``, соответствующий обрабатываемому запросу;

``getRequestType()``
....................

Метод ``getRequestType()`` позволяет слушателям узнавать тип запроса.
Например, если слушатель должен быть активен только для мастер-запроса,
добавьте следующий код в начало вашего "слушающего" метода:

.. code-block:: php

    <?php
    use Symfony\Component\HttpKernel\HttpKernelInterface;

    if (HttpKernelInterface::MASTER_REQUEST !== $event->getRequestType()) {
        // немедленно возвращаемся
        return;
    }

.. tip::

    Если вы ещё не знакомы с Диспетчером Событий Symfony2 (Event Dispatcher),
    прочитайте сначала секцию :ref:`event_dispatcher`.

.. index::
   single: Event; kernel.request

.. _kernel-core-request:

Событие ``kernel.request``
........................

*Класс события*: :class:`Symfony\\Component\\HttpKernel\\Event\\GetResponseEvent`

Цель этого события - либо незамедлительно вернуть объект ``Response``,
или же подготовить переменные, чтобы можно было вызвать контроллер после
события. Любой слушатель может вернуть объект ``Response`` при помощи
метода события ``setResponse()``. В этом случае, все остальные слушатели
не будут вызываться.

Это событие используется в ``FrameworkBundle`` для заполнения атрибута ``_controller``
в объекте ``Request`` при помощи класса
:class:`Symfony\\Bundle\\FrameworkBundle\\EventListener\\RouterListener`. RequestListener
использует объект, реализующий интерфейс :class:`Symfony\\Component\\Routing\\RouterInterface`
для согласования объекта ``Request`` и определения наименования Контроллера (которое хранится
в атрибуте ``_controller`` объекта ``Request``).

.. index::
   single: Event; kernel.controller

Событие ``kernel.controller``
...........................

*Класс события*: :class:`Symfony\\Component\\HttpKernel\\Event\\FilterControllerEvent`

Это событие не используется в ``FrameworkBundle``, но оно может быть точкой входа,
используемой для модификации исполняемого контроллера:

.. code-block:: php

    <?php
    use Symfony\Component\HttpKernel\Event\FilterControllerEvent;

    public function onKernelController(FilterControllerEvent $event)
    {
        $controller = $event->getController();
        // ...

        // the controller can be changed to any PHP callable
        $event->setController($controller);
    }

.. index::
   single: Event; kernel.view

Событие ``kernel.view``
.....................

*Класс события*: :class:`Symfony\\Component\\HttpKernel\\Event\\GetResponseForControllerResultEvent`

Это событие не используется в ``FrameworkBundle``, но оно может быть использовано
для реализации подсистемы view. Это событие вызывается *только* если Контроллер
*не* возвращает объект ``Response``. Назначение этого события - разрешить конвертацию
возвращаемых значений в объект ``Response``.

Значение, возвращаемое Контроллером доступно при помощи метода ``getControllerResult``:

.. code-block:: php

    <?php
    use Symfony\Component\HttpKernel\Event\GetResponseForControllerResultEvent;
    use Symfony\Component\HttpFoundation\Response;

    public function onKernelView(GetResponseForControllerResultEvent $event)
    {
        $val = $event->getReturnValue();
        $response = new Response();
        // код получения объекта Response из полученного значения

        $event->setResponse($response);
    }

.. index::
   single: Event; kernel.response

Событие ``kernel.response``
.........................

*Класс события*: :class:`Symfony\\Component\\HttpKernel\\Event\\FilterResponseEvent`

Назначение этого события - позволить другим системам модифицировать или
заменять объект ``Response`` после его создания:

.. code-block:: php

    <?php
    public function onKernelResponse(FilterResponseEvent $event)
    {
        $response = $event->getResponse();
        // .. modify the response object
    }

``FrameworkBundle`` регистрирует несколько слушателей:

* :class:`Symfony\\Component\\HttpKernel\\EventListener\\ProfilerListener`:
  собирает данные для текущего запроса;

* :class:`Symfony\\Bundle\\WebProfilerBundle\\EventListener\\WebDebugToolbarListener`:
  внедряет Web Debug Toolbar;

* :class:`Symfony\\Component\\HttpKernel\\EventListener\\ResponseListener`:
  устанавливает ``Content-Type`` ответа, основываясь на формате запроса;

* :class:`Symfony\\Component\\HttpKernel\\EventListener\\EsiListener`:
  добавляет заголовок ``Surrogate-Control``, в случае если ответ необходимо
  парсить на предмет наличия ESI тагов.

.. index::
   single: Event; kernel.exception

.. _kernel-kernel.exception:

Событие ``kernel.exception``
..........................

*Класс события*: :class:`Symfony\\Component\\HttpKernel\\Event\\GetResponseForExceptionEvent`

``FrameworkBundle`` регистрирует :class:`Symfony\\Component\\HttpKernel\\EventListener\\ExceptionListener`,
который перенаправляет ``Request`` в указанные Контроллер (определяется
значением параметра ``exception_listener.controller``, указывается в нотации ``class::method``).

Слушатель этого события может создавать объект ``Response``, создавать новый объект
``Exception`` или же ничего не делать:

.. code-block:: php

    <?php
    use Symfony\Component\HttpKernel\Event\GetResponseForExceptionEvent;
    use Symfony\Component\HttpFoundation\Response;

    public function onKernelException(GetResponseForExceptionEvent $event)
    {
        $exception = $event->getException();
        $response = new Response();
        // Настраиваем объект Response, основываясь на перехваченном исключении
        $event->setResponse($response);

        // как вариант - вы можете создать новое исключение
        // $exception = new \Exception('Some special exception');
        // $event->setException($exception);
    }

.. index::
   single: Event Dispatcher

.. _`book-internals-event-dispatcher`:

Диспетчер событий (Event Dispatcher)
------------------------------------

Объектно-ориентированный код прошёл длинный путь по обеспечению расширяемости
кода. Путём создания узкоспециализированных классов, ваш код становится более
гибким и разработчик может расширять его при помощи дочерних классов, чтобы
изменять их поведение. Но что, если требуется использовать его изменения
совместно с другими разработчиками, которые также создают свои дочерние классы?
Здесь использование наследования уже не столь удобно.

Рассмотрим реальный пример, в котором вам нужно создать систему плагинов для
вашего проекта. Плагин должен иметь возможность добавлять методы или же делать
что-то до или после выполнения некоторого метода, не пересекаясь с прочими
плагинами. Эту задачу непросто решить при помощи одиночного наследования,
да и множественное наследование (если бы оно было возможно в PHP) имеет
свои недостатки.

Диспетчер событий Symfony2 реализует шаблон проектирования `Observer`_ простым
и эффективным способом, позволяя создавать, например, что-то вроде системы плагинов,
которую упоминали выше, и делая ваш проект действительно расширяемым.

Рассмотрим ещё один простой пример из `Symfony2 HttpKernel component`_.
Когда создаётся объект ``Response``, было бы здорово позволить другим
системам проекта модифицировать его (например, добавить заголовки для
кэширования) перед последующим использованием. Для того, чтобы достичь этого,
ядро Symfony2 создаёт событие - ``kernel.response``. Вот как это работает:

* *Слушатель* (listener, PHP объект) сообщает центральному *диспетчеру*, что
  он собирается слушать (ожидать) событие ``kernel.response``;

* В какой-то момент ядро Symfony2 просит объект *диспетчера* отправить событие
  ``kernel.response``, и вместе с ним - объект ``Response``;

* Диспетчер уведомляет (фактически вызывает метод) всех слушателей события
  ``kernel.response``, позволяя каждому из них выполнить модификацию объекта
  ``Response``.

.. index::
   single: Event Dispatcher; Events

.. _event_dispatcher:

События
~~~~~~~

Когда сообщение отправлено, оно идентифицируется по уникальному имени
(например, ``kernel.response``), которое могут ожидать некоторое число
слушателей. Также создаётся экземпляр класса
:class:`Symfony\\Component\\EventDispatcher\\Event`, который затем передаётся
всем слушателям. Как вы увидите чуть позже, объект ``Event`` часто содержит
данные о направляемом событии.

.. index::
   pair: Event Dispatcher; Naming conventions

Соглашения по именованию
........................

Уникальным именем для события может быть любая строка, но желательно следование
нескольким простым правилам:

* Допустимые символы: буквы в нижнем регистре, цифры, точка (``.``), подчерк (``_``);

* Добавляйте префикс пространства имён с точкой на конце (например, ``kernel.``);

* Оканчивайте имя глаголом, который обозначает действие (например, ``request``).

Вот пара примеров хороших имён для событий:

* ``kernel.response``
* ``form.pre_set_data``

.. index::
   single: Event Dispatcher; Event Subclasses

Объекты событий
...............

Когда диспетчер уведомляет слушателей, он передаёт им объект ``Event``. Базовый
класс ``Event`` очень прост: он содержит метод для прекращения воспроизведения
(:ref:`event propagation<event_dispatcher-event-propagation>`) и ничего более.

Зачастую, необходимо передавать в объекте ``Event`` также данные о событии,
чтобы слушатели могли их обработать тем или иным образом. В случае события
``kernel.response``, объект ``Event``, передаваемый каждому слушателю, фактически
имеет тип :class:`Symfony\\Component\\HttpKernel\\Event\\FilterResponseEvent`,
дочерний по отношению к ``Event`` класс. Этот класс содержит методы, такие
как ``getResponse`` и ``setResponse``, позволяющие слушателям получать и даже
заменять объект ``Response``.

Мораль этой истории в следующем: при создании слушателя некоторого события,
объект ``Event``, который будет передан этому слушателю, может быть
специализированным дочерним классом и иметь дополнительные методы для получения
данных события и их обработки.

Диспетчер
~~~~~~~~~~~~~~

Диспетчер - это центральный объект системы обработки событий. Как правило,
создаётся единственный диспетчер, который обслуживает реестр слушателей. Когда
событие поступает к диспетчеру - он уведомляет всех слушателей, подписанных
на это событие.

.. code-block:: php

    <?php
    use Symfony\Component\EventDispatcher\EventDispatcher;

    $dispatcher = new EventDispatcher();

.. index::
   single: Event Dispatcher; Listeners

Подключаем Слушателей
~~~~~~~~~~~~~~~~~~~~~

Для того, чтобы отреагировать на некое существующее событие, вам необходимо
подключить слушателя к диспетчеру, чтобы последний имел возможность сообщить
о появлении нужного события. Вызов метода диспетчера ``addListener()`` ассоциирует
любую исполнимую функцию/метод с событием:

.. code-block:: php

    <?php
    $listener = new AcmeListener();
    $dispatcher->addListener('foo.action', array($listener, 'onFooAction'));

Метод ``addListener()`` получает три аргумента:

* Наименование события, которое слушатель будет ожидать;

* Некий объект (функцию, в общем же случае PHP callable), который будет вызван
  при наступлении события;

* Опциональный приоритет (чем больше - тем более важный), который определяет
  очерёдность вызова слушателей (по умолчанию ``0``). Если два слушателя имеют
  одинаковый приоритет, они выполняются в порядке их добавления.

.. note::

    `PHP callable`_ - это переменная, которая может быть использована в функции
    ``call_user_func()`` и возвращает ``true`` при проверке с помощью функции
    ``is_callable()``. Это может быть, в том числе, и экземпляр замыкания (``\Closure``),
    строка с именем функции или массив, представляющий собой метод объекта или же
    метод класса.

    Ранее вы уже видели как PHP объект может быть зарегистрирован в качестве слушателя.
    Вы также можете регистрировать Замыкания (`Closures`_) в качестве слушателей:

    .. code-block:: php

        <?php
        use Symfony\Component\EventDispatcher\Event;

        $dispatcher->addListener('foo.action', function (Event $event) {
            // этот код будет вызван при обработке события foo.action
        });

Когда слушатель зарегистрирован диспетчером, он ожидает наступления события.
В примере выше, когда появляется событие ``foo.action``, диспетчер вызывает
метод ``AcmeListener::onFooAction`` и передаёт объекту ``Event`` один аргумент:

.. code-block:: php

    <?php
    use Symfony\Component\EventDispatcher\Event;

    class AcmeListener
    {
        // ...

        public function onFooAction(Event $event)
        {
            // do something
        }
    }

.. tip::

    Если вы используете Symfony2 MVC framework, слушатели могут быть зарегистрированы
    при помощи :ref:`конфигурации <dic-tags-kernel-event-listener>`. В качестве бонуса,
    объект слушателя будет создан лишь когда будет нужен.

Во многих случаях, слушателю передаётся специализированный дочерний класс ``Event``.
Это даёт слушателю доступ к информации о событии. Сверяйтесь с документацией или
реализацией каждого конкретного события для определения какой именно экземпляр
``Symfony\Component\EventDispatcher\Event`` будет передан. Например, событие
``kernel.event`` передаёт экземпляр класса ``Symfony\Component\HttpKernel\Event\FilterResponseEvent``:

.. code-block:: php

    <?php
    use Symfony\Component\HttpKernel\Event\FilterResponseEvent

    public function onKernelResponse(FilterResponseEvent $event)
    {
        $response = $event->getResponse();
        $request = $event->getRequest();

        // ...
    }

.. _event_dispatcher-closures-as-listeners:

.. index::
   single: Event Dispatcher; Creating and Dispatching an Event

Создание и обработка события
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В дополнение к регистрации слушателей для уже существующих событий, вы можете
создавать и вызывать свои собственные события. Это может быть удобно при создании
сторонних библиотек и если вы хотите чтобы различные компоненты вашей системы
были гибкими и независимыми.

Статический класс ``Events``
............................

Предположим, вы хотите создать новое событие - ``store.order`` - которое
создаётся всякий раз, когда в вашем приложении создаётся заказ. Для того,
чтобы поддерживать порядок в приложении, начнём с создания класса ``StoreEvents``,
который будет определять ваше событие:

.. code-block:: php

    <?php
    namespace Acme\StoreBundle;

    final class StoreEvents
    {
        /**
         * Событие store.order создаётся всякий раз, когда в системе создаётся заказ.
         *
         * Слушатель получит экземпляр Acme\StoreBundle\Event\FilterOrderEvent
         *
         * @var string
         */
        const onStoreOrder = 'store.order';
    }

Отметим также, что этот класс по сути свой ничего *не делает*. Назначение
класса ``StoreEvents`` - централизация данных о событии. Слушателям этого события
будет передаваться специализированный класс ``FilterOrderEvent``.

Создание объекта события
........................

Позднее, когда вы будете отправлять это событие, вы создадите экземпляр
класса ``Event`` и передадите этот экземпляр всем слушателям события. Если
вы не хотите передавать никакой дополнительной информации слушателям, вы
можете использовать класс ``Symfony\Component\EventDispatcher\Event``.
В большинстве же случаев, вы *будете* передавать информацию о событии слушателям.
Для этого необходимо создать новый класс, который будет наследоваться от
класса ``Symfony\Component\EventDispatcher\Event``.

В этом примере, каждый слушатель будет должен получить доступ к некоторому объекту
``Order``. Создадим класс ``Event``, который реализует такое поведение:

.. code-block:: php

    <?php
    namespace Acme\StoreBundle\Event;

    use Symfony\Component\EventDispatcher\Event;
    use Acme\StoreBundle\Order;

    class FilterOrderEvent extends Event
    {
        protected $order;

        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        public function getOrder()
        {
            return $this->order;
        }
    }

Каждый слушатель теперь имеет доступ к объекту ``Order`` при помощи метода
``getOrder``.

Отправка события
...............

Метод :method:`Symfony\\Component\\EventDispatcher\\EventDispatcher::dispatch`
уведомляет всех слушателей о событии. Он принимает два аргумента:
наименование события для отправки и экземпляр ``Event`` для передачи
каждому слушателю этого события:

.. code-block:: php

    <?php
    use Acme\StoreBundle\StoreEvents;
    use Acme\StoreBundle\Order;
    use Acme\StoreBundle\Event\FilterOrderEvent;

    // заказ как-то создаётся или получается
    $order = new Order();
    // ...

    // создаём FilterOrderEvent и его отправка
    $event = new FilterOrderEvent($order);
    $dispatcher->dispatch(StoreEvents::onStoreOrder, $event);

Объект ``FilterOrderEvent`` создаётся и передаётся в метод ``dispatch``.
Теперь, любой слушатель события ``store.order`` будет получать ``FilterOrderEvent``
и соответственно иметь доступ к объекту ``Order`` при помощи метода ``getOrder``:

.. code-block:: php

    <?php
    // какой-то слушатель, подписанный на событие store.order методом onStoreOrder
    use Acme\StoreBundle\Event\FilterOrderEvent;

    public function onStoreOrder(FilterOrderEvent $event)
    {
        $order = $event->getOrder();
        // далее выполняются какие-то действия с заказом
    }

Внутри объекта Диспетчера событий
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Если вы взглянете на класс ``EventDispatcher``, вы увидите, что этот класс
работает не как Singleton (нет статического метода ``getInstance()``). Это
сделано преднамеренно, так как вам, возможно, потребуется иметь несколько
конкурирующих диспетчеров в рамках одного запроса. Но это также означает, что
вам нужен способ для передачи диспетчеру объектов, которые нужно подключить или
которые надо уведомить о событии.

Общепринятой практикой является внедрение объекта диспетчера в ваши объекты,
т.е. внедрение зависимости.

Вы можете использовать внедрение в конструктор::

    class Foo
    {
        protected $dispatcher = null;

        public function __construct(EventDispatcher $dispatcher)
        {
            $this->dispatcher = $dispatcher;
        }
    }

Или же внедрение через метод (setter injection)::

    class Foo
    {
        protected $dispatcher = null;

        public function setEventDispatcher(EventDispatcher $dispatcher)
        {
            $this->dispatcher = $dispatcher;
        }
    }

Выбор того или иного метода - это дело вкуса. Многие предпочитают метод с
конструктором, так как объекты полностью инициализируются во время создания.
Но когда у вас имеется длинный список зависимостей, использовать метод-сеттер
это тоже вариант, особенно для опциональных зависимостей.

.. tip::

    Если вы используете внедрение зависимости как мы делали в двух примерах выше,
    вы можете использовать `Symfony2 Dependency Injection component`_ для
    того чтобы управлять внедрением службы ``event_dispatcher`` для этих
    объектов.

        .. code-block:: yaml

            # src/Acme/HelloBundle/Resources/config/services.yml
            services:
                foo_service:
                    class: Acme/HelloBundle/Foo/FooService
                    arguments: [@event_dispatcher]

.. index::
   single: Event Dispatcher; Event subscribers

Подписка на события
~~~~~~~~~~~~~~~~~~~

Типичный способ ожидать возникновение события - зарегистрировать *слушателя события*
при помощи диспетчера. Этот слушатель может слушать одно или несколько событий и
уведомляется каждый раз при отправке нужного события.

Альтернативным способом для ожидания событий - использование *подписчика события*.
Подписчик - это PHP класс, который имеет возможность сообщить диспетчеру
на какие события он подписывается. Подписчик должен реализовывать
интерфейс :class:`Symfony\\Component\\EventDispatcher\\EventSubscriberInterface`,
который требует наличие одного статического метода ``getSubscribedEvents``.
Рассмотрим пример подписчика, который подписывается на события
``kernel.response`` и ``store.order``:

.. code-block:: php

    <?php
    namespace Acme\StoreBundle\Event;

    use Symfony\Component\EventDispatcher\EventSubscriberInterface;
    use Symfony\Component\HttpKernel\Event\FilterResponseEvent;

    class StoreSubscriber implements EventSubscriberInterface
    {
        static public function getSubscribedEvents()
        {
            return array(
                'kernel.response' => 'onKernelResponse',
                'store.order'     => 'onStoreOrder',
            );
        }

        public function onKernelResponse(FilterResponseEvent $event)
        {
            // ...
        }

        public function onStoreOrder(FilterOrderEvent $event)
        {
            // ...
        }
    }

Этот класс похож на класс слушателя, за исключением того, что он сам может
сообщить диспетчеру, на какие именно события он подписывается (будет слушать).
Для регистрации подписчика в диспетчере необходимо использовать метод
:method:`Symfony\\Component\\EventDispatcher\\EventDispatcher::addSubscriber`:

.. code-block:: php

    <?php
    use Acme\StoreBundle\Event\StoreSubscriber;

    $subscriber = new StoreSubscriber();
    $dispatcher->addSubscriber($subscriber);

Диспетчер автоматически зарегистрирует подписчика для каждого события,
возвращаемого методом ``getSubscribedEvents``. Этот метод возвращает массив,
индексами которого служат наименования событий, а значениями служат либо
наименования методов, которые будут вызваны, либо массивы с именем метода и
его приоритетом при обработке события.

.. tip::

    Если вы используете Symfony2 MVC framework, подписчики можно регистрировать
    при помощи :ref:`конфигурации <dic-tags-kernel-event-subscriber>`. В
    качестве приятного бонуса, экземпляр подписчика будет создан лишь
    когда будет нужен.

.. index::
   single: Event Dispatcher; Stopping event flow

.. _event_dispatcher-event-propagation:

Прекращение обработки событий
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В некоторых случаях, один из слушателей может затребовать прекращение обработки
события другими слушателями. Другими словами, слушатель должен иметь возможность
сообщить диспетчеру, что он должен остановить обработку события всеми оставшимися
слушателями (не уведомлять их о событии). Этого можно достигнуть внутри слушателя
при помощи метода :method:`Symfony\\Component\\EventDispatcher\\Event::stopPropagation`:

.. code-block:: php

   <?php
   use Acme\StoreBundle\Event\FilterOrderEvent;

   public function onStoreOrder(FilterOrderEvent $event)
   {
       // ...

       $event->stopPropagation();
   }

Теперь, все слушатели ``store.order``, которые ещё не были уведомлены о событии,
уведомляться уже *не* будут.

.. index::
   single: Profiler

Профайлер
---------

Профайлер Symfony2, если он активирован, собирает полезную информацию о каждом
запросе, выполненном к вашему приложение и сохраняет его для последующего анализа.
Использование профайлера в девелоперском окружении поможет вам в отладке
кода и увеличении быстродействия; используйте его в продуктовой среде для
обнаружения проблем "по факту".

Вам вряд ли придётся часто взаимодействовать с профайлером непосредственно,
так как Symfony2 предоставляет визуализатор по типу Web Debug Toolbar и
Web Profiler. Если вы используете Symfony2 Standard Edition, профайлер,
дебаг-панель и веб-профайлер уже настроены и подключены.

.. note::

    Профайлер собирает информацию обо всех запросах (простые запросы,
    перенаправления, исключения, Ajax запросы, ESI запросы; а также
    о всех HTTP методах и обо всех форматах). Это означает, что для
    одного URL вы можете иметь много профилированных данных (по одному
    на каждую пару запрос/ответ).

.. index::
   single: Profiler; Visualizing

Визуализация данных профайлера
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Использование Web Debug Toolbar
...............................

В dev окружении web debug toolbar расположен в низу каждой страницы. Он отображает
обобщённые данные профайлера и предоставляет доступ к полезной информации,
когда что-либо работает не так как ожидалось.

Если обобщённых данных не хватает, вы можете кликнуть на ссылку с токеном (
строка из 13 случайных символов) и перейти на страницу Web Profiler.

.. note::

    Если токен не кликается, это означает, что маршруты профайлера не
    зарегистрированы (см. ниже информацию о конфигурировании).

Анализ данных в Web Profiler
............................

Web Profiler - это инструмент визуализации данных профилирования, который
вы можете использовать в разработке для отладки вашего кода и увеличения
его быстродействия; но его также можно использовать для отслеживания проблем
в продуктовой среде. Он предоставляет всю информацию, собранную профайлером,
в своём веб-интерфейсе.

.. index::
   single: Profiler; Using the profiler service

Доступ к данным профайлера
..........................

Вам не обязательно использовать визуализатор для доступа к данным
профайлера. Как же вам получить доступ к информации профайлера
для некоторого запроса по факту его выполнения? Когда профайлер сохраняет
данные о запросе, он также ассоциирует с ними некоторый токен; этот
токен доступен в заголовке ответа ``X-Debug-Token``::

    $profile = $container->get('profiler')->loadProfileFromResponse($response);

    $profile = $container->get('profiler')->loadProfile($token);

.. tip::

    Когда профайлер активирован, но нет web debug toolbar, или же когда
    вы хотите получить токен для Ajax запроса, используйте, например,
    Firebug для того, чтобы получить заголовок ``X-Debug-Token``.

Используйте метод ``find()``, для получения доступа к токенам по какому-либо
критерию::

    // получить 10 последних токенов
    $tokens = $container->get('profiler')->find('', '', 10);

    // получить последние 10 токенов для всех URL, содержащих /admin/
    $tokens = $container->get('profiler')->find('', '/admin/', 10);

    // получить последние 10 токенов для локальных запросов
    $tokens = $container->get('profiler')->find('127.0.0.1', '', 10);

Если вы хотите манипулировать данными профайлера на другой машине, используйте
методы ``export()`` и ``import()``::

    // в prod окружении
    $profile = $container->get('profiler')->loadProfile($token);
    $data = $profiler->export($profile);

    // в dev окружении
    $profiler->import($data);

.. index::
   single: Profiler; Visualizing

Конфигурирование
................

Конфигурация по умолчанию содержит разумные настройки профайлера, дебаг-панели
(web debug toolbar) и веб-профайлера (web profiler). Ниже приведён пример
конфигурации для dev окружения:

.. configuration-block::

    .. code-block:: yaml

        # загрузка профайлера
        framework:
            profiler: { only_exceptions: false }

        # активация веб-профайлера
        web_profiler:
            toolbar: true
            intercept_redirects: true
            verbose: true

    .. code-block:: xml

        <!-- xmlns:webprofiler="http://symfony.com/schema/dic/webprofiler" -->
        <!-- xsi:schemaLocation="http://symfony.com/schema/dic/webprofiler http://symfony.com/schema/dic/webprofiler/webprofiler-1.0.xsd"> -->

        <!-- загрузка профайлера -->
        <framework:config>
            <framework:profiler only-exceptions="false" />
        </framework:config>

        <!-- активация веб-профайлера -->
        <webprofiler:config
            toolbar="true"
            intercept-redirects="true"
            verbose="true"
        />

    .. code-block:: php

        <?php
        // загрузка профайлера
        $container->loadFromExtension('framework', array(
            'profiler' => array('only-exceptions' => false),
        ));

        // активация веб-профайлера
        $container->loadFromExtension('web_profiler', array(
            'toolbar' => true,
            'intercept-redirects' => true,
            'verbose' => true,
        ));

Если ``only-exceptions`` имеет значение ``true``, профайлер собирает данные
только при возникновении исключений.

Если ``intercept-redirects`` имеет значение ``true``, профайлер перехватывает
перенаправления и предоставляет вам возможность наблюдать собранные данные
перед перенаправлением.

Если ``verbose`` имеет значение ``true``, Web Debug Toolbar отображает большое
количество данных. Если присвоить ``verbose`` значение ``false``, вторичная
информация не будет отображаться.

Если вы активировали web profiler, вам также необходимо подключить его маршруты:

.. configuration-block::

    .. code-block:: yaml

        _profiler:
            resource: @WebProfilerBundle/Resources/config/routing/profiler.xml
            prefix:   /_profiler

    .. code-block:: xml

        <import resource="@WebProfilerBundle/Resources/config/routing/profiler.xml" prefix="/_profiler" />

    .. code-block:: php

        $collection->addCollection($loader->import("@WebProfilerBundle/Resources/config/routing/profiler.xml"), '/_profiler');

Так как профайлер выполняет дополнительную работу для каждого запроса, вы,
возможно, захотите активировать его в продуктовой среде лишь в некоторых случаях.
Опция ``only-exceptions`` устанавливает лимит профилирования в 500 страниц, но
что, если вы захотите получить информацию, когда IP клиента имеет некоторое
определённое значение или если запрашивается строго определённая часть сайта?
Вы можете использовать request matcher:

.. configuration-block::

    .. code-block:: yaml

        # активирует профайлер для запросов из подсети 192.168.0.0/24
        framework:
            profiler:
                matcher: { ip: 192.168.0.0/24 }

        # активирует профайлер только для URL /admin
        framework:
            profiler:
                matcher: { path: "^/admin/" }

        # комбинирование правил
        framework:
            profiler:
                matcher: { ip: 192.168.0.0/24, path: "^/admin/" }

        # использование пользовательской службы matcher
        framework:
            profiler:
                matcher: { service: custom_matcher }

    .. code-block:: xml

        <!-- активирует профайлер для запросов из подсети 192.168.0.0/24 -->
        <framework:config>
            <framework:profiler>
                <framework:matcher ip="192.168.0.0/24" />
            </framework:profiler>
        </framework:config>

        <!-- активирует профайлер только для URL /admin -->
        <framework:config>
            <framework:profiler>
                <framework:matcher path="^/admin/" />
            </framework:profiler>
        </framework:config>

        <!-- комбинирование правил -->
        <framework:config>
            <framework:profiler>
                <framework:matcher ip="192.168.0.0/24" path="^/admin/" />
            </framework:profiler>
        </framework:config>

        <!-- использование пользовательской службы matcher -->
        <framework:config>
            <framework:profiler>
                <framework:matcher service="custom_matcher" />
            </framework:profiler>
        </framework:config>

    .. code-block:: php

        <?php
        // активирует профайлер для запросов из подсети 192.168.0.0/24
        $container->loadFromExtension('framework', array(
            'profiler' => array(
                'matcher' => array('ip' => '192.168.0.0/24'),
            ),
        ));

        // активирует профайлер только для URL /admin
        $container->loadFromExtension('framework', array(
            'profiler' => array(
                'matcher' => array('path' => '^/admin/'),
            ),
        ));

        // комбинирование правил
        $container->loadFromExtension('framework', array(
            'profiler' => array(
                'matcher' => array('ip' => '192.168.0.0/24', 'path' => '^/admin/'),
            ),
        ));

        # использование пользовательской службы matcher
        $container->loadFromExtension('framework', array(
            'profiler' => array(
                'matcher' => array('service' => 'custom_matcher'),
            ),
        ));

Читайте в книге рецептов
------------------------

* :doc:`/cookbook/testing/profiling`
* :doc:`/cookbook/profiler/data_collector`
* :doc:`/cookbook/event_dispatcher/class_extension`
* :doc:`/cookbook/event_dispatcher/method_behavior`

.. _Observer: http://ru.wikipedia.org/wiki/Наблюдатель_(шаблон_проектирования)
.. _`Symfony2 HttpKernel component`: https://github.com/symfony/HttpKernel
.. _Closures: http://php.net/manual/en/functions.anonymous.php
.. _`Symfony2 Dependency Injection component`: https://github.com/symfony/DependencyInjection
.. _PHP callable: http://www.php.net/manual/en/language.pseudo-types.php#language.types.callback

.. toctree::
    :hidden:

    Translation source: 2011-09-27 676d511
    Corrected from: 2011-10-27 6935901
    Corrected from: 2011-11-28 26d17e3
