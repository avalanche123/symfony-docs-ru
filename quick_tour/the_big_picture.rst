Общая картина
=============

Вы хотите попробовать Symfony2 но на это у вас есть всего около 10 минут? Эта первая часть учебника была написана именно для вас. В ней объясняется, как начать работу с Symfony2 и показывает структуру простого проекта.

Если вы когда-нибудь использовали какой-либо веб-фреймворк прежде, вы будете чувствовать себя в Symfony2 как дома.

.. index::
   pair: Песочница; Загрузка

Загрузка и Установка
--------------------

В первую очередь, убедитесь что у вас установлен как минимум PHP 5.3.2 и корректно настроен для работы с web сервером, таким как Apache.

Готовы? Давайте начнем с загрузки Symfony. Для быстрого старта мы будем использовать "песочницу Symfony". Это Symfony, который содержит все необходимые библиотеки и несколько простых контроллеров; также включена базовая конфигурация. Наибольшее преимущество песочницы перед другими типами инсталляции в том, что вы можете сразу же начать экспериментировать с Symfony.

Загрузите `sandbox`_, и распакуйте ее в корневую директорию web сервера. Сейчас у вас должна быть директория ``sandbox/``::

    www/ <- ваша корневая web директория
        sandbox/ <- распакованый архив
            app/
                cache/
                config/
                logs/
            src/
                Application/
                    HelloBundle/
                        Controller/
                        Resources/
                vendor/
                    symfony/
            web/

.. index::
   single: Инсталляция; Проверка

Проверка Конфигурации
---------------------

Для того чтобы избежать головной боли в последствии, проверьте, может ли быть запущен Symfony проект у вас – для этого откройте следующий URL:

    http://localhost/sandbox/web/check.php

Внимательно прочитайте вывод скрипта и исправьте все проблемы которые он найдет.

Теперь запросите вашу первую "реальную" страничку на Symfony:

    http://localhost/sandbox/web/index_dev.php/

Symfony должен поблагодарить за ваши затраченные усилия!

Ваше Первое Приложение
----------------------

Песочница содержит простое ":term:`приложение`" Hello world и мы будем использовать его для того чтобы узнать побольше о Symfony. Откройте следующий URL для того чтобы Symfony мог поприветствовать вас (замените Fabien на ваше имя):

    http://localhost/sandbox/web/index_dev.php/hello/Fabien

Что происходит в этом месте? Давайте разберем URL:

.. index:: Front Контроллер

* ``index_dev.php``: Это "front контроллер". Это единая точка входа для приложения – она обрабатывает все запросы пользователя;

* ``/hello/Fabien``: это виртуальный путь к ресурсу, который хочет получить пользователь.

Ваша обязанность как разработчика - написать код, который отражает запрос (``/hello/Fabien``) с соответствующим ресурсом (``Hello
Fabien!``).

.. index::
   single: Конфигурация

Конфигурация
~~~~~~~~~~~~

Но как Symfony связывает запрос с вашим кодом? Просто считывая некоторый конфигурационный файл.

Все конфигурационные файлы Symfony2 могут быть написаны на PHP, XML, или `YAML`_
(YAML это простой формат, который очень упрощает описание конфигурационных настроек).

.. tip::
   По умолчанию в песочнице выбран формат YAML, но вы можете легко переключить его на XML или PHP отредактировав файл ``app/AppKernel.php``. Вы можете переключить формат следуя инструкциям внизу файла ``app/AppKernel.php`` (в руководстве конфигурация показана во всех поддерживаемых форматах).

.. index::
   single: Маршрутизация
   pair: Конфигурация; Маршрутизация

Маршрутизация
~~~~~~~~~~~~~

Symfony проводит маршрутизацию запроса анализируя файл конфигурации маршрутов:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        homepage:
            pattern:  /
            defaults: { _controller: FrameworkBundle:Default:index }

        hello:
            resource: HelloBundle/Resources/config/routing.yml

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://www.symfony-project.org/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.symfony-project.org/schema/routing http://www.symfony-project.org/schema/routing/routing-1.0.xsd">

            <route id="homepage" pattern="/">
                <default key="_controller">FrameworkBundle:Default:index</default>
            </route>

            <import resource="HelloBundle/Resources/config/routing.xml" />
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->addRoute('homepage', new Route('/', array(
            '_controller' => 'FrameworkBundle:Default:index',
        )));
        $collection->addCollection($loader->import("HelloBundle/Resources/config/routing.php"));

        return $collection;

Первые несколько линий файла конфигурации маршрутов определяют, какой код будет вызван, когда пользователь запросит ресурс "``/``". Более интересно выглядит последняя часть, которая импортирует другой конфигурационный файл, который выглядит следующим образом:

.. configuration-block::

    .. code-block:: yaml

        # src/Application/HelloBundle/Resources/config/routing.yml
        hello:
            pattern:  /hello/:name
            defaults: { _controller: HelloBundle:Hello:index }

    .. code-block:: xml

        <!-- src/Application/HelloBundle/Resources/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://www.symfony-project.org/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.symfony-project.org/schema/routing http://www.symfony-project.org/schema/routing/routing-1.0.xsd">

            <route id="hello" pattern="/hello/:name">
                <default key="_controller">HelloBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        // src/Application/HelloBundle/Resources/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->addRoute('hello', new Route('/hello/:name', array(
            '_controller' => 'HelloBundle:Hello:index',
        )));

        return $collection;

Here we go! Как вы можете видеть, шаблон "``/hello/:name``" (строка которая начинается с двоеточия как ``:name`` это метка, плэйсхолдер) отображается на контроллер, который определен через значение ``_controller``.

.. index::
   single: Контроллер
   single: MVC; Контроллер

Контроллеры
~~~~~~~~~~~

Контроллер ответственный за возврат представления ресурса (как правило HTML) и определен как PHP класс:

.. code-block:: php
   :linenos:

    // src/Application/HelloBundle/Controller/HelloController.php

    namespace Application\HelloBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class HelloController extends Controller
    {
        public function indexAction($name)
        {
            return $this->render('HelloBundle:Hello:index.php', array('name' => $name));
        }
    }

Код довольно простой, но давайте разберем его по строкам:

* *строка 3*: Symfony использует все преимущества PHP 5.3, все контроллеры находятся в пространствах имен (пространство имен - это первая часть значения переменной маршрутизации ``_controller``: ``HelloBundle``).

* *строка 7*: Имя контроллера - это конкатенация второй части значения ``_controller`` переменной маршрутизации (``Hello``) и слова ``Controller``. Он расширяет встроенный класс ``Controller``, который предоставляет полезные сокращения (как мы далее убедимся в этом руководстве).

* *строка 9*: Each controller is made of several actions. As per the
  configuration, the hello page is handled by the ``index`` action (the third
  part of the ``_controller`` routing value). This method receives the
  resource placeholder values as arguments (``$name`` in our case).

* *строка  11*: Метод ``render()`` загружает и интерпретирует шаблон (``HelloBundle:Hello:index``) с переменными, передаваемыми в качестве второго аргумента.

Но что такое :term:`бандл`? Весь код, который вы пишите в Symfony проекте организован в банлах. В понимании Symfony, бандл - это структурированный набор файлов (PHP файлы, таблицы стилей, JavaScript-ы, изображения, ...) которые реализуют отдельную функциональность (блог, форум, ...) которой легко можно поделиться с другими разработчиками. В нашем примере, у нас есть один бандл, ``HelloBundle``.

Шаблоны
~~~~~~~

Итак, контроллер отображает шаблон ``HelloBundle:Hello:index.php``. Но что скрыто в имени шаблона? ``HelloBundle`` это имя бандла, ``Hello`` это контроллер, и ``index.php`` имя файла шаблона. Шаблон по сути состоит из HTML или простых PHP выражений:

.. code-block:: html+php

    # src/Application/HelloBundle/Resources/views/Hello/index.php
    <?php $view->extend('HelloBundle::layout.php') ?>

    Hello <?php echo $name ?>!

Поздравляем! Вы разобрались в первом кусочке Symfony кода. Не так уж и сложно, правда? Symfony делает web-разработку более быстрой и приятной.

.. index::
   single: Окружение
   single: Конфигурация; Окружение

Окружения
---------

Now that you have a better understanding on how Symfony works, have a closer
look at the bottom of the page; you will notice a small bar with the Symfony
and PHP logos. It is called the "Web Debug Toolbar" and it is the developer's
best friend. Of course, such a tool must not be displayed when you deploy your
application to your production servers. That's why you will find another front
controller in the ``web/`` directory (``index.php``), optimized for the production
environment:

    http://localhost/sandbox/web/index.php/hello/Fabien

And if you have ``mod_rewrite`` installed, you can even omit the ``index.php``
part of the URL:

    http://localhost/sandbox/web/hello/Fabien

Last but not least, on the production servers, you should point your web root
directory to the ``web/`` directory to secure your installation and have an even
better looking URL:

    http://localhost/hello/Fabien

To make the production environment as fast as possible, Symfony maintains a
cache under the ``app/cache/`` directory. When you make changes, you need to
manually remove the cached files. That's why you should always use the
development front controller (``index_dev.php``) when working on a project.

Final Thoughts
--------------

The 10 minutes are over. By now, you should be able to create your own simple
routes, controllers, and templates. As an exercise, try to build something
more useful than the Hello application! But if you are eager to learn more
about Symfony, you can read the next part of this tutorial right away, where
we dive more into the templating system.

.. _sandbox: http://symfony-reloaded.org/code#sandbox
.. _YAML:    http://www.yaml.org/
