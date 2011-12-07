.. index::
   single: PHP Templates

Как использовать PHP шаблоны вмесро Twig
========================================

Не смотря на то, что Symfony2 по умолчанию использует шаблоны Twig в частве
шаблонного движка, вы можете использовать PHP шаблоны, если захотите. Оба шаблонных
движка одинаково поддерживаются в Symfony2. Symfony2 также добавляет к PHP
шаблонам несколько удобных фич, чтобы сделать их более мощными.

Отображение PHP шаблонов
-----------------------

Если вы хотите использовать PHP шаблоны, во-первых, убедитесь что вы их активировали
в настройках приложения:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            # ...
            templating:    { engines: ['twig', 'php'] }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config ... >
            <!-- ... -->
            <framework:templating ... >
                <framework:engine id="twig" />
                <framework:engine id="php" />
            </framework:templating>
        </framework:config>

    .. code-block:: php

        $container->loadFromExtension('framework', array(
            // ...
            'templating'      => array(
                'engines' => array('twig', 'php'),
            ),
        ));

Теперь вы можете использовать PHP шаблоны взамен Twig, просто указывая
расширение ``.php`` для ваших шаблонов, а не ``.twig``. Контроллер из
примера ниже отображает шаблон ``index.html.php``:

.. code-block:: php

    <?php
    // src/Acme/HelloBundle/Controller/HelloController.php

    public function indexAction($name)
    {
        return $this->render('AcmeHelloBundle:Hello:index.html.php', array('name' => $name));
    }

.. index::
  single: Templating; Layout
  single: Layout

Декорирование шаблонов
----------------------

Чаще всего, шаблоны используют некоторые общие элементы, например всем известные
header и footer. В Symfony2 этот вопрос решается несколько иначе - шаблон может
быть декорирован другим шаблоном.

Шаблон ``index.html.php`` будет декорироваться шаблоном ``layout.html.php``
благодаря вызову метода ``extend()``:

.. code-block:: html+php

    <!-- src/Acme/HelloBundle/Resources/views/Hello/index.html.php -->
    <?php $view->extend('AcmeHelloBundle::layout.html.php') ?>

    Hello <?php echo $name ?>!

Нотация ``AcmeHelloBundle::layout.html.php`` выглядит знакомо, не так ли? Это
такая же нотация, которая используется для ссылки на шаблон. Часть ``::``
означает, что элемент "контроллер", таким образом соответствующий файл
распложен напрямую в директории ``views/``.

Теперь давайте взглянем на файл ``layout.html.php``:

.. code-block:: html+php

    <!-- src/Acme/HelloBundle/Resources/views/layout.html.php -->
    <?php $view->extend('::base.html.php') ?>

    <h1>Hello Application</h1>

    <?php $view['slots']->output('_content') ?>

Декорирующий шаблон (layout) в свою очередь декорирован другим шаблоном (``::base.html.php``).
Symfony2 поддерживает множественные уровни декорирования: layout может быть декорирован
другим шаблоном более высокого уровня. Когда часть "bundle" в наименовании шаблона
пуста, он ищется в директории ``app/Resources/views/``. Она содержит глобальные
шаблоны уровня приложения:

.. code-block:: html+php

    <!-- app/Resources/views/base.html.php -->
    <!DOCTYPE html>
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
            <title><?php $view['slots']->output('title', 'Hello Application') ?></title>
        </head>
        <body>
            <?php $view['slots']->output('_content') ?>
        </body>
    </html>

Для обоих шаблонов, выражение ``$view['slots']->output('_content')`` заменяется
контентом дочернего шаблона: ``index.html.php`` и ``layout.html.php`` соответственно
(о слотах подробнее поговорим в следующей секции).

Как вы можете видеть, Symfony2 предоставляет методы посредством объекта ``$view``.
В шаблоне переменная ``$view``  всегда доступна и представляет собой специальный объект,
которые предоставляет пакет методов, которые собственно и крутят винтики в движке
шаблонизатора.

.. index::
   single: Templating; Slot
   single: Slot

Работаем со Слотами
------------------

Слот - это некоторый кусочек кода, определённый в шаблоне и который можно
повторно использовать в любом декорирующем шаблоне. В шаблоне ``index.html.php``
определён слот ``title``:

.. code-block:: html+php

    <!-- src/Acme/HelloBundle/Resources/views/Hello/index.html.php -->
    <?php $view->extend('AcmeHelloBundle::layout.html.php') ?>

    <?php $view['slots']->set('title', 'Hello World Application') ?>

    Hello <?php echo $name ?>!

Базовый шаблон уже имеет код для вывода заголовка:

.. code-block:: html+php

    <!-- app/Resources/views/base.html.php -->
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <title><?php $view['slots']->output('title', 'Hello Application') ?></title>
    </head>

Метод ``output()`` вставляет контент слота и опционально принимает значение по умолчанию,
которое будет использовано, если слот не будет определён. ``_content`` - это
особый слот, который содержит рендер дочернего шаблона.

Для больших слотов можно использовать расширенный синтаксис:

.. code-block:: html+php

    <?php $view['slots']->start('title') ?>
        Some large amount of HTML
    <?php $view['slots']->stop() ?>

.. index::
   single: Templating; Include

Включение других шаблонов
-------------------------

Наилучшим способом сделать доступным в других шаблонах некоторый кусочек кода -
это включить его в другие шаблорны.

Создадим шаблон ``hello.html.php``:

.. code-block:: html+php

    <!-- src/Acme/HelloBundle/Resources/views/Hello/hello.html.php -->
    Hello <?php echo $name ?>!

И изменим шаблон ``index.html.php`` таким образом чтобы он его подключал:

.. code-block:: html+php

    <!-- src/Acme/HelloBundle/Resources/views/Hello/index.html.php -->
    <?php $view->extend('AcmeHelloBundle::layout.html.php') ?>

    <?php echo $view->render('AcmeHelloBundle:Hello:hello.html.php', array('name' => $name)) ?>

Метод ``render()`` вычисляет и возвращает значение другого шаблона (это такой
же метод, который используется в контроллере).

.. index::
   single: Templating; Embedding Pages

Встраивание Контроллеров
---------------------------

Что, если вы захотите встроить результат выполнения другого контроллера в шаблон?
Это очень удобно при работе с Ajax, или же когда встраиваемый шаблон требует
некоторые переменные, не доступные в главном шаблоне.

Если вы создадите действие ``fancy`` и захотите включить его в
шаблон ``index.html.php``, просто используйте такой код:

.. code-block:: html+php

    <!-- src/Acme/HelloBundle/Resources/views/Hello/index.html.php -->
    <?php echo $view['actions']->render('AcmeHelloBundle:Hello:fancy', array('name' => $name, 'color' => 'green')) ?>

Строка ``AcmeHelloBundle:Hello:fancy`` ссылается на действие ``fancy`` контроллера ``Hello``:

.. code-block:: php

    <?php
    // src/Acme/HelloBundle/Controller/HelloController.php

    class HelloController extends Controller
    {
        public function fancyAction($name, $color)
        {
            // create some object, based on the $color variable
            $object = ...;

            return $this->render('AcmeHelloBundle:Hello:fancy.html.php', array('name' => $name, 'object' => $object));
        }

        // ...
    }

Но где же определён элемент массива ``$view['actions']``? Как и ``$view['slots']``,
здесь вызывается хелпер и в следующей секции об этом будет чуть подробнее.

.. index::
   single: Templating; Helpers

Использование хелперов в шаблонах
----------------------

Система шаблонов Symfony2 может быть легко и просто при помощи хелперов. Хелперы
это PHP объекты, которые предоставляют удобные фичи в контексте шаблонов. ``actions``
и ``slots`` - это два хелпера из числа встроенных в Symfony2.

Создание ссылок между страницами
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Говоря о веб-приложениях, создание ссылок между страницами - это крайне необходимая
функция. Вместо того, чтобы хордкодить URLы в шаблонах, нужно использовать
хелпер ``router``, который знает как генерировать URL, основываясь на конфигурацию
маршрутизатора. Если использовать этот подход, любой URL может быть легко
обновлён при помощи изменения конфигурации:

.. code-block:: html+php

    <a href="<?php echo $view['router']->generate('hello', array('name' => 'Thomas')) ?>">
        Greet Thomas!
    </a>

Метод ``generate()`` принимает имя маршрута и массив параметров в качестве аргументов.
Имя маршрута - это ключ, по которому определяется маршрут, а параметры - это значения
плэйсхолдеров из шаблона маршрута:

.. code-block:: yaml

    # src/Acme/HelloBundle/Resources/config/routing.yml
    hello: # The route name
        pattern:  /hello/{name}
        defaults: { _controller: AcmeHelloBundle:Hello:index }

Работа с ресурсами: картинки, JavaScript, CSS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Чем бы был интернет без картинок, джаваскриптов и стилей? Symfony2 предоставляет
вам таг ``assets`` для работы с ними:

.. code-block:: html+php

    <link href="<?php echo $view['assets']->getUrl('css/blog.css') ?>" rel="stylesheet" type="text/css" />

    <img src="<?php echo $view['assets']->getUrl('images/logo.png') ?>" />

Главной обязанностью хелпера ``assets`` - сделать ваше приложение более
портируемым. Благодаря этому хелперу вы можете перемещать корень вашего
приложения внутри web root не меняя ничего у коде шаблонов.

Экранирование
---------------

При использовании PHP шаблонов необходимо экранировать все переменные::

    <?php echo $view->escape($var) ?>

По умолчанию, метод ``escape()`` полагает, что переменная выводится в контексте HTML.
Второй аргумент метода позволяет изменить контекст. Например, выводя что-то в JavaScript,
используйте ``js`` контекст::

    <?php echo $view->escape($var, 'js') ?>

.. toctree::
    :hidden:

    Translation source: 2011-12-06 4cd15f3
