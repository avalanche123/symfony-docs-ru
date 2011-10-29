.. index::
   single: Контейнер служб
   single: Контейнер внедрения зависимости

Контейнер служб
=================

В современном PHP приложении множество объектов. Один объект может облегчать
отправку email'ов, другой - сохранять информацию в базе данных. В вашем приложении
вы можете создать объект, который ведёт учёт товаров, или же объект, который
обрабатывает данные от сторонних API. Здесь важно понимание того, что современное
приложение выполняет множество функций и состоит из множества объектов,
реализующих эти функции.

В этой главе мы поговорим об особом объекте в Symfony2, который позволяет вам
создавать экземпляры, систематизировать и получать различные объекты вашего
приложения. Этот объект, называемый контейнером служб, позволит вам стандартизировать
и централизовать способ создания объектов в вашем приложении. Контейнер
делает вашу жизнь проще, быстрее и делает особый акцент на архитектуре, которая
предоставляет независимый, готовый к повторному использованию код. Так как
все классы ядра Symfony2 используют контейнер, вы узнаете, как расширять,
настраивать и использовать любой объект Symfony2. Контейнер служб в значительной
степени определяет скорость и расширяемость Symfony2.

И, в конце концов, настройка и использование контейнера служб весьма проста.
В конце этой главы вы будете с лёгкостью создавать ваши собственные объекты
при помощи контейнера и настраивать объекты из сторонних пакетов. Вы будете
писать более тестируемый и менее запутанный код, который можно будет использовать
повторно лишь потому, что контейнер служб делает написание хорошего кода
простым.

.. index::
   single: Контейнер служб; Что такое служба?

Что такое служба?
------------------

Если вкратце, :term:`Служба` - это некий PHP объект, который
выполняет какую-либо "глобальную" задачу. Это наименование используется в
компьютерной науке для описания объекта, который создан с некоторой целью
(например, отправлять email'ы). Каждая служба используется в любом модуле
приложения, где бы вам ни понадобился функционал, который предоставляет служба.
Вам не требуется делать ничего особенного, для того чтобы создать службу:
просто создайте PHP класс, решающий некую конкретную задачу. Поздравляем,
Вы только что создали службу!

.. note::

    Как правило, PHP объект является службой, если он используется в вашем
    приложении глобально. Единственная служба ``Mailer`` используется для
    отправки email сообщений, в то время как множество объектов ``Message``,
    которые содержат эти сообщения, службами *не* являются. Точно так же,
    объект ``Product`` не является службой, но объект, который сохраняет
    ``Product`` в базу данных - *является* службой.

Так где же выгода? Мыслить в терминах "служб" полезно, когда вы начинаете
думать о распределении каждого кусочка функционала в вашем приложении по
ряду служб. Так как каждая служба выполняет единственную функцию, вы можете
легко получить доступ к любой службе и использовать её возможности там,
где это требуется. Каждая служба легко тестируется и настраивается, так
как она не зависит от прочего функционала. Такое разделение на службы
называется `Сервис-ориентированная архитектура`_ и она не уникальна ни в
рамках Symfony2, ни даже в масштабе всего PHP. Структурирование вашего
приложения в виде набора независимых служб - это хорошо известная,
а также хорошо зарекомендовавшая себя практика. Знание этой архитектуры
будет полезно любому хорошему разработчику вне зависимости от языка, на
котором он программирует.

.. index::
   single: Контейнер служб; Что такое контейнер служб?

Что такое контейнер служб?
----------------------------

:term:`Контейнер служб` (или же *контейнер внедрения зависимости*) - это также
PHP объект, который управляет созданием служб (т.е. объектов).  Например, положим
у вас есть простой PHP класс, который отправляет email сообщения. Не имея
контейнера служб, вы будете вынуждены вручную создавать этот объект там
где вам это потребуется:

.. code-block:: php

    <?php
    use Acme\HelloBundle\Mailer;

    $mailer = new Mailer('sendmail');
    $mailer->send('ryan@foobar.net', ... );

Выглядит не сложно. Ваш воображаемый класс ``Mailer`` позволяет указать
метод, используемый для отправки сообщений (например,  ``sendmail``, ``smtp``
и т.д.). Но что будет, если потребуется использовать эту службу где-то ещё?
Естественно вам не захочется конфигурировать объект ``Mailer`` *каждый раз*,
когда вам потребуется его использовать. Что, если вам потребуется изменить
транспорт с ``sendmail`` на ``smtp`` во всём вашем приложении? Потребовалось
бы искать все места, где создаётся ``Mailer`` и изменять их.

.. index::
   single: Контейнер служб; Настройка служб

Создание/настройка служб в контейнере
-------------------------------------

Наилучшим решением на практике - разрешить контейнеру служб создать объект
``Mailer`` для вас. Для этого контейнер необходимо *обучить* - как создавать
объект ``Mailer``. Это выполняется при помощи конфигурации, которую можно
выполнить в форматах YAML, XML или PHP:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        services:
            my_mailer:
                class:        Acme\HelloBundle\Mailer
                arguments:    [sendmail]

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <services>
            <service id="my_mailer" class="Acme\HelloBundle\Mailer">
                <argument>sendmail</argument>
            </service>
        </services>

    .. code-block:: php

        <?php
        // app/config/config.php
        use Symfony\Component\DependencyInjection\Definition;

        $container->setDefinition('my_mailer', new Definition(
            'Acme\HelloBundle\Mailer',
            array('sendmail')
        ));

.. note::

    При инициализации Symfony2 он создаёт контейнер служб, используя
    конфигурацию приложения (по умолчанию ``app/config/config.yml``).
    Файл, который будет загружен, определяется методом
    ``AppKernel::registerContainerConfiguration()``, который загружает
    файл, относящийся к конкретному окружению (например, ``config_dev.yml``
    для ``dev`` или же ``config_prod.yml`` для ``prod``).

Экземпляр объекта ``Acme\HelloBundle\Mailer`` теперь можно получить через
контейнер служб. А сам контейнер доступен в любом традиционном контроллере
Symfony2 при помощи вспомогательного метода ``get()``:

.. code-block:: php

    <?php
    class HelloController extends Controller
    {
        // ...

        public function sendEmailAction()
        {
            // ...
            $mailer = $this->get('my_mailer');
            $mailer->send('ryan@foobar.net', ... );
        }
    }

Когда запрашивается служба ``my_mailer``, контейнер создаёт её объект и возвращает
её. Это ещё одно преимущество от использования контейнера служб. А именно, служба
не создаётся вплоть до того момента, когда она будет нужна вам. Если вы определите
службу, но нигде её не используете - она никогда не будет создана. Это экономит
память и делает ваше приложение быстрее. Это также означает, что вы можете определять
сколько угодно служб без ущерба быстродействию приложения - службы, которые
не используются - не будут и созданы.

В качестве приятного бонуса, служба ``Mailer`` будет создана лишь однажды
и один и тот же её экземпляр будет возвращаться всякий раз, когда вы запрашиваете
данную службу. Как правило, вы именно этого и ожидаете (такой подход более
гибок и удобен), однако ниже вы узнаете как настроить службу таким образом,
чтобы она могла иметь несколько экземпляров одновременно.

.. _book-service-container-parameters:

Параметры службы
------------------

Создание новой службы (т.е. объекта) при помощи контейнера происходит
очень просто. Параметры делают службы более гибкими:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        parameters:
            my_mailer.class:      Acme\HelloBundle\Mailer
            my_mailer.transport:  sendmail

        services:
            my_mailer:
                class:        %my_mailer.class%
                arguments:    [%my_mailer.transport%]

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <parameters>
            <parameter key="my_mailer.class">Acme\HelloBundle\Mailer</parameter>
            <parameter key="my_mailer.transport">sendmail</parameter>
        </parameters>

        <services>
            <service id="my_mailer" class="%my_mailer.class%">
                <argument>%my_mailer.transport%</argument>
            </service>
        </services>

    .. code-block:: php

        <?php
        // app/config/config.php
        use Symfony\Component\DependencyInjection\Definition;

        $container->setParameter('my_mailer.class', 'Acme\HelloBundle\Mailer');
        $container->setParameter('my_mailer.transport', 'sendmail');

        $container->setDefinition('my_mailer', new Definition(
            '%my_mailer.class%',
            array('%my_mailer.transport%')
        ));

Конечный результат такой же, как и раньше - отличие лишь в том, *как*
определена служба. Заключив строки ``my_mailer.class`` и ``my_mailer.transport``
между символами процента (``%``), вы указали контейнеру искать параметры
с этими именами. Когда контейнер создан, он ищет значение для каждого параметра
и использует их при создании служб.

Назначение параметров - передача информации внутрь служб. Конечно же, нет ничего
зазорного в создании служб без параметров, тем не менее параметры дают некоторые
преимущества:

* разделение и структурирование всех опций службы;

* значения параметров могут быть использованы для множественного определения служб;

* при создании службы в пакете (об этом будет чуть ниже), использование
  параметров позволяет легко настроить службу в вашем приложении.

Выбор - использовать или не использовать параметры - целиком лежит на вас.
Качественные пакеты от сторонних разработчиков *всегда* используют параметры,
так как они делают службы более конфигурируемыми. Для служб в вашем приложении,
тем не менее, вам может и не потребоваться та гибкость, которую даёт использование
параметров.

Массивы параметров
~~~~~~~~~~~~~~~~~~

Параметры - это не обязательно строки, это также могут быть и массивы. В формате XML
вы должны использовать атрибут type="collection" для параметров-массивов.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        parameters:
            my_mailer.gateways:
                - mail1
                - mail2
                - mail3
            my_multilang.language_fallback:
                en:
                    - en
                    - fr
                fr:
                    - fr
                    - en

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <parameters>
            <parameter key="my_mailer.gateways" type="collection">
                <parameter>mail1</parameter>
                <parameter>mail2</parameter>
                <parameter>mail3</parameter>
            </parameter>
            <parameter key="my_multilang.language_fallback" type="collection">
                <parameter key="en" type="collection">
                    <parameter>en</parameter>
                    <parameter>fr</parameter>
                </parameter>
                <parameter key="fr" type="collection">
                    <parameter>fr</parameter>
                    <parameter>en</parameter>
                </parameter>
            </parameter>
        </parameters>

    .. code-block:: php

        <?php
        // app/config/config.php
        use Symfony\Component\DependencyInjection\Definition;

        $container->setParameter('my_mailer.gateways', array('mail1', 'mail2', 'mail3'));
        $container->setParameter('my_multilang.language_fallback',
                                 array('en' => array('en', 'fr'),
                                       'fr' => array('fr', 'en'),
                                ));

Импорт конфигураций контейнера
------------------------------

.. tip::

    В этом разделе мы будем ссылаться на файлы конфигурации служб, как на некоторые
    *ресурсы*. Это подчёркивает тот факт, что, хотя большинство ресурсов конфигурации
    будут представлены в виде файлов (например, YAML, XML, PHP), Symfony2 настолько
    гибок, что конфигурация может быть загружена практически отовсюду (например,
    из базы данных или даже через внешний веб-сервис).

Контейнер служб создаётся с использованием одного конфигурационного ресурса
(по умолчанию ``app/config/config.yml``). Все прочие ресурсы для служб
(включая ядро Symfony2 и настройки сторонних пакетов) должны импортироваться
тем или иным способом. Это даёт вам абсолютную гибкость в настройке служб
в вашем приложении.

Настройки служб могут быть импортированы двумя различными способами. Во-первых,
мы поговорим о методе, который вы будете в основном использовать в вашем приложении:
директива ``imports``. В следующей секции мы рассмотрим другой, более гибкий метод,
наиболее предпочтительный для импорта настроек служб из сторонних приложений.

.. index::
   single: Контейнер служб; Импорт

.. _service-container-imports-directive:

Импорт конфигурации при помощи директивы ``imports``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ранее вы разместили определение службы ``my_mailer`` напрямую в файле
конфигурации приложения (``app/config/config.yml``). Поскольку класс
``Mailer`` располагается в пакете ``AcmeHelloBundle``, имеет смысл
разместить определение контейнера ``my_mailer`` внутри этого пакета.

Во-первых, переместите определение ``my_mailer`` в новый файл внутри ``AcmeHelloBundle``.
Если директории ``Resources`` или ``Resources/config`` отсутствуют - создайте их.

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/services.yml
        parameters:
            my_mailer.class:      Acme\HelloBundle\Mailer
            my_mailer.transport:  sendmail

        services:
            my_mailer:
                class:        %my_mailer.class%
                arguments:    [%my_mailer.transport%]

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/services.xml -->
        <parameters>
            <parameter key="my_mailer.class">Acme\HelloBundle\Mailer</parameter>
            <parameter key="my_mailer.transport">sendmail</parameter>
        </parameters>

        <services>
            <service id="my_mailer" class="%my_mailer.class%">
                <argument>%my_mailer.transport%</argument>
            </service>
        </services>

    .. code-block:: php

        <?php
        // src/Acme/HelloBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;

        $container->setParameter('my_mailer.class', 'Acme\HelloBundle\Mailer');
        $container->setParameter('my_mailer.transport', 'sendmail');

        $container->setDefinition('my_mailer', new Definition(
            '%my_mailer.class%',
            array('%my_mailer.transport%')
        ));

Само определение службы осталось без изменений, изменилось только расположение
файла конфигурации. Конечно же, контейнер пока ничего не знает о новом ресурсе.
К счастью, вы можете легко импортировать файл ресурса при помощи ключевого
слова ``imports`` в конфигурации приложения:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        imports:
            hello_bundle:
                resource: @AcmeHelloBundle/Resources/config/services.yml

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <imports>
            <import resource="@AcmeHelloBundle/Resources/config/services.xml"/>
        </imports>

    .. code-block:: php

        <?php
        // app/config/config.php
        $this->import('@AcmeHelloBundle/Resources/config/services.php');

Директива ``imports`` позволяет приложению подключать ресурсы конфигурации
контейнера служб из различных мест (как правило, из пакетов).
Расположение ``resource`` для файлов - это абсолютный путь к этому файлу.
Специальный синтаксис ``@AcmeHello`` соответствует пути к директории пакета
``AcmeHelloBundle``. Это помогает указывать путь к ресурсу не заботясь о том,
что пакет ``AcmeHelloBundle`` может быть в будущем перемещён в другое место.

.. index::
   single: Контейнер служб; Расширение конфигурации

.. _service-container-extension-configuration:

Импорт конфигурации при помощи расширений контейнера
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

При разработке с использованием Symfony2 чаще всего вы будете использовать
директиву ``imports`` для импорта конфигурации контейнера из пакетов, которые
вы создали специально для вашего приложения. Конфигурация контейнера для сторонних
пакетов, включая службы ядра Symfony2, как правило, загружается при помощи
другого метода, более гибкого и просто для настройки в вашем приложении.

Вот как работает этот метод. Внутри каждого пакета его службы определяются
очень похожим образом, как вы делали это ранее. А именно, пакет использует
один или более файлов конфигурации (как правило, XML) для указания параметров
и служб этого пакета. Тем не менее, вместо того, чтобы импортировать каждый ресурс
непосредственно в файл конфигурации вашего приложения при помощи директивы
``imports``, вы можете вызвать *расширение контейнера служб* внутри пакета,
которое выполнит эту работу за вас. Расширение контейнера служб - это PHP
класс, созданный автором пакета для выполнения следующих функций:

* Импорта всех ресурсов контейнера служб, необходимых для конфигурации
  всех служб пакетов;

* Предоставления простой и понятной конфигурации, при помощи которой пакет
  можно настроить, не взаимодействуя напрямую с конфигурацией
  контейнера служб пакета.

Другими словами, расширение контейнера служб настраивает службы пакета от
вашего имени. И, как вы скоро увидите, расширение предоставляет удобный
высокоуровневый интерфейс для настройки пакета.

Возьмём в качестве примера ``FrameworkBundle`` - основу Symfony2. Наличие
следующего кода в конфигурации вашего приложения вызывает расширение контейнера
служб внутри ``FrameworkBundle``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            secret:          xxxxxxxxxx
            charset:         UTF-8
            form:            true
            csrf_protection: true
            router:        { resource: "%kernel.root_dir%/config/routing.yml" }
            # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config charset="UTF-8" secret="xxxxxxxxxx">
            <framework:form />
            <framework:csrf-protection />
            <framework:router resource="%kernel.root_dir%/config/routing.xml" />
            <!-- ... -->
        </framework>

    .. code-block:: php

        <?php
        // app/config/config.php
        $container->loadFromExtension('framework', array(
            'secret'          => 'xxxxxxxxxx',
            'charset'         => 'UTF-8',
            'form'            => array(),
            'csrf-protection' => array(),
            'router'          => array('resource' => '%kernel.root_dir%/config/routing.php'),
            // ...
        ));

Когда парсится конфигурационный файл, контейнер ищет расширение, которое может
обработать директиву ``framework``. Вызывается расширение, которое находится
в ``FrameworkBundle`` и загружается конфигурация служб для ``FrameworkBundle``.
Если вы удалите ключ ``framework`` из конфигурации приложения - основные
сервисы ядра Symfony2 не будут загружены. Суть же заключается в том, что всё
находится под вашим контролем: Symfony2 не содержит никакой магии и не делает
ничего такого, чего бы вы не контролировали.

Конечно же, вы можете делать много больше, чем просто активировать расширение
контейнера служб для ``FrameworkBundle``. Каждое расширение позволяет вам легко
настроить пакет, не заботясь о том, как именно определены его службы.

В нашем случае, расширение позволяет настроить ``charset``, ``error_handler``,
``csrf_protection``, ``router`` и многое другое. Внутри ``FrameworkBundle``
использует опции, указанные в настройках приложения для определения и конфигурирования
своих служб. Пакет позаботится о создании всех необходимых настроек ``parameters`` и
``services`` для контейнера служб, при этом также сохраняя гибкость настройки.
В качестве бонуса большинство расширений контейнера служб также выполняют валидацию -
уведомляют вас об отсутствующих или же имеющих неправильный тип параметрах.

Когда вы устанавливаете или настраиваете пакет - смотрите его документацию,
чтобы узнать как установить и настроить его службы. Опции, доступные для
основных пакетов ядра вы можете посмотреть в :doc:`справочнике </reference/index>`.

.. note::

   Контейнер служб распознаёт лишь директивы ``parameters``, ``services``,
   и ``imports``. Все остальные директивы обрабатываются расширениями.

.. index::
   single: Контейнер служб; Внедрение служб

Использование одних служб внутри других (Внедрение служб)
---------------------------------------------------------

Рассмотренная выше служба ``my_mailer`` проста: она принимает лишь один аргумент
конструктора, который легко настраивается. Как вы увидите, свою силу контейнер
показывает, когда вам нужно создать службу, которая зависит от одной или нескольких
служб контейнера.

Давайте начнём с примера. Предположим у вас есть новая служба ``NewsletterManager``,
которая помогает подготавливать и рассылать email-сообщения на некоторый набор адресов.
Службу ``my_mailer`` было бы неплохо использовать для отправки сообщений
внутри службы ``NewsletterManager``. Таким образом, класс может выглядеть примерно так:

.. code-block:: php

    <?php
    namespace Acme\HelloBundle\Newsletter;

    use Acme\HelloBundle\Mailer;

    class NewsletterManager
    {
        protected $mailer;

        public function __construct(Mailer $mailer)
        {
            $this->mailer = $mailer;
        }

        // ...
    }

Не используя контейнер служб, мы может создать ``NewsletterManager`` внутри
контроллера:

.. code-block:: php

    <?php
    public function sendNewsletterAction()
    {
        $mailer = $this->get('my_mailer');
        $newsletter = new Acme\HelloBundle\Newsletter\NewsletterManager($mailer);
        // ...
    }

Такой подход, в общем-то, не плох, но что, если потребуется добавить к конструктору
класса ``NewsletterManager`` второй или даже третий аргумент? Что, если вы решите
выполнить рефакторинг и переименуете класс? В обоих случаях вам потребовалось бы найти
все места, где создаются экземпляры ``NewsletterManager`` и изменить их. И тут контейнер
служб предоставляет вам удобное решение:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/services.yml
        parameters:
            # ...
            newsletter_manager.class: Acme\HelloBundle\Newsletter\NewsletterManager

        services:
            my_mailer:
                # ...
            newsletter_manager:
                class:     %newsletter_manager.class%
                arguments: [@my_mailer]

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/services.xml -->
        <parameters>
            <!-- ... -->
            <parameter key="newsletter_manager.class">Acme\HelloBundle\Newsletter\NewsletterManager</parameter>
        </parameters>

        <services>
            <service id="my_mailer" ... >
              <!-- ... -->
            </service>
            <service id="newsletter_manager" class="%newsletter_manager.class%">
                <argument type="service" id="my_mailer"/>
            </service>
        </services>

    .. code-block:: php

        <?php
        // src/Acme/HelloBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;

        // ...
        $container->setParameter('newsletter_manager.class', 'Acme\HelloBundle\Newsletter\NewsletterManager');

        $container->setDefinition('my_mailer', ... );
        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%',
            array(new Reference('my_mailer'))
        ));

В YAML, специальный синтаксис ``@my_mailer`` сообщает контейнеру, что
нужно искать службу ``my_mailer`` и передать этот объект в конструктор
``NewsletterManager``. В этом случае служба ``my_mailer`` должна существовать.
Если её определение не будет найдено, будет вызвано исключение. Вы также можете
пометить зависимости опциональными - это будет обсуждаться в следующей секции.

Использование ссылок на службы (внедрение служб) - это мощный инструмент, который
позволяет создавать независимые классы служб с чётко определёнными зависимостями. В
этом примере, службе ``newsletter_manager`` для функционирования необходима служба
``my_mailer``. Когда вы определите эту зависимость в контейнере служб, он позаботится
о создании всех необходимых объектов.

Опциональные зависимости
~~~~~~~~~~~~~~~~~~~~~~~~

Внедрение зависимостей в конструктор - это прекрасный способ удостовериться,
что зависимость доступна для использования. Если у вас есть необязательные
зависимости для класса, то лучшим выбором будет использование "setter injection".
Это означает, что внедрение зависимости производится при помощи некоторого
метода, а не в конструкторе. Класс будет выглядеть следующим образом:

.. code-block:: php

    <?php
    namespace Acme\HelloBundle\Newsletter;

    use Acme\HelloBundle\Mailer;

    class NewsletterManager
    {
        protected $mailer;

        public function setMailer(Mailer $mailer)
        {
            $this->mailer = $mailer;
        }

        // ...
    }

Внедрение зависимости при помощи метода требует также изменения синтаксиса:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/services.yml
        parameters:
            # ...
            newsletter_manager.class: Acme\HelloBundle\Newsletter\NewsletterManager

        services:
            my_mailer:
                # ...
            newsletter_manager:
                class:     %newsletter_manager.class%
                calls:
                    - [ setMailer, [ @my_mailer ] ]

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/services.xml -->
        <parameters>
            <!-- ... -->
            <parameter key="newsletter_manager.class">Acme\HelloBundle\Newsletter\NewsletterManager</parameter>
        </parameters>

        <services>
            <service id="my_mailer" ... >
              <!-- ... -->
            </service>
            <service id="newsletter_manager" class="%newsletter_manager.class%">
                <call method="setMailer">
                     <argument type="service" id="my_mailer" />
                </call>
            </service>
        </services>

    .. code-block:: php

        <?php
        // src/Acme/HelloBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;

        // ...
        $container->setParameter('newsletter_manager.class', 'Acme\HelloBundle\Newsletter\NewsletterManager');

        $container->setDefinition('my_mailer', ... );
        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%'
        ))->addMethodCall('setMailer', array(
            new Reference('my_mailer')
        ));

.. note::

    Подходы к внедрению служб, представленные в этой секции называются "constructor injection"
    и "setter injection". Контейнер служб Symfony2 также поддерживает "property injection".

Делаем ссылки на службы опциональными
-------------------------------------

Иногда, службы могут иметь опциональные зависимости, т.е. зависимость
не требуется, для того, чтобы служба правильно работала. В примере выше
служба ``my_mailer`` должна существовать, в противном случае будет сгенерирована
ошибка (исключение). Изменив определение службы ``newsletter_manager`` вы
можете сделать зависимость необязательной. Контейнер будет внедрять её
лишь когда эта зависимость существует, в случае же если она не существует,
никаких действий производиться не будет:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/services.yml
        parameters:
            # ...

        services:
            newsletter_manager:
                class:     %newsletter_manager.class%
                arguments: [@?my_mailer]

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/services.xml -->

        <services>
            <service id="my_mailer" ... >
              <!-- ... -->
            </service>
            <service id="newsletter_manager" class="%newsletter_manager.class%">
                <argument type="service" id="my_mailer" on-invalid="ignore" />
            </service>
        </services>

    .. code-block:: php

        <?php
        // src/Acme/HelloBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;
        use Symfony\Component\DependencyInjection\ContainerInterface;

        // ...
        $container->setParameter('newsletter_manager.class', 'Acme\HelloBundle\Newsletter\NewsletterManager');

        $container->setDefinition('my_mailer', ... );
        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%',
            array(new Reference('my_mailer', ContainerInterface::IGNORE_ON_INVALID_REFERENCE))
        ));

В YAML, специальный синтаксис ``@?`` сообщает контейнеру служб, что зависимость не
обязательная. И, конечно же, конструктор ``NewsletterManager`` должен быть переписан,
чтобы поддерживать опциональную зависимость:

.. code-block:: php

    <?php
    // ...
    public function __construct(Mailer $mailer = null)
    {
        // ...
    }

Основные службы Symfony и службы от сторонних разработчиков
----------------------------------------------------------

Так как Symfony2 и все сторонние пакеты настраивают и получают свои службы
при помощи контейнера, вы можете легко получить доступ к ним или даже использовать
их в своих собственных службах. Для простоты Symfony2 по умолчанию не требует,
чтобы контроллеры были бы определены как службы. Кроме того, Symfony2 внедряет
в ваш контроллер контейнер служб целиком. Например, для работы с пользовательской
сессией, Symfony2 предоставляет службу ``session``, которая позволяет получить
доступ к сессии внутри обычного контроллера:

.. code-block:: php

    <?php
    public function indexAction($bar)
    {
        $session = $this->get('session');
        $session->set('foo', $bar);

        // ...
    }

В Symfony2 вы постоянно будете пользоваться службами, предоставляемыми ядром Symfony
или же прочими пакетами от сторонних разработчиков, для выполнения таких задач как
отображение шаблонов (``templating``), отправку майлов (``mailer``) или доступ
к переменным запроса (``request``).

Вы можете пойти ещё дальше и использовать эти службы внутри ваших служб,
созданных для вашего приложения. Дайте изменим класс ``NewsletterManager``,
чтобы он использовал стандартный ``mailer`` Symfony2 (вместо ``my_mailer``).
Давайте также внедрим службу шаблонизатора в ``NewsletterManager``, чтобы
можно было создавать контент электронных писем из шаблонов:

.. code-block:: php

    <?php
    namespace Acme\HelloBundle\Newsletter;

    use Symfony\Component\Templating\EngineInterface;

    class NewsletterManager
    {
        protected $mailer;

        protected $templating;

        public function __construct(\Swift_Mailer $mailer, EngineInterface $templating)
        {
            $this->mailer = $mailer;
            $this->templating = $templating;
        }

        // ...
    }

Настройку контейнера выполнить не сложно:

.. configuration-block::

    .. code-block:: yaml

        services:
            newsletter_manager:
                class:     %newsletter_manager.class%
                arguments: [@mailer, @templating]

    .. code-block:: xml

        <service id="newsletter_manager" class="%newsletter_manager.class%">
            <argument type="service" id="mailer"/>
            <argument type="service" id="templating"/>
        </service>

    .. code-block:: php

        <?php
        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%',
            array(
                new Reference('mailer'),
                new Reference('templating')
            )
        ));

Служба ``newsletter_manager`` теперь имеет доступ к службам ``mailer``
и ``templating``. Это типичный путь по созданию служб, специфичных для вашего
приложения, который позволяет использовать всю мощь различных служб Фреймворка.

.. tip::

    Удостоверьтесь, что в конфигурации вашего приложения присутствует раздел
    ``swiftmailer``. Как указано в секции :ref:`service-container-extension-configuration`,
    ключ ``swiftmailer`` внедряет расширение из ``SwiftmailerBundle``, которое
    регистрирует службу ``mailer``.

.. index::
   single: Service Container; Advanced configuration

Продвинутая конфигурация контейнера
-----------------------------------

Как вы могли видеть ранее, определение служб в контейнере выполняется легко,
в основном с применением ключа конфигурации ``service`` и нескольких
параметров. Тем не менее, контейнер имеет ещё несколько дополнительных
инструментов, с помощью которых можно получить дополнительную функциональность,
создавать более сложные службы и выполнять операции после создания контейнера.

Публичные и приватные службы
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Как правило, при определении служб, вы рассчитываете получить доступ к ним в вашем
приложении. Такие службы называются *публичными* (``public``). Например, служба ``doctrine``
из состава DoctrineBundle является публичной и вы можете получить к ней доступ
следующим образом::

   $doctrine = $container->get('doctrine');

Тем не менее, имеются случаи, когда вы не захотите, чтобы ваши службы были публичными.
Как правило, это случается, когда служба определена лишь для того, чтобы
быть использованной в качестве аргумента для другой службы.

.. note::

    Если вы используете приватные службы в качестве аргумента более чем
    для одной службы, в результате будут созданы два различных экземпляра
    этой приватной службы (т.е. ``new PrivateFooBar()``).

Служба должна быть приватной, когда вы не хотите, чтобы она была доступна
напрямую из кода.

Например:

.. configuration-block::

    .. code-block:: yaml

        services:
           foo:
             class: Acme\HelloBundle\Foo
             public: false

    .. code-block:: xml

        <service id="foo" class="Acme\HelloBundle\Foo" public="false" />

    .. code-block:: php

        <?php
        $definition = new Definition('Acme\HelloBundle\Foo');
        $definition->setPublic(false);
        $container->setDefinition('foo', $definition);

Теперь служба определена как приватная и в *не можете* получить к ней доступ напрямую::

    $container->get('foo');

Тем не менее, даже если служба обозначена как приватная, вы ещё можете использовать
её псевдоним (alias, см. ниже) для доступа к ней (при помощи этого псевдонима).

.. note::

   По умолчанию все службы - публичные.

Псевдонимы
~~~~~~~~~~

При использовании основных или же сторонних пакетов в вашем приложении, вы
возможно захотите использовать ярлычки для доступа к некоторым их службам. Вы
можете сделать это при помощи псевдонимов и даже можете создавать псевдонимы
для приватных служб.

.. configuration-block::

    .. code-block:: yaml

        services:
           foo:
             class: Acme\HelloBundle\Foo
           bar:
             alias: foo

    .. code-block:: xml

        <service id="foo" class="Acme\HelloBundle\Foo"/>

        <service id="bar" alias="foo" />

    .. code-block:: php

        <?php
        $definition = new Definition('Acme\HelloBundle\Foo');
        $container->setDefinition('foo', $definition);

        $containerBuilder->setAlias('bar', 'foo');

Это означает, что при использовании контейнера, вы можете получить доступ
к службе ``foo`` запрашивая службу ``bar``::

    $container->get('bar'); // В итоге получите службу foo

Подключение файлов
~~~~~~~~~~~~~~~~~~

Также возможны случаи, когда вам будет необходимо подключить некоторый
файл прямо перед загрузкой службы. Для этого вы можете воспользоваться
директивой ``file``:

.. configuration-block::

    .. code-block:: yaml

        services:
           foo:
             class: Acme\HelloBundle\Foo\Bar
             file: %kernel.root_dir%/src/path/to/file/foo.php

    .. code-block:: xml

        <service id="foo" class="Acme\HelloBundle\Foo\Bar">
            <file>%kernel.root_dir%/src/path/to/file/foo.php</file>
        </service>

    .. code-block:: php

        $definition = new Definition('Acme\HelloBundle\Foo\Bar');
        $definition->setFile('%kernel.root_dir%/src/path/to/file/foo.php');
        $container->setDefinition('foo', $definition);

Имейте в виду, что symfony будет подключать файл при помощи PHP функции
``require_once``, поэтому файл будет подключаться один единственный раз
в рамках каждого запроса.

.. _book-service-container-tags:

Таги (``tags``)
~~~~~~~~~~~~~~~

Точно также как ваша запись в блоге может быть снабжена тагами, так и
любая служба может иметь свои таги. В контейнере служб таг означает,
что служба используется для некоторых специфических функций. Давайте
рассмотрим пример:

.. configuration-block::

    .. code-block:: yaml

        services:
            foo.twig.extension:
                class: Acme\HelloBundle\Extension\FooExtension
                tags:
                    -  { name: twig.extension }

    .. code-block:: xml

        <service id="foo.twig.extension" class="Acme\HelloBundle\Extension\FooExtension">
            <tag name="twig.extension" />
        </service>

    .. code-block:: php

        <?php
        $definition = new Definition('Acme\HelloBundle\Extension\FooExtension');
        $definition->addTag('twig.extension');
        $container->setDefinition('foo.twig.extension', $definition);

Таг ``twig.extension`` - это специализированный таг, который ``TwigBundle``
использует во время конфигурирования. Присваивая службе таг ``twig.extension``,
``TwigBundle`` будет знать, что служба ``foo.twig.extension`` должна быть
зарегистрирована в качестве расширения Twig. Другими словами, Twig таким
образом ищет все службы с тагом ``twig.extension`` и автоматически регистрирует
их как расширения Twig.

Таги, таким образом, являются способом сообщить Symfony2 или другим
сторонним пакетам, что ваша служба должна быть зарегистрирована или
использована некоторым особым способом внутри целевого пакета.

Ниже представлен список тагов, доступных в ядре Symfony2.
Каждый из этих тагов имеет свой собственный эффект на вашу службу и
многие таги требуют наличия дополнительных параметров (помимо параметра
``name``).

* assetic.filter
* assetic.templating.php
* data_collector
* form.field_factory.guesser
* kernel.cache_warmer
* kernel.event_listener
* monolog.logger
* routing.loader
* security.listener.factory
* security.voter
* templating.helper
* twig.extension
* translation.loader
* validator.constraint_validator

Дополнительно читайте в книге рецептов:
---------------------------------------

* :doc:`/cookbook/service_container/factories`
* :doc:`/cookbook/service_container/parentservices`
* :doc:`/cookbook/controller/service`

.. _`Сервис-ориентированная архитектура`: http://ru.wikipedia.org/wiki/Сервис-ориентированная_архитектура

.. toctree::
    :hidden:

    Translation source: 2011-10-16 7e2d2b2
    Corrected from: 2011-10-29 cc7fef0
