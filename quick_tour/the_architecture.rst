Архитектура (needs correction)
================

Вы мой герой! Кто бы мог подумать что вы всё ещё будете здесь после первых трёх
частей? Ваши усилия скоро будут вознаграждены. В первых трёх частях мы глубоко
не вникали в архитектуру фреймворка. Но так как она выделяет Symfony2 из толпы
фреймворков, давайте сейчас же в неё погрузимся.

.. index::
   single: Directory Structure

Структура папок
------------------

Структура папок приложения (:term:`application`) на Symfony2 довольно гибкая,
но типичная и рекомендованная структура папок показана в песочнице Symfony2:

* ``app/``: Эта папка содержит конфигурацию приложения;

* ``src/``: Весь PHP код хранится здесь;

* ``web/``: Это папка должна быть корневой web директорией.

Web директория
~~~~~~~~~~~~~~~~~

Корневая web директория - это дом для всех публичных и статичных файлов, таких
как изображения, таблицы стилей и файлы JavaScript. Здесь также обитает
:term:`front controller`::

    // web/app.php
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->handle(new Request())->send();

Как и любой фронт контроллер, ``app.php`` использует Kernel Class,
``AppKernel``, чтобы начать загрузку приложения.

.. index::
   single: Kernel

Папка приложения
~~~~~~~~~~~~~~~~~~~~~~~~~

Класс ``AppKernel`` это главная входная точка конфигурации приложения, поэтому
он содержится в директории ``app/``.

Этот класс должен реализовывать четыре метода:

* ``registerRootDir()``: Возвращает корневую папку конфигурации;

* ``registerBundles()``: Возвращает массив всех бандлов, необходимых для
  запуска приложения (см. ссылку ``Application\HelloBundle\HelloBundle``);

* ``registerBundleDirs()``: Возвращает массив, связывающий пространства имён и
  их домашние директории;

* ``registerContainerConfiguration()``: Загружает конфигурацию  (об этом чуть позже);

Обратите внимание на типичную реализацию этих методов для того чтобы лучше
понять гибкость фреймворка.

Чтобы всё это заработало, ядру необходим один файл из директории ``src/``::

    // app/AppKernel.php
    require_once __DIR__.'/../src/autoload.php';

Папка с исходниками
~~~~~~~~~~~~~~~~~~~~

Файл ``src/autoload.php`` ответственен за автозагрузку всех PHP классов,
которые используются в приложении::

    // src/autoload.php
    $vendorDir = __DIR__.'/vendor';

    require_once $vendorDir.'/symfony/src/Symfony/Component/HttpFoundation/UniversalClassLoader.php';

    use Symfony\Component\HttpFoundation\UniversalClassLoader;

    $loader = new UniversalClassLoader();
    $loader->registerNamespaces(array(
        'Symfony'                        => $vendorDir.'/symfony/src',
        'Application'                    => __DIR__,
        'Bundle'                         => __DIR__,
        'Doctrine\\Common\\DataFixtures' => $vendorDir.'/doctrine-data-fixtures/lib',
        'Doctrine\\Common'               => $vendorDir.'/doctrine-common/lib',
        'Doctrine\\DBAL\\Migrations'     => $vendorDir.'/doctrine-migrations/lib',
        'Doctrine\\MongoDB'              => $vendorDir.'/doctrine-mongodb/lib',
        'Doctrine\\ODM\\MongoDB'         => $vendorDir.'/doctrine-mongodb-odm/lib',
        'Doctrine\\DBAL'                 => $vendorDir.'/doctrine-dbal/lib',
        'Doctrine'                       => $vendorDir.'/doctrine/lib',
        'Zend'                           => $vendorDir.'/zend/library',
    ));
    $loader->registerPrefixes(array(
        'Swift_'           => $vendorDir.'/swiftmailer/lib/classes',
        'Twig_Extensions_' => $vendorDir.'/twig-extensions/lib',
        'Twig_'            => $vendorDir.'/twig/lib',
    ));
    $loader->register();

``UniversalClassLoader`` из Symfony2 используется для автозагрузки файлов,
которые относятся либо соотвествуют техническим стандартам `standards`_ для
пространств имён в PHP 5.3 или соглашению `convention`_ о наименованиях для
классов в PEAR. Как вы видите, все зависимости хранятся в папке ``vendor/``,
но это просто соглашение. Можете хранить их где пожелаете, глобально на сервере
или локально в проекте.

.. index::
   single: Bundles

Система бандлов
-----------------

Этот раздел кратко поведает вам об одной из существеннейших и наиболее мощных
особенностей Symfony2, о системе бандлов :term:`bundle`.

Бандл в некотором роде как плагин в других программах. Почему его назвали
*бандл*, а не *плагин*? Потому что *всё что угодно* в Symfony2 это бандл, от
ключевых особенностей фреймворка до кода, который вы пишете для приложения.
Бандлы это высшая каста в Symfony2. Это даёт вам гибкость в применении как уже
встроенных особенностей сторонних бандлов, так и в написании своих собственных.
Бандл позволяет выбрать необходимые для приложения особенности и оптимизировать
их как вы этого хотите.

Приложение составлено из бандлов, объявленных в методе ``registerBundles()``
класса ``AppKernel``::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            new Symfony\Bundle\TwigBundle\TwigBundle(),

            // enable third-party bundles
            new Symfony\Bundle\ZendBundle\ZendBundle(),
            new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            new Symfony\Bundle\DoctrineBundle\DoctrineBundle(),
            //new Symfony\Bundle\DoctrineMigrationsBundle\DoctrineMigrationsBundle(),
            //new Symfony\Bundle\DoctrineMongoDBBundle\DoctrineMongoDBBundle(),

            // register your bundles
            new Application\HelloBundle\HelloBundle(),
        );

        if ($this->isDebug()) {
            $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
        }

        return $bundles;
    }

В дополнение к ``HelloBundle``, о котором мы недавно говорили, заметьте что ядро
также включает ``FrameworkBundle``, ``DoctrineBundle``, ``SwiftmailerBundle`` и
``ZendBundle``. Все они части ядра фрэймворка.

Каждый бандл может быть настроен при помощи конфигурационных файлов, написанных
на YAML, XML, или PHP. Взгляните на конфигурацию по умолчанию:

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
                #assets_version: SomeVersionScheme
            session:
                default_locale: en
                lifetime: 3600

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
            <app:session default-locale="en" lifetime="3600" />
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
                #'assets_version' => "SomeVersionScheme",
            ),
            'session' => array(
                'default_locale' => "en",
                'lifetime' => "3600",
            ),
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

Каждая запись ``app.config`` указывает на настройку для бандла.

Каждое `окружение` (:term:`environment`) может переопределять стандартную
конфигурацию, задавая специфичный конфигурационный файл:

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
                path:     %kernel.logs_dir%/%kernel.environment%.log

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

        // app/config/config_dev.php
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

В предыдущей участке кода вы могли убедиться что приложение состоит из бандлов,
определённых в методе ``registerBundles()``. Но откуда Symfony2 знает где их
искать? Symfony2 и здесь достаточно гибок. Метод ``registerBundleDirs()`` должен
возвратить ассоциативный массив, который связывает пространства имён с любой
доступной папкой (локальной или глобальной)::

    public function registerBundleDirs()
    {
        return array(
            'Application'     => __DIR__.'/../src/Application',
            'Bundle'          => __DIR__.'/../src/Bundle',
            'Symfony\\Bundle' => __DIR__.'/../src/vendor/symfony/src/Symfony/Bundle',
        );
    }

Таким образом, когда вы ссылаетесь на ``HelloBundle`` в имени контроллера или
в имени шаблона, Symfony2 будет искать их в данных директориях.

Теперь вы понимаете почему Symfony2 такой гибкий? Делитесь вашими бандлами
между приложениями, храните их локально или глобально, всё на ваш выбор.

.. index::
   single: Vendors

Применение вендоров
-------------

Скорее всего ваше приложение будет зависеть и от сторонних библиотек. Они должны
хранится в папке ``src/vendor/``. Она уже содержит библиотеки Symfony2,
библиотеку SwiftMailer, Doctrine ORM, систему шаблонизации Twig и выборку из
классов Zend Framework.

.. index::
   single: Configuration Cache
   single: Logs

Кэширование и Логи
--------------

Symfony2 пожалуй одна из быстрейших среди многофункциональных фреймворков. Но
откуда взяться такой скорости когда она анализирует и интерпретирует десятки
YAML и XML для каждого запроса? Отчасти это благодаря системе кэширования.
Конфигурация приложения анализируется только при первом запросе, затем она
компилируется в чистый PHP и хранится в ``cache/`` папке приложения. В среде
разработки Symfony2 достаточно умён чтобы очищать кэш когда вы измените файл.
Но в производственной среде, когда вы изменяете код или конфигурацию, то
ответственность по очистке кэша перекладывается на вас.

Когда разрабатывается web приложение, многое может пойти не так. Логи в ``logs/``
в папке приложения расскажут вам всё о запросах и помогут быстро решить проблемы.

.. index::
   single: CLI
   single: Command Line

Интерфейс командной строки
--------------------------

Все приложения идут с интерфейсом командной строки (``консоль``), который
помогает обслуживать приложение. Он предоставляет команды, которые увеличивают
вашу продуктивность, автоматизируя частые и повторяющиеся задачи.

Запустите консоль без агрументов, чтобы получить представление о её возможностях:

.. code-block:: bash

    $ php app/console

Опция ``--help`` поможет вам уточнить возможности использования команды:

.. code-block:: bash

    $ php app/console router:debug --help

Заключительное слово
--------------------

Называйте меня сумасшедшим, но после прочтения этой части, вам должно быть
комфортно перемещать любые вещи и при этом заставить Symfony2 работать на вас.
В Symfony2 всё сделано так, чтобы вы смогли настроить его на ваше усмотрение.
Так что, переименовывайте и перемещайте директории как вам угодно.

Для начала этого достаточно. Вам ещё предстоит многому научиться, от
тестирования до отправки почты, чтобы стать мастером Symfony2. Готовы
погрузиться в чтение сейчас? Следуйте на официальную страницу руководств
(`guides`_) и выбирайте любую тему.

.. _standards:  http://groups.google.com/group/php-standards/web/psr-0-final-proposal
.. _convention: http://pear.php.net/
.. _guides:     http://www.symfony-reloaded.org/learn
