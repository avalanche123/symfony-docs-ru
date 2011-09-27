.. index::
   single: Создание страниц

Создание страниц в Symfony2
==========================

Создание новой страницы в Symfony2 это простой процесс, состоящий из 2 шагов:

* *Создание маршрута*: Маршрут определяет URL (например ``/about``) для вашей
  страницы, а также контроллер (PHP функция), который Symfony2 должен выполнить,
  когда URL входящего запроса совпадет шаблоном маршрута;

* *Создание контроллера*: Контроллер – это PHP функция, которая принимает входящий
  запрос и преобразует его в объект ``Response``, который будет возвращен
  пользователю.

Нам нравится такой подход, потому что он соответствует тому, как работает Web.
Каждое взаимодействие в Web инициализируется HTTP запросом. Забота вашего
приложения – интерпретировать запрос и вернуть соответствующий ответ.

Symfony2 следует этой философии и предоставляет вам инструменты и соглашения,
для того чтобы ваше приложение оставалось хорошо структурированным при росте
его посещаемости и сложности.

Звучит просто? Давайте копнём по глубже!

.. index::
   single: Создание страницы; Пример

Страница "Hello Symfony!"
-------------------------

Давайте начнем с классического приложения "Hello World!". Когда вы закончите работу
над ним, пользователь приложения будет иметь возможность получить персональное
приветствие, (например "Hello Symfony"), перейдя по следующему URL:

.. code-block:: text

    http://localhost/app_dev.php/hello/Symfony

Вы также сможете заменить ``Symfony`` на другое имя и получить новое приветствие.
Для создания этой страницы мы пройдем простой путь из двух шагов.

.. note::

    Данное руководство подразумевает, что вы уже скачали Symfony2 и настроили ваш
    веб-сервер. URL, указанный выше, подразумевает, что localhost указывает на
    web-директорию вашего нового Symfony2 проекта. Если же вы ещё не выполнили
    этих шагов, рекомендуется их выполнить, прежде чем вы продолжите чтение.
    Дополнительную информацию вы можете найти в главе
    :doc:`Установка и настройка Symfony2</book/installation>`.

Прежде чем начать: создание Пакета (bundle)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Прежде чем начать, вам необходимо создать пакет (:term:`bundle`). В Symfony2 пакет
напоминает plugin, за исключением того, что весь код вашего приложения будет
расположен внутри такого пакета.

Вообще говоря, пакет – это не более чем директория, которая содержит все что
относится к какой-то специфической функции, включая PHP-классы, настройки и
даже стили и файлы Javascript (см. :ref:`page-creation-bundles`).

Для создания пакета с именем ``AcmeHelloBundle`` (демо-пакет, который
вы создадите в ходе прочтения данной статьи), необходимо выполнить
следующую команду и следовать инструкциям, которые появятся на
экране (установите все опции по-умолчанию):

.. code-block:: bash

    php app/console generate:bundle --namespace=Acme/HelloBundle --format=yml

За кулисами же произойдёт вот что: будет создана директория для пакета
``src/Acme/HelloBundle``. Также в файл ``app/AppKernel.php`` автоматически
будет добавлена строка, которая зарегистрирует вновь созданный пакет:

.. code-block:: php

    <?php

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Acme\HelloBundle\AcmeHelloBundle(),
        );
        // ...

        return $bundles;
    }

Теперь, когда вы создали и инициализировали пакет, вы можете приступить к созданию
вашего приложения.

Шаг 1: Создание маршрута
~~~~~~~~~~~~~~~~~~~~~~~~

По умолчанию, конфигурационный файл маршрутизатора в приложении Symfony2,
располагается в ``app/config/routing.yml``. Для конфигурирования маршрутизатора,
а также любых прочих конфигураций Symfony2, вы можете также использовать
XML или PHP формат.

Если вы посмотрите в основной конфигурационный файл, вы увидите, что Symfony
уже добавил запись для сгенерированного ``AcmeHelloBundle``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        AcmeHelloBundle:
            resource: "@AcmeHelloBundle/Resources/config/routing.yml"
            prefix:   /

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <import resource="@AcmeHelloBundle/Resources/config/routing.xml" prefix="/" />
        </routes>

    .. code-block:: php

        <?php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->addCollection(
            $loader->import('@AcmeHelloBundle/Resources/config/routing.php'),
            '/',
        );

        return $collection;

Эта запись очень проста: она сообщает Symfony, что необходимо загрузить конфигурацию
маршрутизатора из файла ``Resources/config/routing.yml``, который расположен в
пакете ``AcmeHelloBundle``. Это означает, что вы можете размешать конфигурацию
маршрутизатора непосредственно в ``app/config/routing.yml`` или же хранить
маршруты внутри пакета и импортировать их оттуда.

Теперь, когда файл ``routing.yml`` импортирован из пакета, добавьте новый маршрут,
который определит URL страницы, которую вы собираетесь создать:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/routing.yml
        hello:
            pattern:  /hello/{name}
            defaults: { _controller: AcmeHelloBundle:Hello:index }

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="hello" pattern="/hello/{name}">
                <default key="_controller">AcmeHelloBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        <?php

        // src/Acme/HelloBundle/Resources/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('hello', new Route('/hello/{name}', array(
            '_controller' => 'AcmeHelloBundle:Hello:index',
        )));

        return $collection;

Маршрут состоит из двух основных частей: шаблона (``pattern``), с которым
сравнивается URL, а также массива параметров по умолчанию (``defaults``), в котором
указывается контроллер, который необходимо выполнить. Заполнитель ``{name}``
в шаблоне – это метасимвол (wildcard). Он означает, что URL ``/hello/Ryan``,
``/hello/Fabien``, а также прочие, похожие на них, будут соответствовать этому
же маршруту. Параметр, определённый заполнителем ``{name}``, также будет передан
в контроллер, так что вы сможете использовать его, чтобы поприветствовать
пользователя.

.. note::

  Система маршрутизации имеет еще множество замечательных функций для создания
  гибких и функциональных структур URL в приложении. За дополнительной информацией
  вы можете обратиться к главе :doc:`Маршрутизация</book/routing>`.

Шаг 2: Создание Контроллера
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Когда URI вида ``/hello/Ryan`` обнаруживается приложением в запросе, маршрут
``hello`` совпадёт с ним и будет вызван контроллер ``AcmeHelloBundle:Hello:index``.
Следующим вашим шагом будет создание этого контроллера.

Контроллер ``AcmeHelloBundle:Hello:index`` - это *логическое* имя контроллера
и оно соответствует методу ``indexAction`` PHP-класса, именуемого
``Acme\HelloBundle\Controller\Hello``. Приступим к созданию этого файла внутри
``AcmeHelloBundle``:

.. code-block:: php

    <?php

    // src/Acme/HelloBundle/Controller/HelloController.php
    namespace Acme\HelloBundle\Controller;

    use Symfony\Component\HttpFoundation\Response;

    class HelloController
    {
    }

В действительности, контроллер – это не что иное, как метод PHP класса, который вы
создаёте, а Symfony выполняет. Это то место, где ваш код, используя информацию из
запроса, создает запрошенный ресурс. За исключением некоторых особых случаев,
результатом работы контроллера всегда является объект Symfony2 ``Response``.

Создайте метод ``indexAction``, который Symfony выполнит, когда сработает
маршрут ``hello``:

.. code-block:: php

    <?php

    // src/Acme/HelloBundle/Controller/HelloController.php

    // ...
    class HelloController
    {
        public function indexAction($name)
        {
            return new Response('<html><body>Hello '.$name.'!</body></html>');
        }
    }

Этот контроллер предельно прост: он создает новый объект ``Response``, чьим
первым аргументом является контент, который будет использован для создания
ответа (в нашем случае это маленькая HTML-страница, код которой мы указали прямо
в контроллере).

Примите наши поздравления! После создания всего лишь маршрута и контроллера,
вы уже имеете полноценную страницу! Если вы все настроили корректно, ваше
приложение должно поприветствовать вас:

.. code-block:: text

    http://localhost/app_dev.php/hello/Ryan

.. tip::

    Вы также можете отобразить ваше приложение в :ref:`"продуктовом (prod)" окружении<environments-summary>`,
    посетив следующий URL:

    .. code-block:: text

        http://localhost/app.php/hello/Ryan

    Если вы увидите ошибку, то скорее всего вам всего лишь необходимо
    очистить кэш, выполнив команду:

    .. code-block:: bash

        php app/console cache:clear --env=prod --no-debug

Не обязательным (но, как правило, востребованным) третьим шагом является создание шаблона.

.. note::

   Контроллер – это главная точка входа для вашего кода и ключевой ингредиент
   при создании страниц. Больше информации о контроллерах вы можете найти
   в главе :doc:`Контроллер</book/controller>`.

Необязательный шаг 3: Создание шаблона
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Шаблоны позволяют нам вынести разметку страниц (HTML код, как правило) в
отдельный файл и повторно использовать различные части шаблона страницы.
Вместо того чтобы писать код внутри контроллера, воспользуемся шаблоном:

.. code-block:: php
    :linenos:

    <?php

    // src/Acme/HelloBundle/Controller/HelloController.php
    namespace Acme\HelloBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class HelloController extends Controller
    {
        public function indexAction($name)
        {
            return $this->render('AcmeHelloBundle:Hello:index.html.twig', array('name' => $name));

            // render a PHP template instead
            // return $this->render('AcmeHelloBundle:Hello:index.html.php', array('name' => $name));
        }
    }

.. note::

   Для того, чтобы использовать метод ``render()``, необходимо отнаследоваться от
   класса ``Symfony\Bundle\FrameworkBundle\Controller\Controller``
   (API docs: :class:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller`),
   который добавляет несколько методов для быстрого вызова часто употребляемых
   функций контроллера. В предыдущем примере это достигается путём добавления выражения
   ``use`` в строке 6 и, затем, наследованием от класса ``Controller`` в строке 8.

Метод ``render()`` создает объект ``Response``, заполненный результатом обработки (рендеринга)
шаблона. Как и в любом другом контроллере, вы, в конце концов, вернете объект ``Response``.

Обратите внимание, что есть две различные возможности рендеринга шаблонов.
Symfony2 по умолчанию, поддерживает 2 языка шаблонов: классические PHP-шаблоны и
простой, но мощный язык шаблонов `Twig`_. Но не пугайтесь, вы свободны в выборе
того или иного из них, кроме того вы можете использовать оба в рамках одного проекта.

Контроллер отображает шаблон ``AcmeHelloBundle:Hello:index.html.twig``, который
назван с использованием следующих соглашений:

    **BundleName**:**ControllerName**:**TemplateName**

Это, так называемое, логическое имя шаблона, которое соответствует физическому файлу
на основании следующих соглашений:

    **/путь/к/BundleName**/Resources/views/**ControllerName**/**TemplateName**

В нашем случае ``AcmeHelloBundle`` - это наименование пакета, ``Hello`` - это
контроллер и ``index.html.twig`` - это шаблон:

.. configuration-block::

    .. code-block:: jinja
       :linenos:

        {# src/Acme/HelloBundle/Resources/views/Hello/index.html.twig #}
        {% extends '::base.html.twig' %}

        {% block body %}
            Hello {{ name }}!
        {% endblock %}

    .. code-block:: php

        <!-- src/Acme/HelloBundle/Resources/views/Hello/index.html.php -->
        <?php $view->extend('::base.html.php') ?>

        Hello <?php echo $view->escape($name) ?>!

Давайте рассмотрим подробнее шаблон Twig:

* *строка 2*: Токен ``extends`` определяет родительский шаблон. Таким образом, сам шаблон
  однозначным образом определяет родителя (layout) внутрь которого он будет помещен.

* *строка 4*: Токен ``block`` означает, что всё внутри него будет помещено в блок с
  именем ``body``. Как вы увидите ниже, это уже обязанность родительского шаблона
  (``base.html.twig``) – полностью отобразить блок ``body``.

Родительский шаблон, ``::base.html.twig``, не включает в себя ни **имени пакета**,
ни **имени контроллера** (отсюда и двойное двоеточие в начале имени (::)). Это означает,
что шаблон располагается вне пакета в директории ``app``:

.. configuration-block::

    .. code-block:: html+jinja

        {# app/Resources/views/base.html.twig #}
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title>{% block title %}Welcome!{% endblock %}</title>
                {% block stylesheets %}{% endblock %}
                <link rel="shortcut icon" href="{{ asset('favicon.ico') }}" />
            </head>
            <body>
                {% block body %}{% endblock %}
                {% block javascripts %}{% endblock %}
            </body>
        </html>

    .. code-block:: php

        <!-- app/Resources/views/base.html.php -->
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title><?php $view['slots']->output('title', 'Welcome!') ?></title>
                <?php $view['slots']->output('stylesheets') ?>
                <link rel="shortcut icon" href="<?php echo $view['assets']->getUrl('favicon.ico') ?>" />
            </head>
            <body>
                <?php $view['slots']->output('_content') ?>
                <?php $view['slots']->output('stylesheets') ?>
            </body>
        </html>

Базовый шаблон определяет HTML-разметку и отображает блок ``body``, который
вы определили в шаблоне ``index.html.twig``. Так как вы не определили блок
``title`` в дочернем шаблоне, он останется со значением по умолчанию - "Welcome!".

Шаблоны являются мощным инструментом по организации и отображению контента ваших страниц.
Шаблон может отобразить всё что угодно от HTML разметки, до CSS-кода или что контроллеру
будет угодно.

В жизненном цикле обработки запроса, шаблонизатор - это всего лишь опциональный
инструмент. Не забывайте, что цель каждого контроллера - вернуть объект ``Response``.
Шаблоны являются мощным инструментом, но они опциональны, всего лишь инструмент для
создания контента для объекта ``Response``.

.. index::
   single: Структура директорий

Структура директорий
-----------------------

Всего лишь после прочтения нескольких коротких секций вы уже уяснили философию
создания и отображения страниц в Symfony2. Поэтому без лишних слов мы приступим к
изучению того, как организованы и структурированы проекты Symfony2. К концу этой
секции вы будете знать, где найти и куда поместить различные типы файлов.
И более того, будет понимать – почему!

По умолчанию каждое Symfony приложение (:term:`application`), изначально созданное быть
очень гибким, имеет одну и ту же базовую (и рекомендуемую) структуру директорий:

* ``app/``: Эта директория содержит настройки приложения;

* ``src/``: Весь PHP код проекта находится в этой директории;

* ``vendor/``: Здесь размещаются сторонние библиотеки;

* ``web/``: Это корневая директория, видимая web-серверу и содержащая доступные пользователям файлы;

Директория Web
~~~~~~~~~~~~~~~~~

Web-директория – это дом для всех публично-доступных статических файлов,
таких как изображения, таблицы стилей и JavaScript файлы.
Тут также располагаются все фронт-контроллеры (:term:`front controller`):

.. code-block:: php

    <?php

    // web/app.php
    require_once __DIR__.'/../app/bootstrap.php.cache';
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->loadClassCache();
    $kernel->handle(Request::createFromGlobals())->send();

Файл фронт-контроллера (в примере выше – ``app.php``)- это PHP файл, который
выполняется, когда используется Symfony2 приложение и в его обязанности входит
использование Kernel-класса(``AppKernel``), для запуска приложения.

.. tip::

    Наличие фронт-контроллера означает возможность использования более гибких URL,
    отличных от тех, что используются в типичном "плоском" PHP-приложении.
    Когда используется фронт-контроллер, URL формируется следующим образом:

    .. code-block:: text

        http://localhost/app.php/hello/Ryan

    Фронт-контроллер ``app.php`` выполняется и "внутренний:" URL ``/hello/Ryan``
    направляется внутри приложения с использованием конфигурации маршрутизатора.
    С использованием правил mod_rewrite для Apache вы можете перенаправлять
    все запросы (на физически не существующие URL) на app.php, чтобы явно не
    указывать его в URL:

    .. code-block:: text

        http://localhost/hello/Ryan

Хотя фронт-контроллеры имеют важное значение при обработке каждого запроса,
вам нечасто придется модифицировать их или вообще вспоминать об их существовании.
Мы еще вкратце упомянем о них в секции, где говорится об Окружениях (`Окружения`_).

Директория приложения (``app``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Как вы уже видели во фронт-контроллере, класс ``AppKernel`` – это точка входа
приложения и он отвечает за его конфигурацию. Как таковой, этот класс расположен
в директории ``app/``.

Этот класс должен реализовывать два метода, которые определяют всё, что Symfony
необходимо знать о вашем приложении. Вам даже не нужно беспокоиться о реализации
этих методов, когда начинаете работу – они уже реализованы и содержат код по умолчанию.

* ``registerBundles()``: Возвращает массив всех пакетов, необходимых для запуска
  приложения (см. :ref:`page-creation-bundles`);

* ``registerContainerConfiguration()``: Загружает главный конфигурационный файл
  (см. секцию `Конфигурация приложения`_).

Изо дня в день вы будете использовать директорию ``app/`` в основном для того,
чтобы модифицировать конфигурацию и настройки маршрутизатора в директории
``app/config/`` (см. `Конфигурация приложения`_). Также в ``app/`` содержится кэш
(``app/cache``), директория для логов (``app/logs``) и директория для ресурсов
уровня приложения (``app/Resources``). Об этих директориях подробнее будет
рассказано в других главах.

.. _autoloading-introduction-sidebar:

.. sidebar:: Автозагрузка

    При инициализации приложения подключается особый файл: ``app/autoload.php``.
    Этот файл отвечает за автозагрузку всех файлов из директорий ``src/`` и
    ``vendor/``.

    Благодаря автозагрузке, вам больше не придется беспокоиться об использовании
    выражений ``include`` или ``require``. Вместо этого, Symfony2 использует
    пространства имен классов, чтобы определить их расположение и автоматически
    подключить файл класса, в случае если класс вам понадобится.

    Автозагрузчик уже настроен для того, чтобы искать ваши классы в директории
    ``src/``. Для того чтобы автозагрузка работала, имя класса и путь к его
    файлу должны следовать следующему шаблону:

    .. code-block:: text

        Class Name:
            Acme\HelloBundle\Controller\HelloController
        Path:
            src/Acme/HelloBundle/Controller/HelloController.php

    Как правило, единственная ситуация, когда вам необходимо беспокоиться
    о файле ``app/autoload.php`` - когда вы добавляете новую стороннюю
    библиотеку в директорию ``vendor/``. Если вы хотите узнать больше об
    автозагрузке, обратите внимание на статью
    :doc:`Как автоматически загружать классы</cookbook/tools/autoloader>`.

Директория исходных кодов проекта (``src``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Если вкратце, то директория ``src/`` содержит весь код *вашего* приложения
(PHP-код, шаблоны, конфигурационные файлы, стили и т.д.). Во время разработки,
бОльшую часть работ вы будете выполнять внутри одного или нескольких пакетов,
которые вы создадите именно в этой директории.

Но что же собственно из себя представляет сам пакет (:term:`bundle`)?

.. _page-creation-bundles:

Система пакетов
-----------------

Пакет чем-то схож с плагином, но он ещё лучше. Ключевое отличие состоит в том,
что *всё есть пакет* в Symfony2, включая функционал ядра и код вашего приложения.
Пакеты – это граждане высшего сорта в Symfony2. Они дают вам возможность
использовать уже готовые пакеты, которые вы можете найти на сайте `symfony2bundles.org`_.
Вы также можете там выкладывать свои пакеты. Они также дают возможность легко и
просто выбрать, какие именно функции подключить в вашем приложении.

.. note::

   Здесь мы рассмотрим лишь основы, более детальную информацию по пакетам вы можете
   найти в статье :doc:`пакеты</cookbook/bundles/best_practices>` в "книге рецептов".

Пакет это просто структурированный набор файлов и директорий, который реализует
одну конкретную функцию. Вы можете создать ``BlogBundle`` или ``ForumBundle`` или же
пакет для управления пользователями (такие пакеты уже есть и даже с открытым исходным
кодом). Каждая директория содержит все необходимое для реализации этой конкретной
функции, включая PHP файлы, шаблоны, стили, клиентские скрипты, тесты и все что
ещё потребуется. Каждый аспект реализации функции находится в своём пакете и каждая
функция располагается в своем собственном пакете.

Приложение состоит из пакетов, которые объявлены в методе ``registerBundles()``
класса ``AppKernel``:

.. code-block:: php

    <?php

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            new Symfony\Bundle\SecurityBundle\SecurityBundle(),
            new Symfony\Bundle\TwigBundle\TwigBundle(),
            new Symfony\Bundle\MonologBundle\MonologBundle(),
            new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            new Symfony\Bundle\DoctrineBundle\DoctrineBundle(),
            new Symfony\Bundle\AsseticBundle\AsseticBundle(),
            new Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle(),
            new JMS\SecurityExtraBundle\JMSSecurityExtraBundle(),
        );

        if (in_array($this->getEnvironment(), array('dev', 'test'))) {
            $bundles[] = new Acme\DemoBundle\AcmeDemoBundle();
            $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
            $bundles[] = new Sensio\Bundle\DistributionBundle\SensioDistributionBundle();
            $bundles[] = new Sensio\Bundle\GeneratorBundle\SensioGeneratorBundle();
        }

        return $bundles;
    }

Используя метод ``registerBundles()``, вы получаете полный контроль над теми пакетами,
которые используются вашим приложением (включая пакеты, входящие в состав ядра Symfony).

.. tip::

   Вообще говоря, пакет может располагаться где угодно, если автозагрузчик
   (``app/autoload.php``) можно настроить таким образом, чтобы этот пакет
   мог быть загружен.

Создание пакета
~~~~~~~~~~~~~~~~~

Symfony Standard Edition содержит удобную команду для создания пакета. Тем не менее,
создать пакет вручную лишь немногим больше и ничуть не сложнее.

Чтобы показать вам, как проста система пакетов, давайте создадим новый
пакет, назовём его ``AcmeTestBundle`` и активируем его.

.. tip::

    ``Acme`` это всего-лишь формальное имя, которое должно быть заменено
    на наименование некоего вендора, которое будет представлять вашу организацию
    (например ``ABCTestBundle`` для компании ``ABC``).

В первую очередь, создадим директорию ``src/Acme/TestBundle/`` и добавим
в неё файл ``AcmeTestBundle.php``:

.. code-block:: php

    <?php

    // src/Acme/TestBundle/AcmeTestBundle.php
    namespace Acme\TestBundle;

    use Symfony\Component\HttpKernel\Bundle\Bundle;

    class AcmeTestBundle extends Bundle
    {
    }

.. tip::

   Наименование класса ``AcmeTestBundle`` следует стандарту
   :ref:`Именования Пакетов<bundles-naming-conventions>`. Вы также можете сократить
   наименование пакета до ``TestBundle``, назвав класс ``TestBundle``
   (и переименовав файл в ``TestBundle.php``).

Этот пустой класс – единственное, что необходимо создать для минимальной комплектации
пакета. Не смотря на то, что класс пуст, он обладает большим потенциалом и позволяет
настраивать поведение пакета.

Теперь, когда мы создали пакет, его нужно активировать в классе ``AppKernel``:

.. code-block:: php

    <?php

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...

            // register your bundles
            new Acme\TestBundle\AcmeTestBundle(),
        );
        // ...

        return $bundles;
    }

И, хотя наш новый пакет пока ничего не делает, ``AcmeTestBundle`` готов к
использованию.

Symfony также предлагает интерфейс для командной строки для создания
базового скелетона пакета:

.. code-block:: bash

    php app/console generate:bundle --namespace=Acme/TestBundle

Каркас пакета создаёт базовый контроллер, шаблон и маршрут, которые можно
настроить впоследствии. Мы еще вернёмся к инструментам командной строки позже.

.. tip::

   Когда создаёте новый пакет, или используете сторонние пакеты, убедитесь,
   что пакет активирован в registerBundles(). При использовании же команды
   ``generate:bundle`` - это действие производится автоматически.

Структура директории пакета
~~~~~~~~~~~~~~~~~~~~~~~~~~

Структура директории пакета проста и гибка. По умолчанию, система пакетов
следует некоторым соглашениям, которые помогают поддерживать стилевое единообразие
во всех пакетах Symfony2. Давайте взглянем на пакет ``AcmeHelloBundle``, так
как он содержит наиболее основные элементы пакета:

* ``Controller/`` содержит контроллеры пакета (например ``HelloController.php``);

* ``Resources/config/`` дом для конфигурационных файлов, включая конфигурацию
  маршрутизатора (например ``routing.yml``);

* ``Resources/views/`` шаблоны, сгруппированные по имени контроллера (например
  ``Hello/index.html.twig``);

* ``Resources/public/`` публично доступные ресурсы (картинки, стили…),
  которые будут скопированы или связаны символической ссылкой с ``web/`` директорией
  при помощи консольной команды ``assets:install``;

* ``Tests/`` содержит все тесты пакета.

Пакет может быть как маленьким, так и большим – в зависимости от задачи,
которую он реализует. Он содержит лишь те файлы, которые нужны – и ничего более.

В других главах книги вы также узнаете, как работать с базой данных, как
создавать и валидировать формы, создавать файлы переводов, писать тесты и
много чего ещё. Все эти объекты в пакете имеют определенную роль и место.

Конфигурация приложения
-------------------------

Приложение состоит из набора пакетов, реализующих все необходимые функции вашего приложения.
Каждый пакет может быть настроен при помощи конфигурационных файлов, написанных на YAML,
XML или PHP. По умолчанию, основной конфигурационный файл расположен в директории
``app/config/`` и называется ``config.yml``, ``config.xml`` или ``config.php``, в
зависимости от предпочитаемого вами формата:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        imports:
            - { resource: parameters.ini }
            - { resource: security.yml }

        framework:
            secret:          %secret%
            charset:         UTF-8
            router:          { resource: "%kernel.root_dir%/config/routing.yml" }
            form:            true
            csrf_protection: true
            validation:      { enable_annotations: true }
            templating:      { engines: ['twig'] } #assets_version: SomeVersionScheme
            session:
                default_locale: %locale%
                auto_start:     true

        # Twig Configuration
        twig:
            debug:            %kernel.debug%
            strict_variables: %kernel.debug%

        # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <imports>
            <import resource="parameters.ini" />
            <import resource="security.yml" />
        </imports>

        <framework:config charset="UTF-8" secret="%secret%">
            <framework:router resource="%kernel.root_dir%/config/routing.xml" />
            <framework:form />
            <framework:csrf-protection />
            <framework:validation annotations="true" />
            <framework:templating assets-version="SomeVersionScheme">
                <framework:engine id="twig" />
            </framework:templating>
            <framework:session default-locale="%locale%" auto-start="true" />
        </framework:config>

        <!-- Twig Configuration -->
        <twig:config debug="%kernel.debug%" strict-variables="%kernel.debug%" />

        <!-- ... -->

    .. code-block:: php

        <?php

        $this->import('parameters.ini');
        $this->import('security.yml');

        $container->loadFromExtension('framework', array(
            'secret'          => '%secret%',
            'charset'         => 'UTF-8',
            'router'          => array('resource' => '%kernel.root_dir%/config/routing.php'),
            'form'            => array(),
            'csrf-protection' => array(),
            'validation'      => array('annotations' => true),
            'templating'      => array(
                'engines' => array('twig'),
                #'assets_version' => "SomeVersionScheme",
            ),
            'session' => array(
                'default_locale' => "%locale%",
                'auto_start'     => true,
            ),
        ));

        // Twig Configuration
        $container->loadFromExtension('twig', array(
            'debug'            => '%kernel.debug%',
            'strict_variables' => '%kernel.debug%',
        ));

        // ...

.. note::

   Подробнее о том, как загружать каждый файл/формат будет рассказано
   в следующей секции - `Окружения`_.

Каждый параметр верхнего уровня, например ``framework`` или ``twig``,
определяет настройки конкретного пакета. Например, ключ ``framework`` определяет
настройки ядра Symfony ``FrameworkBundle`` и включает настройки маршрутизации,
шаблонизатора и прочих ключевых систем.

Пока же нам не стоит беспокоиться о конкретных настройках в каждой секции.
Файл настроек по умолчанию содержит все необходимые параметры. По ходу чтения
прочей документации вы ознакомитесь со всеми специфическими настройками.

.. sidebar:: Форматы конфигураций

    Во всех главах книги все примеры конфигураций будут показаны во всех трех
    форматах (YAML, XML and PHP). Каждый из них имеет свои достоинства и недостатки.
    Выбор же формата целиком зависит о ваших предпочтений:

    * *YAML*: Простой, понятный и читабельный;

    * *XML*: В разы более мощный, нежели YAML, к тому же многие современные IDE поддерживают автозавершение в XML;

    * *PHP*: Очень мощный, но менее читабельный, чем стандартные форматы конфигурационных файлов.

.. index::
   single: Окружения; Введение

.. _environments-summary:

Окружения
------------

Приложение можно запускать в различных окружениях. Различные окружения используют
один и тот же PHP код (за исключением фронт-контроллера), но могут иметь совершенно
различные настройки. Например, ``dev`` окружение ведет лог ошибок и замечаний, в
то время как ``prod`` окружение логгирует только ошибки. В ``dev`` некоторые
файлы пересоздаются при каждом запросе, но кэшируются в ``prod`` окружении.
В то же время, все окружения одновременно доступны на одной и той же машине.

Проект Symfony2 по умолчанию имеет три окружения (``dev``, ``test`` и ``prod``),
хотя создать новое окружение не сложно. Вы можете смотреть ваше приложение
в различных окружениях просто меняя фронт-контроллеры в браузере. Для того
чтобы отобразить приложение в dev окружении, откройте его при помощи
фронт контроллера ``app_dev.php``:

.. code-block:: text

    http://localhost/app_dev.php/hello/Ryan

Если же вы хотите посмотреть, как поведёт себя приложение в продуктовом
окружении, вы можете вызвать фронт-контроллер ``prod``:

.. code-block:: text

    http://localhost/app.php/hello/Ryan

Так как ``prod`` окружение оптимизировано для скорости, настройки, маршруты
и шаблоны Twig компилируются в плоские PHP классы и кэшируются. Когда вы
хотите посмотреть изменения в продуктовом окружении, вам потребуется удалить
эти файлы чтобы они пересоздались автоматически::

    php app/console cache:clear --env=prod --no-debug

.. note::

   Если вы откроете файл ``web/app.php``, вы обнаружите, что он однозначно
   настроен на использование ``prod`` окружения::

       $kernel = new AppKernel('prod', false);

   Вы можете создать новый фронт-контроллер для нового окружения просто
   скопировав этот файл и изменив ``prod`` на другое значение.

.. note::

    Тестовое окружение (``test``) используется при запуске автотестов и его
    нельзя напрямую открыть через браузер. Подробнее об это можно почитать в главе
    :doc:`Тестирование</book/testing>`.

.. index::
   single: Окружения; Конфигурация

Настройка окружений
~~~~~~~~~~~~~~~~~~~~~~~~~

Класс ``AppKernel`` отвечает за загрузку конфигурационных файлов:

.. code-block:: php

    <?php

    // app/AppKernel.php
    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load(__DIR__.'/config/config_'.$this->getEnvironment().'.yml');
    }

Вы уже знаете, что расширение ``.yml`` может быть изменено на ``.xml`` или ``.php``,
если вы предпочитаете использовать XML или PHP для файлов конфигурации. Имейте
также в виду, что каждое окружение загружает свои собственные настройки.
Рассмотрим конфигурационный файл для ``dev`` окружения.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_dev.yml
        imports:
            - { resource: config.yml }

        framework:
            router:   { resource: "%kernel.root_dir%/config/routing_dev.yml" }
            profiler: { only_exceptions: false }

        # ...

    .. code-block:: xml

        <!-- app/config/config_dev.xml -->
        <imports>
            <import resource="config.xml" />
        </imports>

        <framework:config>
            <framework:router resource="%kernel.root_dir%/config/routing_dev.xml" />
            <framework:profiler only-exceptions="false" />
        </framework:config>

        <!-- ... -->

    .. code-block:: php

        <?php

        // app/config/config_dev.php
        $loader->import('config.php');

        $container->loadFromExtension('framework', array(
            'router'   => array('resource' => '%kernel.root_dir%/config/routing_dev.php'),
            'profiler' => array('only-exceptions' => false),
        ));

        // ...

Ключ ``imports`` похож по действию на выражение ``include`` в PHP и
гарантирует что главный конфигурационный файл (``config.yml``) будет загружен
в первую очередь. Остальной код корректирует конфигурацию по умолчанию для
увеличения порога логгирования и прочих настроек, специфичных для
процесса разработки.

Оба окружения – ``prod`` и ``test`` следуют той же модели: каждое окружение
импортирует базовые настройки и модифицирует их значения для своих нужд. Это
соглашение позволяет повторно использовать большую часть настроек и изменять
лишь те из них, которые требует окружение.

Заключение
-------

Поздравляем! Вы усвоили все фундаментальные аспекты Symfony2 и обнаружили,
какими лёгкими и в то же время гибкими они могут быть. И, поскольку на подходе
ещё *много интересного*, обязательно запомните следующие положения:

* создание страниц – это три простых шага, включающих **маршрут**, **контроллер**
  и (опционально) **шаблон**;

* каждое приложение должно содержать 4 основных директории: ``web/`` (ассеты и фронт-контроллеры),
  ``app/`` (настройки), ``src/`` (ваши пакеты), и ``vendor/`` (сторонние библиотеки)
  (также ещё имеется директория ``bin/`` для обновления вендоров);

* Каждая функция в Symfony2 (включая ядро фреймворка) должна располагаться внутри *пакета*,
  который представляет собой структурированный набор файлов, реализующих эту функцию;

* *настройки* каждого пакета располагаются в директории ``app/config`` и могут быть записаны
  в формате YAML, XML или PHP;

* каждое **окружение** доступно через свой отдельный фронт-контроллер
  (например ``app.php`` и ``app_dev.php``) и загружает отдельный файл
  настроек;

Далее, каждая глава книги познакомит вас с все более и более мощными инструментами и
более глубокими концепциями. Чем больше вы знаете о Symfony2, тем больше вы будете
ценить гибкость его архитектуры и его обширные возможности для быстрой разработки
приложений.

.. _`Twig`: http://twig.sensiolabs.org
.. _`symfony2bundles.org`: http://symfony2bundles.org/
.. _`Symfony Standard Edition`: http://symfony.com/download
