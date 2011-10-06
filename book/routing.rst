.. index::
   single: Маршрутизация

Маршрутизация
=============

Каждое серьёзное приложение должно обязательно иметь "красивые" URL.
Это означает, что это приложение должно оставить позади страшненькие URL
типа ``index.php?article_id=57`` в пользу таких ``/read/intro-to-symfony``.

Однако гибкость в этом вопросе - ещё более важна, нежели красота. Что, если
вам нужно изменить URL ``/blog`` на ``/news``? Сколько ссылок вам придётся
отыскать и обновить для этого? Если же вы используете маршрутизатор Symfony,
подобные изменения делать легко.

Маршрутизатор Symfony2 позволяет вам определить креативные URL, которые
вы сможете привязать к различным областям вашего приложения. По прочтению
этой главы вы сможете делать следующее:

* Создавать сложные маршруты, соответствующие контроллерам;
* Генерировать URL в шаблонах и контроллерах;
* Загрузить ресурсы для маршрутизации из пакетов (или из любых других источников);
* Отлаживать маршруты.

.. index::
   single: Маршрутизация; Основы

Маршрутизация в действии
------------------------

*Маршрут* по сути это связующее звено между шаблоном URL и контроллером.
Например, предположим, вам нужно искать URL похожие на ``/blog/my-post`` или
``/blog/all-about-symfony`` и отправлять их на обработку в контроллер, который
найдёт и отобразит эти записи блога. Соответствующий этой задаче маршрут - прост:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        blog_show:
            pattern:   /blog/{slug}
            defaults:  { _controller: AcmeBlogBundle:Blog:show }

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog_show" pattern="/blog/{slug}">
                <default key="_controller">AcmeBlogBundle:Blog:show</default>
            </route>
        </routes>

    .. code-block:: php

        <?php
        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog_show', new Route('/blog/{slug}', array(
            '_controller' => 'AcmeBlogBundle:Blog:show',
        )));

        return $collection;

Шаблон, определяемый маршрутом ``blog_show`` работает как выражение ``/blog/*``, где
метасимволом является имя ``slug``. Для URL ``/blog/my-blog-post`` переменная ``slug``
получает значение ``my-blog-post``, которое будет доступно для использования в
контроллере.

Параметр ``_controller`` - это служебный ключ, который сообщает Symfony, какой
именно контроллер должен быть выполнен, когда маршрут совпадает с URL. Строка
``_controller``, называется :ref:`логическим именем<controller-string-syntax>`.
Логическое имя указывает на некоторый РHP-класс и его метод:

.. code-block:: php

    <?php
    // src/Acme/BlogBundle/Controller/BlogController.php

    namespace Acme\BlogBundle\Controller;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class BlogController extends Controller
    {
        public function showAction($slug)
        {
            $blog = // используйте переменную $slug, для того чтобы получить запись из базы данных

            return $this->render('AcmeBlogBundle:Blog:show.html.twig', array(
                'blog' => $blog,
            ));
        }
    }

Поздравляем! Вы только что создали ваш первый маршрут и связали его с контроллером.
Теперь, когда вы посетите страницу ``/blog/my-post``, будет выполнен контроллер
``showAction`` и переменная ``$slug`` будет равна ``my-post``.

Это и есть цель маршрутизатора Symfony2: устанавливать соответствие между
URL запроса и контроллером. Далее в этой главе вы узнаете все возможные трюки,
которые позволяют легко писать маршруты даже для сложных URL.

.. index::
   single: Маршрутизация; Что под капотом

Маршрутизация; Что под капотом
------------------------------

Когда создаётся запрос к вашему приложению, он содержит точный адрес ресурса,
который запрашивается клиентом. Этот адрес называется URL (или URI) и может
выглядеть следующим образом: ``/contact``, ``/blog/read-me`` или ещё каким-то
похожим образом. Давайте рассмотрим следующий HTTP-запрос в качестве примера:

.. code-block:: text

    GET /blog/my-blog-post

Цель системы маршрутизации Symfony2 - разбор этого URL и определение того,
какой контроллер должен быть выполнен. Процесс целиком выглядит так:

#. Запрос обрабатывается фронт-контроллером Symfony2 (например ``app.php``);

#. Ядро Symfony2 (``Kernel``), вызывает маршрутизатор для анализа запроса;

#. Маршрутизатор устанавливает соответствие между входящим URL и некоторым маршрутом
   и возвращает информацию о маршруте, включая данные о том, какой контроллер требуется
   выполнить;

#. Ядро Symfony2 выполняет контроллер, который в конечном итоге возвращает объект ``Response``.

.. figure:: /images/request-flow.png
   :align: center
   :alt: Symfony2 request flow

   Слой маршрутизации - это инструмент, который транслирует входящий URL в контроллер,
   который нужно выполнить для его обработки.

.. index::
   single: Маршрутизация; Создание маршрутов

Создание маршрутов
------------------

Symfony загружает все маршруты, определённые для вашего приложения, из одного
файла настроек. Как правило, этот файл называется ``app/config/routing.yml``,
но при желании наименование файла конфигурации можно изменить на другое (в том
числе на файл формата XML или PHP) в конфигурационном файле приложения:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            # ...
            router:        { resource: "%kernel.root_dir%/config/routing.yml" }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config ...>
            <!-- ... -->
            <framework:router resource="%kernel.root_dir%/config/routing.xml" />
        </framework:config>

    .. code-block:: php

        <?php
        // app/config/config.php
        $container->loadFromExtension('framework', array(
            // ...
            'router'        => array('resource' => '%kernel.root_dir%/config/routing.php'),
        ));

.. tip::

    Не смотря на то, что все маршруты загружаются из одного файла, обычной
    практикой является подключение дополнительных ресурсов внутри этого
    файла (см. секцию :ref:`routing-include-external-resources`).

Базовая настройка маршрута
~~~~~~~~~~~~~~~~~~~~~~~~~~

Определить новый маршрут не сложно, типичное приложение будет иметь много различных
маршрутов. Самый простой маршрут состоит из двух частей: шаблона URL (``pattern``)  и
массива ``defaults``:

.. configuration-block::

    .. code-block:: yaml

        _welcome:
            pattern:   /
            defaults:  { _controller: AcmeDemoBundle:Main:homepage }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="_welcome" pattern="/">
                <default key="_controller">AcmeDemoBundle:Main:homepage</default>
            </route>

        </routes>

    ..  code-block:: php

        <?php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('_welcome', new Route('/', array(
            '_controller' => 'AcmeDemoBundle:Main:homepage',
        )));

        return $collection;

Этот маршрут соответствует главной странице (``/``) и ставит ей в соответствие
контроллер ``AcmeDemoBundle:Main:homepage``. Symfony2 переводит строку
``_controller`` в имя функции, которую необходимо выполнить. Этот процесс
будет объясняться в секции :ref:`controller-string-syntax`.

.. index::
   single: Маршрутизация; Заполнители

Маршрутизация и Заполнители (Placeholders)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Конечно же система маршрутизации поддерживает и более интересные
маршруты. Многие маршруты будут содержать один или более заполнителей
(placeholders):

.. configuration-block::

    .. code-block:: yaml

        blog_show:
            pattern:   /blog/{slug}
            defaults:  { _controller: AcmeBlogBundle:Blog:show }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog_show" pattern="/blog/{slug}">
                <default key="_controller">AcmeBlogBundle:Blog:show</default>
            </route>
        </routes>

    .. code-block:: php

        <?php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog_show', new Route('/blog/{slug}', array(
            '_controller' => 'AcmeBlogBundle:Blog:show',
        )));

        return $collection;

Шаблон будет соответствовать любому URL похожему на ``/blog/*``. Что ещё более
важно - значение, соответствующее заполнителю ``{slug}``, будет доступно в
вашем контроллере. Другими словами, если дан URL ``/blog/hello-world``,
переменная ``$slug`` со значением ``hello-world`` будет доступна в контроллере.
Эту возможность можно использовать, например, для загрузки записи блога,
соответствующей этой строке.

Тем не менее, этот шаблон *не будет соответствовать* URL ``/blog``. Это
вызвано тем фактом, что заполнитель по умолчанию является обязательным
параметром. Однако, если добавить заполнителю значение по умолчанию в
массив ``defaults``.

Обязательные и Опциональные Заполнители
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Для того, чтобы разнообразить процесс, давайте создадим новый маршрут, который
отображает список всех записей в блоге для нашего воображаемого приложения:

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog
            defaults:  { _controller: AcmeBlogBundle:Blog:index }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
            </route>
        </routes>

    .. code-block:: php

        <?php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
        )));

        return $collection;

Пока что этот маршрут выглядит проще простого - он не содержит заполнителей
и соответствует лишь одному URL ``/blog``. Ну а если вам потребуется, чтобы
данный маршрут поддерживал постраничную навигацию и URL ``/blog/2`` отображал
вторую страницу с записями блога? Добавим к маршруту заполнитель ``{page}``:

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog/{page}
            defaults:  { _controller: AcmeBlogBundle:Blog:index }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog/{page}">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
            </route>
        </routes>

    .. code-block:: php

        <?php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog/{page}', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
        )));

        return $collection;

Подобно заполнителю ``{slug}``, значение соответствующее ``{page}`` будет
доступно внутри контроллера. Это значение может быть использовано для того,
чтобы определить, какой набор записей блога отобразить для данной страницы.

Но погодите-ка! Так как заполнитель по умолчанию обязателен, маршрут теперь
не сможет соответствовать просто ``/blog``. Вместо этого, если вы захотите
отобразить первую страницу, вам нужно будет использовать URL ``/blog/1``!
Поскольку это совершенно неприемлемо, потребуется изменить параметр ``{page}``
и сделать его опциональным. Сделать это можно, включив его в массив ``defaults``:

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog/{page}
            defaults:  { _controller: AcmeBlogBundle:Blog:index, page: 1 }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog/{page}">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
                <default key="page">1</default>
            </route>
        </routes>

    .. code-block:: php

        <?php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog/{page}', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
            'page' => 1,
        )));

        return $collection;

Добавив ``page`` в массив ``defaults``, вы сделали заполнитель ``{page}``
необязательным. URL ``/blog`` будет соответствовать маршруту и значение
параметра ``page`` будет равно ``1``. URL ``/blog/2`` также будет соответствовать
этому маршруту, присваивая параметру ``page`` значение ``2``. Отлично.

+---------+------------+
| /blog   | {page} = 1 |
+---------+------------+
| /blog/1 | {page} = 1 |
+---------+------------+
| /blog/2 | {page} = 2 |
+---------+------------+

.. index::
   single: Маршрутизация; Ограничения

Добавляем Ограничения
~~~~~~~~~~~~~~~~~~~

Давайте взглянем на маршруты, которые мы добавили ранее:

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog/{page}
            defaults:  { _controller: AcmeBlogBundle:Blog:index, page: 1 }

        blog_show:
            pattern:   /blog/{slug}
            defaults:  { _controller: AcmeBlogBundle:Blog:show }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog/{page}">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
                <default key="page">1</default>
            </route>

            <route id="blog_show" pattern="/blog/{slug}">
                <default key="_controller">AcmeBlogBundle:Blog:show</default>
            </route>
        </routes>

    .. code-block:: php

        <?php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog/{page}', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
            'page' => 1,
        )));

        $collection->add('blog_show', new Route('/blog/{show}', array(
            '_controller' => 'AcmeBlogBundle:Blog:show',
        )));

        return $collection;

Можете определить тут проблему? Обратите внимание, что оба маршрута имеют похожие
шаблоны и соответствуют URL вида ``/blog/*``. Маршрутизатор Symfony всегда будет
выбирать *первый* совпавший маршрут, который он найдёт. Другими словами, маршрут
``blog_show`` *никогда* не совпадёт и не будет вызван соответствующий контроллер.
Вместо этого URL вида ``/blog/my-blog-post`` будет соответствовать первому маршруту
(``blog``) и возвращать бессмысленное для параметра ``{page}`` значение
``my-blog-post``.

+--------------------+-------+-----------------------+
| URL                | route | parameters            |
+====================+=======+=======================+
| /blog/2            | blog  | {page} = 2            |
+--------------------+-------+-----------------------+
| /blog/my-blog-post | blog  | {page} = my-blog-post |
+--------------------+-------+-----------------------+

Решением этой проблемы является добавление *ограничений* в маршрут. Маршруты в этом
примере будут работать, если шаблон ``/blog/{page}`` будет соответствовать URL
лишь в том случае, когда ``{page}`` будет целым числом. К счастью, ограничения в виде
регулярных выражений легко могут быть добавлены к любому параметру. Например:

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog/{page}
            defaults:  { _controller: AcmeBlogBundle:Blog:index, page: 1 }
            requirements:
                page:  \d+

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog/{page}">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
                <default key="page">1</default>
                <requirement key="page">\d+</requirement>
            </route>
        </routes>

    .. code-block:: php

        <?php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog/{page}', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
            'page' => 1,
        ), array(
            'page' => '\d+',
        )));

        return $collection;

Ограничение ``\d+`` - это регулярное выражение, которое сообщает маршрутизатору, что
значение параметра ``{page}`` должно быть числовым. Маршрут ``blog`` по-прежнему
будет соответствовать URL вида ``/blog/2`` (так как 2 это число), но он уже
не будет соответствовать URL вида ``/blog/my-blog-post`` (так как ``my-blog-post``
*не является числом*).

В результате URL ``/blog/my-blog-post`` будет соответствовать маршруту ``blog_show``.

+--------------------+-----------+-----------------------+
| URL                | route     | parameters            |
+====================+===========+=======================+
| /blog/2            | blog      | {page} = 2            |
+--------------------+-----------+-----------------------+
| /blog/my-blog-post | blog_show | {slug} = my-blog-post |
+--------------------+-----------+-----------------------+

.. sidebar:: Более ранний маршрут всегда выигрывает

    Что же означает тот факт, что порядок маршрутов очень важен? Если маршрут
    ``blog_show`` будет расположен выше маршрута ``blog``, то URL ``/blog/2``
    будет соответствовать маршруту ``blog_show`` вместо маршрута ``blog``
    так как параметр ``{slug}`` не имеет ограничений. Используя правильный порядок
    и разумные ограничения вы сможете сделать всё что вам угодно.

Так как ограничения для параметров - это регулярные выражения, сложность и гибкость
каждого ограничения лежит на вашей совести. Предположим, что главная страница
вашего приложения доступна на двух различных языках, в зависимости от URL:

.. configuration-block::

    .. code-block:: yaml

        homepage:
            pattern:   /{culture}
            defaults:  { _controller: AcmeDemoBundle:Main:homepage, culture: en }
            requirements:
                culture:  en|fr

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="homepage" pattern="/{culture}">
                <default key="_controller">AcmeDemoBundle:Main:homepage</default>
                <default key="culture">en</default>
                <requirement key="culture">en|fr</requirement>
            </route>
        </routes>

    .. code-block:: php

        <?php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('homepage', new Route('/{culture}', array(
            '_controller' => 'AcmeDemoBundle:Main:homepage',
            'culture' => 'en',
        ), array(
            'culture' => 'en|fr',
        )));

        return $collection;

Для входящих запросов, часть URL, соответствующая ``{culture}`` должна удовлетворять
регулярному выражению ``(en|fr)``:

+-----+-----------------------------+
| /   | {culture} = en              |
+-----+-----------------------------+
| /en | {culture} = en              |
+-----+-----------------------------+
| /fr | {culture} = fr              |
+-----+-----------------------------+
| /es | *не соответствует маршруту* |
+-----+-----------------------------+

.. index::
   single: Маршрутизация; Ограничения для HTTP-метода

Добавляем Ограничения для HTTP-метода
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В дополнение к URL, вы также можете проверять *HTTP-метод* входящего запроса
(GET, HEAD, POST, PUT, DELETE). Предположим у вас есть форма контактов с двумя
контроллерами - один для отображения формы (GET запрос) и другой - для обработки
формы, когда она отправлена пользователем (POST запрос). Ограничения для HTTP-метода
можно задать следующим образом:

.. configuration-block::

    .. code-block:: yaml

        contact:
            pattern:  /contact
            defaults: { _controller: AcmeDemoBundle:Main:contact }
            requirements:
                _method:  GET

        contact_process:
            pattern:  /contact
            defaults: { _controller: AcmeDemoBundle:Main:contactProcess }
            requirements:
                _method:  POST

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="contact" pattern="/contact">
                <default key="_controller">AcmeDemoBundle:Main:contact</default>
                <requirement key="_method">GET</requirement>
            </route>

            <route id="contact_process" pattern="/contact">
                <default key="_controller">AcmeDemoBundle:Main:contactProcess</default>
                <requirement key="_method">POST</requirement>
            </route>
        </routes>

    .. code-block:: php

        <?php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('contact', new Route('/contact', array(
            '_controller' => 'AcmeDemoBundle:Main:contact',
        ), array(
            '_method' => 'GET',
        )));

        $collection->add('contact_process', new Route('/contact', array(
            '_controller' => 'AcmeDemoBundle:Main:contactProcess',
        ), array(
            '_method' => 'POST',
        )));

        return $collection;

Пренебрегая тем, что оба представленных выше маршрута имеют идентичные шаблоны
(``/contact``), первый маршрут будет соответствовать только GET-запросам, а второй,
в свою очередь, будет соответствовать только POST-запросам. Это означает, что
вы сможете отображать и отправлять форму, используя один и тот же URL и использовать
различные контроллеры для каждого из этих действий.

.. note::

    Если ограничения на ``_method`` не указаны, маршрут будет соответствовать *любому*
    методу.

Как и любые другие ограничения, ограничения для ``_method`` обрабатываются как
регулярные выражения. Для того, чтобы соответствовать как ``GET`` *так и* ``POST``
запросам, вы можете использовать ограничение ``GET|POST``.

.. index::
   single: Маршрутизация; Продвинутые примеры использования
   single: Routing; _format parameter

.. _advanced-routing-example:

Продвинутая Маршрутизация в Примерах
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

На текущий момент, вы имеете всю необходимую информацию, для создания
сложных структур маршрутизации в Symfony. Ниже мы покажем вам, насколько
гибкой может быть система маршрутизации:

.. configuration-block::

    .. code-block:: yaml

        article_show:
          pattern:  /articles/{culture}/{year}/{title}.{_format}
          defaults: { _controller: AcmeDemoBundle:Article:show, _format: html }
          requirements:
              culture:  en|fr
              _format:  html|rss
              year:     \d+

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="article_show" pattern="/articles/{culture}/{year}/{title}.{_format}">
                <default key="_controller">AcmeDemoBundle:Article:show</default>
                <default key="_format">html</default>
                <requirement key="culture">en|fr</requirement>
                <requirement key="_format">html|rss</requirement>
                <requirement key="year">\d+</requirement>
            </route>
        </routes>

    .. code-block:: php

        <?php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('homepage', new Route('/articles/{culture}/{year}/{title}.{_format}', array(
            '_controller' => 'AcmeDemoBundle:Article:show',
            '_format' => 'html',
        ), array(
            'culture' => 'en|fr',
            '_format' => 'html|rss',
            'year' => '\d+',
        )));

        return $collection;

Как вы можете видеть, этот маршрут сработает лишь в том случае, если ``{culture}``
в URL будет либо ``en`` либо ``fr`` и ``{year}`` будет числом. Этот маршрут также
показывает, что вы можете использовать помимо слэша (``/``) точку между двумя
заполнителями. URL, соответствующий этому маршруту может выглядеть следующим образом:

 * ``/articles/en/2010/my-post``
 * ``/articles/fr/2010/my-post.rss``

.. _book-routing-format-param:

.. sidebar:: Особый параметр маршрута ``_format``

    Этот пример также использует особый параметр маршрута - ``_format``.
    При использовании этого параметра, соответствующее значение становится
    "форматом запроса" в объекте ``Request``. В конечном счёте, формат запроса
    используется для таких действий, как установка ``Content-Type`` для ответа
    (например формат запроса ``json`` трансформируется в ``Content-Type``
    ``application/json``). Этот параметр также может быть использован в
    контроллере для отображения различных шаблонов для каждого возможного
    его значения. Параметр ``_format`` является весьма удобным решением
    при необходимости отображать один и тот же контент в различных форматах.

Специальные параметры маршрута
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Как вы наверное обратили внимание, каждый параметр маршрута или значение по
умолчанию в конечном итоге доступен в виде аргумента в методе контроллера. В дополнение
к этому есть также три специальных параметра, каждый из которых добавляет уникальные
возможности внутри вашего приложения:

* ``_controller``: Как вы уже знаете, этот параметр используется для того, чтобы определить
  какой контроллер будет выполнен, когда маршрут совпадает с URL;

* ``_format``: Используется для определения запрашиваемого формата
  (см. :ref:`параметр маршрута _format<book-routing-format-param>`);

* ``_locale``: Используется для того, чтобы установить локаль в сессии
  (см. :ref:`локаль в URL<book-translation-locale-url>`);

.. index::
   single: Маршрутизация; Контроллеры
   single: Контроллеры; Формат Именования

.. _controller-string-syntax:

Шаблон Именования Контроллера
-----------------------------

Каждый маршрут должен иметь параметр ``_controller``, который определяет,
какой именно контроллер будет выполнен, когда соответствующий маршрут
совпадёт с URL. Этот параметр использует простой строковый шаблон, именуемый
*логическим именем контроллера*, которому Symfony ставит в соответствие
PHP метод и класс. Шаблон состоит из трёх частей, разделённых двоеточием:

    **пакет**:**контроллер**:**действие**

Например, если ``_controller`` имеет значение ``AcmeBlogBundle:Blog:show``, то
это означает следующее:

+----------------+------------------+-------------+
| Bundle         | Controller Class | Method Name |
+================+==================+=============+
| AcmeBlogBundle | BlogController   | showAction  |
+----------------+------------------+-------------+

Контроллер может выглядеть так:

.. code-block:: php

    <?php
    // src/Acme/BlogBundle/Controller/BlogController.php

    namespace Acme\BlogBundle\Controller;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class BlogController extends Controller
    {
        public function showAction($slug)
        {
            // ...
        }
    }

Обратите внимание, что Symfony добавляет строку ``Controller`` у имени класса
(``Blog`` => ``BlogController``) и ``Action`` к имени метода (``show`` => ``showAction``).

Вы также можете ссылаться на этот класс, используя полное имя класса и метода:
``Acme\BlogBundle\Controller\BlogController::showAction``. Но, если вы следуете
нескольким простым соглашениям, логическое имя будет более удобно.

.. note::

    В дополнение к использованию логического имени и полного имени класса,
    Symfony поддерживает третий тип ссылок на контроллер. Этот метод
    использует одно двоеточие в качестве разделителя (например ``service_name:indexAction``)
    и ссылается на котроллер, определённый как сервис (см. :doc:`/cookbook/controller/service`).

Параметры маршрута и Аргументы контроллера
------------------------------------------

Параметры маршрута (например ``{slug}``) очень важны, так как каждый
параметр будет доступен в качестве аргумента в методе-контроллере:

.. code-block:: php

    <?php
    public function showAction($slug)
    {
      // ...
    }

Фактически, все ``defaults`` объединяются со значениями параметров и формируют
один массив. Каждый ключ такого массива доступен в качестве аргумента внутри
контроллера.

Другими словами, для каждого аргумента вашего метода-контроллера, Symfony
ищет параметр маршрута с тем же именем и присваивает его значение этому
аргументу. В продвинутом примере ранее любая комбинация (в любом порядке)
следующих переменных может быть использована в качестве аргументов
метода ``showAction()``:

* ``$culture``
* ``$year``
* ``$title``
* ``$_format``
* ``$_controller``

Так как заполнители и массив ``defaults`` объединяются, даже переменная ``$_controller``
становится доступна. Более подробно это описано в секции :ref:`route-parameters-controller-arguments`.

.. tip::

    Вы также можете использовать переменную ``$_route``, которая содержит
    имя соответствующего маршрута.

.. index::
   single: Маршрутизация; Подключение внешних ресурсов

.. _routing-include-external-resources:

Подключение внешних ресурсов для маршрутизации
----------------------------------------------

Все маршруты загружаются посредством одного конфигурационного файла, обычно
это файл ``app/config/routing.yml`` (см. `Создание маршрутов`_ выше). На практике
же вы вероятно захотите загружать маршруты из других мест, например из ваших
пакетов. И это становится возможным при помощи "импорта" файла маршрутов:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        acme_hello:
            resource: "@AcmeHelloBundle/Resources/config/routing.yml"

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <import resource="@AcmeHelloBundle/Resources/config/routing.xml" />
        </routes>

    .. code-block:: php

        <?php
        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;

        $collection = new RouteCollection();
        $collection->addCollection($loader->import("@AcmeHelloBundle/Resources/config/routing.php"));

        return $collection;

.. note::

   При импорте ресурсов из YAML, ключ (например ``acme_hello``) не имеет практического
   значения. Просто убедитесь, что этот ключ уникален и нигде далее не переопределяется.

Ключ ``resource`` загружает указанный ресурс с маршрутами. В данном примере
ресурс - это полный путь к файлу, где ``@AcmeHelloBundle`` это ярлык, означающий
путь к пакету. Импортируемый файл может выглядеть следующим образом:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/routing.yml
       acme_hello:
            pattern:  /hello/{name}
            defaults: { _controller: AcmeHelloBundle:Hello:index }

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="acme_hello" pattern="/hello/{name}">
                <default key="_controller">AcmeHelloBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        <?php
        // src/Acme/HelloBundle/Resources/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('acme_hello', new Route('/hello/{name}', array(
            '_controller' => 'AcmeHelloBundle:Hello:index',
        )));

        return $collection;

Маршруты из этого файла анализируются и загружаются также, как и основной
файл.

Префикс для импортируемого ресурса
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вы также можете указать "префикс" для импортируемого маршрута. Например,
предположим, что вы хотите чтобы маршрут ``acme_hello`` имел следующий вид:
``/admin/hello/{name}`` вместо обычного ``/hello/{name}``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        acme_hello:
            resource: "@AcmeHelloBundle/Resources/config/routing.yml"
            prefix:   /admin

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <import resource="@AcmeHelloBundle/Resources/config/routing.xml" prefix="/admin" />
        </routes>

    .. code-block:: php

        <?php
        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;

        $collection = new RouteCollection();
        $collection->addCollection($loader->import("@AcmeHelloBundle/Resources/config/routing.php"), '/admin');

        return $collection;

Строка ``/admin`` теперь будет добавлена вначале каждого маршрута, загружаемого
из указанного ресурса:

.. index::
   single: Маршрутизация; Отладка

Отображение и Отладка маршрутов
------------------------------

По мере добавления и настройки маршрутов, было бы удобно иметь возможность
визуализировать и получать детальную информацию о них. Для того, чтобы
увидеть все маршруты вашего приложения, вы можете воспользоваться консольной командой
``router:debug``. Выполните эту команду из корня вашего проекта:

.. code-block:: bash

    php app/console router:debug

Эта команда отобразит удобный список *всех* настроенных маршрутов вашего
приложения:

.. code-block:: text

    homepage              ANY       /
    contact               GET       /contact
    contact_process       POST      /contact
    article_show          ANY       /articles/{culture}/{year}/{title}.{_format}
    blog                  ANY       /blog/{page}
    blog_show             ANY       /blog/{slug}

Вы также можете получить более подробную информацию о конкретном маршруте,
указав его имя после команды ``router:debug``:

.. code-block:: bash

    php app/console router:debug article_show

.. index::
   single: Маршрутизация; Генерация URL

Генерация URL
---------------

Система маршрутизации также должна позволять генерировать URL. На практике,
маршрутизация - это двунаправленная система: устанавливает как соответствие URL
с контроллером (+ параметры), так и обратно - превращает маршрут (+ параметры) в URL. Методы
:method:`Symfony\\Component\\Routing\\Router::match` и
:method:`Symfony\\Component\\Routing\\Router::generate` формируют эту
двунаправленную систему. Рассмотрим маршрут ``blog_show``, описанный выше:

.. code-block:: php

    <?php
    $params = $router->match('/blog/my-blog-post');
    // array('slug' => 'my-blog-post', '_controller' => 'AcmeBlogBundle:Blog:show')

    $uri = $router->generate('blog_show', array('slug' => 'my-blog-post'));
    // /blog/my-blog-post

Для того, чтобы сгенерировать URL, вам необходимо указать имя маршрута (``blog_show``)
и параметры, используемые в шаблоне этого маршрута. Имея эту информацию, можно
сгенерировать любой URL:

.. code-block:: php

    <?php
    class MainController extends Controller
    {
        public function showAction($slug)
        {
          // ...

          $url = $this->get('router')->generate('blog_show', array('slug' => 'my-blog-post'));
        }
    }

В следующей секции вы узнаете как генерировать URL в шаблоне.

.. index::
   single: Маршрутизация; Абсолютные URL

Генерация Абсолютных URL
~~~~~~~~~~~~~~~~~~~~~~~~

По умолчанию, маршрутизатор генерирует относительные URL (например ``/blog``).
Для того, чтобы сгенерировать абсолютный URL, просто укажите "true" в качестве
третьего аргумента метода ``generate()``:

.. code-block:: php

    <?php
    $router->generate('blog_show', array('slug' => 'my-blog-post'), true);
    // http://www.example.com/blog/my-blog-post

.. note::

    Хост, который используется для генерации абсолютного URL - это хост
    из текущего объекта ``Request``. Этот параметр определяется автоматически,
    основываясь на информации о сервере, которую предоставляет PHP. При
    создании абсолютных URL для скриптов, запущенных из командной строки,
    вам необходимо вручную установить желаемый хост для объекта ``Request``:

    .. code-block:: php

        $request->headers->set('HOST', 'www.example.com');

.. index::
   single: Маршрутизация; Генерация URL в шаблоне

Генерация URL содержащих строку запроса (Query String)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Метод ``generate`` принимает массив значений для генерации URL. Если вы передадите
лишний (не указанный в определении маршрута) параметр, он будет добавлен как query string::

    $router->generate('blog', array('page' => 2, 'category' => 'Symfony'));
    // /blog/2?category=Symfony

Генерация URL в шаблоне
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Типичное место, где вам потребуется генерировать URL - это шаблон. Выполнить
эту операцию можно, воспользовавшись функцией-помощником:

.. configuration-block::

    .. code-block:: html+jinja

        <a href="{{ path('blog_show', { 'slug': 'my-blog-post' }) }}">
          Read this blog post.
        </a>

    .. code-block:: php

        <a href="<?php echo $view['router']->generate('blog_show', array('slug' => 'my-blog-post')) ?>">
            Read this blog post.
        </a>

Абсолютные URL также можно генерировать, но уже при помощи другой функции:

.. configuration-block::

    .. code-block:: html+jinja

        <a href="{{ url('blog_show', { 'slug': 'my-blog-post' }) }}">
          Read this blog post.
        </a>

    .. code-block:: php

        <a href="<?php echo $view['router']->generate('blog_show', array('slug' => 'my-blog-post'), true) ?>">
            Read this blog post.
        </a>

Заключение
----------

Маршрутизатор - это система, ставящая в соответствие URL из входящего
запроса контроллеру, который будет вызван для обработки запроса. Он
позволяет использовать в приложении "красивые" URL и поддерживать приложение
независимым от URL-ов. Маршрутизация - это двунаправленный механизм, и
позволяет также генерировать URL.

Дополнительная информация из Книги Рецептов
-------------------------------------------

* :doc:`/cookbook/routing/scheme`
