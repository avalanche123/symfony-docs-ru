Symfony2 против чистого PHP
========================

**Почему использовать Symfony2 лучше, чем простой PHP файл, который можно просто открыть и писать код не задумываясь?**

Если раньше вы никогда не пользовались PHP-фреймворками, не знакомы с философией
Model-View-Controller (здесь и далее MVC) или же удивлены *суматохой* вокруг Symfony2,
то эта глава создана специально для вас! Вместо того чтобы *рассказать* вам о том, что
Symfony2 позволит разрабатывать быстрее и качественнее, чем при использовании чистого PHP,
мы просто покажем вам это.

В этой главе, вы создадите простенькое приложение на чистом PHP и выполните его оптимизацию.
Вы совершите своеобразное путешествие свозь время, наблюдая за развитием web-разработки на
протяжении последних лет от плоского PHP до сегодняшнего уровня.

В конце концов, вы увидите, как Symfony2 поможет вам избавиться от рутинных задач и вернуть
вам контроль над кодом.

Простой блог на чистом PHP
-------------------------

В этой главе вы создадите простое приложение - блог, используя лишь обычный PHP.
Чтобы начать, создайте страницу, которая отображает записи в блоге, которые
были сохранены в базе данных. Писать на чистом PHP проще простого:

.. code-block:: html+php

    <?php
    // index.php

    $link = mysql_connect('localhost', 'myuser', 'mypassword');
    mysql_select_db('blog_db', $link);

    $result = mysql_query('SELECT id, title FROM post', $link);
    ?>

    <html>
        <head>
            <title>List of Posts</title>
        </head>
        <body>
            <h1>List of Posts</h1>
            <ul>
                <?php while ($row = mysql_fetch_assoc($result)): ?>
                <li>
                    <a href="/show.php?id=<?php echo $row['id'] ?>">
                        <?php echo $row['title'] ?>
                    </a>
                </li>
                <?php endwhile; ?>
            </ul>
        </body>
    </html>

    <?php
    mysql_close($link);

Такой код быстро пишется, также быстро выполняется, и, по мере роста вашего приложения,
становится совершенно неподдерживаемым. Таким образом, тут имеется несколько проблем,
которые требуется решить:

* **Нет обработчика ошибок**: А что, если подключение к базе данных отвалится?

* **Плохая организация кода**: По мере роста приложения, этот файл будет все больше и больше,
  в то же время поддерживать его будет всё сложнее и сложнее. Где вы должны будете разместить код,
  который обрабатывает отправку формы? Как вы будете проверять входные данные? А куда разместить
  код для отправки email'ов?

* **Сложность (а скорее даже невозможность) повторного использования кода**: Так как весь код
  располагается в одном файле, нет никакой возможности повторного использования любой части
  приложения для других страниц блога.

.. Внимание::
    Другая проблема, не упомянутая выше, заключается в том, что вы фактически привязаны к
    базе данных MySQL. В данной главе этот вопрос не рассматривается, но, тем не менее,
    Symfony2 изначально интегрирована с ORM `Doctrine`_, библиотекой, отвечающей за
    абстракцию от баз данных и соответствие данных между СУБД и вашими сущностями (mapping).

Давайте же поработаем над разрешением поставленных выше проблем.

Изоляция представления
~~~~~~~~~~~~~~~~~~~~~~~~~~

При разделении "логики" приложения от кода, который подготавливает HTML "представление"
страницы - общая структура приложения сразу же выигрывает:

.. code-block:: html+php

    <?php
    // index.php

    $link = mysql_connect('localhost', 'myuser', 'mypassword');
    mysql_select_db('blog_db', $link);

    $result = mysql_query('SELECT id, title FROM post', $link);

    $posts = array();
    while ($row = mysql_fetch_assoc($result)) {
        $posts[] = $row;
    }

    mysql_close($link);

    // include the HTML presentation code
    require 'templates/list.php';

HTML код теперь расположен в отдельном файле (``templates/list.php``), который
главным образом представляет собой HTML-файл, который использует PHP-синтаксис
"для шаблонов":

.. code-block:: html+php

    <html>
        <head>
            <title>List of Posts</title>
        </head>
        <body>
            <h1>List of Posts</h1>
            <ul>
                <?php foreach ($posts as $post): ?>
                <li>
                    <a href="/read?id=<?php echo $post['id'] ?>">
                        <?php echo $post['title'] ?>
                    </a>
                </li>
                <?php endforeach; ?>
            </ul>
        </body>
    </html>

По договорённости, файл, который содержит всю логику приложения - ``index.php`` -
называется "контроллер". Термин :term:`controller` - это слово, которое вы будете
частенько слышать вне зависимости от языка программирования или же фреймворка,
который используете. В действительности же, речь идёт о части *вашего* кода,
который обрабатывает пользовательский ввод и готовит ответ.

В нашем случае, контроллер получает данные из базы и подключает шаблон, для того
чтобы отобразить их. С изоляцией контроллера, вы получили возможность поменять
*лишь* шаблон, если вам вдруг понадобится отобразить записи блога в другом формате
(например ``list.json.php`` для использования JSON-формата).

Изоляция логики Приложения (Домена)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Пока наше приложение содержало всего одну страницу. Но что же делать, если
нужно добавить вторую страницу, которая использует то же подключение к базе
данных или даже тот же массив постов из блога? Давайте преобразуем код,
изолировав базовую логику от функций доступа к БД - поместим их в новый
файл под названием ``model.php``:

.. code-block:: html+php

    <?php
    // model.php

    function open_database_connection()
    {
        $link = mysql_connect('localhost', 'myuser', 'mypassword');
        mysql_select_db('blog_db', $link);

        return $link;
    }

    function close_database_connection($link)
    {
        mysql_close($link);
    }

    function get_all_posts()
    {
        $link = open_database_connection();

        $result = mysql_query('SELECT id, title FROM post', $link);
        $posts = array();
        while ($row = mysql_fetch_assoc($result)) {
            $posts[] = $row;
        }
        close_database_connection($link);

        return $posts;
    }

.. Совет::

   Имя файла ``model.php`` использовано не случайно - логика и доступ к данным
   приложения традиционно известен как уровень "модели". В правильно организованном
   приложении бОльшая часть кода представляющая собой "бизнес-логику" должна
   быть расположена в модели (в противовес расположению её в контроллере). И,
   в отличие от нашего примера, лишь часть модели отвечает за доступ к БД
   (а бывает и вообще не отвечает).

Контроллер (``index.php``) теперь выглядит очень просто:

.. code-block:: html+php

    <?php
    require_once 'model.php';

    $posts = get_all_posts();

    require 'templates/list.php';

Теперь, в обязанности контроллера вменяется получение данных из модели приложения
и вызов шаблона для отображения данных. Это очень простой пример паттерна
model-view-controller.

Изоляция разметки (Layout)
~~~~~~~~~~~~~~~~~~~~

На текущий момент, приложение разделено на три различных части, предлагающих
различные преимущества и возможности по повторному использованию почти любого кода
для других страниц.

Пока что мы *не можем* повторно использовать - это разметка страницы (layout).
Исправим это упущение, создав файл ``layout.php``:

.. code-block:: html+php

    <!-- templates/layout.php -->
    <html>
        <head>
            <title><?php echo $title ?></title>
        </head>
        <body>
            <?php echo $content ?>
        </body>
    </html>

Шаблон (``templates/list.php``) может быть упрощён, так как будет "расширять"
базовую разметку:

.. code-block:: html+php

    <?php $title = 'List of Posts' ?>

    <?php ob_start() ?>
        <h1>List of Posts</h1>
        <ul>
            <?php foreach ($posts as $post): ?>
            <li>
                <a href="/read?id=<?php echo $post['id'] ?>">
                    <?php echo $post['title'] ?>
                </a>
            </li>
            <?php endforeach; ?>
        </ul>
    <?php $content = ob_get_clean() ?>

    <?php include 'layout.php' ?>

Теперь вы знаете методологию, которая позволяет повторно использовать разметку-layout.
К сожалению, для того чтобы достичь этого, вы вынуждены использовать несколько страшненьких
PHP-функций (``ob_start()``, ``ob_get_clean()``) в шаблоне. Symfony2 использует
компонент ``Templating``, который позволяет достичь этого просто и прозрачно. Скоро
вы увидите - как именно.

Добавляем страницу блога "show"
-------------------------

Страница блога "list" была оптимизирована таким образом, чтобы код был лучше
организован и позволял повторное использование. Для того чтобы доказать, что
все оптимизации были не зря, добавим страницу "show", которая отображает один пост
идентифицируемый по параметру запроса - ``id``.

Для начала, создадим новую функцию в файле ``model.php``, которая получает
одиночную запись по её id:

    // model.php
    function get_post_by_id($id)
    {
        $link = open_database_connection();

        $id = mysql_real_escape_string($id);
        $query = 'SELECT date, title, body FROM post WHERE id = '.$id;
        $result = mysql_query($query);
        $row = mysql_fetch_assoc($result);

        close_database_connection($link);

        return $row;
    }

Далее, создадим новый файл, который назовем ``show.php`` - контроллер для
нашей новой страницы:

.. code-block:: html+php

    <?php
    require_once 'model.php';

    $post = get_post_by_id($_GET['id']);

    require 'templates/show.php';

И, наконец, создадим новый шаблон - ``templates/show.php`` - для отображения
одного поста из блога:

.. code-block:: html+php

    <?php $title = $post['title'] ?>

    <?php ob_start() ?>
        <h1><?php echo $post['title'] ?></h1>

        <div class="date"><?php echo $post['date'] ?></div>
        <div class="body">
            <?php echo $post['body'] ?>
        </div>
    <?php $content = ob_get_clean() ?>

    <?php include 'layout.php' ?>

Создание второй страницы выполнено легко и непринужденно, и мы избежали
дублирования кода. Тем не менее, эта страница добавляет даже больше проблем,
которые фреймворк может решить для вас. Например, отсутствующий или неверный
параметр ``id`` вызовет фатальную ошибку приложения. Было бы лучше, если бы
в этом случае отображалась страница 404, но сейчас мы не можем легко достичь
такого эффекта. И ещё ложка дёгтя - ведь вы забыли "очистить" параметр ``id``
при помощи функции ``mysql_real_escape_string()`` - так что вся ваша база
данных подвергается риску SQL-инъекции.

Другая серьёзная проблема заключается в том, что каждый файл-контроллер
должен подключать файл ``model.php``. А что если к каждому контроллеру неожиданно
придется подключить дополнительный файл или же выполнить другую глобальную операцию
(например, связанную с безопасностью)? При нынешней организации, этот код необходимо
добавить в каждый контроллер. Если вы забудете включить что-нибудь в один из файлов,
остаётся лишь надеяться, что это не скажется на безопасности приложения...

"Front Controller" вам в помощь
----------------------------------

Решение указанных выше проблем является использование :term:`front controller`:
единственного PHP-файла, который будет обрабатывать *любой* запрос. При использовании
front controller (далее просто фронт-контроллер) URI для вашего приложения изменяются
незначительно, но становятся более гибкими:

.. code-block:: text

    Без фронт-контроллера
    /index.php          => Список постов (выполняется index.php)
    /show.php           => Отдельный пост (выполняется show.php)

    При использовании index.php в качестве фронт-контроллера
    /index.php          => Список постов (выполняется index.php)
    /index.php/show     => Отдельный пост (выполняется index.php)

.. Совет::
    Часть URI, включающая ``index.php``, может быть опущена, при использовании
    rewrite rules веб-сервера Apache (или их эквивалента для прочих веб-серверов).
    В этом случае результирующий URI для страницы с постом блога будет просто ``/show``.

При использовании фронт-контроллера, один PHP файл (``index.php`` в нашем случае)
обрабатывает *любой* запрос. Для страницы с одним постом ``/index.php/show``
будет выполнять файл ``index.php``, который теперь несёт ответственность за
маршрутизацию запроса, основываясь на полном URI. Как вы скоро увидите
фронт-контроллер - это очень мощный инструмент.

Создание фронт-контроллера
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Внимание! Прямо сейчас вы стоите на пороге **большого** шага для вашего приложения.
Имея один файл, который принимает все запросы, вы можете централизованно обрабатывать
вопросы, связанные, к примеру, с безопасностью, загрузкой конфигурации, маршрутизацией.
В нашем приложении ``index.php`` теперь должен быть достаточно умён, чтобы отобразить
страницу со списком постов *или* страницу отельного поста, основываясь на URI запроса:

.. code-block:: html+php

    <?php
    // index.php

    // Загружаем и инициализируем глобальные библиотеки
    require_once 'model.php';
    require_once 'controllers.php';

    // Внутренняя маршрутизация
    $uri = $_SERVER['REQUEST_URI'];
    if ($uri == '/index.php') {
        list_action();
    } elseif ($uri == '/index.php/show' && isset($_GET['id'])) {
        show_action($_GET['id']);
    } else {
        header('Status: 404 Not Found');
        echo '<html><body><h1>Page Not Found</h1></body></html>';
    }

Для улучшения структуры приложения, оба контроллера (ранее ``index.php`` и ``show.php``)
превратились в функции, и каждая из них была помещена в файл ``controllers.php``:

.. code-block:: php

    function list_action()
    {
        $posts = get_all_posts();
        require 'templates/list.php';
    }

    function show_action($id)
    {
        $post = get_post_by_id($id);
        require 'templates/show.php';
    }

Став фронт-контроллером ``index.php`` получил совершенно новую роль, включая
загрузку библиотек ядра и маршрутизацию, которая сейчас заключается в вызове
одного из двух контроллеров (функции ``list_action()`` и ``show_action()``).
На самом деле, этот фронт-контроллер уже, в плане обработки запросов и маршрутизации,
начинает себя вести сходным образом, как и контроллер Symfony2.

.. Совет::

   Другое достоинство фронт-контроллера - это гибкие URL. Обратите внимание,
   что URL для страницы, отображающей отдельный пост блога, в любой момент
   может быть изменён с ``/show`` на ``/read``, изменив код всего лишь в
   одном месте. Ранее же нам бы потребовалось переименовать файл целиком.
   В Symfony2 URLы ещё более гибки.

К этому времени, приложение разрослось с одного PHP-файла до целой структуры,
которая хорошо организована и позволяет повторное использование кода. Вы
должны быть счастливы, но до полного удовлетворения ещё далеко. К примеру,
система "маршрутизации" ненадёжна и не может определить, что страница list
(``/index.php``) должна быть доступна через ``/`` (если используются Apache
rewrite rules). Также, вместо того чтобы разрабатывать блог, куча времени была
потрачена на "архитектуру" кода (например, маршрутизация, вызовы контроллеров,
шаблоны и т.п.). Еще больше времени нужно, чтобы обрабатывать отправку форм,
валидацию введённых данных, логгирование и безопасность. Почему мы должны
заново изобретать решения для этих рутинных проблем?

Прикосновение к Symfony2
~~~~~~~~~~~~~~~~~~~~~~~

Symfony2 идёт на помощь. Перед тем, как начать использовать Symfony2, вам
нужно указать PHP как и где найти классы Symfony2. Это достигается путём
использования автозагрузчика, который предоставляет Symfony. Автозагрузчик
- это инструмент, который позволяет использовать PHP-классы, не подключая
файлы их содержащие явно.

Во-первых, `скачать symfony`_ и поместите файлы в директорию ``vendor/symfony/``.
Затем, создайте файл ``app/bootstrap.php``. Используйте его для подключения
(``require``) двух файлов приложения и конфигурирования автозагрузчика:

.. code-block:: html+php

    <?php
    // bootstrap.php
    require_once 'model.php';
    require_once 'controllers.php';
    require_once 'vendor/symfony/src/Symfony/Component/ClassLoader/UniversalClassLoader.php';

    $loader = new Symfony\Component\ClassLoader\UniversalClassLoader();
    $loader->registerNamespaces(array(
        'Symfony' => __DIR__.'/vendor/symfony/src',
    ));

    $loader->register();

Это покажет автозагрузчику, где живут классы ``Symfony``. Теперь вы можете
начать пользоваться классами Symfony, не используя оператор ``require`` для
файлов, содержащих требуемые классы.

Ядром философии Symfony является идея, что основная задача приложения - это
интерпретировать каждый запрос и возвратить ответ. Для этого Symfony2 предоставляет
два класса: :class:`Symfony\\Component\\HttpFoundation\\Request` и
:class:`Symfony\\Component\\HttpFoundation\\Response`. Эти классы являются
объектно-ориентированным представлением необработанного HTTP-запроса, который
подлежит обработке и соответствующего ему HTTP-ответа, который будет возвращен клиенту.
Используйте их для улучшения блога:

.. code-block:: html+php

    <?php
    // index.php
    require_once 'app/bootstrap.php';

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;

    $request = Request::createFromGlobals();

    $uri = $request->getPathInfo();
    if ($uri == '/') {
        $response = list_action();
    } elseif ($uri == '/show' && $request->query->has('id')) {
        $response = show_action($request->query->get('id'));
    } else {
        $html = '<html><body><h1>Page Not Found</h1></body></html>';
        $response = new Response($html, 404);
    }

    // Вывод заголовков и отправка ответа
    $response->send();

Контроллеры теперь отвечают за возврат объекта ``Response``. Для того чтобы
упростить процесс создания ответа, вы можете добавить новую функцию ``render_template()``,
которая, между прочим, действует практически как движок шаблонов Symfony2:

.. code-block:: php

    // controllers.php
    use Symfony\Component\HttpFoundation\Response;

    function list_action()
    {
        $posts = get_all_posts();
        $html = render_template('templates/list.php', array('posts' => $posts));

        return new Response($html);
    }

    function show_action($id)
    {
        $post = get_post_by_id($id);
        $html = render_template('templates/show.php', array('post' => $post));

        return new Response($html);
    }

    // Функция-помощник для отображения шаблонов
    function render_template($path, array $args)
    {
        extract($args);
        ob_start();
        require $path;
        $html = ob_get_clean();

        return $html;
    }

Получив в помощь небольшую часть Symfony2, приложение стало более гибким и
надёжным. ``Request`` предоставляет надёжный способ получить информацию о запросе.
К примеру, метод ``getPathInfo()`` возвращает "очищенный" URI (всегда возвращает
``/show`` и никогда ``/index.php/show``). Таким образом, даже если пользователь
откроет в браузере ``/index.php/show``, приложение выполнит ``show_action()``.

Объект ``Response`` предоставляет гибкость в построении HTTP-ответа, позволяя добавлять
HTTP заголовки и контент страницы посредством объектно-ориентированного интерфейса.
И, хотя в этом приложении пока что ответы весьма просты, эта гибкость выплатит вам дивиденды
по мере роста приложения.

Простое приложение на Symfony2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Блог начал свой *длинный* путь, но он всё ещё содержит слишком много кода для
такого небольшого приложения. Следуя по пути, мы изобрели простую систему
маршрутизации и метод, использующий ``ob_start()`` и ``ob_get_clean()`` для
отображения шаблонов. Если, по каким-либо соображениям, вы хотите продолжить
создание этого "фреймворка" с нуля, вы можете по крайней мере использовать
самостоятельные компоненты Symfony - `Routing`_ и `Templating`_, которые решают
эти проблемы.

Вместо того чтобы заново решать типовые проблемы, вы можете предоставить
Symfony2 заботу о них. Вот пример простого приложения, построенного с
использованием Symfony2:

.. code-block:: html+php

    <?php
    // src/Acme/BlogBundle/Controller/BlogController.php

    namespace Acme\BlogBundle\Controller;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class BlogController extends Controller
    {
        public function listAction()
        {
            $posts = $this->get('doctrine')->getEntityManager()
                ->createQuery('SELECT p FROM AcmeBlogBundle:Post p')
                ->execute();

            return $this->render('AcmeBlogBundle:Post:list.html.php', array('posts' => $posts));
        }

        public function showAction($id)
        {
            $post = $this->get('doctrine')
                ->getEntityManager()
                ->getRepository('AcmeBlogBundle:Post')
                ->find($id);

            if (!$post) {
                // cause the 404 page not found to be displayed
                throw $this->createNotFoundException();
            }

            return $this->render('AcmeBlogBundle:Post:show.html.php', array('post' => $post));
        }
    }

Эти два контроллера всё ещё легковесны. Каждый из них использует библиотеку Doctrine
ORM для получения объектов из базы данных и компонент ``Templating`` для отображения
шаблона и возврата объекта ``Response``. Шаблон list теперь стал ещё немного проще:

.. code-block:: html+php

    <!-- src/Acme/BlogBundle/Resources/views/Blog/list.html.php -->
    <?php $view->extend('::layout.html.php') ?>

    <?php $view['slots']->set('title', 'List of Posts') ?>

    <h1>List of Posts</h1>
    <ul>
        <?php foreach ($posts as $post): ?>
        <li>
            <a href="<?php echo $view['router']->generate('blog_show', array('id' => $post->getId())) ?>">
                <?php echo $post->getTitle() ?>
            </a>
        </li>
        <?php endforeach; ?>
    </ul>

Layout практически не изменился:

.. code-block:: html+php

    <!-- app/Resources/views/layout.html.php -->
    <html>
        <head>
            <title><?php echo $view['slots']->output('title', 'Default title') ?></title>
        </head>
        <body>
            <?php echo $view['slots']->output('_content') ?>
        </body>
    </html>

.. Внимание::

    Мы оставляем шаблон show вам в качестве самостоятельного упражнения,
    так как он будет не сложнее шаблона list.

Когда движок Symfony2 (который называется ``Kernel`` - ядро) загружается,
он нуждается в "карте", по которой он будет узнавать - какой контроллер требуется
выполнить, основываясь на информации из запроса. Конфигурация маршрутизатора
предоставляет ему эту информацию в следующем формате:

.. code-block:: yaml

    # app/config/routing.yml
    blog_list:
        pattern:  /blog
        defaults: { _controller: AcmeBlogBundle:Blog:list }

    blog_show:
        pattern:  /blog/show/{id}
        defaults: { _controller: AcmeBlogBundle:Blog:show }

Теперь, когда Symfony2 берёт на себя повседневные задачи, фронт-контроллер
стал предельно простым. Поскольку он теперь делает так мало, вам никогда
не придется трогать его после создания (а если вы используете дистрибутив Symfony2,
то вам даже не придётся создавать его!):

.. code-block:: html+php

    <?php
    // web/app.php
    require_once __DIR__.'/../app/bootstrap.php';
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->handle(Request::createFromGlobals())->send();

Единственная забота фронт-контроллера - инициализация движка Symfony2
(``Kernel``) и передача ему объекта ``Request`` для последующей обработки.
Ядро Symfony2 использует карту маршрутизации для определения - какой контроллер
необходимо выполнить. Как и раньше, метод контроллера отвечает за возврат
конечного объекта ``Response``.

Для визуального представления процесса обработки запроса в Symfony2 -
посмотрите диаграмму :ref:`процесс обработки запроса<request-flow-figure>`.

В чём польза Symfony2
~~~~~~~~~~~~~~~~~~~~~~~

В последующих главах вы узнаете больше обо всех аспектах работы с Symfony и
рекомендуемой структуре проекта. Сейчас же давайте посмотрим - как миграция
блога с обычного PHP на Symfony2 улучшает жизнь:

* Теперь ваше приложение имеет **простой, понятный и единообразно организованный код**
  (хотя Symfony не требует этого от вас). Это поощряет **повторное использование**
  и позволяет новым разработчикам становиться продуктивными быстрее.

* 100% кода, который вы написали - для *вашего* приложения. Вам
  **не нужно разрабатывать или поддерживать низкоуровневые инструменты**,
  такие как :ref:`автозагрузка<autoloading-introduction-sidebar>`,
  :doc:`маршрутизация</book/routing>`, или рендеринг :doc:`контроллеров</book/controller>`.

* Symfony2 предоставляет вам **доступ к инструментам с открытым кодом**,
  таким как Doctrine, и компонентам Templating, Security, Form, Validation and Translation.

* Приложение теперь использует **гибчайшие URLы** благодаря компоненту ``Routing``.

* Архитектура Symfony2, центрированная на HTTP, дает вам доступ к
  мощным инструментам, таким как **HTTP кеширование**, базирующееся на
  **внутреннем HTTP-кэше Symfony2** или более ещё более мощным инструментам,
  таким как `Varnish`_. Об этом будет рассказано в главе о
  :doc:`кэшировании</book/http_cache>`.

И, возможно самое лучшее, используя Symfony2 вы получаете доступ к целому
набору **качественных инструментов с открытым исходным кодом, разработанных
участниками коммьюнити**! Дополнительную информацию вы можете получить на сайте
`Symfony2Bundles.org`_

Лучшие шаблоны
----------------

Если вы выбрали Symfony2, то приготовьтесь встретиться с шаблонизатором
`Twig`_, который делает шаблоны быстрыми в разработке и лёгкие в понимании.
Это означает, что приложение будет содержать ещё меньше кода! Давайте,
к примеру, взглянем на шаблон списка, написанный на Twig:

.. code-block:: html+jinja

    {# src/Acme/BlogBundle/Resources/views/Blog/list.html.twig #}

    {% extends "::layout.html.twig" %}
    {% block title %}List of Posts{% endblock %}

    {% block body %}
        <h1>List of Posts</h1>
        <ul>
            {% for post in posts %}
            <li>
                <a href="{{ path('blog_show', { 'id': post.id }) }}">
                    {{ post.title }}
                </a>
            </li>
            {% endfor %}
        </ul>
    {% endblock %}

Соответствующий шаблон ``layout.html.twig`` ещё проще:

.. code-block:: html+jinja

    {# app/Resources/views/layout.html.twig #}

    <html>
        <head>
            <title>{% block title %}Default title{% endblock %}</title>
        </head>
        <body>
            {% block body %}{% endblock %}
        </body>
    </html>

Twig отлично интегрирован с Symfony2. В то время, как PHP шаблоны будут
всегда поддерживаться в Symfony2, мы также будем продолжать обсуждения преимуществ
Twig. Больше информации о Twig вы найдете в :doc:`главе о шаблонах</book/templating>`.

Дополнительная информация в Cookbook
----------------------------

* :doc:`/cookbook/templating/PHP`
* :doc:`/cookbook/controller/service`

.. _`Doctrine`: http://www.doctrine-project.org
.. _`скачать symfony`: http://symfony.com/download
.. _`Routing`: https://github.com/symfony/Routing
.. _`Templating`: https://github.com/symfony/Templating
.. _`Symfony2Bundles.org`: http://symfony2bundles.org
.. _`Twig`: http://twig.sensiolabs.org
.. _`Varnish`: http://www.varnish-cache.org
.. _`PHPUnit`: http://www.phpunit.de
