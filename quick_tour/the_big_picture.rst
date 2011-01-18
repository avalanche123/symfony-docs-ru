Общая картина
=============

Итак вы хотите попробовать Symfony2, но в наличии у вас не более 10 минут?
Первая часть этого учебника написана для вас. Она объяснит как быстро начать
с Symfony2, показав структуру простого готового проекта.

Если вы когда-нибудь использовали какой-либо веб-фреймворк прежде, вы будете
чувствовать себя в Symfony2 как дома.

.. index::
   pair: Sandbox; Download

Загрузка и установка
--------------------

В первую очередь, убедитесь что у вас установлен как минимум PHP 5.3.2 и он
настроен для работы с web сервером, таким как Apache.

Готовы? Давайте начнем с загрузки Symfony2. Для быстрого старта мы будем
использовать "Symfony2 песочницу". Это Symfony2, который содержит все
необходимые библиотеки и несколько простых контроллеров; также в неё включена
базовая конфигурация. Наибольшее преимущество песочницы перед другими типами
инсталляции в том, что вы можете сразу же начать экспериментировать с Symfony2.

Загрузите песочницу (`sandbox`_), и распакуйте её в корневую директорию web
сервера. Сейчас у вас должна быть папка ``sandbox/``::

    www/ <- ваша корневая web директория
        sandbox/ <- распакованный архив
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
   single: Installation; Check

Проверка конфигурации
---------------------

Для того чтобы избежать головной боли в будущем, проверьте, сможет ли ваша
система запустить Symfony2 без проблем – для этого откройте следующий URL:

    http://localhost/sandbox/web/check.php

Внимательно прочитайте вывод скрипта и исправьте все проблемы, которые он найдет.

Теперь запросите вашу первую "реальную" страничку на Symfony2:

    http://localhost/sandbox/web/app_dev.php/

Symfony2 должен поблагодарить вас за приложенные усилия!

Создаём первое приложение
----------------------

Песочница поставляется с простым Hello World ":term:`application`".
Мы будем использовать его чтобы узнать больше о Symfony2. Проследуйте по этому
URL чтобы Symfony2 вас поприветствовал (замените Fabien на своё имя):

    http://localhost/sandbox/web/app_dev.php/hello/Fabien

Что происходит в этом месте? Давайте разберём URL:

.. index:: Front Controller

* ``app_dev.php``: Это "front controller". Уникальная точка входа для приложения,
  которая отвечает на все запросы пользователя;

* ``/hello/Fabien``: Это "виртуальный" путь ресурса, к которому пользователь
  хочет получить доступ.

От вас как от разработчика требуется написать код, который сопоставит
пользовательский запрос (``/hello/Fabien``) и ассоциированный с ним ресурс
(``HelloFabien!``).

.. index::
   single: Configuration

Конфигурация
~~~~~~~~~~~~

Но как Symfony2 связывает запрос с вашим кодом? Просто прочитав некоторый файл
конфигурации.

Все конфигурационные файлы Symfony2 могут быть написаны на PHP, XML, или `YAML`_
(YAML это простой формат, который очень упрощает описание конфигурационных настроек).

.. tip::

    Песочница настроена на YAML, но вы легко сможете переключиться на XML или PHP
    изменив файл ``app/AppKernel.php``. You can switch now by looking at
    the bottom of this file for instructions (the tutorials show the
    configuration for all supported formats).

.. index::
   single: Routing
   pair: Configuration; Routing

Маршрутизация
~~~~~~~~~~~~~

Symfony2 проводит маршрутизацию запроса, анализируя файл конфигурации маршрутов:

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
        $collection->add('homepage', new Route('/', array(
            '_controller' => 'FrameworkBundle:Default:index',
        )));
        $collection->addCollection($loader->import("HelloBundle/Resources/config/routing.php"));

        return $collection;
Первые несколько строк файла настроек маршрутизации определяют, какой код вызвать
когда пользователь запросит ресурс "``/``". Более интересна концовка, которая
внедряет следующий файл настроек маршрутизации, который состоит из:

.. configuration-block::

    .. code-block:: yaml

        # src/Application/HelloBundle/Resources/config/routing.yml
        hello:
            pattern:  /hello/{name}
            defaults: { _controller: HelloBundle:Hello:index }

    .. code-block:: xml

        <!-- src/Application/HelloBundle/Resources/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://www.symfony-project.org/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.symfony-project.org/schema/routing http://www.symfony-project.org/schema/routing/routing-1.0.xsd">

            <route id="hello" pattern="/hello/{name}">
                <default key="_controller">HelloBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        // src/Application/HelloBundle/Resources/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('hello', new Route('/hello/{name}', array(
            '_controller' => 'HelloBundle:Hello:index',
        )));

        return $collection;

Ну, вот! Теперь вы видите, паттерн ресурса "``/hello/:name``" (строка,
начинающаяся с двоеточия ``:name``, называется заполнитель) сопоставляется с
контроллером, заданным через значение ``_controller``.

.. index::
   single: Controller
   single: MVC; Controller

Контроллеры
~~~~~~~~~~~

Контроллер отвественен за возвращение представления ресурса (зачастую это HTML)
и определён как PHP класс:

.. code-block:: php
   :linenos:

    // src/Application/HelloBundle/Controller/HelloController.php

    namespace Application\HelloBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class HelloController extends Controller
    {
        public function indexAction($name)
        {
            return $this->render('HelloBundle:Hello:index.twig', array('name' => $name));

            // render a PHP template instead
            // return $this->render('HelloBundle:Hello:index.php', array('name' => $name));
        }
    }

Код довольно простой, но давайте разберём его по строкам:

* *строка 3*: Symfony2 использует преимущства новых возможностей PHP 5.3 и
  потому все контроллеры находятся в пространствах имён (пространство имён это
  первая часть значения переменной для маршрутизации ``_controller``: ``HelloBundle``).

* *строка 7*: Имя контроллера состоит из второй части значения переменной для
  маршрутизации ``_controller`` (``Hello``) и слова ``Controller``. Он
  расширяет встроенный класс ``Controller``, который обеспечивает полезные
  сокращения (их мы увидим позже в этом руководстве).

* *строка 9*: Каждый контроллер состоит из нескольких действий. Согласно
  конфигурации, страница hello обрабатывается действием ``index`` (третья часть
  значения переменной для маршрутизации ``_controller``). Этот метод получает
  через аргументы значения заполнителя для данного ресурса (``$name`` в нашем случае).

* *строка 11*: Метод ``render()`` загружает и заполняет шаблон
  (``HelloBundle:Hello:index.twig``) переменными, переданными вторым аргументом.

Но что такое :term:`bundle`? Весь код, написанный в Symfony2 упорядочен
через бандлы. На языке Symfony2 бандл это структурированный набор файлов
(файлы PHP, таблицы стилей, JavaScripts, изображения, ...), который
реализует одну функцию (блог, форум, ...) и который с лёгкостью может быть
распространён среди других разработчиков. В нашем примере только один бандл ``HelloBundle``.

Шаблоны
~~~~~~~

Итак, контроллер заполняет шаблон ``HelloBundle:Hello:index.twig``. Но что оно
значит? ``HelloBundle`` это имя бандла, ``Hello`` это контроллер, а ``index.twig``
это имя шаблона. Изначально песочница использует движок шаблонов Twig:

.. code-block:: jinja

    {# src/Application/HelloBundle/Resources/views/Hello/index.twig #}
    {% extends "HelloBundle::layout.twig" %}

    {% block content %}
        Hello {{ name }}!
    {% endblock %}

Поздравляем! Вы увидели первый кусочек кода для Symfony2. Это не было трудно,
не так ли? Symfony2 позволяет внедрять сайты удобно и быстро.

.. index::
   single: Environment
   single: Configuration; Environment

Работаем с Окружениями (Environments)
-------------------------

Теперь, когда вы лучше разбираетесь в работе Symfony2, давайте взглянем на
нижнюю часть страницы; вы увидите небольшую полоску со значками Symfony2 и PHP.
Она называется "Web панель отладки" ("Web Debug Toolbar") - лучший друг
разработчика. Конечно, такой инструмент не должен отображаться, когда вы начнёте
разворачивать приложение на производственном сервере. Для этого обратите
внимание на другой контроллер в папке ``web/`` (``app.php``), оптимизированный
для производственного окружения:

    http://localhost/sandbox/web/app.php/hello/Fabien

Если вы используете Apache с включённым ``mod_rewrite``, вы можете отказаться от
использования ``app.php`` части в URL:

    http://localhost/sandbox/web/hello/Fabien

И это ещё не всё, на производственном сервере, вам следует сделать корневой web
директорией папку ``web/`` чтобы обезопасить установку и получить даже более
красивый URL:

    http://localhost/hello/Fabien

Чтобы сделать производственное окружение быстрым насколько это возможно,
Symfony2 делает кэш в папке ``app/cache/``. Когда вы вносите изменения в код или
конфигурацию, вам необходимо вручную удалить кэш файлы. Вот почему лучше
использовать фронт контроллер для разработки (``app_dev.php``) когда работаете
над проектом.

Заключительное слово
--------------

10 минут истекли. Теперь вы должны быть способны создать свои простые маршруты,
контроллеры и шаблоны. Для закрепления материала, попробуйте создать что-либо
более полезное чем приложение Hello! Но если вы стремитесь узнать больше о
Symfony2, прочтите следующую часть руководства прямо сейчас, в котором мы глубже
затронем систему шаблонов.

.. _sandbox: http://symfony-reloaded.org/code#sandbox
.. _YAML:    http://www.yaml.org/
