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

* ``registerContainerConfiguration()``: RВозвращает главный объект конфигурации (об этом подробнее ниже);

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

The ``UniversalClassLoader`` from Symfony is used to autoload files that
respect either the technical interoperability `standards`_ for PHP 5.3
namespaces or the PEAR naming `convention`_ for classes. Как вы можете видеть, все зависимости хранятся в директории ``vendor/``, но это только соглашение. Вы можете хранить их где захотите, глобально на вашем сервере или локально в ваших проектах.

.. index::
   single: Бандлы

Система Бандлов
-----------------

This section starts to scratch the surface of one of the greatest and more
powerful features of Symfony, its :term:`bundle` system.

A bundle is kind of like a plugin in other software. But why is it called
bundle and not plugin then? Because everything is a bundle in Symfony, from
the core framework features to the code you write for your application.
Bundles are first-class citizens in Symfony. This gives you the flexibility to
use pre-built features packaged in third-party bundles or to distribute your
own bundles. It makes it so easy to pick and choose which features to enable
in your application and optimize them the way you want.

An application is made up of bundles as defined in the ``registerBundles()``
method of the ``AppKernel`` class::

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

Along side the ``HelloBundle`` we have already talked about, notice that the
kernel also enables ``FrameworkBundle``, ``DoctrineBundle``,
``SwiftmailerBundle``, and ``ZendBundle``. They are all part of the core
framework.

Each bundle can be customized via configuration files written in YAML, XML, or
PHP. Have a look at the default configuration:

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

Each entry like ``app.config`` defines the configuration for a bundle.

Each :term:`environment` can override the default configuration by providing a
specific configuration file:

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

As we have seen in the previous part, an application is made of bundles as
defined in the ``registerBundles()`` method but how does Symfony know where to
look for bundles? Symfony is quite flexible in this regard. The
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

So, when you reference the ``HelloBundle`` in a controller name or in a template
name, Symfony will look for it under the given directories.

Do you understand now why Symfony is so flexible? Share your bundles between
applications, store them locally or globally, your choice.

.. index::
   single: Vendors

Vendors
-------

Odds are your application will depend on third-party libraries. Those should
be stored in the ``src/vendor/`` directory. It already contains the Symfony
libraries, the SwiftMailer library, the Doctrine ORM, the Propel ORM, the Twig
templating system, and a selection of the Zend Framework classes.

.. index::
   single: Cache
   single: Logs

Cache and Logs
--------------

Symfony is probably one of the fastest full-stack frameworks around. But how
can it be so fast if it parses and interprets tens of YAML and XML files for
each request? This is partly due to its cache system. The application
configuration is only parsed for the very first request and then compiled down
to plain PHP code stored in the ``cache/`` application directory. In the
development environment, Symfony is smart enough to flush the cache when you
change a file. But in the production one, it is your responsibility to clear
the cache when you update your code or change its configuration.

When developing a web application, things can go wrong in many ways. The log
files in the ``logs/`` application directory tell you everything about the
requests and helps you fix the problem in no time.

.. index::
   single: CLI
   single: Command Line

The Command Line Interface
--------------------------

Each application comes with a command line interface tool (``console``) that
helps you maintain your application. It provides commands that boost your
productivity by automating tedious and repetitive tasks.

Run it without any arguments to learn more about its capabilities:

.. code-block:: bash

    $ php app/console

The ``--help`` option helps you discover the usage of a command:

.. code-block:: bash

    $ php app/console router:debug --help

Final Thoughts
--------------

Call me crazy, but after reading this part, you should be comfortable with
moving things around and making Symfony work for you. Everything is done in
Symfony to stand out of your way. So, feel free to rename and move directories
around as you see fit.

And that's all for the quick tour. From testing to sending emails, you still
need to learn of lot to become a Symfony master. Ready to dig into these
topics now? Look no further, go to the official `guides`_ page and pick any
topic you want.

.. _standards:  http://groups.google.com/group/php-standards/web/psr-0-final-proposal
.. _convention: http://pear.php.net/
.. _guides:     http://www.symfony-reloaded.org/learn
