Вид
===

После прочтения первой части данного руководства, вы решили что стоит потратить на Symfony2 еще 10 минут. Отлично. Во второй части вы больше узнаете о системе шаблонов в Symfony2. Как вы могли видеть ранее, Symfony2 использует PHP в качестве шаблонного движка по-умолчанию, но добавляет некоторые высокоуровневые особенности, что делает его более мощным.

Вместо PHP, вы также можете использовать `Twig`_ (он делает ваши шаблоны более выразительными и более дружественными для веб дизайнеров). Если вы предпочитаете использовать `Twig`, прочитайте альтернативную главу :doc:`View with Twig <the_view_with_twig>`.

.. index::
  single: Шаблон; Макет
  single: Макет

Декорирование Шаблонов
----------------------

Довольно часто, шаблоны в проектах используют общие элементы, такие, как всем известные header и footer. В Symfony, мы подходим к этому вопросы по-другому: шаблон может быть декорирован другим шаблоном.

Шаблон ``index`` декорирован шаблоном ``layout.php``, благодаря вызову ``extend()``:

.. code-block:: html+php

    <!-- src/Application/HelloBundle/Resources/views/Hello/index.php -->
    <?php $view->extend('HelloBundle::layout') ?>

    Hello <?php echo $name ?>!

Нотация ``HelloBundle::layout`` звучит знакомо, не так ли? Это такая же нотация, как и для привязки шаблона. Часть ``::`` просто обозначает, что контроллер не указан, и, следовательно, соответствующий файл хранится напрямую во ``views/``.

Сейчас, давайте взглянем на файл ``layout.php``:

.. code-block:: html+php

    <!-- src/Application/HelloBundle/Resources/views/layout.php -->
    <?php $view->extend('::layout') ?>

    <h1>Hello Application</h1>

    <?php $view['slots']->output('_content') ?>

Макет - это он сам, декорированный другим макетом (``::layout``). Symfony поддерживает сложные уровни декорирования: макет может декорировать себя другим макетом. Когда часть с названием бандла в имени шаблона пуста, виды будут браться из директории ``app/views/``. В этой директории содержатся глобальные виды всего вашего проекта:

.. code-block:: html+php

    <!-- app/views/layout.php -->
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
            <title><?php $view['slots']->output('title', 'Hello Application') ?></title>
        </head>
        <body>
            <?php $view['slots']->output('_content') ?>
        </body>
    </html>

Для обоих макетов, выражение ``$view['slots']->output('_content')`` заменяется содержимым дочернего шаблона, ``index.php`` и ``layout.php`` соответственно (больше о слотах в следующей секции).

Как вы можете видеть, Symfony предоставляет метод загадочного объекта ``$view``. В шаблоне, переменная ``$view`` всегда доступна и ссылается на специальный объект, который предоставляет группу методов и свойств которые заставляют механизм шаблонов "тикать как часы".

.. index::
   single: Шаблон; Слот
   single: Слот

Слоты
-----

Слот – это кусочек кода, определенный в шаблоне, который может быть использован в любом макете, декорирующем шаблон. Определим слот ``title`` в шаблоне index:

.. code-block:: html+php

    <!-- src/Application/HelloBundle/Resources/views/Hello/index.php -->
    <?php $view->extend('HelloBundle::layout') ?>

    <?php $view['slots']->set('title', 'Hello World app') ?>

    Hello <?php echo $name ?>!

Базовый макет уже содержит код для вывода в title:

.. code-block:: html+php

    <!-- app/views/layout.php -->
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <title><?php $view['slots']->output('title', 'Hello Application') ?></title>
    </head>

Метод ``output()`` вставляет содержимое слота и может принимать значение по умолчанию, если слот не установлен. А ``_content`` представляет собой специальный слот, который содержит обработанный дочерний шаблон.

Для больших слотов, также существует расширенный синтаксис:

.. code-block:: html+php

    <?php $view['slots']->start('title') ?>
        Some large amount of HTML
    <?php $view['slots']->stop() ?>

.. index::
   single: Шаблон; Включать

Включение сторонних шаблонов
-----------------------
Лучшим способом, для того чтобы кусочек кода можно было использовать во многих различных шаблонах, будет определить шаблон, который может быть включен в любой другой шаблон.

Создайте шаблон ``hello.php``:

.. code-block:: html+php

    <!-- src/Application/HelloBundle/Resources/views/Hello/hello.php -->
    Hello <?php echo $name ?>!

И измените шаблон ``index.php`` чтобы подключить его:

.. code-block:: html+php

    <!-- src/Application/HelloBundle/Resources/views/Hello/index.php -->
    <?php $view->extend('HelloBundle::layout') ?>

    <?php echo $view->render('HelloBundle:Hello:hello', array('name' => $name)) ?>

Метод ``render()`` вычисляет и возвращает содержимое другого шаблона (это точно такой же метод, который используется в контроллере).

.. index::
   single: Шаблон; Встроенные Страницы

Встраивание других Действий
-------------------

Что если вы хотите включить результат других действий в шаблон?
Это очень удобно работая с Ajax, или когда включаемый шаблон обращается к некоторой переменной, которая недоступна в главном шаблоне.

Если вы создаете действие ``fancy``, и хотите включить его в шаблон ``index``, просто используйте следующий код:

.. code-block:: html+php

    <!-- src/Application/HelloBundle/Resources/views/Hello/index.php -->
    <?php $view['actions']->output('HelloBundle:Hello:fancy', array('name' => $name, 'color' => 'green')) ?>

Здесь, строка ``HelloBundle:Hello:fancy`` соответствует действию ``fancy`` контроллера ``Hello``::

    // src/Application/HelloBundle/Controller/HelloController.php

    class HelloController extends Controller
    {
        public function fancyAction($name, $color)
        {
            // create some object, based on the $color variable
            $object = ...;

            return $this->render('HelloBundle:Hello:fancy', array('name' => $name, 'object' => $object));
        }

        // ...
    }

Но где объявлен элемент массива ``$view['actions']``? Подобно ``$view['slots']``, он называется хелпером шаблона, и следующая секция расскажет вам о них больше.

.. index::
   single: Шаблон; Хелперы

Хелперы Шаблонов
----------------

Система шаблонов Symfony может быстро и просто быть расширена через хелперы. Хелперы это PHP объекты, которые предоставляют полезные особенности в контексте шаблона. Действия ``actions`` и слоты ``slots`` - это два встроенных в Symfony хелпера.

Связи между Страницами
~~~~~~~~~~~~~~~~~~~~~~

Говоря о веб приложениях, создание ссылок меджу страницами - это необходимость. Вместо того, чтобы жестко прописывать URL в шаблоне, хелпер маршрутов ``router`` знает как генерировать URL-ы базируясь на конфигурации маршрутов. Таким образом, все ваши URL-ы могут бытьлегко обновлены через изменение конфигурации:

.. code-block:: html+php

    <a href="<?php echo $view['router']->generate('hello', array('name' => 'Thomas')) ?>">
        Greet Thomas!
    </a>

Метод ``generate()`` принимает имя маршрута и массив аргументов. Имя маршрута это основной ключ, по которому маршруты упоминаются и аргументы это значения меток-заполнителей описания маршрута	:

.. code-block:: yaml

    # src/Application/HelloBundle/Resources/config/routing.yml
    hello: # The route name
        pattern:  /hello/:name
        defaults: { _bundle: HelloBundle, _controller: Hello, _action: index }

Использование Ассетов: изображений, JavaScript, и таблиц стилей
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Каким бы был интернет без изображений, скриптов и стилей?
Symfony предлагает три хелпера для упрощения работы с ними: ``assets``,
``javascripts``, и ``stylesheets``:

.. code-block:: html+php

    <link href="<?php echo $view['assets']->getUrl('css/blog.css') ?>" rel="stylesheet" type="text/css" />

    <img src="<?php echo $view['assets']->getUrl('images/logo.png') ?>" />

Главное назначение хелпера ``assets`` - сделать ваше приложение более переносимым.
Благодаря этому хелперу, вы можете переносить корень приложения куда вам угодно в рамках корневой директории web сервера без изменений в коде ваших шаблонов.

Таким же образом вы можете управлять стилями и яваскриптами при помощи хелперов ``stylesheets`` и ``JavaScripts``:

.. code-block:: html+php

    <?php $view['javascripts']->add('js/product.js') ?>
    <?php $view['stylesheets']->add('css/product.css') ?>

Метод ``add()`` определяет зависимости. Для вывода ассетов, вам также необходимо добавить следующий код в ваш основной шаблон:

.. code-block:: html+php

    <?php echo $view['javascripts'] ?>
    <?php echo $view['stylesheets'] ?>

Заключительное Слово
--------------------

Система шаблонов Symfony простая, но очень эффективная. Благодаря layout'ам,
slot'ам, шаблонам и включению действий, очень легко можно организовать ваши шаблоны логично и гибко.

Вы работаете с Symfony всего приблизительно 20 минут, и вы уже можете проделывать с ней удивительные вещи. Это сила Symfony. Изучить основы легко, но скоро вы поймете, что эта простота скрывает очень гибкую архитектуру.

Но не будем забегать вперед. Для начала вам нужно изучить немного больше о контроллере и это будет темой следующей части данного руководства. Готовы выделить еще 10 минут для Symfony?


.. _Twig: http://www.twig-project.org/
