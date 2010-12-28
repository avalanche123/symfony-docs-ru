Архитектура
================

Вы мой герой! Кто бы мог подумать что вы все еще будете здесь после первых трех частей? Ваши усилия скоро будут вознаграждены. В первых трех частях мы глубоко не вникали в архитектуру фреймворка. Так как она выделяет Symfony2 из толпы фреймворков, давайте сейчас же в нее погрузимся.

.. index::
   single: Directory Structure

Структура Директорий
-----------------------

The directory structure of a Symfony2 :term:`application` is rather flexible
but the directory structure of the sandbox reflects the typical and recommended
structure of a Symfony2 application:

* ``app/``: В этой категории содержится конфигурация приложения;

* ``src/``: Весь PHP код содержится в этой директории;

* ``web/``: Это корневая web директория проекта.

Web Директория
~~~~~~~~~~~~~~~~~

The web root directory is the home of all public and static files like images,
stylesheets, and JavaScript files. It is also where each :term:`front controller`
lives::

    // web/app.php
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->handle(new Request())->send();

Like any front controller, ``app.php`` uses a Kernel Class, ``AppKernel``, to
bootstrap the application.

.. index::
   single: Kernel

Директория Приложения
~~~~~~~~~~~~~~~~~~~~~~~~~

Класс ``AppKernel`` это главная входная точка конфигурации приложения как такового, и содержится в директории ``app/``.

Этот класс должен реализовывать четыре метода:

* ``registerRootDir()``: Returns the configuration root directory;

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
        'Symfony'                        => $vendorDir.'/symfony/src',
        'Application'                    => __DIR__,
        'Bundle'                         => __DIR__,
        'Doctrine\\Common\\DataFixtures' => $vendorDir.'/doctrine-data-fixtures/lib',
        'Doctrine\\Common'               => $vendorDir.'/doctrine-common/lib',
        'Doctrine\\DBAL\\Migrations'     => $vendorDir.'/doctrine-migrations/lib',
        'Doctrine\\ODM\\MongoDB'         => $vendorDir.'/doctrine-mongodb/lib',
        'Doctrine\\DBAL'                 => $vendorDir.'/doctrine-dbal/lib',
        'Doctrine'                       => $vendorDir.'/doctrine/lib',
        'Zend'                           => $vendorDir.'/zend/library',
    ));
    $loader->registerPrefixes(array(
        'Swift_' => $vendorDir.'/swiftmailer/lib/classes',
        'Twig_'  => $vendorDir.'/twig/lib',
    ));
    $loader->register();

The ``UniversalClassLoader`` from Symfony2 is used to autoload files that
respect either the technical interoperability `standards`_ for PHP 5.3
namespaces or the PEAR naming `convention`_ for classes. As you can see
here, all dependencies are stored under the ``vendor/`` directory, but this is
just a convention. You can store them wherever you want, globally on your
server or locally in your projects.

.. index::
   single: Bundles

Система Бандлов
-----------------

This section starts to scratch the surface of one of the greatest and most
powerful features of Symfony2, the :term:`bundle` system.

A bundle is kind of like a plugin in other software. So why is it called
bundle and not plugin? Because *everything* is a bundle in Symfony2, from
the core framework features to the code you write for your application.
Bundles are first-class citizens in Symfony2. This gives you the flexibility to
use pre-built features packaged in third-party bundles or to distribute your
own bundles. It makes it easy to pick and choose which features to enable
in your application and optimize them the way you want.

An application is made up of bundles as defined in the ``registerBundles()``
method of the ``AppKernel`` class::

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

In addition to the ``HelloBundle`` that we have already talked about, notice
that the kernel also enables ``FrameworkBundle``, ``DoctrineBundle``,
``SwiftmailerBundle``, and ``ZendBundle``. They are all part of the core
framework.

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

As we have seen in the previous part, an application is made up of bundles
defined in the ``registerBundles()`` method. But how does Symfony2 know where
to look for bundles? Symfony2 is quite flexible in this regard. The
``registerBundleDirs()`` method must return an associative array that maps
namespaces to any valid directory (local or global ones)::

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
   single: Vendors

Using Vendors
-------------

Odds are that your application will depend on third-party libraries. Those
should be stored in the ``src/vendor/`` directory. This directory already
contains the Symfony2 libraries, the SwiftMailer library, the Doctrine ORM,
the Twig templating system, and a selection of the Zend Framework classes.

.. index::
   single: Configuration Cache
   single: Logs

Кэширование и Логи
--------------

Symfony2 is probably one of the fastest full-stack frameworks around. But how
can it be so fast if it parses and interprets tens of YAML and XML files for
each request? This is partly due to its cache system. The application
configuration is only parsed for the very first request and then compiled down
to plain PHP code stored in the ``cache/`` application directory. In the
development environment, Symfony2 is smart enough to flush the cache when you
change a file. But in the production environment, it is your responsibility
to clear the cache when you update your code or change its configuration.

When developing a web application, things can go wrong in many ways. The log
files in the ``logs/`` application directory tell you everything about the
requests and help you fix the problem quickly.

.. index::
   single: CLI
   single: Command Line

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

And that's all for the quick tour. From testing to sending emails, you still
need to learn a lot to become a Symfony2 master. Ready to dig into these topics
now? Look no further - go to the official `guides`_ page and pick any topic you
want.

.. _standards:  http://groups.google.com/group/php-standards/web/psr-0-final-proposal
.. _convention: http://pear.php.net/
.. _guides:     http://www.symfony-reloaded.org/learn
