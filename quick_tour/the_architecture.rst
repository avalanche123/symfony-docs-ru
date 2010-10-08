Архитектура
================

Вы мой герой! Кто бы мог подумать что вы все еще будете здесь после первых трех частей? Ваши усилия скоро будут вознаграждены. В первых трех частях мы глубоко не вникали в архитектуру фреймворка. Так как она выделяет Symfony2 из толпы фреймворков, давайте сейчас же в нее погрузимся.

.. index::
   single: Структура Директорий

Структура Директорий
-----------------------

Структура директорий приложения :term:`application` на Symfony довольно гибкая но структура директорий песочницы отражает типовую и рекомендованную структуру приложения Symfony:

* ``app/``: В этой категории содержится конфигурация приложения;

* ``src/``: Весь PHP код содержится в этой директории;

* ``web/``: Это корневая web директория проекта.

Web Директория
~~~~~~~~~~~~~~~~~

Корневая web директория - это домашняя директория для всех публичных и статических файлов типа изображений, стилей и javascript-файлов. Она также содержит боевые фронт-контроллеры:

.. code-block:: html+php

    <!-- web/index.php -->
    <?php

    require_once __DIR__.'/../app/AppKernel.php';

    $kernel = new AppKernel('prod', false);
    $kernel->handle()->send();

Как любой фронт-контроллер, ``index.php`` использует Kernel Class, ``AppKernel``, для запуска приложения.

.. index::
   single: Ядро

Директория Приложения
~~~~~~~~~~~~~~~~~~~~~~~~~

Класс ``AppKernel`` это главная входная точка конфигурации приложения как такового, и содержится в директории ``app/``.

Этот класс должен реализовывать четыре метода:

* ``registerRootDir()``: Возвращает корневую директорию;

* ``registerBundles()``: Возвращает массив всех бандлов, необходимых для запуска приложения (обратите внимание на ``Application\HelloBundle\HelloBundle``);

* ``registerBundleDirs()``: Возвращает массив ассоциаций пространств имен и их домашних директорий;

* ``registerContainerConfiguration()``: Возвращает главный объект конфигурации (об этом подробнее ниже);

Обратите внимание на реализацию этих методов по умолчанию для того чтобы лучше понять гибкость фреймворка.

Для того чтобы это все работало, ядру необходим один файл из директории ``src/``::

    // app/AppKernel.php
    require_once __DIR__.'/../src/autoload.php';

Директория Исходных Кодов
~~~~~~~~~~~~~~~~~~~~

Файл ``src/autoload.php`` ответственный за автозагрузку всех файлов из директории ``src/``::

    // src/autoload.php
    $vendorDir = __DIR__.'/vendor';

    require_once $vendorDir.'/symfony/src/Symfony/Component/HttpFoundation/UniversalClassLoader.php';

    use Symfony\Component\HttpFoundation\UniversalClassLoader;

    $loader = new UniversalClassLoader();
    $loader->registerNamespaces(array(
        'Symfony'                    => $vendorDir.'/symfony/src',
        'Application'                => __DIR__,
        'Bundle'                     => __DIR__,
        'Doctrine\\Common'           => $vendorDir.'/doctrine-common/lib',
        'Doctrine\\DBAL\\Migrations' => $vendorDir.'/doctrine-migrations/lib',
        'Doctrine\\ODM\\MongoDB'     => $vendorDir.'/doctrine-mongodb/lib',
        'Doctrine\\DBAL'             => $vendorDir.'/doctrine-dbal/lib',
        'Doctrine'                   => $vendorDir.'/doctrine/lib',
        'Zend'                       => $vendorDir.'/zend/library',
    ));
    $loader->registerPrefixes(array(
        'Swift_' => $vendorDir.'/swiftmailer/lib/classes',
        'Twig_'  => $vendorDir.'/twig/lib',
    ));
    $loader->register();

Symfony ``UniversalClassLoader`` используется для автозагрузки файлов, которые отвечают всем техническим требованиям `стандартов`_ PHP 5.3 пространств имен или `соглашению`_ по наименованию PEAR для классов. Как вы можете видеть, все зависимости хранятся в директории ``vendor/``, но это только соглашение. Вы можете хранить их где захотите, глобально на вашем сервере или локально в ваших проектах.

.. index::
   single: Бандлы

Система Бандлов
-----------------

В этой секции мы начинаем рассмотрение одной из наиболее существенных и мощных особенностей Symfony, ее системы :term:`бандлов`.

Бандл чем-то похож на плагин в других программах. Но почему тогда его назвали бандл вместо плагин? Потому что все в Symfony это бандлы, начиная от функционала ядра до кода, который вы пишете для своего приложения.
Бандлы - это главные структурные кирпичики в Symfony. Это наделяет вас гибкостью использовать функционал встроенный в сторонние бандлы или же распостранять ваши собственные бандлы. Это позволяет с легкостью вибирать и включать в приложение только нужный функционал, оптимизируя его на свой вкус.

Приложение состоит из бандлов, которые объявлены в методе ``registerBundles()`` класса ``AppKernel``::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),

            // enable third-party bundles
            new Symfony\Bundle\ZendBundle\ZendBundle(),
            new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            new Symfony\Bundle\DoctrineBundle\DoctrineBundle(),
            //new Symfony\Bundle\DoctrineMigrationsBundle\DoctrineMigrationsBundle(),
            //new Symfony\Bundle\DoctrineMongoDBBundle\DoctrineMongoDBBundle(),
            //new Symfony\Bundle\PropelBundle\PropelBundle(),
            //new Symfony\Bundle\TwigBundle\TwigBundle(),

            // register your bundles
            new Application\AppBundle\AppBundle(),
        );

        if ($this->isDebug()) {
            $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
        }

        return $bundles;
    }

Отметьте, что вместе с ``HelloBundle``, о котором мы уже говорили, что ядро также подключает ``FrameworkBundle``, ``DoctrineBundle``,
``SwiftmailerBundle``, и ``ZendBundle``. Они входят в состав ядра фреймворка.

Каждый бандл может быть настроен при помощи конфигурационных файлов, написанных на YAML, XML, или PHP. Взгляните на конфигурацию по умолчанию:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        app.config:
            charset:       UTF-8
            error_handler: null
            csrf_secret:   xxxxxxxxxx
            router:        { resource: "%kernel.root_dir%/config/routing.yml" }
            validation:    { enabled: true, annotations: true }
            templating:
                escaping:       htmlspecialchars
                #assets_version: SomeVersionScheme
            #user:
            #    default_locale: fr
            #    session:
            #        name:     SYMFONY
            #        type:     Native
            #        lifetime: 3600

        ## Twig Configuration
        #twig.config:
        #    auto_reload: true

        ## Doctrine Configuration
        #doctrine.dbal:
        #    dbname:   xxxxxxxx
        #    user:     xxxxxxxx
        #    password: ~
        #doctrine.orm: ~

        ## Swiftmailer Configuration
        #swiftmailer.config:
        #    transport:  smtp
        #    encryption: ssl
        #    auth_mode:  login
        #    host:       smtp.gmail.com
        #    username:   xxxxxxxx
        #    password:   xxxxxxxx

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <app:config csrf-secret="xxxxxxxxxx" charset="UTF-8" error-handler="null">
            <app:router resource="%kernel.root_dir%/config/routing.xml" />
            <app:validation enabled="true" annotations="true" />
            <app:templating escaping="htmlspecialchars" />
            <!--
            <app:user default-locale="fr">
                <app:session name="SYMFONY" type="Native" lifetime="3600" />
            </app:user>
            //-->
        </app:config>

        <!-- Twig Configuration -->
        <!--
        <twig:config auto_reload="true" />
        -->

        <!-- Doctrine Configuration -->
        <!--
        <doctrine:dbal dbname="xxxxxxxx" user="xxxxxxxx" password="" />
        <doctrine:orm />
        -->

        <!-- Swiftmailer Configuration -->
        <!--
        <swiftmailer:config
            transport="smtp"
            encryption="ssl"
            auth_mode="login"
            host="smtp.gmail.com"
            username="xxxxxxxx"
            password="xxxxxxxx" />
        -->

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('app', 'config', array(
            'charset'       => 'UTF-8',
            'error_handler' => null,
            'csrf-secret'   => 'xxxxxxxxxx',
            'router'        => array('resource' => '%kernel.root_dir%/config/routing.php'),
            'validation'    => array('enabled' => true, 'annotations' => true),
            'templating'    => array(
                'escaping'        => 'htmlspecialchars'
                #'assets_version' => "SomeVersionScheme",
            ),
            #'user' => array(
            #    'default_locale' => "fr",
            #    'session' => array(
            #        'name' => "SYMFONY",
            #        'type' => "Native",
            #        'lifetime' => "3600",
            #    )
            #),
        ));

        // Twig Configuration
        /*
        $container->loadFromExtension('twig', 'config', array('auto_reload' => true));
        */

        // Doctrine Configuration
        /*
        $container->loadFromExtension('doctrine', 'dbal', array(
            'dbname'   => 'xxxxxxxx',
            'user'     => 'xxxxxxxx',
            'password' => '',
        ));
        $container->loadFromExtension('doctrine', 'orm');
        */

        // Swiftmailer Configuration
        /*
        $container->loadFromExtension('swiftmailer', 'config', array(
            'transport'  => "smtp",
            'encryption' => "ssl",
            'auth_mode'  => "login",
            'host'       => "smtp.gmail.com",
            'username'   => "xxxxxxxx",
            'password'   => "xxxxxxxx",
        ));
        */

Каждая запись наподобие ``app.config`` определяет конфигурацию бандла.

Каждое окружение :term:`environment` может перекрывать конфигурацию по умолчанию путем создания специфичного конфигурационного файла:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_dev.yml
        imports:
            - { resource: config.yml }

        app.config:
            router:   { resource: "%kernel.root_dir%/config/routing_dev.yml" }
            profiler: { only_exceptions: false }

        webprofiler.config:
            toolbar: true
            intercept_redirects: true

        zend.config:
            logger:
                priority: debug
                path:     %kernel.root_dir%/logs/%kernel.environment%.log

    .. code-block:: xml

        <!-- app/config/config_dev.xml -->
        <imports>
            <import resource="config.xml" />
        </imports>

        <app:config>
            <app:router resource="%kernel.root_dir%/config/routing_dev.xml" />
            <app:profiler only-exceptions="false" />
        </app:config>

        <webprofiler:config
            toolbar="true"
            intercept-redirects="true"
        />

        <zend:config>
            <zend:logger priority="info" path="%kernel.logs_dir%/%kernel.environment%.log" />
        </zend:config>

    .. code-block:: php

        // app/config/config.php
        $loader->import('config.php');

        $container->loadFromExtension('app', 'config', array(
            'router'   => array('resource' => '%kernel.root_dir%/config/routing_dev.php'),
            'profiler' => array('only-exceptions' => false),
        ));

        $container->loadFromExtension('webprofiler', 'config', array(
            'toolbar' => true,
            'intercept-redirects' => true,
        ));

        $container->loadFromExtension('zend', 'config', array(
            'logger' => array(
                'priority' => 'info',
                'path'     => '%kernel.logs_dir%/%kernel.environment%.log',
            ),
        ));

Как мы видели в предыдущей части, приложение состоит из бандлов объявленных в методе ``registerBundles()``, но откуда Symfony знает где искать бандлы? Symfony очень гибкий в этом плане. Метод ``registerBundleDirs()`` возвращает ассоциативный массив, который отображает пространство имен для любого допустимого каталога (локального или глобального)::

    public function registerBundleDirs()
    {
        return array(
            'Application'     => __DIR__.'/../src/Application',
            'Bundle'          => __DIR__.'/../src/Bundle',
            'Symfony\\Bundle' => __DIR__.'/../src/vendor/symfony/src/Symfony/Bundle',
        );
    }

Таким образом, когда вы ссылаетесь в имени контроллера или шаблона на ``HelloBundle``, Symfony будет искать их в данных директориях.

Теперь вы понимаете Symfony такой гибкий? Делитесь вашими бандлами между приложениями, храните их локально или глобально, на ваше усмотрение.

.. index::
   single: Вендоры

Вендоры
-------

Скорее всего ваше приложение будет зависеть от сторонних библиотек. Они должны храниться в директории ``src/vendor/``. Она уже содержит библиотеки Symfony, библиотеку SwiftMailer, Doctrine ORM, Propel ORM, систему шаблонов Twig и избранное из классов Zend Framework.

.. index::
   single: Cache
   single: Logs

Кеширование и Логи
--------------

Symfony, вероятно, это один из самых быстрых фреймворков. Но как он может быть таким быстрым, если он постоянно должен парсить и интерпретировать десятки YAML и XML файлов при каждом запросе? Частично это обязанность системы кеширования. Конфигурация приложения парсится только для первого запроса и после этого компилируется в обычный PHP код, который хранится в директории приложения ``cache/``. В окружении для разработки, Symfony сбрасывает кэш когда вы изменяете файл. Но в главном окружении, это уже ваша обязанность чистить кэш, когда вы обновляете ваш код или конфигурацию.

Пр разработке web приложения, вещи могут пойти не так, как надо разными способами. Файлы логов в директории приложения ``logs/`` раскажут вам все про запросы и помогут быстро устранить проблемы.

.. index::
   single: CLI
   single: Командная строка

Интерфейс Командной Строки
--------------------------

В состав каждого приложения входит интерфейс командной строки (``консоль``), который помогает вам обслуживать ваше приложение. Консоль предоставляет команды, которые увеличивают вашу продуктивность, автоматизируя частые и повторяющиеся задачи.

Запустите консоль без агрументов, для того чтобы получить представление о ее возможностях:

.. code-block:: bash

    $ php app/console

Опция ``--help`` поможет вам уточнить способ использования любой команды:

.. code-block:: bash

    $ php app/console router:debug --help

Заключительное Слово
--------------------

Называйте меня сумасшедшим, но после прочтения этой части, вы должны уметь заставить работать Symfony на вас быстро и комфортно. В Symfony все сделано так, чтобы вы могли настроить его на ваше усмотрение. Так что, перемещайте директории как вам угодно, не стесняйтесь.

И это все для быстрого тура. От тестирования до отправки электронной почты, вам все еще многое предстоит узнать чтобы стать мастером Symfony. Готовы погрузиться в изучение этих тем сейчас? Не откладывайте на потом, переходите к официальным страницам `руководств` страницам и выбирайте любую интересующую вас тему.

.. _standards:  http://groups.google.com/group/php-standards/web/psr-0-final-proposal
.. _convention: http://pear.php.net/
.. _guides:     http://www.symfony-reloaded.org/learn
