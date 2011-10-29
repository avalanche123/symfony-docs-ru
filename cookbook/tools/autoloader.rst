.. index::
   pair: Автозагрузчик; Конфигурирование

Как автоматически загружать классы
===================================

В случаях когда используются неопределенные классы, PHP использует
механизм автозагрузки которому поручает загрузку файла описывающего
класс. С Symfony2 поставляется универсальный автозагрузчик, который
может загружать классы из файлов, которые реализуют одно из следующих
соглашений:

* Технические `стандарты`_ взаимодействия для имен пространств
  и классов PHP 5.3;
* Соглашения именования классов в `PEAR`_.

Если ваши классы и библиотеки 3-х лиц которыми вы пользуетесь в проекте
следуют данным стандартам, автозагрузчик Symfony2 единственный автозагрузчик
который вам когда-либо понадобиться.

Использование
-------------

.. versionadded:: 2.1
   The ``useIncludePath`` method was added in Symfony 2.1.

Регистрация класса :class:`Symfony\\Component\\ClassLoader\\UniversalClassLoader`
автозагрузки проста:

.. code-block:: php

    <?php
    require_once '/path/to/src/Symfony/Component/ClassLoader/UniversalClassLoader.php';

    use Symfony\Component\ClassLoader\UniversalClassLoader;

    $loader = new UniversalClassLoader();

    // В качестве последней инстанции ищем в include_path.
    $loader->useIncludePath(true);

    $loader->register();

С целью улучшения быстродействия - пути к классам могут кэшироваться в памяти
при помощи APC - для этого необходимо регистрировать класс
:class:`Symfony\\Component\\ClassLoader\\ApcUniversalClassLoader`:

.. code-block:: php

    <?php
    require_once '/path/to/src/Symfony/Component/ClassLoader/UniversalClassLoader.php';
    require_once '/path/to/src/Symfony/Component/ClassLoader/ApcUniversalClassLoader.php';

    use Symfony\Component\ClassLoader\ApcUniversalClassLoader;

    $loader = new ApcUniversalClassLoader('apc.prefix.');
    $loader->register();

Автозагрузчик полезен только при условии того, что вы добавите несколько библиотек для
автозагрузки.

.. note::
    Автозагрузчик автоматически регистрируется приложением на Symfony2 (см.
    ``app/autoload.php``).

Если классы которые требуется автоматически загружать используют пространства имён,
применяйте методы :method:`Symfony\\Component\\ClassLoader\\UniversalClassLoader::registerNamespace`
или
:method:`Symfony\\Component\\ClassLoader\\UniversalClassLoader::registerNamespaces`:

.. code-block:: php

    <?php
    $loader->registerNamespace('Symfony', __DIR__.'/vendor/symfony/src');

    $loader->registerNamespaces(array(
        'Symfony' => __DIR__.'/../vendor/symfony/src',
        'Monolog' => __DIR__.'/../vendor/monolog/src',
    ));

    $loader->register();

Для классов которые используют соглашения об именовании в стиле
PEAR, используйте метод :method:`Symfony\\Component\\ClassLoader\\UniversalClassLoader::registerPrefix`
или :method:`Symfony\\Component\\ClassLoader\\UniversalClassLoader::registerPrefixes`:

.. code-block:: php

    <?php
    $loader->registerPrefix('Twig_', __DIR__.'/vendor/twig/lib');

    $loader->registerPrefixes(array(
        'Swift_' => __DIR__.'/vendor/swiftmailer/lib/classes',
        'Twig_'  => __DIR__.'/vendor/twig/lib',
    ));

    $loader->register();

.. note::
    Некоторые библиотеки требуют чтобы их корневой каталог также был
    включен в качестве пути для поиска PHP (``set_include_path()``).

Классы находящиеся в подпространствах имён или в суб-иерархии классов PEAR
можно легко сгруппировать в подмножества, которые можно использовать в
больших проектах:

.. code-block:: php

    <?php
    $loader->registerNamespaces(array(
        'Doctrine\\Common'           => __DIR__.'/vendor/doctrine-common/lib',
        'Doctrine\\DBAL\\Migrations' => __DIR__.'/vendor/doctrine-migrations/lib',
        'Doctrine\\DBAL'             => __DIR__.'/vendor/doctrine-dbal/lib',
        'Doctrine'                   => __DIR__.'/vendor/doctrine/lib',
    ));

    $loader->register();

В этом примере, при попытке использования класса в пространстве имен ``Doctrine\Common``
или его потомков, автозагрузчик в первую очередь просмотрит в поисках класса каталог
``doctrine-common``, и в последнюю очередь каталог ``Doctrine`` (который сконфигурирован
последним). В данном случае порядок регистрации классов важен.

.. _стандарты: http://groups.google.com/group/php-standards/web/psr-0-final-proposal
.. _PEAR:      http://pear.php.net/manual/en/standards.php

.. toctree::
    :hidden:

    Translation source: n/a
    Corrected from: 2011-10-29 4459652
